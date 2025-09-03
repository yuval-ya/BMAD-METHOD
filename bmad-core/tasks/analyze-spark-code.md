# Analyze Spark Code

**Purpose**: Review user's Spark application code to identify performance issues, anti-patterns, and optimization opportunities.

**Elicit**: true

## Input Requirements

Please provide the following information:

1. **Code Repository Path**:
   - Local path to your Spark application code
   - Example: `/path/to/spark-project/`
   - Or: GitHub repository URL

2. **Programming Language**:
   - [ ] Scala
   - [ ] Python (PySpark)
   - [ ] Java
   - [ ] SQL
   - [ ] Mixed (specify which files)

3. **Application Type**:
   - [ ] Batch processing
   - [ ] Streaming (Structured Streaming)
   - [ ] ETL pipeline
   - [ ] Machine Learning
   - [ ] Data analytics
   - [ ] Other: ___________

4. **Analysis Focus** (select one or more):
   - [ ] Performance optimization
   - [ ] Memory usage patterns
   - [ ] Shuffle optimization
   - [ ] I/O efficiency
   - [ ] Resource utilization
   - [ ] Code quality
   - [ ] Best practices compliance
   - [ ] Error handling

5. **Known Issues** (optional):
   - Performance problems experienced
   - Error messages encountered
   - Resource constraints observed

## Code Analysis Workflow

### Phase 1: Code Discovery and Structure

**Step 1: Scan Code Repository**
```bash
# Find Spark application files
find {repository-path} -name "*.scala" -o -name "*.py" -o -name "*.java" -o -name "*.sql"

# Identify main application files
grep -r "SparkSession\|SparkContext\|spark.sql" {repository-path}
```

**Step 2: Analyze Project Structure**
```
# Check for:
- Main application entry points
- Configuration files
- Build files (pom.xml, build.sbt, requirements.txt)
- Resource files
- Test files
```

### Phase 2: Performance Anti-Pattern Detection

**Step 3: Identify Common Anti-Patterns**

**Inefficient Operations**:
```scala
// Anti-pattern: collect() on large datasets
val results = largeDataFrame.collect()

// Anti-pattern: count() in loops
for (i <- 1 to n) {
  val count = df.count() // Expensive operation in loop
}

// Anti-pattern: Unnecessary shuffles
df.groupBy("col1").count().groupBy("col2").sum("count")
```

**Memory Issues**:
```python
# Anti-pattern: Broadcasting large variables
large_dict = {...}  # Very large dictionary
broadcast_dict = spark.sparkContext.broadcast(large_dict)

# Anti-pattern: Caching everything
df.cache().count()  # Unnecessary caching
df2.cache().show()  # Caching rarely used data
```

**I/O Inefficiencies**:
```scala
// Anti-pattern: Reading same data multiple times
val df1 = spark.read.parquet("path/to/data")
val df2 = spark.read.parquet("path/to/data") // Duplicate read

// Anti-pattern: Small files problem
df.write.mode("overwrite").parquet("output/") // Creates many small files
```

### Phase 3: Resource Utilization Analysis

**Step 4: Analyze Resource Usage Patterns**

**CPU Utilization**:
- Identify CPU-intensive operations
- Look for unnecessary computations
- Check for proper parallelization

**Memory Usage**:
- Detect memory-intensive operations
- Identify caching strategies
- Look for memory leaks

**Network I/O**:
- Analyze shuffle operations
- Check join strategies
- Identify broadcast opportunities

### Phase 4: Spark-Specific Optimizations

**Step 5: DataFrame/RDD Operations Review**

**DataFrame Optimizations**:
```scala
// Check for proper column pruning
df.select("needed_col1", "needed_col2") // Good
df.select("*").filter(...) // Potentially inefficient

// Analyze predicate pushdown
df.filter($"date" > "2023-01-01").select(...) // Good
df.select(...).filter($"date" > "2023-01-01") // Less efficient
```

**Join Optimizations**:
```python
# Analyze join strategies
large_df.join(small_df, "key") # Check if broadcast join is better
large_df.join(broadcast(small_df), "key") # Explicit broadcast

# Check join conditions
df1.join(df2, df1.col("id") == df2.col("id")) # Equi-join
df1.join(df2, df1.col("id") > df2.col("id")) # Non-equi join (expensive)
```

**Aggregation Patterns**:
```scala
// Check aggregation efficiency
df.groupBy("key").agg(sum("value1"), avg("value2")) // Efficient
df.groupBy("key").sum("value1").join(df.groupBy("key").avg("value2")) // Inefficient
```

### Phase 5: Configuration Analysis

**Step 6: Review Spark Configuration**

**Configuration Files**:
```properties
# Check spark-defaults.conf
spark.sql.adaptive.enabled true
spark.sql.adaptive.coalescePartitions.enabled true
spark.sql.adaptive.skewJoin.enabled true

# Check application-specific configs
spark.executor.memory 4g
spark.executor.cores 2
spark.executor.instances 10
```

