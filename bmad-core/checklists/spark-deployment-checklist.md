# Spark Deployment Checklist

## Pre-Deployment Validation

### MCP Setup
- [ ] MCP Apache Spark History Server installed and configured
- [ ] EMR cluster connection established and tested
- [ ] Cursor MCP configuration added to `.cursor/mcp.json`
- [ ] MCP tools tested with `get_application` command
- [ ] Authentication and permissions verified

### EMR Cluster Configuration
- [ ] EMR cluster properly sized for workload
- [ ] Appropriate instance types selected (compute vs memory optimized)
- [ ] Spot instances configured if cost optimization needed
- [ ] Auto-scaling policies configured appropriately
- [ ] Security groups allow necessary communication
- [ ] VPC and subnet configuration verified

### Spark Configuration
- [ ] `spark.executor.memory` set appropriately for instance type
- [ ] `spark.executor.cores` optimized for CPU utilization
- [ ] `spark.driver.memory` sufficient for application needs
- [ ] `spark.sql.shuffle.partitions` tuned for data size
- [ ] Adaptive Query Execution enabled (`spark.sql.adaptive.enabled=true`)
- [ ] Dynamic allocation configured if beneficial
- [ ] Kryo serialization enabled for performance
- [ ] Appropriate logging level set

### Data and Storage
- [ ] Input data accessible from EMR cluster
- [ ] Output paths have proper write permissions
- [ ] S3 bucket policies allow EMR access
- [ ] Data partitioning strategy optimized
- [ ] File formats chosen for performance (Parquet recommended)
- [ ] Compression settings optimized

### Application Code
- [ ] Code reviewed for performance anti-patterns
- [ ] Memory-intensive operations identified and optimized
- [ ] Caching strategy implemented where beneficial
- [ ] Error handling implemented
- [ ] Logging configured appropriately
- [ ] Unit tests pass
- [ ] Integration tests completed

## Deployment Execution

### Pre-Launch
- [ ] Application JAR/Python files uploaded to S3
- [ ] Dependencies verified and available
- [ ] Configuration files reviewed
- [ ] Launch script tested
- [ ] Monitoring dashboard prepared

### Launch Process
- [ ] EMR cluster started (if not persistent)
- [ ] Application submitted successfully
- [ ] Initial logs show normal startup
- [ ] Spark UI accessible
- [ ] Resource allocation as expected

### Initial Monitoring
- [ ] Application appears in Spark History Server
- [ ] MCP connection to application successful
- [ ] Key metrics baseline established
- [ ] No immediate errors in logs
- [ ] Resource utilization within expected ranges

## Post-Deployment Validation

### Performance Verification
- [ ] Application completed successfully OR is running as expected
- [ ] Total runtime within acceptable range
- [ ] Resource utilization efficient (>70% CPU, <90% memory)
- [ ] No excessive GC time (< 10% of total time)
- [ ] Minimal task failures and retries
- [ ] Data skew within acceptable limits

### Quality Assurance
- [ ] Output data validated for correctness
- [ ] Data quality checks passed
- [ ] Schema validation successful
- [ ] Row counts match expectations
- [ ] Business logic validation completed

### Monitoring Setup
- [ ] Performance monitoring alerts configured
- [ ] Log aggregation working
- [ ] Metrics collection enabled
- [ ] Dashboard updated with new application
- [ ] Notification channels tested

## Troubleshooting Checklist

### If Application Fails to Start
- [ ] Check EMR cluster status and logs
- [ ] Verify application JAR/files accessibility
- [ ] Review Spark configuration for conflicts
- [ ] Check resource availability on cluster
- [ ] Validate input data paths and permissions

### If Application Runs Slowly
- [ ] Use `*analyze-spark-job {app-id}` for comprehensive analysis
- [ ] Check for data skew using MCP tools
- [ ] Review resource utilization patterns
- [ ] Analyze stage-level performance
- [ ] Check for shuffle bottlenecks

### If Application Fails During Execution
- [ ] Use `*debug-job-failure {app-id}` for systematic analysis
- [ ] Review executor logs using `*fetch-emr-logs {cluster-id}`
- [ ] Check for OutOfMemory errors
- [ ] Analyze task failure patterns
- [ ] Review data quality issues

### If Resource Issues Occur
- [ ] Use `*cluster-health-check {cluster-id}` for cluster analysis
- [ ] Monitor CPU and memory utilization
- [ ] Check for disk space issues
- [ ] Review network connectivity
- [ ] Analyze executor allocation patterns

## Performance Optimization

### After Initial Deployment
- [ ] Baseline performance metrics captured
- [ ] Performance report generated using MCP analysis
- [ ] Bottlenecks identified and prioritized
- [ ] Optimization plan created
- [ ] Configuration tuning completed

### Continuous Improvement
- [ ] Regular performance reviews scheduled
- [ ] Comparative analysis with previous runs
- [ ] Code optimization opportunities identified
- [ ] Infrastructure scaling evaluated
- [ ] Cost optimization reviewed

## Documentation and Handoff

### Documentation
- [ ] Deployment guide updated
- [ ] Configuration parameters documented
- [ ] Performance baselines recorded
- [ ] Troubleshooting runbook updated
- [ ] Monitoring procedures documented

### Knowledge Transfer
- [ ] Operations team trained on monitoring
- [ ] Development team aware of performance characteristics
- [ ] Escalation procedures defined
- [ ] MCP tools usage documented
- [ ] Performance optimization playbook created

## Security and Compliance

### Security Validation
- [ ] Data encryption at rest and in transit
- [ ] Access controls properly configured
- [ ] Network security groups restrictive
- [ ] Audit logging enabled
- [ ] Sensitive data handling verified

### Compliance
- [ ] Data retention policies implemented
- [ ] Privacy requirements met
- [ ] Regulatory compliance validated
- [ ] Data lineage documented
- [ ] Backup and recovery procedures tested

## Sign-off

### Technical Approval
- [ ] Development team sign-off
- [ ] Operations team sign-off
- [ ] Performance benchmarks met
- [ ] Security review passed
- [ ] Documentation complete

### Business Approval
- [ ] Business requirements met
- [ ] Performance SLAs satisfied
- [ ] Cost targets achieved
- [ ] Quality standards met
- [ ] Go-live approval received

---

**Checklist Completed By:** _________________  
**Date:** _________________  
**Application ID:** _________________  
**EMR Cluster ID:** _________________
