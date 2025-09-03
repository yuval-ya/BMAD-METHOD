# Switch MCP Cluster Configuration

**Purpose**: Help users easily switch their MCP Apache Spark History Server configuration to analyze different EMR clusters.

**Elicit**: false

## 🔄 **Why Switch Clusters?**

The MCP Apache Spark History Server is configured for **one specific cluster** at a time. When you want to analyze a different cluster, you need to:

1. **Update the MCP configuration** to point to the new cluster's Spark History Server
2. **Restart the MCP server** with the new settings
3. **Verify the connection** works with the new cluster

## 📋 **Pre-Switch Checklist**

Before switching clusters, let's gather the information you need:

- [ ] **Target cluster ID**: `j-XXXXXXXXXXXXX`
- [ ] **Target cluster region**: `us-east-1`, `us-west-2`, etc.
- [ ] **Target cluster's Spark History Server URL**: `http://cluster-master:18080` or public DNS
- [ ] **Current MCP process ID** (so we can stop it cleanly)

## 🎯 **Quick Switch Guide**

### **Step 1: Find Your Target Cluster**

If you don't know which cluster to switch to:

```bash
# List all available clusters
aws emr list-clusters --region {your-region} --active

# Get cluster details including master public DNS
aws emr describe-cluster --region {your-region} --cluster-id j-XXXXXXXXXXXXX
```

### **Step 2: Get Cluster's Spark History Server URL**

```bash
# Get the master node's public DNS name
CLUSTER_ID="j-XXXXXXXXXXXXX"
REGION="us-east-1"  # Your cluster's region

MASTER_DNS=$(aws emr describe-cluster \
  --region $REGION \
  --cluster-id $CLUSTER_ID \
  --query 'Cluster.MasterPublicDnsName' \
  --output text)

echo "Spark History Server URL: http://${MASTER_DNS}:18080"
```

### **Step 3: Update MCP Configuration**

**Option A: Update existing config.yaml**

```bash
# Navigate to your MCP server directory
cd ~/mcp-apache-spark-history-server  # or wherever you installed it

# Backup current config
cp config.yaml config.yaml.backup

# Edit the configuration
nano config.yaml  # or your preferred editor
```

Update these values in `config.yaml`:

```yaml
servers:
  spark-history:
    spark_history_server_url: 'http://NEW-MASTER-DNS:18080'
    # Update other cluster-specific settings if needed
```

**Option B: Create cluster-specific configs**

```bash
# Create configs for different clusters
cp config.yaml config-cluster-1.yaml
cp config.yaml config-cluster-2.yaml

# Edit each config for specific clusters
# config-cluster-1.yaml -> points to j-CLUSTER1XXXXX
# config-cluster-2.yaml -> points to j-CLUSTER2XXXXX
```

### **Step 4: Stop Current MCP Server**

```bash
# Find the current MCP process
ps aux | grep "spark-mcp"

# Stop it gracefully (replace XXXX with actual process ID)
kill XXXX

# Or if you started it in background with our command:
pkill -f "spark-mcp"
```

### **Step 5: Start MCP with New Configuration**

**Option A: Using default config.yaml (if you updated it)**

```bash
uvx --from mcp-apache-spark-history-server spark-mcp > mcp-server.log 2>&1 &
```

**Option B: Using specific cluster config**

```bash
# Start with specific cluster config
uvx --from mcp-apache-spark-history-server spark-mcp --config config-cluster-2.yaml > mcp-server.log 2>&1 &
```

### **Step 6: Verify New Connection**

```bash
# Check if MCP server started successfully
tail -f mcp-server.log

# Test connection to new cluster
curl "http://NEW-MASTER-DNS:18080/api/v1/applications" | head -20
```

### **Step 7: Test in Cursor**

1. **Restart Cursor** (to refresh MCP connection)
2. **Try a simple MCP command**: `@spark-emr-expert` then `*analyze-spark-job {app-id}`
3. **Verify** you're seeing data from the new cluster

## 🚀 **Advanced: Quick Cluster Switching Script**

Create a script to make switching even easier:

