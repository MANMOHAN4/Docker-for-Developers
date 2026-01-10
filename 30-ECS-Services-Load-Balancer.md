# Deploying ECS Services with Load Balancer for High Availability

## What is an ECS Service?

An **ECS Service** manages the deployment and scaling of tasks. It ensures:

- The specified number of tasks are always running
- Failed tasks are automatically replaced
- Load is distributed across tasks
- Scaling happens based on metrics
- Updates are rolled out gradually

## Service Architecture

```
Users
  ↓
Load Balancer (Port 80 - HTTP)
  ↓
Target Group
  ↓
ECS Service (Manages 2-5 tasks)
  ↓
Tasks (Running containers)
```

## Creating an ECS Service

### Step 1: Navigate to Services

1. Open ECS Console
2. Click on your cluster (e.g., "dev-cluster")
3. Click "Create Service"

### Step 2: Service Configuration

```yaml
Launch Type: Fargate
Task Definition: api-app (select your task definition)
Task Definition Revision: Latest
Service Name: api-service
Desired Number of Tasks: 2
```

**Why 2 tasks minimum?**

- If one task crashes, the other keeps serving requests
- Provides high availability
- Allows rolling updates

### Step 3: Deployment Configuration

```yaml
Deployment Strategy: Rolling Update
Maximum Percent: 200%
Minimum Healthy Percent: 100%
```

**What this means:**

- When updating, can run up to 200% of desired tasks (4 tasks if desired is 2)
- Always keep at least 100% healthy (2 out of 2)
- Ensures zero-downtime deployments

### Step 4: Networking

```yaml
VPC: Default VPC
Subnets: Select available subnets
Security Groups: Select or create
Assign Public IP: Enabled
```

### Step 5: Load Balancing

```yaml
Load Balancer Type: Application Load Balancer
Container Port: 8000 (from task definition)
Load Balancer Port: 80 (HTTP)
Create New Load Balancer: Yes
Load Balancer Name: api-lb
```

**ALB (Application Load Balancer):**

- Works at Layer 7 (Application/HTTP)
- Can route based on hostname, path, headers
- Best for microservices

### Step 6: Target Group Configuration

```yaml
Target Type: IP (for Fargate)
Health Check Path: /health
Health Check Interval: 30 seconds
Health Check Timeout: 5 seconds
Healthy Threshold: 2
Unhealthy Threshold: 3
```

**Health Check Explanation:**

- Every 30 seconds, ALB sends request to `/health`
- If 2 consecutive checks pass → Task is healthy
- If 3 consecutive checks fail → Task is unhealthy and replaced
- If no response in 5 seconds → Counts as failure

### Step 7: Auto-Scaling Configuration

```yaml
Enable Auto-Scaling: Yes
Minimum Tasks: 2
Maximum Tasks: 5
Target Metric: Average CPU Utilization
Target Value: 70%
Scale-Out Cooldown: 300 seconds
Scale-In Cooldown: 300 seconds
```

**Auto-Scaling Policy:**

- If average CPU > 70% → Scale out (add tasks)
- If average CPU < 70% → Scale in (remove tasks)
- Always keep minimum 2 tasks running
- Never exceed 5 tasks
- Wait 5 minutes before scaling again (cooldown)

### Step 8: Review and Create

Review all settings:

```
Service Name: api-service
Task Definition: api-app:1
Desired Tasks: 2
Load Balancer: api-lb (Port 80 → 8000)
Auto-Scaling: Enabled (2-5 tasks, CPU target 70%)
```

Click "Create Service" and wait for deployment.

## What Happens After Creation

### Phase 1: Infrastructure Setup (5-10 minutes)

ECS automatically creates:

1. **Load Balancer**: Receives traffic on port 80
2. **Target Group**: Groups tasks for load distribution
3. **Security Groups**: Configures network access
4. **Listener**: Maps load balancer port 80 to target group

### Phase 2: Task Startup (1-2 minutes)

1. **Pull Image**: Download Docker image from ECR
2. **Create Container**: Spin up the container
3. **Start Application**: Application becomes ready
4. **Health Checks**: Begin checking container health
5. **Register**: Task registers with load balancer

### Phase 3: Production Ready

- Load balancer receives traffic on DNS name
- Distributes across all healthy tasks
- Auto-scaling monitors metrics
- Failed tasks automatically replaced

## Verifying Service Deployment

### In AWS Console

