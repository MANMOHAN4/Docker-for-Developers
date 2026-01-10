# Setting Up Amazon Elastic Container Registry (ECR)

## What is ECR?

**ECR** = Elastic Container Registry

- AWS's Docker image repository
- Similar to Docker Hub, but managed by Amazon
- Integrated with AWS services
- Private repositories by default
- Secure and reliable

## Why Use ECR Instead of Docker Hub?

1. **Integration**: Works seamlessly with AWS ECS
2. **Security**: Built into AWS security model
3. **Performance**: Faster pulling in AWS
4. **Private by Default**: Your images are private
5. **No Public Exposure**: Don't expose images unless you want to

## Creating an ECR Repository

### Step 1: Navigate to ECR

1. Open AWS Console
2. Search for "ECR" (Elastic Container Registry)
3. Click on "Repositories"

### Step 2: Create Repository

1. Click "Create Repository"
2. **Repository Name**: Enter `backend` (or your app name)
3. **Visibility**: Leave as Private
4. Click "Create Repository"

### Result

You'll get a private repository URL:

```
[account-id].dkr.ecr.[region].amazonaws.com/[repo-name]
```

Example:

```
123456789012.dkr.ecr.ap-south-1.amazonaws.com/backend
```

## Pushing Images to ECR

### Step 1: Authenticate Docker

```bash
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin [account-id].dkr.ecr.ap-south-1.amazonaws.com
```

Replace:

- `ap-south-1` with your region
- `[account-id]` with your AWS account ID

### Step 2: Build Your Image

```bash
docker build -t backend:latest .
```

### Step 3: Tag Image for ECR

```bash
docker tag backend:latest [account-id].dkr.ecr.ap-south-1.amazonaws.com/backend:latest
```

### Step 4: Push to ECR

```bash
docker push [account-id].dkr.ecr.ap-south-1.amazonaws.com/backend:latest
```

## Complete Example

```bash
# 1. Authenticate
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.ap-south-1.amazonaws.com

# 2. Build
docker build -t backend:latest .

# 3. Tag
docker tag backend:latest 123456789012.dkr.ecr.ap-south-1.amazonaws.com/backend:latest

# 4. Push
docker push 123456789012.dkr.ecr.ap-south-1.amazonaws.com/backend:latest
```

## Viewing Images in ECR

1. Go to ECR console
2. Click on your repository
3. See:
   - Image URI
   - Push date
   - Image size
   - Digest (unique identifier)

## Image Tags

Tag images for different versions:

```bash
docker tag backend:latest [uri]:v1.0.0
docker tag backend:latest [uri]:production
docker tag backend:latest [uri]:latest

docker push [uri]:v1.0.0
docker push [uri]:production
docker push [uri]:latest
```

## Automatic Image Scanning

ECR can scan images for vulnerabilities:

1. In repository settings
2. Enable "Scan on push"
3. Images automatically scanned for security issues
4. View results in "Image scanning" tab

## Cleanup

To save costs, delete unused images:

1. Click on image
2. Click "Delete"
3. Confirm deletion

**Note**: You can't delete images currently in use by ECS services.

## Best Practices

1. **Use meaningful tags**: `v1.0.0`, `latest`, `production`
2. **Don't tag `latest` in production**: Use specific versions
3. **Enable scanning**: Catch security vulnerabilities
4. **Regular cleanup**: Delete old, unused images
5. **Keep private**: ECR repositories are private by default
6. **Use access policies**: Control who can access images

## Key-takeaways

ECR provides:

- Private Docker image storage
- AWS integration
- Security scanning
- Easy tagging and versioning
- Foundation for ECS deployments

Next step: Use these images in ECS task definitions and services.
