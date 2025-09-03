# Query EMR Steps

**Purpose**: Analyze EMR step execution, performance, and identify bottlenecks in job processing pipeline.

**Elicit**: true

## Input Requirements

Please provide the following information:

1. **EMR Cluster ID**:
   - Format: `j-XXXXXXXXXXXXX`
   - Example: `j-8P7Z16NUDD4H`

2. **AWS Region**:
   - Example: `us-west-2`
   - Or: Use default configured region

3. **Step Analysis Scope** (select one or more):
   - [ ] All steps (comprehensive analysis)
   - [ ] Failed steps only
   - [ ] Running steps only
   - [ ] Completed steps only
   - [ ] Specific step ID: ****\_\_\_****
   - [ ] Steps from last 24 hours
   - [ ] Steps matching pattern: ****\_\_\_****

4. **Analysis Focus**:
   - [ ] Performance bottlenecks
   - [ ] Failure analysis
   - [ ] Resource utilization
   - [ ] Step dependencies
   - [ ] Cost optimization
   - [ ] Execution patterns

5. **Output Detail Level**:
   - [ ] Summary overview
   - [ ] Detailed step analysis
   - [ ] Performance metrics
   - [ ] Complete diagnostic report

## Step Analysis Workflow

### Phase 1: Step Discovery and Overview

**Step 1: List All Steps**

```bash
# Get comprehensive step list
aws emr list-steps \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'Steps[*].{
        StepId: Id,
        Name: Name,
        State: Status.State,
        StartTime: Status.Timeline.StartDateTime,
        EndTime: Status.Timeline.EndDateTime,
        ActionOnFailure: ActionOnFailure,
        HadoopJarStep: HadoopJarStep.Jar
    }' \
    --output table
```

**Step 2: Step Summary Statistics**

```bash
# Calculate step statistics
total_steps=$(aws emr list-steps --region {region} --cluster-id {cluster-id} --query 'length(Steps)' --output text)
completed_steps=$(aws emr list-steps --region {region} --cluster-id {cluster-id} --query 'length(Steps[?Status.State==`COMPLETED`])' --output text)
failed_steps=$(aws emr list-steps --region {region} --cluster-id {cluster-id} --query 'length(Steps[?Status.State==`FAILED`])' --output text)
running_steps=$(aws emr list-steps --region {region} --cluster-id {cluster-id} --query 'length(Steps[?Status.State==`RUNNING`])' --output text)

echo "Step Execution Summary for Cluster {cluster-id}"
echo "=============================================="
echo "Total Steps: $total_steps"
echo "Completed: $completed_steps"
echo "Failed: $failed_steps"
echo "Running: $running_steps"
echo "Success Rate: $(echo "scale=2; $completed_steps * 100 / $total_steps" | bc)%"
```

### Phase 2: Detailed Step Analysis

**Step 3: Analyze Individual Steps**

```bash
# Get detailed information for each step
aws emr list-steps \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'Steps[*]' \
    --output json > steps_raw_data.json

# Process each step for detailed analysis
jq -r '.[] | @base64' steps_raw_data.json | while read step; do
    step_data=$(echo "$step" | base64 --decode)
    step_id=$(echo "$step_data" | jq -r '.Id')
    step_name=$(echo "$step_data" | jq -r '.Name')
    step_state=$(echo "$step_data" | jq -r '.Status.State')

    echo "Analyzing Step: $step_name ($step_id)"
    echo "State: $step_state"

    # Get detailed step information
    aws emr describe-step \
        --region {region} \
        --cluster-id {cluster-id} \
        --step-id $step_id \
        --query 'Step.{
            Name: Name,
            State: Status.State,
            StateReason: Status.StateChangeReason.Message,
            StartTime: Status.Timeline.StartDateTime,
            EndTime: Status.Timeline.EndDateTime,
            ActionOnFailure: ActionOnFailure,
            HadoopJarStep: HadoopJarStep
        }' \
        --output json
    echo "---"
done
```

**Step 4: Performance Metrics Calculation**

```bash
# Calculate execution times for completed steps
echo "Step Performance Analysis:"
echo "========================="

aws emr list-steps \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'Steps[?Status.State==`COMPLETED`]' \
    --output json | jq -r '.[] |
    {
        name: .Name,
        id: .Id,
        start: .Status.Timeline.StartDateTime,
        end: .Status.Timeline.EndDateTime
    } |
    @base64' | while read step; do

    step_info=$(echo "$step" | base64 --decode)
    name=$(echo "$step_info" | jq -r '.name')
    start_time=$(echo "$step_info" | jq -r '.start')
    end_time=$(echo "$step_info" | jq -r '.end')

    if [ "$start_time" != "null" ] && [ "$end_time" != "null" ]; then
        duration_seconds=$(( $(date -d "$end_time" +%s) - $(date -d "$start_time" +%s) ))
        duration_minutes=$(( duration_seconds / 60 ))
        echo "$name: ${duration_minutes}m ${duration_seconds}s"
    fi
done | sort -k2 -nr | head -10
```

### Phase 3: Failure Analysis

