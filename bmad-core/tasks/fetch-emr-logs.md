# Fetch EMR Logs

**Purpose**: Retrieve and analyze EMR cluster logs to identify issues, performance problems, and system-level insights.

**Elicit**: true

## Input Requirements

Please provide the following information:

1. **EMR Cluster ID**: 
   - Format: `j-XXXXXXXXXXXXX`
   - Example: `j-8P7Z16NUDD4H`

2. **AWS Region**: 
   - Example: `us-west-2`

3. **Log Types to Fetch** (select one or more):
   - [ ] Application logs (Spark driver/executor logs)
   - [ ] System logs (EMR cluster logs)
   - [ ] Bootstrap action logs
   - [ ] Step logs
   - [ ] Hadoop/YARN logs
   - [ ] All logs

4. **Time Range** (optional):
   - Start time: `YYYY-MM-DD HH:MM:SS`
   - End time: `YYYY-MM-DD HH:MM:SS`
   - Or: Last N hours/days

5. **Analysis Focus**:
   - [ ] Error investigation
   - [ ] Performance analysis
   - [ ] Resource utilization
   - [ ] Configuration issues
   - [ ] Network problems
   - [ ] Storage issues

## Log Retrieval Workflow

### Phase 1: Cluster Information Gathering

**Step 1: Verify Cluster Access**
```bash
aws emr describe-cluster --cluster-id {cluster-id} --region {region}
```

**Step 2: Get Cluster Details**
```bash
aws emr list-instances --cluster-id {cluster-id} --region {region}
```

**Step 3: Check Log URI**
```bash
# Get S3 log URI from cluster description
aws emr describe-cluster --cluster-id {cluster-id} --region {region} \
  --query 'Cluster.LogUri' --output text
```

### Phase 2: Log Collection Strategy

**Step 4: Determine Log Locations**

**Application Logs**:
- Path: `s3://{log-bucket}/logs/{cluster-id}/containers/`
- Contains: Spark driver/executor logs, application stderr/stdout

**System Logs**:
- Path: `s3://{log-bucket}/logs/{cluster-id}/node/`
- Contains: EMR system logs, bootstrap logs, daemon logs

**Step Logs**:
- Path: `s3://{log-bucket}/logs/{cluster-id}/steps/`
- Contains: Individual step execution logs

### Phase 3: Log Retrieval

**Step 5: Download Application Logs**
```bash
# Create local directory
mkdir -p ./emr-logs/{cluster-id}/applications

# Download Spark application logs
aws s3 sync s3://{log-bucket}/logs/{cluster-id}/containers/ \
  ./emr-logs/{cluster-id}/applications/ \
  --region {region}
```

**Step 6: Download System Logs**
```bash
# Download system logs
aws s3 sync s3://{log-bucket}/logs/{cluster-id}/node/ \
  ./emr-logs/{cluster-id}/system/ \
  --region {region}
```

**Step 7: Download Step Logs**
```bash
# Download step logs
aws s3 sync s3://{log-bucket}/logs/{cluster-id}/steps/ \
  ./emr-logs/{cluster-id}/steps/ \
  --region {region}
```

### Phase 4: Log Analysis

**Step 8: Parse Application Logs**
```bash
# Find Spark driver logs
find ./emr-logs/{cluster-id}/applications -name "*driver*" -type f

# Find executor logs
find ./emr-logs/{cluster-id}/applications -name "*executor*" -type f

# Extract error patterns
grep -r "ERROR\|WARN\|Exception\|Failed" ./emr-logs/{cluster-id}/applications/
```

**Step 9: Analyze System Performance**
```bash
# Check resource manager logs
grep -r "OutOfMemory\|GC\|CPU" ./emr-logs/{cluster-id}/system/

# Check disk space issues
grep -r "No space left\|disk full" ./emr-logs/{cluster-id}/system/

# Network connectivity issues
grep -r "Connection\|timeout\|refused" ./emr-logs/{cluster-id}/system/
```

**Step 10: Step Execution Analysis**
```bash
# Check step failures
grep -r "FAILED\|ERROR" ./emr-logs/{cluster-id}/steps/

# Check step performance
grep -r "duration\|time" ./emr-logs/{cluster-id}/steps/
```

### Phase 5: Log Pattern Analysis

**Step 11: Common Error Patterns**

**OutOfMemory Errors**:
```bash
grep -r "java.lang.OutOfMemoryError" ./emr-logs/{cluster-id}/ | head -20
```

**Shuffle Failures**:
```bash
grep -r "shuffle\|FetchFailedException" ./emr-logs/{cluster-id}/ | head -20
```

