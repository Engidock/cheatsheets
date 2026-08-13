# AWS Cheatsheet

Complete, detailed reference guide for Amazon Web Services (AWS) cloud computing — CLI commands, configuration snippets, and best practices for day-to-day work.

## 🎯 AWS Fundamentals

### AWS Account & Setup

Initial Setup checklist:

1. Create AWS Account
2. Enable MFA on root account
3. Create IAM users for daily use
4. Set up billing alerts
5. Enable CloudTrail for auditing
6. Configure AWS Organizations (for multiple accounts)
7. Set up cost allocation tags

AWS CLI Installation:

```bash
# macOS
brew install awscli

# Linux
sudo apt-get install awscli

# Windows
# Download installer from AWS

# Configure credentials
aws configure
# Enter: Access Key ID, Secret Access Key, Region, Output format

# Verify installation
aws sts get-caller-identity
```

AWS Regions & Availability Zones:

```bash
aws ec2 describe-regions --query 'Regions[].RegionName' --output table
aws ec2 describe-availability-zones --region us-east-1

# Set default region
export AWS_DEFAULT_REGION=us-east-1
aws configure set region us-west-2
```

## 🖥️ EC2 - Elastic Compute Cloud

### EC2 Instance Management

Create & Launch Instances:

```bash
# Launch instance with CLI
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t2.micro \
  --key-name MyKeyPair \
  --security-groups MySecurityGroup \
  --count 1

# Launch with IAM role
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --iam-instance-profile Name=MyRole \
  --instance-type t2.micro

# Launch with user data script
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --user-data file://script.sh
```

Instance Operations:

```bash
# List instances
aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,State.Name,PublicIpAddress]' --output table

# Start/Stop/Reboot
aws ec2 start-instances --instance-ids i-0123456789abcdef0
aws ec2 stop-instances --instance-ids i-0123456789abcdef0
aws ec2 reboot-instances --instance-ids i-0123456789abcdef0

# Terminate instance
aws ec2 terminate-instances --instance-ids i-0123456789abcdef0

# Get instance metadata
aws ec2 describe-instances --instance-ids i-0123456789abcdef0
```

Security Groups:

```bash
# Create security group
aws ec2 create-security-group \
  --group-name MySecurityGroup \
  --description "My security group"

# Add inbound rule (HTTP)
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Add inbound rule (SSH)
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

# Remove rule
aws ec2 revoke-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

### Key Pairs & Access

Key Pair Management:

```bash
# Create key pair
aws ec2 create-key-pair --key-name MyKeyPair --query 'KeyMaterial' --output text > MyKeyPair.pem
chmod 400 MyKeyPair.pem

# List key pairs
aws ec2 describe-key-pairs

# Delete key pair
aws ec2 delete-key-pair --key-name MyKeyPair

# SSH into instance
ssh -i MyKeyPair.pem ec2-user@ec2-public-ip
ssh -i MyKeyPair.pem ubuntu@ec2-public-ip
```

## 📦 S3 - Simple Storage Service

### S3 Bucket Operations

Bucket Management:

```bash
# Create bucket
aws s3 mb s3://my-bucket-name
aws s3 mb s3://my-bucket-name --region us-west-2

# List buckets
aws s3 ls
aws s3api list-buckets --query 'Buckets[].Name'

# Delete empty bucket
aws s3 rb s3://my-bucket-name

# Delete non-empty bucket
aws s3 rb s3://my-bucket-name --force

# List bucket contents
aws s3 ls s3://my-bucket-name
aws s3 ls s3://my-bucket-name --recursive
```

File Operations:

```bash
# Upload file
aws s3 cp myfile.txt s3://my-bucket-name/

# Upload directory
aws s3 cp mydir/ s3://my-bucket-name/ --recursive

# Download file
aws s3 cp s3://my-bucket-name/myfile.txt ./

# Download directory
aws s3 cp s3://my-bucket-name/ ./ --recursive

# Sync directories
aws s3 sync s3://my-bucket-name/ ./
aws s3 sync ./ s3://my-bucket-name/

