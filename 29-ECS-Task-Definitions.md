# Defining ECS Tasks and Creating Task Definitions for Container Deployment

## What is a Task Definition?

A **Task Definition** is a blueprint that describes how to run Docker containers in ECS. It's similar to a `docker run` command but stored in AWS.

### Task Definition Analogy

```
Docker Command:
docker run -e NODE_ENV=production -p 8000:8000 --name api 123456.dkr.ecr.ap-south-1.amazonaws.com/backend:latest

↓ Becomes ↓

Task Definition:
{
  "containerName": "api",
  "image": "123456.dkr.ecr.ap-south-1.amazonaws.com/backend:latest",
  "environment": [{"name": "NODE_ENV", "value": "production"}],
  "portMappings": [{"containerPort": 8000, "hostPort": 8000, "protocol": "tcp"}]
}
```

## Task Definition Components

### 1. Container Image

The Docker image to run, typically from ECR:

```
123456.dkr.ecr.ap-south-1.amazonaws.com/backend:latest
```

### 2. Container Name

A descriptive name for your container:

```
api
worker
queue-consumer
```

### 3. Port Mappings

Which ports to expose:

```
Container Port: 8000 (port inside container)
Host Port: 8000 (port on the task)
Protocol: TCP or UDP
```

### 4. CPU and Memory

Resource allocation for the task:

```
CPU: 256 (256 CPU units = 0.25 vCPU)
Memory: 512 MB
```

**Valid CPU/Memory combinations for Fargate:**

| CPU  | Memory Options                  |
| ---- | ------------------------------- |
| 256  | 512, 1024, 2048                 |
| 512  | 1024-4096 (increments of 1024)  |
| 1024 | 2048-8192 (increments of 1024)  |
| 2048 | 4096-16384 (increments of 1024) |
| 4096 | 8192-30720 (increments of 1024) |

### 5. Environment Variables

Pass configuration to your application:

```yaml
Environment Variables:
  - NODE_ENV: production
  - DATABASE_URL: postgres://db:5432/myapp
  - API_KEY: ${secrets_manager:api-key}
```

### 6. Logging

Configure where logs are sent:

```yaml
Log Driver: awslogs (CloudWatch Logs)
Log Group: /ecs/api-app
Log Stream Prefix: ecs
```

### 7. Health Checks

Monitor container health:

```yaml
Health Check Command:
  CMD-SHELL: curl -f http://localhost:8000/health || exit 1

Interval: 30 seconds
Timeout: 5 seconds
Start Period: 30 seconds
Max Retries: 3
```

The health check will:

- Every 30 seconds, make a GET request to `/health`
- If no response in 5 seconds, consider it a failure
- Wait 30 seconds before starting checks (start period)
- After 3 failures, mark container as unhealthy

### 8. Architecture

Specify the CPU architecture:

- **x86_64**: Intel/AMD processors (Windows, most Linux)
- **ARM64**: ARM-based processors (Mac M1/M2, Graviton)

**Important**: Build your Docker image on the same architecture as you'll run it on ECS.

## Creating a Task Definition

### Step 1: Navigate to Task Definitions

1. Open ECS Console
2. Click "Task Definitions"
3. Click "Create new task definition"

### Step 2: Basic Configuration

```
Task Definition Family: api-app
Launch Type: Fargate
Operating System: Linux
CPU: 256
Memory: 512 MB
Task Execution Role: Create new role (default)
```

### Step 3: Container Configuration

1. Click "Add container"
2. **Container name**: `api`
3. **Image URI**: Paste from ECR

Example:

```
123456.dkr.ecr.ap-south-1.amazonaws.com/backend:latest
```

### Step 4: Port Mappings

1. **Container Port**: 8000
2. **Protocol**: TCP
3. **Name**: api

### Step 5: Environment Variables

Add environment variables needed by your app:

```
NODE_ENV: production
REDIS_URL: redis://redis:6379
DATABASE_URL: postgres://db:5432/myapp
```

### Step 6: Logging Configuration

```
Log Driver: awslogs
Log Group: /ecs/api-app
Log Stream Prefix: ecs
```

This sends all container logs to CloudWatch.

### Step 7: Health Check

```
Command: CMD-SHELL,curl -f http://localhost:8000/health || exit 1
Interval: 30 seconds
Timeout: 5 seconds
Start Period: 30 seconds
Max Retries: 3
```

### Step 8: Create

Click "Create" to save the task definition.

## Complete Task Definition Example

```yaml
Family: api-app
Revision: 1
Network Mode: awsvpc
Requires Compatibilities: [FARGATE]
CPU: 256
Memory: 512

Containers:
  - Name: api
    Image: 123456.dkr.ecr.ap-south-1.amazonaws.com/backend:latest
    PortMappings:
      - ContainerPort: 8000
        Protocol: tcp
    Environment:
      - NODE_ENV: production
      - REDIS_URL: redis://redis:6379
    LogConfiguration:
      LogDriver: awslogs
      Options:
        awslogs-group: /ecs/api-app
        awslogs-region: ap-south-1
        awslogs-stream-prefix: ecs
    HealthCheck:
      Command:
        - CMD-SHELL
        - curl -f http://localhost:8000/health || exit 1
      Interval: 30
      Timeout: 5
      StartPeriod: 30
      Retries: 3
```

## Architecture Selection Important Note

**Architecture Mismatch Issue:**

If you build your Docker image on:

- **ARM64 Mac (M1/M2)**: Image is ARM64
- But select x86_64 in Task Definition: Incompatibility error

**Solution**: Match your build architecture to task definition:

```bash
# If building on Mac M1/M2 (ARM)
docker build -t myapp . # Creates ARM64 image

# Then in Task Definition, select: ARM64 (Graviton)
```

Or build for multiple architectures:

```bash
docker buildx build --platform linux/amd64,linux/arm64 -t myapp .
```

## Task Definition Versioning

Each time you update a task definition:

- Previous versions are preserved
- New revision number is created
- Example: `api-app:1`, `api-app:2`, `api-app:3`
- You can rollback by using an older revision

## Viewing Task Definition History

```bash
aws ecs list-task-definitions --family-prefix api-app
aws ecs describe-task-definition --task-definition api-app:1
```

## Best Practices

1. **Use ECR URLs**: Always pull from ECR, not Docker Hub
2. **Set Resource Limits**: Allocate appropriate CPU/memory
3. **Enable Health Checks**: Always include health check endpoint
4. **Use CloudWatch Logs**: Centralize all logging
5. **Set Environment Variables**: Externalize configuration
6. **Version Everything**: Update revision numbers semantically
7. **Test Locally**: Run `docker run` with same config first
8. **Architecture Match**: Ensure build and deployment architecture match

## Common Issues

### Issue: Container Fails to Start

**Check**:

- Is the Docker image accessible from ECR?
- Are credentials correct?
- Is the application listening on the correct port?

### Issue: Health Check Failing

**Check**:

- Does your app have a `/health` endpoint?
- Is the port correct?
- Is the response fast enough (< timeout)?

### Issue: Port Not Accessible

**Check**:

- Is the port correctly mapped?
- Are security groups allowing traffic?
- Is the application actually listening?

## Key-takeaways

Task Definitions:

- Define how containers run in ECS
- Specify image, CPU, memory, ports, environment
- Include health checks and logging
- Support versioning and rollback
- Are the foundation for ECS services

Next step: Create a service that uses this task definition to run and manage multiple container instances.
