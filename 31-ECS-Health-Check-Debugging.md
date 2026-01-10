# Debugging and Fixing ECS Health Check Failures During Deployment

## The Problem: Health Checks Keep Failing

When deploying on ECS, you might see:

```
Task Status: Unhealthy
Reason: Health check failed
Last Status: FAILED
```

Even though your application works locally with Docker.

## Understanding Health Checks

A health check is a verification that your container is:

1. **Running**: The process is active
2. **Responsive**: It can handle requests
3. **Healthy**: It's in a good state

ECS performs health checks by running a command inside the container:

```yaml
Health Check Command: curl -f http://localhost:8000/health
Interval: 30 seconds
Timeout: 5 seconds
```

## Common Health Check Issues

### Issue 1: curl Not Installed

**Symptom:**

```
Error: command not found: curl
```

**Cause:**
Your Docker image (especially Alpine) doesn't have curl installed.

**Solution:**

In your Dockerfile, add:

```dockerfile
FROM alpine:latest

# Install curl
RUN apk add --no-cache curl

# Your application setup
COPY . /app
WORKDIR /app
CMD ["node", "server.js"]
```

**For different base images:**

```dockerfile
# Alpine
RUN apk add --no-cache curl

# Ubuntu/Debian
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

# Node.js
# curl usually pre-installed
```

### Issue 2: Health Endpoint Missing

**Symptom:**

```
Received HTTP 404 Not Found
Task marked unhealthy
```

**Cause:**
Your application doesn't have a `/health` endpoint.

**Solution:**

Add a health check endpoint to your application:

**Node.js/Express:**

```javascript
app.get("/health", (req, res) => {
  res.json({ status: "healthy" });
});
```

**Python/Flask:**

```python
@app.route('/health')
def health():
    return {'status': 'healthy'}
```

**Go:**

```go
http.HandleFunc("/health", func(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(map[string]string{"status": "healthy"})
})
```

**Java/Spring:**

```java
@GetMapping("/health")
public ResponseEntity<Map<String, String>> health() {
    return ResponseEntity.ok(Map.of("status", "healthy"));
}
```

### Issue 3: Wrong Port in Health Check

**Symptom:**

```
Connection refused on port 8000
```

**Cause:**
Health check is looking at wrong port, or application isn't listening.

**Solution:**

Verify in your task definition:

```yaml
Health Check:
  Command: curl -f http://localhost:8000/health || exit 1

Container Port Mapping:
  Container Port: 8000
  Host Port: 8000
```

Ensure application listens on the correct port:

```javascript
const PORT = process.env.PORT || 8000;
app.listen(PORT, () => {
  console.log(`Server listening on port ${PORT}`);
});
```

### Issue 4: Health Check Timeout Too Short

**Symptom:**

```
Health check timeout reached (5 seconds)
```

**Cause:**
Application takes longer than 5 seconds to respond.

**Solution:**

Increase timeout in task definition:

```yaml
Health Check:
  Command: curl -f http://localhost:8000/health || exit 1
  Timeout: 10 seconds (increased from 5)
  Interval: 30 seconds
```

### Issue 5: Firewall/Security Group Blocking

**Symptom:**

```
Connection timeout on health check
```

**Cause:**
Security group doesn't allow health check traffic.

**Solution:**

Check security group inbound rules:

```yaml
Inbound Rules:
  - Type: Custom TCP
    Port: 8000
    Source: 0.0.0.0/0 or sg-xxxxx (ALB security group)
```

## Debugging Health Check Failures

### Step 1: Check Task Logs

```bash
aws logs tail /ecs/api-app --follow
```

Look for errors like:

- "curl: command not found"
- "Connection refused"
- "503 Service Unavailable"

### Step 2: Manually Test Your Image

Build and run locally with same configuration:

```bash
# Build image
docker build -t myapp .

# Run with same ports and environment
docker run -p 8000:8000 \
  -e NODE_ENV=production \
  myapp

# In another terminal, test health check
curl -f http://localhost:8000/health
echo $?  # Should output 0 (success)
```

### Step 3: Check Application Startup

Your app might not be ready when health check runs.

**Solution:** Increase start period:

```yaml
Health Check:
  Start Period: 60 seconds (increased from 30)
  Command: curl -f http://localhost:8000/health || exit 1
  Interval: 30 seconds
```

The start period is how long to wait before STARTING health checks.

### Step 4: Inspect Running Task

If task is running but failing health check:

```bash
# Get task details
aws ecs describe-tasks \
  --cluster dev-cluster \
  --tasks <task-arn>

# Look for health status in output
# "healthStatus": "UNHEALTHY"
# "statusReason": "..."
```

### Step 5: Test with Disabled Health Check

Temporarily disable health check to isolate the problem:

1. Create new task definition revision
2. Remove health check configuration (leave empty)
3. Update service with new revision
4. If service becomes healthy, problem is with health check

Then fix the actual health check command.

## Complete Health Check Setup Example

### Step 1: Update Dockerfile

```dockerfile
FROM node:18-alpine

# Install curl for health checks
RUN apk add --no-cache curl

WORKDIR /app
COPY package*.json ./
RUN npm install --production

COPY . .

EXPOSE 8000

CMD ["node", "server.js"]
```

### Step 2: Add Health Endpoint

```javascript
// server.js
const express = require("express");
const app = express();

app.get("/health", (req, res) => {
  res.status(200).json({
    status: "healthy",
    timestamp: new Date().toISOString(),
  });
});

app.get("/api/data", (req, res) => {
  res.json({ data: "your data" });
});

app.listen(8000, () => {
  console.log("Server running on port 8000");
});
```

### Step 3: Task Definition Configuration

```yaml
Health Check:
  Command:
    - CMD-SHELL
    - curl -f http://localhost:8000/health || exit 1

  Interval: 30 seconds
  Timeout: 5 seconds
  Start Period: 30 seconds
  Max Retries: 3
```

**What this means:**

- Wait 30 seconds after container start
- Every 30 seconds, run health check
- Allow 5 seconds for response
- 3 consecutive failures = unhealthy

### Step 4: Test Locally Before Deployment

```bash
# Build
docker build -t myapp .

# Run
docker run -p 8000:8000 \
  --health-cmd='curl -f http://localhost:8000/health || exit 1' \
  --health-interval=30s \
  --health-timeout=5s \
  --health-start-period=30s \
  --health-retries=3 \
  myapp

# Wait 30 seconds, then test
sleep 30
curl http://localhost:8000/health

# Check health status
docker ps
# Look for "health: starting" → "health: healthy"
```

## Advanced Health Check Strategies

### Readiness Check (Application Ready)

Endpoint to check if app is truly ready:

```javascript
let isReady = false;

app.get("/health/ready", (req, res) => {
  if (!isReady) {
    res.status(503).json({ status: "not ready" });
    return;
  }
  res.json({ status: "ready" });
});

// After database connection, cache warmup, etc
setTimeout(() => {
  isReady = true;
  console.log("Application is ready for traffic");
}, 5000);
```

### Liveness Check (Application Alive)

Endpoint to check if app is still running:

```javascript
app.get("/health/live", (req, res) => {
  res.json({ status: "alive" });
});
```

### Use Readiness for ECS Health Checks

```yaml
Health Check:
  Command: curl -f http://localhost:8000/health/ready || exit 1
  Start Period: 60 seconds (give app time to initialize)
```

## Prevention Tips

1. **Always include curl** in image (especially Alpine)
2. **Always implement /health endpoint** in your app
3. **Test locally first** before pushing to ECS
4. **Use reasonable timeouts** (5-10 seconds typically)
5. **Increase start period** for slow-starting apps
6. **Monitor logs** for actual error messages
7. **Don't hardcode localhost** - use environment variables for port

## Key-takeaways

Health check failures usually stem from:

1. **Missing curl**: Install curl in Dockerfile
2. **Missing /health endpoint**: Add to application
3. **Wrong port**: Verify port matches task definition
4. **Timeout too short**: Increase from 5 to 10 seconds
5. **Start period too short**: Give app time to initialize

**The Fix:**

1. Fix Dockerfile → Push to ECR
2. Create new task definition revision → Update image URL
3. Update service → Force new deployment
4. Monitor logs → Verify healthy status

Most health check failures are simple fixes once you identify the actual error in the logs.