```bash
#!/bin/bash
# save as: switch-spark-cluster.sh

CLUSTER_ID=$1
REGION=${2:-us-east-1}

if [ -z "$CLUSTER_ID" ]; then
    echo "Usage: ./switch-spark-cluster.sh j-XXXXXXXXXXXXX [region]"
    exit 1
fi

echo "🔄 Switching to cluster: $CLUSTER_ID in region: $REGION"

# Get master DNS
echo "📡 Getting cluster master DNS..."
MASTER_DNS=$(aws emr describe-cluster \
  --region $REGION \
  --cluster-id $CLUSTER_ID \
  --query 'Cluster.MasterPublicDnsName' \
  --output text)

if [ "$MASTER_DNS" == "None" ] || [ -z "$MASTER_DNS" ]; then
    echo "❌ Could not get master DNS. Is cluster running?"
    exit 1
fi

echo "🎯 Master DNS: $MASTER_DNS"

# Update config
echo "⚙️ Updating MCP configuration..."
SPARK_URL="http://${MASTER_DNS}:18080"

# Backup current config
cp config.yaml "config-backup-$(date +%Y%m%d-%H%M%S).yaml"

# Update config.yaml
sed -i.bak "s|spark_history_server_url:.*|spark_history_server_url: \"$SPARK_URL\"|" config.yaml

echo "📝 Updated config.yaml with: $SPARK_URL"

# Stop current MCP
echo "🛑 Stopping current MCP server..."
pkill -f "spark-mcp" 2>/dev/null || true
sleep 2

# Start new MCP
echo "🚀 Starting MCP server with new configuration..."
uvx --from mcp-apache-spark-history-server spark-mcp > mcp-server.log 2>&1 &
sleep 3

# Verify
echo "✅ Testing connection..."
if curl -s --connect-timeout 5 "$SPARK_URL/api/v1/applications" > /dev/null; then
    echo "🎉 Successfully switched to cluster: $CLUSTER_ID"
    echo "📊 Spark History Server: $SPARK_URL"
    echo "📝 Please restart Cursor to refresh MCP connection"
else
    echo "⚠️ Warning: Could not verify connection to $SPARK_URL"
    echo "   The cluster might still be starting or have security group restrictions"
fi
```

Make it executable:

```bash
chmod +x switch-spark-cluster.sh

# Usage:
./switch-spark-cluster.sh j-XXXXXXXXXXXXX us-west-2
```

## 🎯 **Multiple Cluster Management**

### **Strategy 1: Config Files per Cluster**

```bash
# Organize configs by cluster
config-production.yaml     # Production cluster
config-development.yaml    # Dev cluster
config-analytics.yaml      # Analytics cluster

# Switch between them:
ln -sf config-production.yaml config.yaml  # Switch to production
# Restart MCP server
```

### **Strategy 2: Environment-Based Switching**

```bash
# Set environment variables
export SPARK_CLUSTER_ID="j-XXXXXXXXXXXXX"
export SPARK_REGION="us-east-1"

# Script reads these and updates config automatically
./update-mcp-config.sh
```

## 🆘 **Troubleshooting**

### **MCP Won't Connect to New Cluster**

```bash
# Check if Spark History Server is accessible
MASTER_DNS="ec2-xx-xx-xx-xx.compute-1.amazonaws.com"
curl -v "http://${MASTER_DNS}:18080"

# Check security groups allow port 18080
aws emr describe-cluster --cluster-id j-XXXXXXXXXXXXX \
  --query 'Cluster.Ec2InstanceAttributes.EmrManagedMasterSecurityGroup'
```

### **Old Data Still Showing**

```bash
# Clear any cached data
rm -rf ~/.cache/mcp-* 2>/dev/null || true

# Restart Cursor completely
# Kill all Cursor processes and restart
```

### **Permission Issues**

```bash
# Verify AWS credentials can access the new cluster
aws emr describe-cluster --cluster-id j-XXXXXXXXXXXXX --region us-east-1

# Check if you have EMR permissions for the new region/cluster
aws sts get-caller-identity
```

## 💡 **Pro Tips**

1. **Keep a cluster inventory**: Document which config goes with which cluster
2. **Use descriptive config names**: `config-prod-analytics.yaml` vs `config-1.yaml`
3. **Test connection before switching**: Verify cluster is accessible
4. **Backup configs**: Always backup before making changes
5. **Restart Cursor**: Always restart Cursor after switching MCP configuration
6. **Monitor logs**: Check `mcp-server.log` for connection issues

## ✅ **Verification Checklist**

After switching clusters, verify:

- [ ] MCP server started without errors (`tail mcp-server.log`)
- [ ] Can access Spark History Server URL in browser
- [ ] Cursor shows new cluster data when using MCP commands
- [ ] Application IDs match the target cluster
- [ ] No old cached data from previous cluster

Your MCP is now configured for the new cluster! 🎉
