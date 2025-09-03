# Describe EMR Cluster

**Purpose**: Get comprehensive information about a specific EMR cluster including configuration, instances, applications, and performance metrics.

**Elicit**: true

## Input Requirements

Please provide the following information:

1. **EMR Cluster ID**:
   - Format: `j-XXXXXXXXXXXXX`
   - Example: `j-8P7Z16NUDD4H`

2. **AWS Region**:
   - Example: `us-west-2`
   - Or: Use default configured region

3. **Analysis Depth** (select one or more):
   - [ ] Basic cluster information
   - [ ] Instance group details
   - [ ] Application configuration
   - [ ] Network and security settings
   - [ ] Auto-scaling configuration
   - [ ] Cost analysis data
   - [ ] Performance metrics
   - [ ] Complete comprehensive analysis

4. **Output Format**:
   - [ ] Detailed JSON for further processing
   - [ ] Summary table for quick review
   - [ ] Formatted report for documentation
   - [ ] All formats

## Cluster Analysis Workflow

### Phase 1: Basic Cluster Information

**Step 1: Get Core Cluster Details**

```bash
# Basic cluster information
aws emr describe-cluster \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'Cluster.{
        ID: Id,
        Name: Name,
        State: Status.State,
        StateReason: Status.StateChangeReason.Message,
        EMRVersion: ReleaseLabel,
        CreatedTime: Status.Timeline.CreationDateTime,
        ReadyTime: Status.Timeline.ReadyDateTime,
        EndTime: Status.Timeline.EndDateTime
    }' \
    --output table
```

**Step 2: Get Cluster Applications**

```bash
# List installed applications
aws emr describe-cluster \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'Cluster.Applications[*].{
        Name: Name,
        Version: Version,
        Args: Args
    }' \
    --output table
```

### Phase 2: Instance and Resource Analysis

**Step 3: Instance Group Configuration**

```bash
# Get instance group details
aws emr list-instance-groups \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'InstanceGroups[*].{
        Name: Name,
        Type: InstanceGroupType,
        InstanceType: InstanceType,
        RequestedCount: RequestedInstanceCount,
        RunningCount: RunningInstanceCount,
        State: Status.State,
        Market: Market,
        BidPrice: BidPrice,
        AutoScaling: AutoScalingPolicy.Status.State
    }' \
    --output table
```

**Step 4: Individual Instance Details**

```bash
# Get detailed instance information
aws emr list-instances \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'Instances[*].{
        InstanceId: Ec2InstanceId,
        Type: InstanceType,
        State: Status.State,
        PrivateIP: PrivateIpAddress,
        PublicIP: PublicIpAddress,
        Market: Market,
        SpotRequestId: SpotInstanceRequestId,
        InstanceGroupId: InstanceGroupId
    }' \
    --output table
```

**Step 5: Instance Fleet Analysis (if applicable)**

```bash
# Check if cluster uses instance fleets
aws emr list-instance-fleets \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'InstanceFleets[*].{
        Name: Name,
        FleetType: InstanceFleetType,
        TargetOnDemand: TargetOnDemandCapacity,
        TargetSpot: TargetSpotCapacity,
        ProvisionedOnDemand: ProvisionedOnDemandCapacity,
        ProvisionedSpot: ProvisionedSpotCapacity,
        State: Status.State
    }' \
    --output table 2>/dev/null || echo "Cluster uses instance groups, not fleets"
```

### Phase 3: Configuration Analysis

**Step 6: Cluster Configuration Details**

```bash
# Get comprehensive cluster configuration
aws emr describe-cluster \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'Cluster.{
        ServiceRole: ServiceRole,
        AutoScalingRole: AutoScalingRole,
        LogUri: LogUri,
        TerminationProtected: TerminationProtected,
        VisibleToAllUsers: VisibleToAllUsers,
        KerberosAttributes: KerberosAttributes,
        ClusterArn: ClusterArn,
        OutpostArn: OutpostArn,
        StepConcurrencyLevel: StepConcurrencyLevel,
        PlacementGroupConfigs: PlacementGroupConfigs
    }' \
    --output json
```

