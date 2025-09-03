# Spark EMR Expert - Getting Started Guide

**Purpose**: Simple, step-by-step guide to get you started with Spark EMR analysis and debugging.

**Elicit**: false

## 🚀 **Quick Start - 3 Simple Steps**

### **Step 1: Setup (One-time only)**

Choose one option:

**Option A: I already have AWS CLI configured**
✅ Skip to Step 2

**Option B: I need to setup AWS CLI**
Use: `*setup-aws-cli`

- I'll guide you through AWS CLI installation
- Help configure your credentials
- Set up the right permissions for EMR

### **Step 2: Find Your Cluster**

Use: `*list-emr-clusters`

- I'll show you all your EMR clusters
- You can filter by region, status, or time period
- I'll give you the cluster IDs you need

### **Step 3: Analyze Your Cluster**

Pick what you want to do:

**🔍 General Cluster Health**

- `*describe-emr-cluster j-XXXXX` - Get cluster overview and health

**⚡ Spark Job Performance**

- `*analyze-spark-job application_123_456` - Deep Spark analysis
- `*performance-report application_123_456` - Generate detailed report

**📊 Step-by-Step Analysis**

- `*query-emr-steps j-XXXXX` - See which jobs ran and how they performed

**💰 Cost Analysis**

- `*aws-cost-analysis j-XXXXX` - Find ways to save money

**🐛 Debug Failed Jobs**

- `*fetch-emr-logs j-XXXXX` - Get error logs and diagnose issues

## 🎯 **Common Workflows**

### **"My Spark job is running slow"**

```
1. *list-emr-clusters (find your cluster)
2. *describe-emr-cluster j-XXXXX (check cluster health)
3. *query-emr-steps j-XXXXX (find your slow job)
4. *analyze-spark-job application_123_456 (deep analysis)
5. *performance-report application_123_456 (get recommendations)
```

### **"My job failed and I don't know why"**

```
1. *list-emr-clusters (find your cluster)
2. *query-emr-steps j-XXXXX (find the failed step)
3. *fetch-emr-logs j-XXXXX (get error details)
4. *debug-job-failure application_123_456 (systematic diagnosis)
```

### **"I want to optimize costs"**

```
1. *list-emr-clusters (see all your clusters)
2. *describe-emr-cluster j-XXXXX (check configuration)
3. *aws-cost-analysis j-XXXXX (get cost recommendations)
```

### **"I want to review my code for performance issues"**

```
1. *analyze-code (I'll review your Spark code)
2. *performance-report application_123_456 (combine with runtime analysis)
```

## 🆘 **Need Help?**

**Don't know your cluster ID?**
→ Use `*list-emr-clusters` first

**Don't know your application ID?**  
→ Use `*query-emr-steps j-XXXXX` to see all applications in your cluster

**Getting permission errors?**
→ Use `*setup-aws-cli` to fix permissions

**Want to see all available commands?**
→ Use `*help`

## 💡 **Pro Tips**

1. **Start Simple**: Begin with `*list-emr-clusters` to see what you have
2. **One Thing at a Time**: Focus on one cluster or job at a time
3. **Use Reports**: Generate `*performance-report` for detailed analysis you can share
4. **Save Time**: I can discover cluster IDs and application IDs for you automatically
5. **Ask Questions**: I'm here to guide you through any analysis

## 📋 **What Each Command Does (Simple Explanations)**

| Command                         | What It Does            | When To Use                 |
| ------------------------------- | ----------------------- | --------------------------- |
| `*setup-aws-cli`                | Sets up AWS access      | First time setup            |
| `*list-emr-clusters`            | Shows all your clusters | Find cluster IDs            |
| `*describe-emr-cluster j-XXXXX` | Cluster health check    | Check if cluster is healthy |
| `*query-emr-steps j-XXXXX`      | Shows jobs that ran     | Find slow or failed jobs    |
| `*analyze-spark-job app_123`    | Deep Spark analysis     | Understand performance      |
| `*fetch-emr-logs j-XXXXX`       | Gets error logs         | Debug failed jobs           |
| `*performance-report app_123`   | Creates detailed report | Share with team             |
| `*aws-cost-analysis j-XXXXX`    | Cost optimization       | Save money                  |
| `*analyze-code`                 | Reviews your code       | Find code issues            |

## 🎯 **Next Steps**

1. **Try it now**: Start with `*list-emr-clusters` to see your clusters
2. **Pick a cluster**: Choose one that interests you
3. **Explore**: Try `*describe-emr-cluster` to learn about it
4. **Go deeper**: Use specific analysis commands based on what you find

Remember: I'm here to guide you through each step! Just tell me what you want to achieve, and I'll recommend the right commands.
