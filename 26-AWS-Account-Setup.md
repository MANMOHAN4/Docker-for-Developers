# Creating and Configuring a New AWS Account for ECS Deployment

## Prerequisites

Before deploying containers on AWS ECS, you need:

1. An AWS account
2. AWS CLI installed on your local machine
3. A Docker image ready to deploy

## Cost Expectations

**Good News**: Learning AWS ECS costs very little!

- **Free Tier**: AWS offers 12 months of free tier (eligible for many services)
- **Expected Cost**: Approximately $0-2 USD for learning/testing
- **Card Verification**: AWS charges $2 INR (~3 cents USD) to verify your card, which is refundable
- **Fair Cost**: Consider it a learning investment - far cheaper than any other cloud platform

## AWS Account Setup

### Step 1: Create AWS Account

1. Visit [aws.amazon.com](https://aws.amazon.com)
2. Click "Create an AWS Account"
3. Provide email and password
4. Add payment information (for card verification)
5. Verify phone number
6. Choose support plan (free tier is fine for learning)

### Step 2: Choose Correct Region

AWS has data centers in different regions worldwide.

**Always choose the region nearest to you**:

- **Asia Pacific (Mumbai)** - ap-south-1 - Closest to India
- **Asia Pacific (Singapore)** - ap-southeast-1
- **Asia Pacific (Tokyo)** - ap-northeast-1
- **US East (N. Virginia)** - us-east-1 - Default
- **Europe (Ireland)** - eu-west-1

Lower latency = better performance.

## Installing AWS CLI

### Mac/Linux

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

Or using Homebrew (Mac):

```bash
brew install awscli
```

### Windows

1. Download installer from [AWS CLI Documentation](https://aws.amazon.com/cli/)
2. Run the MSI installer
3. Follow installation wizard

### Verify Installation

```bash
aws --version
```

Should output something like: `AWS CLI 2.15.0`

## Configuring AWS CLI

AWS CLI needs credentials to access your account.

### Step 1: Create IAM User

1. Log in to AWS Console
2. Search for "IAM" (Identity Access Management)
3. Go to Users section
4. Click "Create User"
5. Username: `docker-deployment` or similar
6. Click Next

### Step 2: Set Permissions

1. Click "Attach policies directly"
2. Search for "AmazonEC2FullAccess" - select it
3. Search for "AmazonECS_FullAccess" - select it
4. Search for "AmazonEC2ContainerRegistryFullAccess" - select it
5. Click "Create User"

### Step 3: Create Access Keys

1. Click on the newly created user
2. Go to "Security credentials" tab
3. Click "Create access key"
4. Choose "CLI"
5. Click "Create access key"
6. **Important**: Save the Access Key ID and Secret Access Key

### Step 4: Configure AWS CLI Locally

```bash
aws configure
```

When prompted:

- **AWS Access Key ID**: Paste your Access Key ID
- **AWS Secret Access Key**: Paste your Secret Access Key
- **Default region name**: Enter `ap-south-1` (or your region)
- **Default output format**: Press Enter (default is fine)

### Verify Configuration

```bash
aws sts get-caller-identity
```

Should output your AWS account information.

## AWS Console Navigation

### Important Services for ECS

1. **ECR** (Elastic Container Registry)
   - Where you store Docker images
   - Search: "Elastic Container Registry"

2. **ECS** (Elastic Container Service)
   - Where you run and manage containers
   - Search: "Elastic Container Service"

3. **IAM** (Identity and Access Management)
   - User and permission management
   - Search: "IAM"

4. **EC2** (Elastic Compute Cloud)
   - Virtual machines and infrastructure
   - Search: "EC2" (for load balancers)

5. **CloudWatch**
   - Logs and monitoring
   - Search: "CloudWatch"

## Setting Up Cost Alerts

Prevent unexpected charges:

### Enable Billing Alerts

1. Go to AWS Billing Console
2. Create a Billing Alarm
3. Set alert threshold to $5 USD
4. Provide email for notifications

## Best Practices for Cost Management

1. **Always use free tier eligible services** when possible
2. **Delete resources** when not using them (delete clusters, services, etc.)
3. **Monitor usage** regularly in AWS Billing Dashboard
4. **Use AWS Cost Explorer** to track spending
5. **Set up billing alerts** to catch unexpected charges
6. **Use Fargate** (serverless) instead of EC2 for better cost control

## What You Can Do with Free Tier

- ECS (limited)
- ECR (12-month free for first 500 MB stored)
- EC2 instances (750 hours/month)
- Load Balancer (limited)
- CloudWatch (limited)

## AWS Free Tier Limits

- **ECS**: Free to use (charges only for underlying compute)
- **Fargate**: $0.04048 per vCPU-hour, $0.004445 per GB-hour (minimal cost)
- **EC2**: 1 t2.micro instance free for 12 months
- **Load Balancer**: Not free, but < $1/day for minimal usage
