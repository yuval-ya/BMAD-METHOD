# List EMR Clusters

**Purpose**: Discover and list available EMR clusters across regions with detailed information for analysis.

**Elicit**: true

## Input Requirements

Please provide the following information:

1. **AWS Region(s) to search** (select one or more):
   - [ ] us-east-1 (N. Virginia)
   - [ ] us-west-2 (Oregon)
   - [ ] eu-west-1 (Ireland)
   - [ ] ap-southeast-1 (Singapore)
   - [ ] All regions (comprehensive search)
   - [ ] Custom region: ****\_\_\_****

2. **Cluster states to include** (select one or more):
   - [ ] RUNNING - Active clusters
   - [ ] WAITING - Idle clusters ready for steps
   - [ ] TERMINATED - Recently terminated clusters
   - [ ] TERMINATED_WITH_ERRORS - Failed clusters
   - [ ] All states

3. **Time range** (optional):
   - [ ] Last 24 hours
   - [ ] Last 7 days
   - [ ] Last 30 days
   - [ ] Custom date range: From **\_\_\_** To **\_\_\_**
   - [ ] All time

4. **Analysis focus**:
   - [ ] Active cluster discovery
   - [ ] Cost analysis preparation
   - [ ] Performance comparison
   - [ ] Troubleshooting investigation
   - [ ] General inventory

## Cluster Discovery Workflow

### Phase 1: Region Discovery

**Step 1: Verify AWS CLI Configuration**

```bash
# Check current AWS configuration
aws configure list

# Verify authentication
aws sts get-caller-identity
```

**Step 2: Determine Target Regions**

```bash
# List all available regions
aws ec2 describe-regions --query 'Regions[].RegionName' --output table

# Check default region
aws configure get region
```

### Phase 2: Cluster Enumeration

**Step 3: List Clusters by Region**

**Single Region Search:**

```bash
# List clusters in specified region
aws emr list-clusters \
    --region {region} \
    --cluster-states RUNNING WAITING TERMINATED \
    --created-after 2024-01-01 \
    --output table

# Get detailed cluster information
aws emr list-clusters \
    --region {region} \
    --query 'Clusters[*].{ID:Id,Name:Name,State:Status.State,Created:Status.Timeline.CreationDateTime,Type:InstanceCollectionType}' \
    --output table
```

**Multi-Region Search:**

```bash
# Search across multiple regions
for region in us-east-1 us-west-2 eu-west-1; do
    echo "=== Region: $region ==="
    aws emr list-clusters \
        --region $region \
        --cluster-states RUNNING WAITING TERMINATED \
        --query 'Clusters[*].{ID:Id,Name:Name,State:Status.State,Region:"'$region'"}' \
        --output table
    echo
done
```

**Comprehensive Global Search:**

```bash
# Search all regions (comprehensive)
regions=$(aws ec2 describe-regions --query 'Regions[].RegionName' --output text)
for region in $regions; do
    echo "Checking region: $region"
    clusters=$(aws emr list-clusters --region $region --max-items 100 2>/dev/null)
    if [ $? -eq 0 ] && [ "$(echo "$clusters" | jq '.Clusters | length')" -gt 0 ]; then
        echo "Found clusters in $region:"
        echo "$clusters" | jq -r '.Clusters[] | "\(.Id) | \(.Name) | \(.Status.State) | \(.Status.Timeline.CreationDateTime)"'
    fi
    echo "---"
done
```

### Phase 3: Detailed Cluster Analysis

**Step 4: Extract Cluster Metadata**

```bash
# Get comprehensive cluster list with metadata
aws emr list-clusters \
    --region {region} \
    --query 'Clusters[*].{
        ID:Id,
        Name:Name,
        State:Status.State,
        StateReason:Status.StateChangeReason.Message,
        Created:Status.Timeline.CreationDateTime,
        Ready:Status.Timeline.ReadyDateTime,
        Ended:Status.Timeline.EndDateTime,
        InstanceType:InstanceCollectionType,
        EMRVersion:ReleaseLabel,
        Applications:Applications[].Name
    }' \
    --output json
```

**Step 5: Filter by Specific Criteria**

**Active Clusters Only:**

```bash
aws emr list-clusters \
    --region {region} \
    --cluster-states RUNNING WAITING \
    --query 'Clusters[?Status.State==`RUNNING` || Status.State==`WAITING`]' \
    --output table
```