# Delete file
aws s3 rm s3://my-bucket-name/myfile.txt

# Delete multiple files
aws s3 rm s3://my-bucket-name/ --recursive
```

S3 Configuration:

```bash
# Enable versioning
aws s3api put-bucket-versioning \
  --bucket my-bucket-name \
  --versioning-configuration Status=Enabled

# Set bucket policy
aws s3api put-bucket-policy \
  --bucket my-bucket-name \
  --policy file://policy.json

# Enable encryption
aws s3api put-bucket-encryption \
  --bucket my-bucket-name \
  --server-side-encryption-configuration file://encryption.json

# Set bucket lifecycle
aws s3api put-bucket-lifecycle-configuration \
  --bucket my-bucket-name \
  --lifecycle-configuration file://lifecycle.json

# Set public access block
aws s3api put-public-access-block \
  --bucket my-bucket-name \
  --public-access-block-configuration file://block.json
```

## 🗄️ RDS - Relational Database Service

### Database Instance Management

Create Database:

```bash
# Create MySQL instance
aws rds create-db-instance \
  --db-instance-identifier mydb \
  --db-instance-class db.t2.micro \
  --engine mysql \
  --master-username admin \
  --master-user-password mypassword123 \
  --allocated-storage 20 \
  --backup-retention-period 7

# Create PostgreSQL instance
aws rds create-db-instance \
  --db-instance-identifier mydb \
  --db-instance-class db.t2.micro \
  --engine postgres \
  --master-username postgres \
  --master-user-password mypassword123

# Create Multi-AZ instance
aws rds create-db-instance \
  --db-instance-identifier mydb \
  --multi-az
```

Database Operations:

```bash
# List instances
aws rds describe-db-instances --query 'DBInstances[].[DBInstanceIdentifier,DBInstanceStatus,Engine]' --output table

# Get endpoint
aws rds describe-db-instances \
  --db-instance-identifier mydb \
  --query 'DBInstances[0].Endpoint.Address'

# Modify instance
aws rds modify-db-instance \
  --db-instance-identifier mydb \
  --db-instance-class db.t2.small \
  --apply-immediately

# Create snapshot
aws rds create-db-snapshot \
  --db-instance-identifier mydb \
  --db-snapshot-identifier mydb-snapshot

# Restore from snapshot
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier mydb-restored \
  --db-snapshot-identifier mydb-snapshot

# Delete instance
aws rds delete-db-instance \
  --db-instance-identifier mydb \
  --skip-final-snapshot
```

## 🌐 VPC - Virtual Private Cloud

### VPC & Networking

VPC Creation:

```bash
# Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Create subnet
aws ec2 create-subnet \
  --vpc-id vpc-123456 \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a

# Create internet gateway
aws ec2 create-internet-gateway

# Attach IGW to VPC
aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-123456 \
  --vpc-id vpc-123456

# Create route table
aws ec2 create-route-table --vpc-id vpc-123456

# Add route
aws ec2 create-route \
  --route-table-id rtb-123456 \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-123456
```

VPC Management:

```bash
# List VPCs
aws ec2 describe-vpcs --output table

# List subnets
aws ec2 describe-subnets --output table

# List security groups
aws ec2 describe-security-groups --output table

# Enable DNS
aws ec2 modify-vpc-attribute \
  --vpc-id vpc-123456 \
  --enable-dns-support

# Enable DNS hostnames
aws ec2 modify-vpc-attribute \
  --vpc-id vpc-123456 \
  --enable-dns-hostnames
```

## 📊 CloudWatch - Monitoring & Logging

### Monitoring & Metrics

CloudWatch Metrics:

```bash
# List metrics
aws cloudwatch list-metrics --namespace AWS/EC2 --output table

# Get metric statistics
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-0123456789abcdef0 \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-02T00:00:00Z \
  --period 3600 \
  --statistics Average,Maximum

# Put custom metric
aws cloudwatch put-metric-data \
  --namespace MyNamespace \
  --metric-name MyMetric \
  --value 10