1. Go to ECS → Cluster → Services
2. Click on your service
3. Check:
   - **Tasks**: Should show 2 tasks running
   - **Health**: Should all be healthy (green)
   - **Deployments**: Should show 1 active deployment

### Load Balancer DNS

1. Go to EC2 → Load Balancers
2. Find your load balancer (api-lb)
3. Copy the DNS name
4. Paste in browser to test

Example:

```
api-lb-1234567890.ap-south-1.elb.amazonaws.com
```

### Health Check Status

Check if health checks are passing:

1. Go to EC2 → Target Groups
2. Click on your target group
3. View "Targets" tab
4. Each task should show:
   - Health Status: **Healthy** ✓
   - Status Description: **Healthy**

### Troubleshooting Health Checks

If health checks are failing:

**Problem: "Health checks failed"**

Solution checklist:

1. Does your app have a `/health` endpoint?
2. Is it responding with HTTP 200?
3. Check security groups allow traffic
4. Check application logs in CloudWatch
5. Verify port mapping (8000) is correct

**View Logs:**

```bash
aws logs tail /ecs/api-app --follow
```

## Updating a Service

### When you push a new image to ECR:

1. Update ECR image URL (push new image)
2. Create new Task Definition revision
3. Update Service to use new revision
4. ECS performs rolling update

### Steps to Update:

1. Build and push new image to ECR
2. Go to ECS → Task Definitions
3. Create new revision (click on existing, then "Create new revision")
4. Update image URL
5. Click "Create"
6. Go to Service → Update Service
7. Select new Task Definition revision
8. Check "Force new deployment"
9. Click "Update Service"

### Rolling Update Process:

```
Initial State (2 tasks - old version):
  ✓ Task 1 (v1)
  ✓ Task 2 (v1)

Phase 1 (2 old + 1 new allowed, since max 200%):
  ✓ Task 1 (v1)
  ✓ Task 2 (v1)
  ✓ Task 3 (v2) - NEW

Phase 2 (verify new is healthy):
  ✓ Task 1 (v1) - STOP
  ✓ Task 2 (v1)
  ✓ Task 3 (v2)

Phase 3 (replace second old task):
  ✓ Task 2 (v1) - STOP
  ✓ Task 4 (v2) - NEW
  ✓ Task 3 (v2)

Final State (2 tasks - new version):
  ✓ Task 3 (v2)
  ✓ Task 4 (v2)
```

**No downtime** because at least 1 task is always healthy (100% minimum).

## Scaling the Service

### Manual Scaling

Go to Service → Edit:

```
Desired Tasks: 2 → 5
```

Click "Update" to scale up.

### Automatic Scaling

Already configured if you enabled auto-scaling:

- Monitors CPU/memory metrics
- Adds tasks if threshold exceeded
- Removes tasks if usage drops
- Respects min/max boundaries

### View Scaling Activity

Go to Service → Auto Scaling → Scaling Activity:

```
5:23 PM - Scaled from 2 to 3 tasks (CPU 72%)
5:18 PM - Scaled from 3 to 2 tasks (CPU 45%)
```

## Security Group Configuration

Ensure traffic can reach your service:

**Inbound Rules Needed:**

```
- Port 80 (HTTP from anywhere)
- Port 443 (HTTPS from anywhere, if using HTTPS)
- Port 8000 (Can be restricted to ALB security group)
```

**Typical Configuration:**

```yaml
Inbound Rules:
  - Type: HTTP (80)
    Source: 0.0.0.0/0 (anywhere)
  - Type: Custom TCP
    Port: 8000
    Source: sg-xxxx (ALB security group)
```

## Cost Optimization

### Minimize Costs:

1. **Right-size tasks**: Don't over-allocate CPU/memory
2. **Set reasonable minimums**: Don't run tasks unnecessarily
3. **Use auto-scaling**: Scale down during low usage
4. **Optimize health check intervals**: Reduce if safe
5. **Clean up**: Delete unused services and load balancers

### Cost Estimate (Fargate):

```
2 tasks × 0.25 vCPU × $0.04/vCPU-hour
2 tasks × 0.5 GB × $0.004/GB-hour
ALB: ~$16/month
= ~$15-20/month for minimal setup
```

## Key-takeaways

ECS Services provide:

- **Managed task orchestration**: Keeps desired number running
- **Load balancing**: Distributes traffic
- **Auto-scaling**: Scales based on metrics
- **Zero-downtime updates**: Rolling deployments
- **Health management**: Replaces failed tasks
- **Monitoring**: Metrics and logs

With a service deployed, your application is production-ready with high availability and automatic scaling.