**Step 7: Network and Security Configuration**

```bash
# Get EC2 and security configuration
aws emr describe-cluster \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'Cluster.Ec2InstanceAttributes.{
        KeyName: Ec2KeyName,
        InstanceProfile: IamInstanceProfile,
        SubnetId: Ec2SubnetId,
        AvailabilityZone: Ec2AvailabilityZone,
        SecurityGroups: {
            Master: EmrManagedMasterSecurityGroup,
            Slave: EmrManagedSlaveSecurityGroup,
            Service: ServiceAccessSecurityGroup,
            Additional: AdditionalMasterSecurityGroups
        }
    }' \
    --output json
```

### Phase 4: Performance and Scaling Analysis

**Step 8: Auto-Scaling Configuration**

```bash
# Check managed scaling policy
aws emr get-managed-scaling-policy \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'ManagedScalingPolicy.{
        MinCapacity: ComputeLimits.MinimumCapacityUnits,
        MaxCapacity: ComputeLimits.MaximumCapacityUnits,
        MaxOnDemand: ComputeLimits.MaximumOnDemandCapacityUnits,
        MaxCore: ComputeLimits.MaximumCoreCapacityUnits,
        UnitType: ComputeLimits.UnitType
    }' \
    --output table 2>/dev/null || echo "No managed scaling policy configured"
```

**Step 9: Step Execution Analysis**

```bash
# Get recent step execution summary
aws emr list-steps \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'Steps[*].{
        StepId: Id,
        Name: Name,
        State: Status.State,
        StartTime: Status.Timeline.StartDateTime,
        EndTime: Status.Timeline.EndDateTime,
        ActionOnFailure: ActionOnFailure
    }' \
    --output table
```

### Phase 5: Cost Analysis Preparation

**Step 10: Instance Cost Analysis**

```bash
# Get instance types for cost calculation
instance_types=$(aws emr list-instances \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'Instances[].InstanceType' \
    --output text | sort | uniq)

echo "Instance types in cluster:"
for instance_type in $instance_types; do
    count=$(aws emr list-instances \
        --region {region} \
        --cluster-id {cluster-id} \
        --query "length(Instances[?InstanceType=='$instance_type'])" \
        --output text)
    echo "$instance_type: $count instances"
done
```

**Step 11: Spot Instance Analysis**

```bash
# Analyze spot instance usage and pricing
aws emr list-instances \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'Instances[?Market==`SPOT`].{
        InstanceId: Ec2InstanceId,
        Type: InstanceType,
        SpotRequestId: SpotInstanceRequestId,
        State: Status.State
    }' \
    --output table

# Get current spot prices for cluster instance types
for instance_type in $instance_types; do
    echo "Current spot price for $instance_type:"
    aws ec2 describe-spot-price-history \
        --region {region} \
        --instance-types $instance_type \
        --max-items 1 \
        --query 'SpotPriceHistory[0].{Price:SpotPrice,AZ:AvailabilityZone,Timestamp:Timestamp}' \
        --output table
done
```

### Phase 6: Application-Specific Analysis

**Step 12: Spark Configuration Analysis**

```bash
# Check if Spark is installed and get configuration
spark_installed=$(aws emr describe-cluster \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'Cluster.Applications[?Name==`Spark`] | length(@)' \
    --output text)

if [ "$spark_installed" -gt 0 ]; then
    echo "Spark is installed. Getting Spark-specific information..."

    # Get Spark version
    aws emr describe-cluster \
        --region {region} \
        --cluster-id {cluster-id} \
        --query 'Cluster.Applications[?Name==`Spark`].{
            Name: Name,
            Version: Version
        }' \
        --output table

    # Check for Spark History Server configuration
    echo "Checking for Spark History Server configuration..."
    aws emr list-steps \
        --region {region} \
        --cluster-id {cluster-id} \
        --query 'Steps[?contains(Name, `Spark`) || contains(Name, `spark`)].{
            StepId: Id,
            Name: Name,
            State: Status.State
        }' \
        --output table
fi
```