**Recent Clusters (Last 7 days):**

```bash
# Calculate date 7 days ago
start_date=$(date -d '7 days ago' -u +%Y-%m-%dT%H:%M:%SZ)

aws emr list-clusters \
    --region {region} \
    --created-after $start_date \
    --output table
```

**Spark-Specific Clusters:**

```bash
# Find clusters with Spark installed
aws emr list-clusters \
    --region {region} \
    --query 'Clusters[?contains(Applications[].Name, `Spark`)]' \
    --output table
```

### Phase 4: Cost and Resource Analysis

**Step 6: Cluster Resource Summary**

```bash
# Get instance count and types for each cluster
for cluster_id in $(aws emr list-clusters --region {region} --cluster-states RUNNING --query 'Clusters[].Id' --output text); do
    echo "Cluster: $cluster_id"
    aws emr describe-cluster \
        --region {region} \
        --cluster-id $cluster_id \
        --query 'Cluster.{
            Name:Name,
            State:Status.State,
            MasterInstanceType:Ec2InstanceAttributes.Ec2InstanceType,
            InstanceCount:Ec2InstanceAttributes.RequestedEc2AvailabilityZones | length,
            Applications:Applications[].Name
        }' \
        --output table
    echo "---"
done
```

**Step 7: Spot Instance Usage Analysis**

```bash
# Check spot instance usage
for cluster_id in $(aws emr list-clusters --region {region} --cluster-states RUNNING --query 'Clusters[].Id' --output text); do
    echo "Analyzing spot usage for cluster: $cluster_id"
    aws emr list-instances \
        --region {region} \
        --cluster-id $cluster_id \
        --query 'Instances[*].{
            InstanceId:Ec2InstanceId,
            Type:InstanceType,
            Market:Market,
            State:Status.State,
            SpotPrice:SpotInstanceRequestId
        }' \
        --output table
done
```

## Output Analysis

### Cluster Discovery Summary

**Step 8: Generate Discovery Report**

**Summary Statistics:**

```bash
# Count clusters by state
aws emr list-clusters \
    --region {region} \
    --query 'Clusters | group_by(@, &Status.State) | map(&{State: [0].Status.State, Count: length(@)}, @)' \
    --output table
```

**Resource Utilization Overview:**

```bash
# Analyze resource distribution
total_clusters=$(aws emr list-clusters --region {region} --query 'length(Clusters)' --output text)
running_clusters=$(aws emr list-clusters --region {region} --cluster-states RUNNING --query 'length(Clusters)' --output text)
terminated_clusters=$(aws emr list-clusters --region {region} --cluster-states TERMINATED --query 'length(Clusters)' --output text)

echo "EMR Cluster Summary for Region: {region}"
echo "=================================="
echo "Total Clusters: $total_clusters"
echo "Running Clusters: $running_clusters"
echo "Terminated Clusters: $terminated_clusters"
echo "Active Percentage: $(echo "scale=2; $running_clusters * 100 / $total_clusters" | bc)%"
```

### Detailed Cluster Information

**Step 9: Export Detailed Data**

```bash
# Export comprehensive cluster data to JSON
aws emr list-clusters \
    --region {region} \
    --query 'Clusters[*].{
        ClusterID: Id,
        ClusterName: Name,
        State: Status.State,
        StateChangeReason: Status.StateChangeReason.Message,
        CreationTime: Status.Timeline.CreationDateTime,
        ReadyTime: Status.Timeline.ReadyDateTime,
        EndTime: Status.Timeline.EndDateTime,
        EMRVersion: ReleaseLabel,
        Applications: Applications[].Name | join(`, `, @),
        LogURI: LogUri,
        ServiceRole: ServiceRole,
        AutoScaling: AutoScalingRole,
        VisibleToAll: VisibleToAllUsers,
        TerminationProtected: TerminationProtected
    }' \
    --output json > emr_clusters_{region}_$(date +%Y%m%d_%H%M%S).json
```

**Step 10: Create Cluster Inventory**