**Step 5: Analyze Failed Steps**

```bash
# Deep dive into failed steps
failed_step_ids=$(aws emr list-steps \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'Steps[?Status.State==`FAILED`].Id' \
    --output text)

if [ ! -z "$failed_step_ids" ]; then
    echo "Failed Step Analysis:"
    echo "===================="

    for step_id in $failed_step_ids; do
        echo "Analyzing failed step: $step_id"

        # Get detailed failure information
        aws emr describe-step \
            --region {region} \
            --cluster-id {cluster-id} \
            --step-id $step_id \
            --query 'Step.{
                Name: Name,
                State: Status.State,
                FailureReason: Status.StateChangeReason.Message,
                FailureCode: Status.StateChangeReason.Code,
                StartTime: Status.Timeline.StartDateTime,
                EndTime: Status.Timeline.EndDateTime,
                HadoopJarStep: HadoopJarStep,
                ActionOnFailure: ActionOnFailure
            }' \
            --output json

        echo "---"
    done
else
    echo "✅ No failed steps found"
fi
```

**Step 6: Identify Step Dependencies and Bottlenecks**

```bash
# Analyze step execution timeline
echo "Step Execution Timeline Analysis:"
echo "================================"

# Get steps with timing information
aws emr list-steps \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'Steps[?Status.Timeline.StartDateTime != null] | sort_by(@, &Status.Timeline.StartDateTime) | [*].{
        Name: Name,
        State: Status.State,
        StartTime: Status.Timeline.StartDateTime,
        EndTime: Status.Timeline.EndDateTime,
        Duration: "calculated"
    }' \
    --output json | jq '.[] |
    {
        name: .Name,
        state: .State,
        start: .StartTime,
        end: .EndTime,
        duration_seconds: (if .EndTime != null then
            (.EndTime | strptime("%Y-%m-%dT%H:%M:%S.%fZ") | mktime) -
            (.StartTime | strptime("%Y-%m-%dT%H:%M:%S.%fZ") | mktime)
        else null end)
    }'
```

### Phase 4: Resource and Cost Analysis

**Step 7: Step Resource Utilization**

```bash
# Analyze resource usage patterns during step execution
echo "Resource Utilization During Steps:"
echo "================================="

# Get step execution periods
aws emr list-steps \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'Steps[?Status.State==`COMPLETED`].[Id,Name,Status.Timeline.StartDateTime,Status.Timeline.EndDateTime]' \
    --output text | while read step_id name start_time end_time; do

    echo "Step: $name"
    echo "Execution Period: $start_time to $end_time"

    # Calculate step duration for cost analysis
    if [ "$start_time" != "None" ] && [ "$end_time" != "None" ]; then
        duration_seconds=$(( $(date -d "$end_time" +%s) - $(date -d "$start_time" +%s) ))
        duration_hours=$(echo "scale=2; $duration_seconds / 3600" | bc)
        echo "Duration: ${duration_hours} hours"

        # Get instance count during this period (approximation)
        instance_count=$(aws emr list-instances \
            --region {region} \
            --cluster-id {cluster-id} \
            --query 'length(Instances)' \
            --output text)
        echo "Approximate instances during execution: $instance_count"
    fi
    echo "---"
done
```

**Step 8: Spark Application Mapping**

```bash
# Map steps to Spark applications for MCP analysis
echo "Spark Application Mapping:"
echo "========================="

spark_steps=$(aws emr list-steps \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'Steps[?contains(Name, `spark`) || contains(Name, `Spark`) || contains(HadoopJarStep.Jar, `spark`)]' \
    --output json)

echo "$spark_steps" | jq -r '.[] |
"Step ID: \(.Id)
Step Name: \(.Name)
JAR: \(.HadoopJarStep.Jar // "N/A")
Args: \(.HadoopJarStep.Args // [] | join(" "))
State: \(.Status.State)
---"'
```

### Phase 5: Performance Optimization Analysis

**Step 9: Identify Optimization Opportunities**

```bash
# Analyze step patterns for optimization
echo "Optimization Opportunities:"
echo "=========================="

# Find long-running steps
echo "Top 5 Longest Running Steps:"
aws emr list-steps \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'Steps[?Status.State==`COMPLETED`]' \
    --output json | jq -r 'sort_by(
        (if .Status.Timeline.EndDateTime != null and .Status.Timeline.StartDateTime != null then
            (.Status.Timeline.EndDateTime | strptime("%Y-%m-%dT%H:%M:%S.%fZ") | mktime) -
            (.Status.Timeline.StartDateTime | strptime("%Y-%m-%dT%H:%M:%S.%fZ") | mktime)
        else 0 end)
    ) | reverse | .[0:5] | .[] |
    "Step: \(.Name) | Duration: \(
        if .Status.Timeline.EndDateTime != null and .Status.Timeline.StartDateTime != null then
            ((.Status.Timeline.EndDateTime | strptime("%Y-%m-%dT%H:%M:%S.%fZ") | mktime) -
             (.Status.Timeline.StartDateTime | strptime("%Y-%m-%dT%H:%M:%S.%fZ") | mktime)) / 60 | floor
        else 0 end
    ) minutes"'

# Check for sequential vs parallel execution opportunities
echo
echo "Step Concurrency Analysis:"
step_concurrency=$(aws emr describe-cluster \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'Cluster.StepConcurrencyLevel' \
    --output text)
echo "Current Step Concurrency Level: $step_concurrency"

# Analyze overlapping execution times
echo "Overlapping step execution analysis would require detailed timeline parsing..."
```