**Step 13: Hadoop/YARN Configuration**

```bash
# Check Hadoop ecosystem applications
aws emr describe-cluster \
    --region {region} \
    --cluster-id {cluster-id} \
    --query 'Cluster.Applications[?contains([`Hadoop`, `Yarn`, `Spark`, `Hive`, `Pig`, `HBase`], Name)].{
        Application: Name,
        Version: Version
    }' \
    --output table
```

## Comprehensive Analysis Report

### Step 14: Generate Detailed Report

**Create Comprehensive JSON Report:**

```bash
# Generate complete cluster analysis
{
    echo "{"
    echo '"cluster_basic_info":'
    aws emr describe-cluster --region {region} --cluster-id {cluster-id} --query 'Cluster'
    echo ','
    echo '"instance_groups":'
    aws emr list-instance-groups --region {region} --cluster-id {cluster-id} --query 'InstanceGroups'
    echo ','
    echo '"instances":'
    aws emr list-instances --region {region} --cluster-id {cluster-id} --query 'Instances'
    echo ','
    echo '"steps":'
    aws emr list-steps --region {region} --cluster-id {cluster-id} --query 'Steps'
    echo ','
    echo '"bootstrap_actions":'
    aws emr list-bootstrap-actions --region {region} --cluster-id {cluster-id} --query 'BootstrapActions' 2>/dev/null || echo 'null'
    echo "}"
} > cluster_analysis_{cluster-id}_$(date +%Y%m%d_%H%M%S).json
```

### Step 15: Performance Metrics Summary

**Calculate Key Performance Indicators:**

```bash
# Calculate cluster utilization metrics
cluster_info=$(aws emr describe-cluster --region {region} --cluster-id {cluster-id})

# Extract timing information
creation_time=$(echo "$cluster_info" | jq -r '.Cluster.Status.Timeline.CreationDateTime')
ready_time=$(echo "$cluster_info" | jq -r '.Cluster.Status.Timeline.ReadyDateTime')
end_time=$(echo "$cluster_info" | jq -r '.Cluster.Status.Timeline.EndDateTime // "null"')

echo "Cluster Performance Summary:"
echo "============================"
echo "Cluster ID: {cluster-id}"
echo "Creation Time: $creation_time"
echo "Ready Time: $ready_time"
echo "End Time: $end_time"

# Calculate provisioning time
if [ "$ready_time" != "null" ] && [ "$creation_time" != "null" ]; then
    provisioning_seconds=$(( $(date -d "$ready_time" +%s) - $(date -d "$creation_time" +%s) ))
    echo "Provisioning Time: $provisioning_seconds seconds ($(($provisioning_seconds/60)) minutes)"
fi

# Get instance counts
total_instances=$(aws emr list-instances --region {region} --cluster-id {cluster-id} --query 'length(Instances)' --output text)
running_instances=$(aws emr list-instances --region {region} --cluster-id {cluster-id} --query 'length(Instances[?Status.State==`RUNNING`])' --output text)

echo "Total Instances: $total_instances"
echo "Running Instances: $running_instances"
echo "Instance Utilization: $(echo "scale=2; $running_instances * 100 / $total_instances" | bc)%"
```

### Step 16: Configuration Recommendations

**Analyze Configuration for Optimization:**

