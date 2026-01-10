# Cleaning Up AWS ECS and ECR Resources to Avoid Unnecessary Costs

## Why Cleanup Matters

Leaving AWS resources running costs money, even if you're not actively using them. It's essential to properly clean up after testing or development.

**Cost Impact:**

- Running load balancer: ~$16/month
- Running ECS tasks: ~$8-20/month
- Storage (ECR images): ~$0.10/GB/month

Unused resources can quickly accumulate costs.

## Cleanup Checklist

The proper order to delete resources is important:

```
1. Disable CloudFront distributions (if using)
   ↓
2. Delete ECS Services
   ↓
3. Delete ECS Clusters
   ↓
4. Delete ECR Repositories
   ↓
5. Delete Load Balancers (if not auto-deleted)
   ↓
6. Delete CloudFormation stacks (if any)
```

## Detailed Cleanup Steps

### Step 1: Disable CloudFront (if applicable)

CloudFront distributions take 15-20 minutes to disable.

1. Go to CloudFront Console
2. Select your distribution
3. Click "Disable" in the menu
4. Confirm and wait for deployment
5. Only after disabled can you delete:
   - Click "Delete"
   - Confirm deletion

### Step 2: Scale Down the Service

Before deleting the service, scale down tasks to zero:

1. Go to ECS → Clusters → Your Cluster
2. Click on your Service
3. Click "Update Service"
4. Set "Desired Number of Tasks" to 0
5. Also update "Maximum Tasks" to 0 in Auto-Scaling
6. Click "Update"
7. Wait for all tasks to be stopped

**Why scale to zero first?**

- Ensures all load balancers are disconnected
- Clean shutdown of containers
- Prevents errors during deletion

### Step 3: Delete the Service

Once all tasks are stopped:

1. Go to ECS → Clusters → Your Cluster
2. Click on your Service
3. Click "Delete"
4. Confirm deletion
5. Wait for deletion to complete (1-2 minutes)

**What gets deleted automatically:**

- Service configuration
- Load Balancer (if created with service)
- Target Group
- Listener rules
- Auto-scaling rules

### Step 4: Delete the Cluster

1. Go to ECS → Clusters
2. Select your Cluster
3. Click "Delete"
4. Confirm deletion
5. Wait for deletion (usually instant)

**Note:** Task definitions are NOT deleted automatically. They persist but incur no cost.

### Step 5: Delete ECR Repository (if desired)

To remove container images and save storage costs:

1. Go to ECR → Repositories
2. Click on your Repository
3. Click "Delete"
4. Type repository name to confirm
5. Click "Delete repository"

**Important:** You can't delete a repository with images inside. ECS might still reference it.

### Step 6: Verify Deletion

Check that resources are gone:

```bash
# Verify clusters are deleted
aws ecs list-clusters

# Verify services are deleted
aws ecs list-services --cluster your-cluster

# Verify repositories are deleted
aws ecr describe-repositories
```

All should return empty results.

## Cleanup via AWS Console

### Visual Cleanup Process

1. **ECS Console:**
   - Click on cluster
   - Note the service name
   - Click delete (wait for all tasks to stop)

2. **Load Balancer Console:**
   - Go to EC2 → Load Balancers
   - Should be auto-deleted with service
   - If not, delete manually

3. **ECR Console:**
   - Go to ECR Repositories
   - Delete image repositories

4. **CloudWatch Logs:**
   - Go to CloudWatch → Log Groups
   - Delete `/ecs/api-app` log group (optional)

## Cleanup via AWS CLI

```bash
# 1. Update service to scale to zero
aws ecs update-service \
  --cluster dev-cluster \
  --service api-service \
  --desired-count 0

# 2. Delete service
aws ecs delete-service \
  --cluster dev-cluster \
  --service api-service \
  --force

# 3. Delete cluster
aws ecs delete-cluster \
  --cluster dev-cluster

# 4. Delete ECR repository
aws ecr delete-repository \
  --repository-name backend \
  --force

# 5. Delete log group
aws logs delete-log-group \
  --log-group-name /ecs/api-app
```

## Automation Script

Create a cleanup script:

```bash
#!/bin/bash

CLUSTER_NAME="dev-cluster"
SERVICE_NAME="api-service"
REPO_NAME="backend"

echo "Starting cleanup..."

# Scale to zero
echo "Scaling service to zero..."
aws ecs update-service \
  --cluster $CLUSTER_NAME \
  --service $SERVICE_NAME \
  --desired-count 0 \
  --region ap-south-1

echo "Waiting 30 seconds for tasks to stop..."
sleep 30

# Delete service
echo "Deleting service..."
aws ecs delete-service \
  --cluster $CLUSTER_NAME \
  --service $SERVICE_NAME \
  --force \
  --region ap-south-1

# Delete cluster
echo "Deleting cluster..."
aws ecs delete-cluster \
  --cluster $CLUSTER_NAME \
  --region ap-south-1

# Delete ECR repo
echo "Deleting ECR repository..."
aws ecr delete-repository \
  --repository-name $REPO_NAME \
  --force \
  --region ap-south-1

echo "Cleanup complete!"
```

## Checking Remaining Costs

### In AWS Billing Console

1. Go to AWS Billing Dashboard
2. View "Bills" section
3. Look for current month charges
4. After cleanup, charges should drop

### Using Cost Explorer

1. Go to Cost Explorer
2. Set date range to current month
3. Group by service
4. Should see no ECS, ALB, or ECR charges after cleanup

## Recovering from Accidental Deletion

### If you deleted something important

Most resources CAN be recovered if deleted recently:

**ECS Clusters:** Can recreate with same name
**ECR Images:** Check if in Recycle Bin (if available)
**CloudWatch Logs:** Can usually recover if recent
**Load Balancers:** Can recreate and reconfigure

**Best Practice:** Always use version control for:

- Task definition JSON
- Service configurations
- Infrastructure as Code (CloudFormation/Terraform)

## Important Reminders

### Things That Cost Money (Keep Track):

1. **ECS Tasks (Fargate)**
   - Running or not, if desired count > 0
   - ~$0.04 per vCPU-hour
   - ~$0.004 per GB-hour

2. **Load Balancers**
   - ~$16/month base
   - Plus data transfer

3. **ECR Storage**
   - ~$0.10 per GB/month

4. **CloudWatch Logs**
   - ~$0.50 per GB ingested
   - ~$0.03 per GB stored

### What's Free:

- ECS task definitions (not running)
- ECS clusters (when empty)
- ECR repositories (empty)

## Cleanup Schedule Recommendation

For development/learning:

- **Immediate**: Stop services after learning session
- **Daily**: Check for running resources
- **Weekly**: Clean up unused resources
- **Monthly**: Review AWS billing

## Key-takeaways

Proper cleanup:

1. **Scale service to zero** (stop all tasks)
2. **Delete service** (removes load balancer)
3. **Delete cluster** (removes orchestration)
4. **Delete ECR repo** (removes images)
5. **Verify deletion** (check billing)

Takes approximately **10-15 minutes** and prevents unnecessary monthly costs.

Remember: **Stopped instances still cost money if desired count > 0**. Always scale to zero before cleanup!