```

CloudWatch Alarms:

```bash
# Create alarm
aws cloudwatch put-metric-alarm \
  --alarm-name my-alarm \
  --alarm-description "CPU utilization alarm" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2

# List alarms
aws cloudwatch describe-alarms --output table

# Delete alarm
aws cloudwatch delete-alarms --alarm-names my-alarm

# Set alarm state
aws cloudwatch set-alarm-state \
  --alarm-name my-alarm \
  --state-value ALARM \
  --state-reason "Testing"
```

CloudWatch Logs:

```bash
# Create log group
aws logs create-log-group --log-group-name /aws/lambda/my-function

# Create log stream
aws logs create-log-stream \
  --log-group-name /aws/lambda/my-function \
  --log-stream-name my-stream

# Put log events
aws logs put-log-events \
  --log-group-name /aws/lambda/my-function \
  --log-stream-name my-stream \
  --log-events timestamp=$(date +%s000),message="Test message"

# Get log events
aws logs get-log-events \
  --log-group-name /aws/lambda/my-function \
  --log-stream-name my-stream
```

## 🔐 IAM - Identity & Access Management

### User & Role Management

User Management:

```bash
# Create user
aws iam create-user --user-name john

# List users
aws iam list-users --output table

# Create access keys
aws iam create-access-key --user-name john

# Attach policy to user
aws iam attach-user-policy \
  --user-name john \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess

# Create inline policy
aws iam put-user-policy \
  --user-name john \
  --policy-name AllowS3 \
  --policy-document file://policy.json

# Delete user
aws iam delete-user --user-name john
```

Role Management:

```bash
# Create role
aws iam create-role \
  --role-name MyRole \
  --assume-role-policy-document file://trust-policy.json

# List roles
aws iam list-roles --output table

# Attach policy to role
aws iam attach-role-policy \
  --role-name MyRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess

# Assume role
aws sts assume-role \
  --role-arn arn:aws:iam::123456789012:role/MyRole \
  --role-session-name my-session
```

Policy Management:

```bash
# Create policy
aws iam create-policy \
  --policy-name MyPolicy \
  --policy-document file://policy.json

# List policies
aws iam list-policies --scope Local --output table

# Get policy details
aws iam get-policy --policy-arn arn:aws:iam::123456789012:policy/MyPolicy

# Get policy version
aws iam get-policy-version \
  --policy-arn arn:aws:iam::123456789012:policy/MyPolicy \
  --version-id v1
```

## ⚙️ Lambda - Serverless Computing

### Lambda Functions

Create Function:

```bash
# Create function from zip
aws lambda create-function \
  --function-name my-function \
  --runtime python3.9 \
  --role arn:aws:iam::123456789012:role/lambda-role \
  --handler index.handler \
  --zip-file fileb://function.zip

# Create function with inline code / environment variables
aws lambda create-function \
  --function-name my-function \
  --runtime python3.9 \
  --role arn:aws:iam::123456789012:role/lambda-role \
  --handler index.handler \
  --zip-file fileb://function.zip \
  --environment Variables={KEY=value}

# List functions
aws lambda list-functions --output table

# Get function info
aws lambda get-function --function-name my-function
```

Invoke Functions:

```bash
# Invoke synchronously
aws lambda invoke \
  --function-name my-function \
  --payload '{"key":"value"}' \
  response.json

# Invoke asynchronously
aws lambda invoke \
  --function-name my-function \
  --invocation-type Event \
  --payload '{"key":"value"}' \
  response.json

# Update function code
aws lambda update-function-code \
  --function-name my-function \
  --zip-file fileb://function.zip

# Update configuration
aws lambda update-function-configuration \
  --function-name my-function \
  --timeout 60 \
  --memory-size 256

# Delete function
aws lambda delete-function --function-name my-function
```

## 📋 Elastic Load Balancing

### Load Balancer Management

ALB (Application Load Balancer):

```bash
# Create ALB
aws elbv2 create-load-balancer \
  --name my-alb \
  --subnets subnet-12345 subnet-67890 \
  --security-groups sg-12345

# Create target group
aws elbv2 create-target-group \
  --name my-targets \
  --protocol HTTP \
  --port 80 \
  --vpc-id vpc-12345