```bash
# Check for common optimization opportunities
echo "Configuration Analysis:"
echo "======================"

# Check termination protection
termination_protected=$(echo "$cluster_info" | jq -r '.Cluster.TerminationProtected')
echo "Termination Protection: $termination_protected"

# Check log URI
log_uri=$(echo "$cluster_info" | jq -r '.Cluster.LogUri // "Not configured"')
echo "Log URI: $log_uri"

# Check auto-scaling
auto_scaling_role=$(echo "$cluster_info" | jq -r '.Cluster.AutoScalingRole // "Not configured"')
echo "Auto-Scaling Role: $auto_scaling_role"

# Check step concurrency
step_concurrency=$(echo "$cluster_info" | jq -r '.Cluster.StepConcurrencyLevel // "1"')
echo "Step Concurrency Level: $step_concurrency"

# Recommendations based on configuration
echo
echo "Recommendations:"
echo "==============="
if [ "$log_uri" = "Not configured" ]; then
    echo "⚠️  Consider configuring log URI for better debugging capabilities"
fi

if [ "$auto_scaling_role" = "Not configured" ]; then
    echo "💡 Consider enabling auto-scaling for cost optimization"
fi

if [ "$step_concurrency" = "1" ]; then
    echo "💡 Consider increasing step concurrency for better throughput"
fi
```

## Integration with Other Analysis

### Step 17: Prepare for Advanced Analysis

**Generate Analysis Commands:**

```bash
echo "# Follow-up Analysis Commands for Cluster {cluster-id}"
echo "# Generated on: $(date)"
echo

# Check if cluster has Spark applications
spark_steps=$(aws emr list-steps --region {region} --cluster-id {cluster-id} --query 'Steps[?contains(Name, `spark`) || contains(Name, `Spark`)].Id' --output text)

if [ ! -z "$spark_steps" ]; then
    echo "# Spark Applications Found - Use MCP Analysis:"
    for step_id in $spark_steps; do
        echo "*analyze-spark-job application_id_from_step_${step_id}"
    done
    echo
fi

echo "# EMR-Specific Analysis Commands:"
echo "*query-emr-steps {cluster-id}"
echo "*check-emr-instances {cluster-id}"
echo "*fetch-emr-logs {cluster-id}"
echo "*aws-cost-analysis {cluster-id}"
echo

# Check if cluster is suitable for performance comparison
cluster_state=$(echo "$cluster_info" | jq -r '.Cluster.Status.State')
if [ "$cluster_state" = "TERMINATED" ] || [ "$cluster_state" = "RUNNING" ]; then
    echo "# Cluster is suitable for performance comparison:"
    echo "*compare-jobs {app-id-1} {app-id-2}  # Compare with other applications"
fi
```

## Troubleshooting

### Common Issues and Solutions

**Cluster Not Found:**

```bash
# Verify cluster ID format and region
aws emr list-clusters --region {region} --query 'Clusters[].Id' --output text | grep {cluster-id}

# Try different regions
for region in us-east-1 us-west-2 eu-west-1; do
    echo "Checking region: $region"
    aws emr describe-cluster --region $region --cluster-id {cluster-id} --query 'Cluster.Id' --output text 2>/dev/null && echo "Found in $region"
done
```

**Permission Errors:**

```bash
# Check EMR permissions
aws iam simulate-principal-policy \
    --policy-source-arn $(aws sts get-caller-identity --query Arn --output text) \
    --action-names elasticmapreduce:DescribeCluster elasticmapreduce:ListInstances \
    --resource-arns "*"
```

## Output Summary

### Final Analysis Report

```
EMR Cluster Analysis Summary
============================
Cluster ID: {cluster-id}
Region: {region}
Analysis Date: $(date)

Basic Information:
- Name: {cluster-name}
- State: {cluster-state}
- EMR Version: {emr-version}
- Applications: {applications}

Resource Configuration:
- Instance Groups: {instance-group-count}
- Total Instances: {total-instances}
- Instance Types: {instance-types}
- Spot Usage: {spot-percentage}%

Performance Metrics:
- Provisioning Time: {provisioning-time}
- Uptime: {uptime}
- Step Success Rate: {step-success-rate}%

Cost Analysis:
- Estimated Hourly Cost: ${estimated-cost}/hour
- Total Runtime Cost: ${total-cost}
- Optimization Potential: {optimization-percentage}%

Recommendations:
{recommendations-list}

Next Steps:
{next-steps}
```

This comprehensive cluster analysis provides the foundation for performance optimization, cost analysis, and troubleshooting activities.