```bash
# Create detailed inventory with instance information
for cluster_id in $(aws emr list-clusters --region {region} --query 'Clusters[].Id' --output text); do
    echo "Processing cluster: $cluster_id"

    # Get cluster details
    cluster_info=$(aws emr describe-cluster --region {region} --cluster-id $cluster_id)

    # Get instance details
    instances=$(aws emr list-instances --region {region} --cluster-id $cluster_id)

    # Combine information
    echo "$cluster_info" | jq --argjson instances "$instances" '.Cluster + {InstanceDetails: $instances.Instances}'
done > detailed_cluster_inventory_{region}_$(date +%Y%m%d_%H%M%S).json
```

## Analysis Integration

### Step 11: Prepare for Further Analysis

**Identify Analysis Targets:**

```bash
# Find clusters suitable for performance analysis
aws emr list-clusters \
    --region {region} \
    --cluster-states RUNNING TERMINATED \
    --query 'Clusters[?contains(Applications[].Name, `Spark`) && Status.Timeline.CreationDateTime >= `2024-01-01`].{
        ID: Id,
        Name: Name,
        State: Status.State,
        Created: Status.Timeline.CreationDateTime,
        EMRVersion: ReleaseLabel
    }' \
    --output table
```

**Generate Command References:**

```bash
# Create ready-to-use commands for discovered clusters
echo "# Available EMR Clusters for Analysis"
echo "# Generated on: $(date)"
echo

aws emr list-clusters --region {region} --query 'Clusters[].[Id,Name,Status.State]' --output text | \
while read cluster_id name state; do
    echo "# Cluster: $name ($state)"
    echo "*describe-emr-cluster $cluster_id"
    echo "*query-emr-steps $cluster_id"
    echo "*check-emr-instances $cluster_id"
    if [ "$state" = "RUNNING" ] || [ "$state" = "WAITING" ]; then
        echo "*fetch-emr-logs $cluster_id"
    fi
    echo
done
```

## Troubleshooting

### Common Issues

**No Clusters Found:**

```bash
# Check different time ranges
aws emr list-clusters --region {region} --created-after 2023-01-01
aws emr list-clusters --region {region} --created-after 2022-01-01

# Check all cluster states
aws emr list-clusters --region {region} --cluster-states STARTING BOOTSTRAPPING RUNNING WAITING TERMINATING TERMINATED TERMINATED_WITH_ERRORS
```

**Permission Errors:**

```bash
# Verify EMR permissions
aws iam simulate-principal-policy \
    --policy-source-arn $(aws sts get-caller-identity --query Arn --output text) \
    --action-names elasticmapreduce:ListClusters \
    --resource-arns "*"
```

**Region Access Issues:**

```bash
# Test region accessibility
aws emr describe-release-label --region {region} --release-label emr-6.15.0
```

## Output Formats

### Summary Report Template

```
EMR Cluster Discovery Report
===========================
Region: {region}
Generated: $(date)
Search Criteria: {criteria}

Cluster Summary:
- Total Clusters Found: {count}
- Running: {running_count}
- Terminated: {terminated_count}
- Failed: {failed_count}

Top 5 Most Recent Clusters:
{cluster_list}

Recommended Actions:
- Use *describe-emr-cluster for detailed analysis
- Use *query-emr-steps for step performance
- Use *aws-cost-analysis for cost optimization
```

## Follow-up Actions

Based on discovered clusters, you can:

1. **Detailed Analysis**: Use `*describe-emr-cluster {cluster-id}` for specific clusters
2. **Performance Review**: Use `*analyze-spark-job {app-id}` for Spark applications
3. **Cost Optimization**: Use `*aws-cost-analysis {cluster-id}` for cost analysis
4. **Health Check**: Use `*check-emr-instances {cluster-id}` for instance analysis
5. **Log Investigation**: Use `*fetch-emr-logs {cluster-id}` for log analysis

## Export and Documentation

**Final Step: Document Findings**

```bash
# Create summary report
{
    echo "# EMR Cluster Discovery Summary"
    echo "Date: $(date)"
    echo "Region: {region}"
    echo
    echo "## Clusters Found:"
    aws emr list-clusters --region {region} --output table
    echo
    echo "## Next Steps:"
    echo "1. Select clusters for detailed analysis"
    echo "2. Run performance analysis on Spark applications"
    echo "3. Analyze costs and optimization opportunities"
} > emr_discovery_report_$(date +%Y%m%d_%H%M%S).md
```

Your EMR clusters are now catalogued and ready for comprehensive analysis!
