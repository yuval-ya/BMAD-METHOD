# Setup Spark History Server MCP

**Purpose**: Guide user through installing and configuring the MCP Apache Spark History Server to connect to their EMR cluster.

**Elicit**: true

## Prerequisites Check

First, let me check what you have available:

1. **Do you have an existing EMR cluster?** (running or terminated)
   - If yes, what is the cluster ID?
   - What AWS region is it in?
   - What's your AWS account ID?

2. **Do you have AWS CLI configured?**
   - Run: `aws sts get-caller-identity`
   - Do you see your account details?

3. **Do you have the required IAM permissions?**
   - `elasticmapreduce:CreatePersistentAppUI`
   - `elasticmapreduce:DescribePersistentAppUI`
   - `elasticmapreduce:GetPersistentAppUIPresignedURL`
   - `elasticmapreduce:ListPersistentAppUIs`

## Installation Steps

### Step 1: Install MCP Server

**Option A: Using uvx (Recommended)**
```bash
# Install and run in background
uvx --from mcp-apache-spark-history-server spark-mcp > mcp-server.log 2>&1 &
```

**Option B: Clone and setup development environment**
```bash
git clone https://github.com/DeepDiagnostix-AI/mcp-apache-spark-history-server.git
cd mcp-apache-spark-history-server

# Install Task runner
brew install go-task  # macOS, see https://taskfile.dev/installation/ for others

# Start MCP server
task start-mcp-bg
```

### Step 2: Configure EMR Connection

Create or update `config.yaml`:

```yaml
servers:
  local:
    default: false
    url: "http://localhost:18080"
  
  emr_persistent_ui:
    default: true
    emr_cluster_arn: "arn:aws:elasticmapreduce:{region}:{account-id}:cluster/{cluster-id}"
    verify_ssl: true
    timeout: 60

mcp:
  transports:
    - streamable-http
  port: "18888"
  debug: true
  address: localhost
```

**Replace these values:**
- `{region}`: Your AWS region (e.g., us-west-2)
- `{account-id}`: Your AWS account ID
- `{cluster-id}`: Your EMR cluster ID

### Step 3: Test Connection

```bash
# Start MCP server
task start-mcp-bg

# Check server status
curl -s http://localhost:18888/mcp

# Optional: Start inspector for testing
task start-inspector-bg
# Access at: http://localhost:6274
```

### Step 4: Verify EMR Integration

```bash
# Test EMR cluster access
aws emr describe-cluster --cluster-id {cluster-id} --region {region}

# Check MCP server logs
tail -f mcp-server.log
```

## Troubleshooting Common Issues

### Permission Denied
```bash
# Error: User is not authorized to perform: elasticmapreduce:CreatePersistentAppUI
# Solution: Request EMR permissions from AWS admin
```

### Port Already in Use
```bash
# Error: [Errno 48] address already in use
# Solution: Stop existing processes
task stop-all
# Then restart
task start-mcp-bg
```

### EMR Cluster Not Found
```bash
# Error: Cluster id 'xxx' is not valid
# Solution: Verify cluster ID and region
aws emr list-clusters --region {region} --cluster-states TERMINATED
```

### Memory Issues with Large Applications
If you encounter timeouts or memory errors:

1. **Increase MCP client timeout** in your client configuration:
```json
{
  "timeout": 300000
}
```

2. **Increase server timeout**:
```bash
export SHS_SERVERS_EMR_PERSISTENT_UI_TIMEOUT=500
```

3. **Enable Spark History Server hybrid store** in `spark-defaults.conf`:
```properties
spark.history.store.hybridStore.enabled true
spark.history.store.hybridStore.maxMemoryUsage 2g
spark.history.store.hybridStore.diskBackend ROCKSDB
spark.history.store.path /tmp/spark-history-store
```

## Success Indicators

You'll know everything is working when you see:
```
✅ MCP Server is available on port 18888
✅ Persistent App UI created successfully
✅ HTTP session established successfully
✅ Cookies: 8 cookie(s) stored
```

## Next Steps

Once setup is complete, you can:
1. Use `*analyze-spark-job {app-id}` to analyze specific applications
2. Use `*fetch-emr-logs {cluster-id}` to retrieve cluster logs
3. Use `*cluster-health-check {cluster-id}` for comprehensive analysis

## Integration Options

### Cursor IDE (Recommended)
Add to your Cursor MCP configuration (`.cursor/mcp.json`):
```json
{
  "spark-history-server": {
    "command": "uvx",
    "args": [
      "--from",
      "mcp-apache-spark-history-server",
      "spark-mcp"
    ],
    "env": {
      "SHS_MCP_TRANSPORT": "stdio"
    },
    "disabled": false,
    "autoApprove": [],
    "timeout": 300000
  }
}
```

**For development setup (if you cloned the repository)**:
```json
{
  "spark-history-server": {
    "command": "uv",
    "args": [
      "run",
      "-m",
      "spark_history_mcp.core.main",
      "--frozen"
    ],
    "env": {
      "SHS_MCP_TRANSPORT": "stdio",
      "SHS_SERVERS_EMR_PERSISTENT_UI_EMR_CLUSTER_ARN": "arn:aws:elasticmapreduce:{region}:{account-id}:cluster/{cluster-id}",
      "SHS_SERVERS_EMR_PERSISTENT_UI_TIMEOUT": "300"
    },
    "disabled": false,
    "autoApprove": [],
    "timeout": 300000
  }
}
```

### Claude Desktop
Copy the configuration to `~/.config/claude/`:
```json
{
  "mcpServers": {
    "spark-history-server": {
      "command": "uvx",
      "args": ["--from", "mcp-apache-spark-history-server", "spark-mcp"],
      "env": {
        "SHS_MCP_TRANSPORT": "stdio"
      }
    }
  }
}
```

### Amazon Q CLI
Follow the setup guide in `examples/integrations/amazon-q-cli/`

### Custom Integration
- **Transport**: `streamable-http`
- **Endpoint**: `http://localhost:18888/mcp`
- **Protocol**: MCP JSON-RPC 2.0

## Available MCP Tools

Once connected, you'll have access to 17 specialized tools:
- **Application Info**: `get_application`
- **Job Analysis**: `list_jobs`, `list_slowest_jobs`
- **Stage Analysis**: `list_stages`, `get_stage`, `get_stage_task_summary`
- **Executor Analysis**: `list_executors`, `get_executor`, `get_executor_summary`
- **Configuration**: `get_environment`
- **SQL Analysis**: `list_slowest_sql_queries`
- **Performance**: `get_job_bottlenecks`
- **Comparison**: `compare_job_performance`, `compare_job_environments`