# Register targets
aws elbv2 register-targets \
  --target-group-arn arn:aws:elasticloadbalancing:... \
  --targets Id=i-123456 Id=i-789012

# Create listener
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:... \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=arn:aws:...
```

Load Balancer Operations:

```bash
# List load balancers
aws elbv2 describe-load-balancers --output table

# Get load balancer details
aws elbv2 describe-load-balancers \
  --load-balancer-arns arn:aws:elasticloadbalancing:...

# Describe target groups
aws elbv2 describe-target-groups --output table

# Check target health
aws elbv2 describe-target-health \
  --target-group-arn arn:aws:elasticloadbalancing:...

# Delete load balancer
aws elbv2 delete-load-balancer \
  --load-balancer-arn arn:aws:elasticloadbalancing:...
```

## 🚀 Auto Scaling

### Auto Scaling Groups

Launch Template:

```bash
# Create launch template
aws ec2 create-launch-template \
  --launch-template-name my-template \
  --launch-template-data '{"ImageId":"ami-12345","InstanceType":"t2.micro"}'

# Create Auto Scaling group
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --launch-template LaunchTemplateName=my-template,Version='$Latest' \
  --min-size 1 \
  --max-size 5 \
  --desired-capacity 2 \
  --vpc-zone-identifier subnet-12345,subnet-67890

# List ASGs
aws autoscaling describe-auto-scaling-groups --output table

# Update ASG
aws autoscaling update-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --desired-capacity 3

# Delete ASG
aws autoscaling delete-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --force-delete
```

## 📦 Container Services

### ECS & ECR

ECR (Container Registry):

```bash
# Create ECR repository
aws ecr create-repository --repository-name my-app

# Get login token
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com

# Push image
docker tag my-app:latest 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest

# List images
aws ecr describe-images --repository-name my-app --output table

# Delete repository
aws ecr delete-repository --repository-name my-app --force
```

ECS (Container Orchestration):

```bash
# Create task definition
aws ecs register-task-definition \
  --family my-task \
  --container-definitions file://task-def.json

# Create ECS cluster
aws ecs create-cluster --cluster-name my-cluster

# Create service
aws ecs create-service \
  --cluster my-cluster \
  --service-name my-service \
  --task-definition my-task \
  --desired-count 2

# List services
aws ecs list-services --cluster my-cluster --output table

# Update service
aws ecs update-service \
  --cluster my-cluster \
  --service my-service \
  --desired-count 4
```

## 🔄 Deployment & Infrastructure

### CloudFormation & CodeDeploy

CloudFormation:

```bash
# Create stack
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-body file://template.yaml

# List stacks
aws cloudformation list-stacks --output table

# Describe stack
aws cloudformation describe-stacks --stack-name my-stack

# Update stack
aws cloudformation update-stack \
  --stack-name my-stack \
  --template-body file://template.yaml

# Delete stack
aws cloudformation delete-stack --stack-name my-stack

# Validate template
aws cloudformation validate-template --template-body file://template.yaml
```

CodeDeploy:

```bash
# Create application
aws deploy create-application --application-name my-app

# Create deployment
aws deploy create-deployment \
  --application-name my-app \
  --deployment-group-name my-group \
  --s3-location bucket=bucket,key=app.zip,bundleType=zip \
  --deployment-config-name CodeDeployDefault.OneAtATime

# List deployments
aws deploy list-deployments --output table

# Get deployment status
aws deploy get-deployment --deployment-id d-XXXXXXXXXX
```

## 💰 Cost Management & Optimization

### Billing & Cost

Cost Management:

```bash
# Get cost and usage
aws ce get-cost-and-usage \
  --time-period Start=2024-01-01,End=2024-01-31 \
  --granularity DAILY \
  --metrics BlendedCost \
  --group-by Type=DIMENSION,Key=SERVICE

# Get reserved instances
aws ec2 describe-reserved-instances --output table

# Describe savings plans
aws savingsplans describe-savings-plans --output table