**Task Failures**:
```bash
grep -r "Task.*failed\|TaskSetManager" ./emr-logs/{cluster-id}/ | head -20
```

**Configuration Issues**:
```bash
grep -r "Configuration\|property\|setting" ./emr-logs/{cluster-id}/ | head -20
```

### Phase 6: Performance Metrics Extraction

**Step 12: Resource Utilization**
```bash
# CPU utilization patterns
grep -r "CPU\|processor" ./emr-logs/{cluster-id}/system/

# Memory usage patterns
grep -r "memory\|heap\|GC" ./emr-logs/{cluster-id}/

# Disk I/O patterns
grep -r "disk\|I/O\|read\|write" ./emr-logs/{cluster-id}/system/
```

**Step 13: Network Analysis**
```bash
# Network connectivity
grep -r "network\|connection\|socket" ./emr-logs/{cluster-id}/

# Data transfer rates
grep -r "transfer\|bandwidth\|throughput" ./emr-logs/{cluster-id}/
```

## Log Analysis Output

### Executive Summary
- **Log Collection Status**: Success/Partial/Failed
- **Total Log Size**: X GB
- **Time Range Covered**: Start - End
- **Critical Issues Found**: Number of errors/warnings
- **Performance Indicators**: Key metrics identified

### Error Analysis
**Critical Errors** (Count: X):
1. Error type and frequency
2. Affected components
3. Time patterns
4. Potential causes

**Warnings** (Count: Y):
1. Warning categories
2. Frequency patterns
3. Impact assessment
4. Recommendations

### Performance Insights
**Resource Utilization**:
- Peak CPU usage: X%
- Memory consumption: Y GB
- Disk I/O patterns: Z MB/s
- Network utilization: W Mbps

**Bottleneck Identification**:
1. Primary bottlenecks found
2. Resource constraints
3. Configuration issues
4. Timing problems

### System Health Indicators
**Node Performance**:
- Master node status
- Core node performance
- Task node utilization
- Spot instance interruptions

**Service Status**:
- YARN ResourceManager
- Spark History Server
- HDFS NameNode
- Other EMR services

### Recommendations

**Immediate Actions**:
1. Critical fixes needed
2. Configuration changes
3. Resource adjustments
4. Code modifications

**Optimization Opportunities**:
1. Performance improvements
2. Cost optimizations
3. Reliability enhancements
4. Monitoring additions

## Integration with MCP Analysis

**Step 14: Correlate with Spark History Server**
```
# Use MCP tools to correlate findings:
# 1. *analyze-spark-job {app-id} - for application-level analysis
# 2. *cluster-health-check {cluster-id} - for cluster-level insights
# 3. *compare-jobs {app-id-1} {app-id-2} - for performance comparison
```

## Troubleshooting Log Retrieval

### Common Issues

**S3 Access Denied**:
```bash
# Check S3 permissions
aws s3 ls s3://{log-bucket}/logs/{cluster-id}/ --region {region}

# Required permissions:
# - s3:GetObject
# - s3:ListBucket
```

**Logs Not Found**:
```bash
# Verify log URI configuration
aws emr describe-cluster --cluster-id {cluster-id} --region {region} \
  --query 'Cluster.LogUri'

# Check if logging was enabled during cluster creation
```

**Large Log Sizes**:
```bash
# Use selective download with filters
aws s3 sync s3://{log-bucket}/logs/{cluster-id}/ \
  ./emr-logs/{cluster-id}/ \
  --exclude "*" \
  --include "*error*" \
  --include "*exception*" \
  --region {region}
```

**Timeout Issues**:
```bash
# Use parallel downloads
aws configure set max_concurrent_requests 10
aws configure set max_bandwidth 100MB/s
```

## Automated Log Analysis Scripts

**Step 15: Generate Analysis Scripts**

Create automated analysis scripts for:
1. Error pattern extraction
2. Performance metric calculation
3. Timeline reconstruction
4. Correlation analysis
5. Report generation

## Follow-up Actions

Based on log analysis results:

1. **Performance Issues**: Use `*optimize-configuration`
2. **Application Problems**: Use `*analyze-spark-job {app-id}`
3. **Code Issues**: Use `*analyze-code`
4. **Cluster Problems**: Use `*cluster-health-check {cluster-id}`
5. **Comparison Analysis**: Use `*compare-jobs` for before/after analysis

## Documentation

Generate comprehensive log analysis report with:
- Summary of findings
- Detailed error analysis
- Performance metrics
- Recommendations
- Follow-up actions
- Log retention policies