**Dynamic Configuration**:
```scala
// Check programmatic configuration
val spark = SparkSession.builder()
  .appName("MyApp")
  .config("spark.sql.shuffle.partitions", "200") // Check if appropriate
  .getOrCreate()
```

### Phase 6: Error Handling and Reliability

**Step 7: Analyze Error Handling**

**Exception Handling**:
```python
# Check for proper error handling
try:
    df = spark.read.parquet("path/to/data")
except Exception as e:
    # Proper error handling
    logger.error(f"Failed to read data: {e}")
    raise
```

**Data Quality Checks**:
```scala
// Check for data validation
val cleanDf = df.filter($"value".isNotNull && $"value" > 0)
// Check for schema validation
```

### Phase 7: Code Quality Assessment

**Step 8: General Code Quality**

**Code Organization**:
- Function/method size and complexity
- Code reusability
- Separation of concerns
- Documentation quality

**Testing**:
- Unit test coverage
- Integration test presence
- Data quality tests

## Analysis Output Structure

### Executive Summary
- **Code Quality Score**: A-F grade
- **Performance Risk Level**: High/Medium/Low
- **Critical Issues**: Number of high-priority problems
- **Optimization Potential**: Estimated performance improvement

### Performance Issues

**Critical Issues** (High Priority):
1. **Issue**: Description of the problem
   - **Location**: File and line number
   - **Impact**: Performance/memory/reliability impact
   - **Recommendation**: Specific fix
   - **Example**: Code snippet showing fix

**Warning Issues** (Medium Priority):
1. **Issue**: Description
   - **Location**: File and line number
   - **Impact**: Potential impact
   - **Recommendation**: Suggested improvement

### Anti-Pattern Detection

**Detected Anti-Patterns**:
1. **Pattern**: Name of anti-pattern
   - **Occurrences**: Number of instances
   - **Files**: List of affected files
   - **Fix**: How to resolve
   - **Example**: Before/after code

### Resource Optimization Opportunities

**Memory Optimizations**:
- Caching strategy improvements
- Memory-efficient operations
- Garbage collection optimizations

**CPU Optimizations**:
- Parallelization improvements
- Algorithm optimizations
- Unnecessary computation removal

**I/O Optimizations**:
- File format recommendations
- Partitioning strategies
- Compression optimizations

### Configuration Recommendations

**Spark Configuration**:
```properties
# Recommended settings based on code analysis
spark.executor.memory 8g
spark.executor.cores 4
spark.sql.shuffle.partitions 400
spark.sql.adaptive.enabled true
```

**Application-Specific**:
- Custom configuration recommendations
- Environment-specific settings
- Performance tuning parameters

### Best Practices Compliance

**Compliance Score**: X/10

**Areas of Improvement**:
1. **Category**: Description of improvement area
   - **Current State**: What's currently implemented
   - **Best Practice**: What should be implemented
   - **Benefit**: Expected improvement

### Code Quality Metrics

**Complexity Analysis**:
- Cyclomatic complexity scores
- Function/method length analysis
- Code duplication detection

**Maintainability**:
- Documentation coverage
- Code organization score
- Dependency analysis

## Specific Language Recommendations

### Scala/Java Specific
- Kryo serialization usage
- Case class optimization
- Collection usage patterns
- Implicit conversion issues

### Python/PySpark Specific
- UDF vs built-in function usage
- Pandas UDF opportunities
- Python version compatibility
- Package dependency analysis

### SQL Specific
- Query optimization opportunities
- Index usage recommendations
- Join order optimization
- Subquery vs CTE usage

## Integration with Runtime Analysis

**Correlation with MCP Data**:
If application ID is available:
1. Match code patterns with runtime performance
2. Correlate identified issues with actual bottlenecks
3. Validate optimization recommendations with metrics

## Actionable Recommendations

### Immediate Actions (High Priority)
1. **Fix**: Specific code change needed
   - **File**: Location of change
   - **Before**: Current code
   - **After**: Optimized code
   - **Expected Impact**: Performance improvement

### Medium-Term Improvements
1. **Refactoring**: Structural improvements
2. **Configuration**: Settings optimization
3. **Architecture**: Design pattern improvements

### Long-Term Enhancements
1. **Framework Upgrades**: Spark version updates
2. **Algorithm Improvements**: Better approaches
3. **Infrastructure**: Platform optimizations

## Follow-up Actions

Based on code analysis results:

1. **Performance Testing**: Use `*analyze-spark-job` after fixes
2. **Configuration Tuning**: Use `*optimize-configuration`
3. **Runtime Validation**: Use `*compare-jobs` before/after changes
4. **Monitoring Setup**: Implement performance tracking

## Code Review Checklist

Generate checklist for:
- [ ] Performance anti-patterns resolved
- [ ] Memory usage optimized
- [ ] I/O operations efficient
- [ ] Error handling adequate
- [ ] Configuration appropriate
- [ ] Best practices followed
- [ ] Tests updated
- [ ] Documentation complete

## Documentation Output

Generate comprehensive code analysis report using `*performance-report` template with:
- All findings and recommendations
- Code snippets and examples
- Performance impact estimates
- Implementation priorities
- Testing strategies