**Step 10: Generate Optimization Recommendations**

```bash
# Generate actionable recommendations
echo "Recommendations Based on Step Analysis:"
echo "======================================"

# Check step failure patterns
failure_rate=$(echo "scale=2; $failed_steps * 100 / $total_steps" | bc)
if (( $(echo "$failure_rate > 10" | bc -l) )); then
    echo "⚠️  High failure rate ($failure_rate%) - Review step configurations and error handling"
fi

# Check step concurrency
if [ "$step_concurrency" -eq 1 ] && [ "$total_steps" -gt 5 ]; then
    echo "💡 Consider increasing step concurrency level for better parallelism"
fi

# Check for optimization patterns
echo "💡 Long-running steps identified - consider:"
echo "   - Breaking down large steps into smaller chunks"
echo "   - Optimizing Spark configurations"
echo "   - Using more appropriate instance types"
echo "   - Implementing checkpointing for fault tolerance"
```

### Phase 6: Integration with MCP Analysis

**Step 11: Prepare MCP Analysis Commands**

```bash
# Generate MCP analysis commands for Spark applications
echo "# MCP Analysis Commands for Spark Applications"
echo "# Generated from EMR Step Analysis"
echo "# Cluster: {cluster-id}"
echo "# Date: $(date)"
echo

# Extract potential Spark application IDs from step configurations
aws emr list-steps \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'Steps[?contains(Name, `spark`) || contains(Name, `Spark`)]' \
    --output json | jq -r '.[] |
"# Step: \(.Name) (\(.Status.State))
# Step ID: \(.Id)
# Potential MCP Analysis (replace with actual application ID):
# *analyze-spark-job application_XXXXXXXXXXXXXXXX_XXXX
# *performance-report application_XXXXXXXXXXXXXXXX_XXXX
"'
```

### Phase 7: Export and Documentation

**Step 12: Generate Comprehensive Report**

```bash
# Create detailed step analysis report
{
    echo "# EMR Step Analysis Report"
    echo "Cluster ID: {cluster-id}"
    echo "Region: {region}"
    echo "Analysis Date: $(date)"
    echo
    echo "## Executive Summary"
    echo "- Total Steps: $total_steps"
    echo "- Success Rate: $(echo "scale=2; $completed_steps * 100 / $total_steps" | bc)%"
    echo "- Failed Steps: $failed_steps"
    echo "- Running Steps: $running_steps"
    echo
    echo "## Step Details"
    aws emr list-steps --region {region} --cluster-id {cluster-id} --output table
    echo
    echo "## Failed Steps Analysis"
    if [ "$failed_steps" -gt 0 ]; then
        for step_id in $failed_step_ids; do
            echo "### Step ID: $step_id"
            aws emr describe-step --region {region} --cluster-id {cluster-id} --step-id $step_id --output table
        done
    else
        echo "No failed steps found."
    fi
    echo
    echo "## Recommendations"
    echo "- Review long-running steps for optimization opportunities"
    echo "- Consider increasing step concurrency if appropriate"
    echo "- Implement proper error handling and retry mechanisms"
    echo "- Use MCP analysis for detailed Spark application performance review"
} > emr_step_analysis_{cluster-id}_$(date +%Y%m%d_%H%M%S).md
```

## Troubleshooting

### Common Issues

**No Steps Found:**

```bash
# Check if cluster has any steps
aws emr list-steps --region {region} --cluster-id {cluster-id} --query 'length(Steps)' --output text

# Check cluster state
aws emr describe-cluster --region {region} --cluster-id {cluster-id} --query 'Cluster.Status.State' --output text
```

**Step Details Not Available:**

```bash
# Verify step ID format
aws emr list-steps --region {region} --cluster-id {cluster-id} --query 'Steps[].Id' --output text

# Check permissions
aws iam simulate-principal-policy \
    --policy-source-arn $(aws sts get-caller-identity --query Arn --output text) \
    --action-names elasticmapreduce:ListSteps elasticmapreduce:DescribeStep \
    --resource-arns "*"
```

## Follow-up Actions

Based on step analysis results:

1. **For Failed Steps**: Use `*fetch-emr-logs {cluster-id}` to get detailed error logs
2. **For Spark Steps**: Use `*analyze-spark-job {app-id}` for performance analysis
3. **For Cost Optimization**: Use `*aws-cost-analysis {cluster-id}` to analyze costs
4. **For Instance Issues**: Use `*check-emr-instances {cluster-id}` for resource analysis
5. **For Configuration**: Use `*optimize-configuration` for tuning recommendations

Your EMR step analysis provides insights into job execution patterns, performance bottlenecks, and optimization opportunities!