# Set up budget alert
aws budgets create-budget \
  --account-id 123456789012 \
  --budget file://budget.json \
  --notifications-with-subscribers file://notifications.json
```

## 🔒 Security Best Practices

### Security Configuration

Security Best Practices:

- Enable MFA on all accounts
- Use IAM roles for EC2 instances
- Enable CloudTrail logging
- Use VPC with private subnets
- Enable encryption at rest and in transit
- Use security groups properly
- Regular key rotation
- Monitor with CloudWatch
- Use AWS Secrets Manager
- Enable GuardDuty for threat detection
- Enable AWS Config for compliance
- Use AWS Systems Manager for patches
- Regular security audits
- Follow principle of least privilege
- Use KMS for encryption

Security Operations:

```bash
# Enable CloudTrail
aws cloudtrail create-trail \
  --name my-trail \
  --s3-bucket-name my-bucket

# Start logging
aws cloudtrail start-logging --trail-name my-trail

# Enable GuardDuty
aws guardduty create-detector --enable

# Create AWS Config rule
aws configservice put-config-rule \
  --config-rule file://rule.json

# List IAM access keys
aws iam list-access-keys --user-name john

# Rotate access keys
aws iam delete-access-key --user-name john --access-key-id AKIA...
```

## 📋 AWS Services Quick Reference

| Service | Use Case | Command Example |
|---|---|---|
| EC2 | Virtual servers | `aws ec2 run-instances` |
| S3 | Object storage | `aws s3 cp file s3://bucket` |
| RDS | Managed database | `aws rds create-db-instance` |
| Lambda | Serverless compute | `aws lambda invoke` |
| ECS | Container orchestration | `aws ecs create-service` |
| DynamoDB | NoSQL database | `aws dynamodb scan` |
| SNS | Notifications | `aws sns publish` |
| SQS | Message queue | `aws sqs send-message` |
| VPC | Network isolation | `aws ec2 create-vpc` |
| CloudFormation | Infrastructure as Code | `aws cloudformation create-stack` |
| IAM | Access control | `aws iam create-user` |
| CloudWatch | Monitoring | `aws cloudwatch get-metric-statistics` |

## 🎓 AWS Common Patterns

- **Web Application**: ALB → EC2 / Lambda → RDS → S3 → CloudFront
- **Microservices**: ECS/EKS → API Gateway → Lambda → DynamoDB → SNS/SQS
- **Data Analytics**: S3 Data Lake → Glue → Athena → Redshift → QuickSight
- **Machine Learning**: SageMaker → S3 → Lambda → Rekognition → Comprehend
- **Disaster Recovery**: S3 backup → Cross-region → RTO/RPO → CloudFormation → Route 53
- **DevOps Pipeline**: CodeCommit → CodeBuild → CodePipeline → CodeDeploy → CloudFormation

## ✅ AWS Best Practices Checklist

**Account & Organization**
- Use AWS Organizations for multiple accounts
- Enable consolidated billing
- Set up cost allocation tags
- Enable CloudTrail globally
- Implement tagging strategy

**Compute & Infrastructure**
- Use Auto Scaling for HA
- Implement load balancing
- Use spot instances for cost savings
- Reserve capacity for baseline load
- Monitor CPU and memory

**Storage & Database**
- Enable versioning on S3
- Use lifecycle policies
- Enable encryption
- Regular backups
- Test restore procedures

**Network & Security**
- Use VPC with private subnets
- Implement security groups properly
- Use NACLs for additional control
- Enable VPC Flow Logs
- Use VPN for admin access

**Monitoring & Logging**
- Enable CloudWatch for all resources
- Set up CloudTrail
- Use VPC Flow Logs
- Configure alarms
- Regular log analysis

## 💡 Cost Optimization

- Right-size instances regularly
- Use Reserved Instances (30-40% savings)
- Use Savings Plans for flexibility
- Terminate unused resources
- Monitor and optimize data transfer

## ⚠️ Never

- Store credentials in code
- Use root account for daily work
- Skip backup and disaster recovery
- Ignore security groups
- Leave unused resources running

---
*Source: adapted from the AWS cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
