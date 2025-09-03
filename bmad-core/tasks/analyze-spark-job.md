# Analyze Spark Job

**Purpose**: Perform comprehensive analysis of a Spark application using MCP tools to identify performance issues, bottlenecks, and optimization opportunities.

**Elicit**: true

## Input Requirements

Please provide the following information:

1. **Spark Application ID**: 
   - Format: `application_XXXXXXXXXXXXXXXX_XXXX` or `spark-XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX`
   - Example: `application_1234567890123_0001`

2. **Analysis Focus** (select one or more):
   - [ ] Performance bottlenecks
   - [ ] Resource utilization
   - [ ] Job failure analysis
   - [ ] SQL query optimization
   - [ ] Configuration review
   - [ ] Executor analysis
   - [ ] Stage-level deep dive

3. **Comparison Baseline** (optional):
   - Previous application ID for comparison
   - Expected performance baseline

## Analysis Workflow

### Phase 1: Application Overview

**Step 1: Get Application Details**
```
Using MCP tool: get_application
- Application metadata
- Status and duration
- Resource allocation
- Attempt details
```

**Step 2: Environment Configuration**
```
Using MCP tool: get_environment
- Spark configuration
- JVM settings
- System properties
- Classpath information
```

### Phase 2: Job-Level Analysis

**Step 3: Job Performance Analysis**
```
Using MCP tool: list_jobs
- All jobs in application
- Job status and duration
- Success/failure rates

Using MCP tool: list_slowest_jobs
- Top N slowest jobs
- Performance metrics
- Resource consumption
```

### Phase 3: Stage-Level Deep Dive

**Step 4: Stage Analysis**
```
Using MCP tool: list_stages
- All stages with status
- Stage dependencies
- Performance summaries

Using MCP tool: list_slowest_stages
- Bottleneck stages
- Task distribution
- Resource usage patterns
```

**Step 5: Task-Level Metrics**
```
Using MCP tool: get_stage_task_summary
For each slow stage:
- Task execution time distribution
- Memory usage patterns
- I/O metrics
- Shuffle statistics
```

### Phase 4: Resource Analysis

**Step 6: Executor Performance**
```
Using MCP tool: list_executors
- Executor allocation
- Active vs inactive executors
- Resource distribution

Using MCP tool: get_executor_summary
- Aggregated metrics
- Memory usage patterns
- Task distribution
- Performance statistics
```

**Step 7: Resource Timeline**
```
Using MCP tool: get_resource_usage_timeline
- Executor allocation over time
- Resource scaling patterns
- Peak usage identification
```

### Phase 5: SQL and Query Analysis

**Step 8: SQL Performance** (if applicable)
```
Using MCP tool: list_slowest_sql_queries
- Slowest SQL queries
- Execution metrics
- Plan analysis
```

### Phase 6: Bottleneck Identification

**Step 9: Comprehensive Bottleneck Analysis**
```
Using MCP tool: get_job_bottlenecks
- Automated bottleneck detection
- Performance recommendations
- Actionable insights
```

### Phase 7: Comparative Analysis (if baseline provided)

**Step 10: Performance Comparison**
```
Using MCP tool: compare_job_performance
- Execution time comparison
- Resource usage deltas
- Performance regression analysis

Using MCP tool: compare_job_environments
- Configuration differences
- Environment changes
- Setting impacts
```

## Analysis Output Structure

### Executive Summary
- **Application Status**: Success/Failure/Running
- **Total Duration**: X minutes Y seconds
- **Resource Efficiency**: High/Medium/Low
- **Primary Bottlenecks**: Top 3 issues identified
- **Optimization Potential**: Estimated improvement %

### Performance Metrics
- **Job Statistics**:
  - Total jobs: X
  - Failed jobs: Y
  - Average job duration: Z seconds
  
- **Stage Statistics**:
  - Total stages: X
  - Slowest stage: Y (Z seconds)
  - Stage failure rate: W%

- **Task Statistics**:
  - Total tasks: X
  - Failed tasks: Y
  - Average task duration: Z ms
  - Task skew factor: W

### Resource Utilization
- **Executor Metrics**:
  - Peak executors: X
  - Average CPU utilization: Y%
  - Memory efficiency: Z%
  - Disk I/O patterns: W GB/s

- **Memory Analysis**:
  - Peak memory usage: X GB
  - Memory spill events: Y
  - GC time percentage: Z%

### Bottleneck Analysis
1. **Primary Bottleneck**: Description and impact
2. **Secondary Issues**: Additional performance concerns
3. **Resource Constraints**: CPU, memory, I/O limitations
4. **Configuration Issues**: Suboptimal settings identified

### Optimization Recommendations

**High Priority**:
1. Specific recommendation with expected impact
2. Configuration changes needed
3. Code modifications required

**Medium Priority**:
1. Additional optimizations
2. Resource scaling suggestions
3. Architecture improvements

**Low Priority**:
1. Fine-tuning opportunities
2. Monitoring enhancements
3. Future considerations

### Code Analysis Integration

If user provides code repository path:
1. **Performance Anti-Patterns**: Identified in code
2. **Optimization Opportunities**: Code-level improvements
3. **Best Practice Violations**: Spark coding standards
4. **Resource Usage Patterns**: Code impact on performance

### Comparative Analysis (if applicable)

**Performance Delta**:
- Execution time change: +/-X%
- Resource usage change: +/-Y%
- Efficiency improvement: +/-Z%

**Configuration Changes**:
- Modified settings: List of changes
- Impact assessment: Positive/negative effects
- Recommendations: Suggested adjustments

## Follow-up Actions

Based on analysis results, suggest:

1. **Immediate Actions**: Critical fixes needed
2. **Configuration Tuning**: Use `*optimize-configuration`
3. **Code Review**: Use `*analyze-code` if repository provided
4. **Cluster Optimization**: Use `*cluster-health-check`
5. **Monitoring Setup**: Ongoing performance tracking

## Error Handling

If analysis encounters issues:

1. **MCP Connection Problems**: Guide through `*setup-mcp`
2. **Permission Issues**: AWS IAM troubleshooting
3. **Timeout Errors**: Increase timeout settings
4. **Memory Issues**: Enable hybrid store configuration
5. **Application Not Found**: Verify application ID and server configuration

## Documentation Output

Generate comprehensive report using `*performance-report {app-id}` template with:
- All analysis findings
- Detailed metrics
- Visualization suggestions
- Action items with priorities
- Future monitoring recommendations
