# Pulumi Cheatsheet

*Infrastructure as Code with Python, Go, TypeScript & More*

## 1. Fundamentals & Setup

### Installation & Configuration

```bash
# macOS (Homebrew)
brew install pulumi

# Windows (Chocolatey)
choco install pulumi

# Linux (curl)
curl -fsSL https://get.pulumi.com | sh

# Docker Installation
docker run -it pulumi/pulumi:latest

# Verify Installation
pulumi version
```

### Initial Setup & Authentication

```bash
# Login to Pulumi Cloud
pulumi login

# Login to Local File Storage
pulumi login file://~/.pulumi

# Login to S3 Backend
pulumi login s3://my-bucket

# Create New Project
pulumi new aws-python
pulumi new aws-typescript
pulumi new gcp-go
pulumi new azure-csharp

# List Available Templates
pulumi new --help

# Create in Current Directory
pulumi new aws-python --dir .

# Logout
pulumi logout
```

### Project Structure

```
my-pulumi-project/
├── Pulumi.yaml
├── Pulumi.dev.yaml
├── Pulumi.prod.yaml
├── __main__.py
├── requirements.txt
├── index.ts
├── package.json
└── .pulumi/
```

### Pulumi.yaml Configuration

```yaml
name: my-infrastructure
runtime: python
description: Infrastructure automation with Pulumi
main: main.py
config:
  aws:region: us-east-1
  database:size: db.t3.micro
  environment: dev
```

### Stack Management

```bash
# Create Stack
pulumi stack init prod

# List All Stacks
pulumi stack ls

# Select Stack
pulumi stack select prod

# Show Stack Info
pulumi stack

# Rename Stack
pulumi stack rename old-name new-name

# Delete Stack
pulumi stack rm prod

# Export Stack
pulumi stack export > stack-backup.json

# Import Stack
pulumi stack import < stack-backup.json

# Set Configuration Values
pulumi config set environment prod
pulumi config set aws:region us-west-2
pulumi config set database:size db.t3.micro
pulumi config set instance:count 5

# Set Secret Values
pulumi config set --secret db_password "password123"
pulumi config set --secret api_key "secret_key"

# Get Configuration Values
pulumi config get environment
pulumi config get --all

# Show All Configuration
pulumi config show
```

## 2. Core Concepts & Programming Model

### Python Program Structure

```python
import pulumi
import pulumi_aws as aws
from pulumi import Config

# Get Configuration
config = Config()
env = config.get("environment") or "dev"
region = config.get("aws:region") or "us-east-1"

# Create VPC
vpc = aws.ec2.Vpc("main-vpc",
    cidr_block="10.0.0.0/16",
    tags={"Environment": env}
)

# Create Subnet
subnet = aws.ec2.Subnet("main-subnet",
    vpc_id=vpc.id,
    cidr_block="10.0.1.0/24",
    availability_zone="us-east-1a"
)

# Create Instance
instance = aws.ec2.Instance("web-server",
    ami="ami-0c55b159cbfafe1f0",
    instance_type="t3.micro",
    subnet_id=subnet.id,
    tags={"Name": "web-server"}
)

# Export Outputs
pulumi.export("vpc_id", vpc.id)
pulumi.export("subnet_id", subnet.id)
pulumi.export("instance_id", instance.id)
```

### TypeScript Program Structure

```typescript
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";

const config = new pulumi.Config();
const env = config.get("environment") || "dev";
const region = config.get("aws:region") || "us-east-1";

// Create VPC
const vpc = new aws.ec2.Vpc("main-vpc", {
    cidrBlock: "10.0.0.0/16",
    tags: { Environment: env }
});

// Create Subnet
const subnet = new aws.ec2.Subnet("main-subnet", {
    vpcId: vpc.id,
    cidrBlock: "10.0.1.0/24",
    availabilityZone: "us-east-1a"
});

// Create Instance
const instance = new aws.ec2.Instance("web-server", {
    ami: "ami-0c55b159cbfafe1f0",
    instanceType: "t3.micro",
    subnetId: subnet.id,
    tags: { Name: "web-server" }
});

// Export Outputs
pulumi.export("vpc_id", vpc.id);
pulumi.export("subnet_id", subnet.id);
pulumi.export("instance_id", instance.id);
```

### Dependencies

```python
# Implicit Dependency (Automatic)
bucket = aws.s3.Bucket("my-bucket")

bucket_object = aws.s3.BucketObject("file.txt",
    bucket=bucket.id,
    content="Hello World"
)

# Explicit Dependency
lambda_fn = aws.lambda_.Function("my-fn",
    code=pulumi.FileArchive("./lambda"),
    opts=pulumi.ResourceOptions(
        depends_on=[bucket]
    )
)

# Multiple Dependencies
resource = aws.some.Resource("name",
    opts=pulumi.ResourceOptions(
        depends_on=[vpc, subnet, sg]
    )
)

# Parent-Child Relationship
parent = aws.resource.Parent("parent")
child = aws.resource.Child("child",
    parent_id=parent.id,
    opts=pulumi.ResourceOptions(parent=parent)
)
```

### Outputs & Exports

```python
# Simple Export
pulumi.export("bucket_name", bucket.id)
pulumi.export("bucket_arn", bucket.arn)

# Multiple Exports
pulumi.export("outputs", {
    "bucket_name": bucket.id,
    "bucket_arn": bucket.arn,
    "endpoint_url": endpoint.url
})

# Reference Outputs from Another Stack
stack_ref = pulumi.StackReference(f"org/project/{stack_name}")
bucket_name = stack_ref.get_output("bucket_name")
database_url = stack_ref.get_output("database_url")

# Conditional Exports
if environment == "prod":
    pulumi.export("database_endpoint", db.endpoint)
    pulumi.export("api_url", api.url)
```

### Output Transformations

```python
# Apply Transformation
upper_name = bucket_name.apply(
    lambda name: name.upper()
)

# Combine Multiple Outputs
combined = pulumi.Output.concat(
    bucket.arn,
    "/",
    bucket.id
)

# Map & Filter Outputs
instance_ids = pulumi.Output.all(
    instance1.id,
    instance2.id
).apply(lambda ids: [id.upper() for id in ids])

# All With Multiple Outputs
config_output = pulumi.Output.all(
    bucket.id,
    bucket.arn,
    vpc.id
).apply(lambda args: {
    "bucket_id": args[0],
    "bucket_arn": args[1],
    "vpc_id": args[2]
})
```

## 3. AWS EC2 & Networking (100+ Operations)

### EC2 Instances

```python
# Basic Instance
instance = aws.ec2.Instance("web-server",
    ami="ami-0c55b159cbfafe1f0",
    instance_type="t3.micro"
)

# Instance with Full Configuration
instance_full = aws.ec2.Instance("app-server",
    ami="ami-0c55b159cbfafe1f0",
    instance_type="t3.small",
    key_name="my-keypair",
    security_groups=["default"],
    associate_public_ip_address=True,
    user_data="""#!/bin/bash
apt-get update
apt-get install -y nodejs
npm start""",
    root_block_device={
        "volume_size": 30,
        "volume_type": "gp3",
        "delete_on_termination": True
    },
    tags={"Name": "app-server", "Environment": "prod"}
)

# Export Instance Details
pulumi.export("instance_id", instance.id)
pulumi.export("instance_ip", instance.public_ip)
```

### VPC & Networking

```python
# Create VPC
vpc = aws.ec2.Vpc("main-vpc",
    cidr_block="10.0.0.0/16",
    enable_dns_hostnames=True,
    enable_dns_support=True,
    tags={"Name": "main-vpc"}
)

# Public Subnet
public_subnet = aws.ec2.Subnet("public-subnet",
    vpc_id=vpc.id,
    cidr_block="10.0.1.0/24",
    availability_zone="us-east-1a",
    map_public_ip_on_launch=True
)

# Private Subnet
private_subnet = aws.ec2.Subnet("private-subnet",
    vpc_id=vpc.id,
    cidr_block="10.0.2.0/24",
    availability_zone="us-east-1b"
)

# Internet Gateway
igw = aws.ec2.InternetGateway("main-igw",
    vpc_id=vpc.id,
    tags={"Name": "main-igw"}
)

# Elastic IP for NAT
eip = aws.ec2.Eip("nat-eip",
    vpc=True,
    depends_on=[igw]
)

# NAT Gateway
nat_gateway = aws.ec2.NatGateway("nat-gateway",
    subnet_id=public_subnet.id,
    allocation_id=eip.id
)

# Public Route Table
public_rt = aws.ec2.RouteTable("public-routes",
    vpc_id=vpc.id,
    routes=[
        aws.ec2.RouteTableRouteArgs(
            cidr_block="0.0.0.0/0",
            gateway_id=igw.id
        )
    ]
)

# Private Route Table
private_rt = aws.ec2.RouteTable("private-routes",
    vpc_id=vpc.id,
    routes=[
        aws.ec2.RouteTableRouteArgs(
            cidr_block="0.0.0.0/0",
            nat_gateway_id=nat_gateway.id
        )
    ]
)

# Associate Public Subnet with Public Route Table
pub_assoc = aws.ec2.RouteTableAssociation("public",
    subnet_id=public_subnet.id,
    route_table_id=public_rt.id
)

# Associate Private Subnet with Private Route Table
priv_assoc = aws.ec2.RouteTableAssociation("private",
    subnet_id=private_subnet.id,
    route_table_id=private_rt.id
)
```

### Security Groups

```python
# Web Server Security Group
web_sg = aws.ec2.SecurityGroup("web-sg",
    vpc_id=vpc.id,
    description="Allow HTTP/HTTPS",
    ingress=[
        aws.ec2.SecurityGroupIngressArgs(
            protocol="tcp",
            from_port=80,
            to_port=80,
            cidr_blocks=["0.0.0.0/0"]
        ),
        aws.ec2.SecurityGroupIngressArgs(
            protocol="tcp",
            from_port=443,
            to_port=443,
            cidr_blocks=["0.0.0.0/0"]
        ),
        aws.ec2.SecurityGroupIngressArgs(
            protocol="tcp",
            from_port=22,
            to_port=22,
            cidr_blocks=["10.0.0.0/8"]
        )
    ],
    egress=[
        aws.ec2.SecurityGroupEgressArgs(
            protocol="-1",
            from_port=0,
            to_port=0,
            cidr_blocks=["0.0.0.0/0"]
        )
    ]
)

# Database Security Group
db_sg = aws.ec2.SecurityGroup("db-sg",
    vpc_id=vpc.id,
    description="Allow database access",
    ingress=[
        aws.ec2.SecurityGroupIngressArgs(
            protocol="tcp",
            from_port=5432,
            to_port=5432,
            security_groups=[web_sg.id]
        )
    ]
)
```

### Auto Scaling

```python
# Launch Template
launch_template = aws.ec2.LaunchTemplate("web",
    image_id="ami-0c55b159cbfafe1f0",
    instance_type="t3.micro",
    key_name="my-keypair",
    security_group_names=["default"]
)

# Auto Scaling Group
asg = aws.autoscaling.Group("web-asg",
    min_size=1,
    max_size=5,
    desired_capacity=2,
    launch_template={
        "id": launch_template.id,
        "version": "$Latest"
    },
    vpc_zone_identifiers=["subnet-1", "subnet-2"],
    tags=[
        {"key": "Name", "value": "web-server", "propagate_at_launch": True}
    ]
)

# Scaling Policy - Scale Up
scale_up = aws.autoscaling.Policy("scale-up",
    adjustment_type="ChangeInCapacity",
    autoscaling_group_name=asg.name,
    scaling_adjustment=1,
    cooldown=300
)

# Scaling Policy - Scale Down
scale_down = aws.autoscaling.Policy("scale-down",
    adjustment_type="ChangeInCapacity",
    autoscaling_group_name=asg.name,
    scaling_adjustment=-1,
    cooldown=300
)
```

## 4. AWS S3 & Storage Services

### S3 Buckets

```python
# Private Bucket
bucket = aws.s3.Bucket("my-bucket",
    acl="private",
    versioning={"enabled": True},
    server_side_encryption_configuration={
        "rule": {
            "apply_server_side_encryption_by_default": {
                "sse_algorithm": "AES256"
            }
        }
    }
)

# Public Bucket with CORS
public_bucket = aws.s3.Bucket("public-bucket",
    acl="public-read",
    cors_rules=[{
        "allowed_headers": ["*"],
        "allowed_methods": ["GET", "HEAD"],
        "allowed_origins": ["https://example.com"],
        "max_age_seconds": 3000
    }]
)

# Block Public Access
pab = aws.s3.BucketPublicAccessBlock("pab",
    bucket=bucket.id,
    block_public_acls=True,
    block_public_policy=True,
    ignore_public_acls=True,
    restrict_public_buckets=True
)

# Upload Text Object
txt_object = aws.s3.BucketObject("file.txt",
    bucket=bucket.id,
    key="file.txt",
    content="Hello, World!"
)

# Upload File Object
file_object = aws.s3.BucketObject("app.zip",
    bucket=bucket.id,
    key="app.zip",
    source=pulumi.FileAsset("./app.zip")
)

# Upload Directory
dir_object = aws.s3.BucketObject("app",
    bucket=bucket.id,
    key="app/",
    source=pulumi.FileArchive("./src")
)

# Lifecycle Policy
lifecycle = aws.s3.BucketLifecycleConfigurationV2("lifecycle",
    bucket=bucket.id,
    rules=[{
        "id": "archive-old",
        "status": "Enabled",
        "transitions": [{
            "days": 90,
            "storage_class": "GLACIER"
        }],
        "expiration": {"days": 365}
    }]
)

# Logging
logging_bucket = aws.s3.Bucket("logging-bucket")
logging = aws.s3.BucketLogging("logging",
    bucket=bucket.id,
    target_bucket=logging_bucket.id,
    target_prefix="logs/"
)
```

### S3 Bucket Policies

```python
import json

# Allow Public Read
bucket_policy = aws.s3.BucketPolicy("policy",
    bucket=bucket.id,
    policy=bucket.arn.apply(lambda arn: json.dumps({
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": f"{arn}/*"
        }]
    }))
)

# Allow Specific AWS Account
account_policy = aws.s3.BucketPolicy("account",
    bucket=bucket.id,
    policy=bucket.arn.apply(lambda arn: json.dumps({
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:root"
            },
            "Action": ["s3:GetObject", "s3:PutObject"],
            "Resource": f"{arn}/*"
        }]
    }))
)
```

## 5. AWS RDS & Database Services

### RDS Instances

```python
# PostgreSQL Instance
postgres = aws.rds.Instance("postgres",
    identifier="mydb",
    database_name="mydb",
    username="admin",
    password="password123",
    instance_class="db.t3.micro",
    engine="postgres",
    engine_version="13.7",
    allocated_storage=20,
    publicly_accessible=False,
    skip_final_snapshot=True,
    backup_retention_period=7,
    backup_window="03:00-04:00",
    maintenance_window="mon:04:00-mon:05:00",
    tags={"Name": "postgres-db"}
)

# MySQL Instance
mysql = aws.rds.Instance("mysql",
    identifier="mysqldb",
    database_name="mydb",
    username="admin",
    password="password123",
    instance_class="db.t3.small",
    engine="mysql",
    engine_version="8.0.28",
    allocated_storage=50,
    multi_az=True,
    storage_encrypted=True
)

# Export Database Endpoint
pulumi.export("db_endpoint", postgres.endpoint)
pulumi.export("db_port", postgres.port)
```

### Aurora & Advanced RDS

```python
# Aurora Cluster
aurora_cluster = aws.rds.Cluster("aurora",
    cluster_identifier="aurora-cluster",
    engine="aurora-postgresql",
    engine_version="13.7",
    master_username="admin",
    master_password="password123",
    database_name="mydb",
    backup_retention_period=7,
    skip_final_snapshot=True
)

# Aurora Instance
aurora_instance = aws.rds.ClusterInstance("aurora-instance",
    cluster_identifier=aurora_cluster.id,
    instance_class="db.t3.small",
    engine=aurora_cluster.engine,
    engine_version=aurora_cluster.engine_version
)

# Database User
db_user = aws.rds.User("admin",
    instance_id=postgres.identifier,
    name="admin",
    password="password123"
)

# RDS Proxy
rds_proxy = aws.rds.Proxy("proxy",
    engine_family="POSTGRESQL",
    role_arn="arn:aws:iam::123456789012:role/proxy",
    auth={
        "auth_scheme": "SECRETS",
        "secret_arn": "arn:aws:secretsmanager:us-east-1:123456789012:secret:db"
    }
)
```

## 6. AWS Lambda & API Gateway

### Lambda Functions

```python
# Create IAM Role
import json

lambda_role = aws.iam.Role("lambda-role",
    assume_role_policy=json.dumps({
        "Version": "2012-10-17",
        "Statement": [{
            "Action": "sts:AssumeRole",
            "Effect": "Allow",
            "Principal": {"Service": "lambda.amazonaws.com"}
        }]
    })
)

# Attach Execution Policy
policy_attachment = aws.iam.RolePolicyAttachment("lambda-policy",
    role=lambda_role.name,
    policy_arn="arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
)

# Lambda from Zip File
lambda_fn = aws.lambda_.Function("hello",
    runtime="python3.9",
    role=lambda_role.arn,
    handler="lambda_function.lambda_handler",
    code=pulumi.FileArchive("./lambda.zip"),
    environment={"variables": {"ENV": "prod"}}
)

# Lambda with Inline Code
lambda_inline = aws.lambda_.Function("inline",
    runtime="python3.9",
    role=lambda_role.arn,
    handler="index.handler",
    code=pulumi.StringAsset("""
def handler(event, context):
    return {
        'statusCode': 200,
        'body': 'Hello, World!'
    }
""")
)

# Lambda with VPC
lambda_vpc = aws.lambda_.Function("vpc-fn",
    runtime="python3.9",
    role=lambda_role.arn,
    handler="index.handler",
    code=pulumi.FileArchive("./lambda.zip"),
    vpc_config={
        "subnet_ids": ["subnet-1", "subnet-2"],
        "security_group_ids": ["sg-1"]
    }
)

# Lambda with Environment Variables
lambda_env = aws.lambda_.Function("with-env",
    runtime="python3.9",
    role=lambda_role.arn,
    handler="index.handler",
    code=pulumi.FileArchive("./lambda.zip"),
    environment={
        "variables": {
            "DB_HOST": "db.example.com",
            "DB_PORT": "5432",
            "API_KEY": "secret"
        }
    }
)
```

### API Gateway

```python
# Create REST API
api = aws.apigateway.RestApi("api",
    description="REST API Gateway",
    binary_media_types=["application/octet-stream"]
)

# Create Resource
items_resource = aws.apigateway.Resource("items",
    rest_api=api.id,
    parent_id=api.root_resource_id,
    path_part="items"
)

# Create GET Method
get_method = aws.apigateway.Method("get-items",
    rest_api=api.id,
    resource_id=items_resource.id,
    http_method="GET",
    authorization="NONE"
)

# Create Lambda Integration
get_integration = aws.apigateway.Integration("get-integration",
    rest_api=api.id,
    resource_id=items_resource.id,
    http_method=get_method.http_method,
    type="AWS_PROXY",
    integration_http_method="POST",
    uri=lambda_fn.invoke_arn.apply(
        lambda arn: f"arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/{arn}/invocations"
    )
)

# Create POST Method
post_method = aws.apigateway.Method("post-items",
    rest_api=api.id,
    resource_id=items_resource.id,
    http_method="POST",
    authorization="NONE"
)

# Deploy API
deployment = aws.apigateway.Deployment("api-deployment",
    rest_api=api.id,
    stage_name="prod",
    opts=pulumi.ResourceOptions(depends_on=[get_integration, post_method])
)

# Lambda Permission
get_permission = aws.lambda_.Permission("api-permission",
    action="lambda:InvokeFunction",
    function=lambda_fn.name,
    principal="apigateway.amazonaws.com",
    source_arn=api.execution_arn.apply(lambda arn: f"{arn}/*")
)

# Export API URL
pulumi.export("api_url", deployment.invoke_url)
```

## 7. Security, IAM & Encryption

### IAM Roles & Policies

```python
# Create IAM Role
role = aws.iam.Role("app-role",
    assume_role_policy=json.dumps({
        "Version": "2012-10-17",
        "Statement": [{
            "Action": "sts:AssumeRole",
            "Effect": "Allow",
            "Principal": {"Service": "ec2.amazonaws.com"}
        }]
    })
)

# Inline Policy
inline_policy = aws.iam.RolePolicy("s3-access",
    role=role.id,
    policy=json.dumps({
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Action": ["s3:GetObject", "s3:PutObject"],
            "Resource": "arn:aws:s3:::bucket/*"
        }]
    })
)

# Managed Policy Attachment
managed_policy = aws.iam.RolePolicyAttachment("lambda-exec",
    role=role.name,
    policy_arn="arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
)

# Instance Profile
instance_profile = aws.iam.InstanceProfile("profile",
    role=role.name
)

# Use in EC2
instance = aws.ec2.Instance("server",
    ami="ami-0c55b159cbfafe1f0",
    instance_type="t3.micro",
    iam_instance_profile=instance_profile.name
)
```

### Encryption & Secrets

```python
# KMS Key
kms_key = aws.kms.Key("main",
    description="Main encryption key",
    enable_key_rotation=True
)

# KMS Alias
kms_alias = aws.kms.Alias("main-alias",
    name="alias/main",
    target_key_id=kms_key.id
)

# Secrets Manager
secret = aws.secretsmanager.Secret("db-password",
    description="Database password"
)

secret_version = aws.secretsmanager.SecretVersion("db-password-v1",
    secret_id=secret.id,
    secret_string="my-secure-password"
)

# S3 Encryption with KMS
encrypted_bucket = aws.s3.Bucket("encrypted",
    server_side_encryption_configuration={
        "rule": {
            "apply_server_side_encryption_by_default": {
                "sse_algorithm": "aws:kms",
                "kms_master_key_id": kms_key.arn
            }
        }
    }
)

# RDS Encryption
rds_encrypted = aws.rds.Instance("secure-db",
    storage_encrypted=True,
    kms_key_id=kms_key.arn
)

# EBS Encryption
ebs_volume = aws.ec2.Volume("encrypted-volume",
    availability_zone="us-east-1a",
    size=100,
    encrypted=True,
    kms_key_id=kms_key.id
)
```

### Network Security (NACL, Flow Logs, WAF)

```python
# Network ACL
nacl = aws.ec2.NetworkAcl("main",
    vpc_id=vpc.id,
    ingress=[
        aws.ec2.NetworkAclIngressArgs(
            protocol="tcp",
            rule_no=100,
            action="allow",
            cidr_block="10.0.0.0/8",
            from_port=0,
            to_port=65535
        ),
        aws.ec2.NetworkAclIngressArgs(
            protocol="tcp",
            rule_no=110,
            action="allow",
            cidr_block="0.0.0.0/0",
            from_port=80,
            to_port=80
        )
    ],
    egress=[
        aws.ec2.NetworkAclEgressArgs(
            protocol="-1",
            rule_no=100,
            action="allow",
            cidr_block="0.0.0.0/0",
            from_port=0,
            to_port=65535
        )
    ]
)

# VPC Flow Logs
flow_logs = aws.ec2.FlowLog("main",
    iam_role_arn="arn:aws:iam::123456789012:role/vpc-flow-logs",
    log_destination="arn:aws:logs:us-east-1:123456789012:log-group:vpc-flow-logs",
    traffic_type="ALL",
    vpc_id=vpc.id
)

# WAF Web ACL
web_acl = aws.wafv2.WebAcl("main",
    scope="REGIONAL",
    default_action={"allow": {}},
    rules=[{
        "name": "RateLimitRule",
        "priority": 1,
        "action": {"block": {}},
        "statement": {
            "rate_based_statement": {"limit": 2000}
        },
        "visibility_config": {
            "cloudwatch_metrics_enabled": True,
            "metric_name": "RateLimitRule",
            "sampled_requests_enabled": True
        }
    }],
    visibility_config={
        "cloudwatch_metrics_enabled": True,
        "metric_name": "WebACL",
        "sampled_requests_enabled": True
    }
)
```

## 8. Monitoring, Logging & Troubleshooting

### CloudWatch Monitoring

```python
# Log Group
log_group = aws.cloudwatch.LogGroup("app-logs",
    retention_in_days=30
)

# Log Stream
log_stream = aws.cloudwatch.LogStream("app-stream",
    log_group_name=log_group.name
)

# Metric Alarm - High CPU
cpu_alarm = aws.cloudwatch.MetricAlarm("high-cpu",
    comparison_operator="GreaterThanThreshold",
    evaluation_periods=2,
    metric_name="CPUUtilization",
    namespace="AWS/EC2",
    period=300,
    statistic="Average",
    threshold=80.0,
    alarm_description="Alert when CPU > 80%",
    dimensions={"InstanceId": instance.id},
    alarm_actions=["arn:aws:sns:us-east-1:123456789012:alerts"]
)

# Metric Alarm - Disk Space
disk_alarm = aws.cloudwatch.MetricAlarm("low-disk",
    comparison_operator="LessThanThreshold",
    evaluation_periods=1,
    metric_name="DiskSpaceUtilization",
    namespace="Custom",
    period=300,
    statistic="Average",
    threshold=10.0
)

# Dashboard
dashboard = aws.cloudwatch.Dashboard("main",
    dashboard_name="main-dashboard",
    dashboard_body=json.dumps({
        "widgets": [{
            "type": "metric",
            "properties": {
                "metrics": [
                    ["AWS/EC2", "CPUUtilization"],
                    ["AWS/RDS", "DatabaseConnections"]
                ],
                "period": 300,
                "stat": "Average",
                "region": "us-east-1",
                "title": "System Metrics"
            }
        }]
    })
)

# SNS Topic
sns_topic = aws.sns.Topic("alerts")
sns_subscription = aws.sns.TopicSubscription("email",
    topic=sns_topic.arn,
    protocol="email",
    endpoint="admin@example.com"
)
```

### CLI Debugging Commands

```bash
# View Detailed Logs
pulumi up -v
pulumi preview -v
pulumi destroy -v

# Enable Debug Mode
PULUMI_DEBUG=true pulumi up

# Check Stack Status
pulumi stack ls
pulumi stack show

# List Resources
pulumi stack --show-urns

# Refresh State
pulumi refresh

# Export State
pulumi stack export > state.json

# Import State
pulumi stack import < state.json

# Show Resource Details
pulumi preview --diff

# Validate Configuration
pulumi config show
```

### Structured Lambda Logging

```python
import logging
import sys
import json

# Configure Logger
logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)

# Console Handler
handler = logging.StreamHandler(sys.stdout)
handler.setFormatter(logging.Formatter(
    '{"timestamp": "%(asctime)s", "level": "%(levelname)s", "message": "%(message)s"}'
))
logger.addHandler(handler)

# Lambda Handler
def lambda_handler(event, context):
    logger.info(f"Received event: {json.dumps(event)}")
    try:
        result = process_event(event)
        logger.info(f"Success: {result}")
        return {
            "statusCode": 200,
            "body": json.dumps(result)
        }
    except Exception as e:
        logger.error(f"Error: {str(e)}", exc_info=True)
        return {
            "statusCode": 500,
            "body": json.dumps({"error": str(e)})
        }
```

## 9. Deployment & CI/CD Pipeline

### Deployment Commands

```bash
# Preview Changes
pulumi preview
pulumi preview --diff

# Deploy Infrastructure
pulumi up
pulumi up --skip-preview
pulumi up --auto-approve

# Deploy with Parameters
pulumi up -c environment=prod
pulumi up -c aws:region=eu-west-1

# Destroy Infrastructure
pulumi destroy
pulumi destroy --auto-approve

# Target Specific Resource
pulumi up --target urn:pulumi:...

# Parallel Deployment (faster)
pulumi up --parallel 10
```

### GitHub Actions

```yaml
name: Deploy Infrastructure
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  preview:
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    steps:
      - uses: actions/checkout@v2
      - uses: pulumi/actions@v3
        with:
          command: preview
          stack-name: dev
        env:
          PULUMI_ACCESS_TOKEN: ${{ secrets.PULUMI_ACCESS_TOKEN }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

  deploy:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v2
      - uses: pulumi/actions@v3
        with:
          command: up
          stack-name: prod
        env:
          PULUMI_ACCESS_TOKEN: ${{ secrets.PULUMI_ACCESS_TOKEN }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### GitLab CI/CD

```yaml
stages:
  - preview
  - deploy

preview:
  stage: preview
  image: pulumi/pulumi:latest
  script:
    - pulumi stack select dev
    - pulumi preview
  only:
    - merge_requests

deploy:
  stage: deploy
  image: pulumi/pulumi:latest
  script:
    - pulumi stack select prod
    - pulumi up --auto-approve
  only:
    - main
  environment:
    name: production
```

## 10. Quick Reference - Essential Commands

| Command | Description |
|---|---|
| `pulumi new aws-python` | Initialize new Pulumi project |
| `pulumi login` | Authenticate with Pulumi Cloud |
| `pulumi stack init prod` | Create new deployment stack |
| `pulumi stack select prod` | Switch active stack |
| `pulumi config set env prod` | Set configuration value |
| `pulumi preview` | Preview infrastructure changes |
| `pulumi up` | Deploy infrastructure |
| `pulumi up --auto-approve` | Deploy without confirmation |
| `pulumi destroy` | Destroy infrastructure |
| `pulumi stack ls` | Show all stacks |
| `pulumi stack export` | Export stack state |
| `pulumi stack --show-urns` | List all resources |
| `pulumi refresh` | Reconcile state with cloud |
| `pulumi stack output` | Display exported values |
| `PULUMI_DEBUG=true pulumi up` | Enable detailed logging |
| `pulumi preview -v` | Show detailed preview |

## 11. Best Practices & Optimization

### Security Best Practices

- Store secrets in Pulumi Cloud or external secret managers
- Use IAM roles for service authentication instead of access keys
- Enable encryption at rest for all data stores
- Use VPC and security groups to restrict network access
- Enable MFA for AWS root account
- Rotate credentials regularly
- Use least privilege principle for IAM policies
- Enable logging and monitoring for all resources
- Use KMS for key management
- Implement resource tagging for access control

### Performance Best Practices

- Use appropriate instance types for workloads
- Enable auto-scaling for variable loads
- Use caching (ElastiCache, CloudFront) strategically
- Optimize database indexes and queries
- Use connection pooling for databases
- Implement CDN for static content
- Monitor and optimize Lambda memory allocation
- Use SQS for async processing
- Implement database read replicas
- Regular performance testing and benchmarking

### Code Quality Practices

- Use type hints (Python) or strong typing (TypeScript)
- Implement proper error handling
- Use descriptive resource names
- Organize code with components and classes
- Avoid hardcoded values - use configuration
- Version control infrastructure code
- Add comprehensive comments and documentation
- Implement linting and code formatting
- Use unit tests for custom components
- Maintain consistent code style

### Anti-Patterns to Avoid

- Storing secrets in code or version control
- Using overly permissive IAM policies
- Hardcoding resource names and IDs
- Mixing multiple cloud providers without separation
- Creating large monolithic stacks
- Ignoring resource deletion failures
- Using default VPC and security groups
- Disabling CloudTrail or audit logging
- Not testing infrastructure changes
- Manually modifying cloud resources outside Pulumi

## 12. Architecture Patterns

### Pattern 1: Three-Tier Web Application

Architecture overview: Presentation Layer (Web) → Application Layer (Compute) → Data Layer (Database)

```python
def create_web_tier(vpc_id, subnet_id):
    sg = aws.ec2.SecurityGroup("web-sg",
        vpc_id=vpc_id,
        ingress=[
            aws.ec2.SecurityGroupIngressArgs(
                protocol="tcp",
                from_port=80,
                to_port=80,
                cidr_blocks=["0.0.0.0/0"]
            )
        ]
    )
    instance = aws.ec2.Instance("web-server",
        ami="ami-0c55b159cbfafe1f0",
        instance_type="t3.micro",
        subnet_id=subnet_id,
        security_groups=[sg.name]
    )
    return instance

def create_app_tier(vpc_id):
    role = aws.iam.Role("lambda-role", ...)
    fn = aws.lambda_.Function("api",
        role=role.arn,
        handler="index.handler"
    )
    return fn

def create_data_tier(vpc_id, subnet_ids):
    db_subnet_group = aws.rds.SubnetGroup("db",
        subnet_ids=subnet_ids
    )
    db = aws.rds.Instance("main-db",
        db_subnet_group_name=db_subnet_group.name
    )
    return db

# Deploy all tiers
web = create_web_tier(vpc.id, subnet.id)
app = create_app_tier(vpc.id)
db = create_data_tier(vpc.id, [subnet1.id, subnet2.id])
```

### Pattern 2: Event-Driven Data Pipeline

```python
# S3 bucket for raw data
raw_data_bucket = aws.s3.Bucket("raw-data",
    acl="private"
)

# SNS topic for notifications
notification_topic = aws.sns.Topic("data-events")

# Lambda for data processing
process_lambda = aws.lambda_.Function("processor",
    runtime="python3.9",
    handler="index.handler",
    code=pulumi.FileArchive("./processor.zip")
)

# S3 notification to Lambda
bucket_notification = aws.s3.BucketNotification("notify",
    bucket=raw_data_bucket.id,
    lambda_functions=[{
        "lambda_function_arn": process_lambda.arn,
        "events": ["s3:ObjectCreated:*"]
    }]
)

# DynamoDB for results
results_table = aws.dynamodb.Table("results",
    attributes=[{"name": "id", "type": "S"}],
    hash_key="id",
    billing_mode="PAY_PER_REQUEST"
)
```

### Pattern 3: Multi-Region Deployment

```python
regions = ["us-east-1", "eu-west-1", "ap-northeast-1"]

for region in regions:
    provider = aws.Provider(f"provider-{region}",
        region=region
    )
    bucket = aws.s3.Bucket(f"app-bucket-{region}",
        opts=pulumi.ResourceOptions(provider=provider)
    )
    instance = aws.ec2.Instance(f"server-{region}",
        ami="ami-0c55b159cbfafe1f0",
        instance_type="t3.micro",
        opts=pulumi.ResourceOptions(provider=provider)
    )
```

## 13. Testing & Infrastructure Validation

### Unit Testing Components

```python
import pytest
import pulumi

def test_vpc_creation():
    def run_pulumi_code():
        vpc = aws.ec2.Vpc("test-vpc",
            cidr_block="10.0.0.0/16"
        )
        pulumi.export("vpc_id", vpc.id)

    stack = pulumi.automation.create_or_select_stack(
        project_name="test",
        stack_name="test",
        program=run_pulumi_code
    )
    up_result = stack.up()
    assert up_result.summary.result == "succeeded"

def test_lambda_function():
    def run_pulumi_code():
        role = aws.iam.Role("lambda-role", ...)
        fn = aws.lambda_.Function("test-fn",
            runtime="python3.9",
            handler="index.handler"
        )
        pulumi.export("function_arn", fn.arn)

    stack = pulumi.automation.create_or_select_stack(...)
    up_result = stack.up()
    outputs = stack.outputs()
    assert "function_arn" in outputs
```

### Validation Testing with boto3

```python
import boto3

# Test S3 bucket exists and is encrypted
def test_s3_encryption():
    s3_client = boto3.client("s3")
    response = s3_client.get_bucket_encryption(
        Bucket="my-bucket"
    )
    assert response["ServerSideEncryptionConfiguration"]

# Test RDS security
def test_rds_security():
    rds_client = boto3.client("rds")
    response = rds_client.describe_db_instances(
        DBInstanceIdentifier="mydb"
    )
    db = response["DBInstances"][0]
    assert db["StorageEncrypted"] == True
    assert db["PubliclyAccessible"] == False

# Test API Gateway
def test_api_health():
    import requests
    response = requests.get("https://api.example.com/health")
    assert response.status_code == 200
```

## 14. Complete End-to-End Example: Serverless REST API

```python
import pulumi
import pulumi_aws as aws
import json

config = pulumi.Config()
table_name = config.get("table_name") or "items"

# Create DynamoDB Table
table = aws.dynamodb.Table("items",
    attributes=[
        {"name": "id", "type": "S"},
        {"name": "created_at", "type": "S"}
    ],
    hash_key="id",
    range_key="created_at",
    billing_mode="PAY_PER_REQUEST"
)

# Create IAM Role
lambda_role = aws.iam.Role("lambda-role",
    assume_role_policy=json.dumps({
        "Version": "2012-10-17",
        "Statement": [{
            "Action": "sts:AssumeRole",
            "Effect": "Allow",
            "Principal": {"Service": "lambda.amazonaws.com"}
        }]
    })
)

# DynamoDB Access Policy
policy = aws.iam.RolePolicy("lambda-policy",
    role=lambda_role.id,
    policy=table.arn.apply(lambda arn: json.dumps({
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Action": ["dynamodb:*"],
            "Resource": [arn]
        }]
    }))
)

# Lambda: GET /items
get_items = aws.lambda_.Function("get-items",
    runtime="python3.9",
    role=lambda_role.arn,
    handler="index.handler",
    code=pulumi.StringAsset("""
import json
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('items')

def handler(event, context):
    response = table.scan()
    return {
        'statusCode': 200,
        'body': json.dumps(response['Items'])
    }
"""),
    environment={"variables": {"TABLE_NAME": table.name}}
)

# Lambda: POST /items
create_item = aws.lambda_.Function("create-item",
    runtime="python3.9",
    role=lambda_role.arn,
    handler="index.handler",
    code=pulumi.StringAsset("""
import json
import boto3
import uuid

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('items')

def handler(event, context):
    body = json.loads(event['body'])
    item_id = str(uuid.uuid4())
    table.put_item(Item={'id': item_id, **body})
    return {
        'statusCode': 201,
        'body': json.dumps({'id': item_id})
    }
"""),
    environment={"variables": {"TABLE_NAME": table.name}}
)

# Create API Gateway
api = aws.apigateway.RestApi("api",
    description="Items API"
)

# Create /items resource
items_resource = aws.apigateway.Resource("items",
    rest_api=api.id,
    parent_id=api.root_resource_id,
    path_part="items"
)

# GET /items method
get_method = aws.apigateway.Method("get-items-method",
    rest_api=api.id,
    resource_id=items_resource.id,
    http_method="GET",
    authorization="NONE"
)

# GET integration
get_integration = aws.apigateway.Integration("get-integration",
    rest_api=api.id,
    resource_id=items_resource.id,
    http_method=get_method.http_method,
    type="AWS_PROXY",
    integration_http_method="POST",
    uri=get_items.invoke_arn.apply(
        lambda arn: f"arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/{arn}/invocations"
    )
)

# POST /items method
post_method = aws.apigateway.Method("post-items-method",
    rest_api=api.id,
    resource_id=items_resource.id,
    http_method="POST",
    authorization="NONE"
)

# POST integration
post_integration = aws.apigateway.Integration("post-integration",
    rest_api=api.id,
    resource_id=items_resource.id,
    http_method=post_method.http_method,
    type="AWS_PROXY",
    integration_http_method="POST",
    uri=create_item.invoke_arn.apply(
        lambda arn: f"arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/{arn}/invocations"
    )
)

# Deploy API
deployment = aws.apigateway.Deployment("api-deployment",
    rest_api=api.id,
    stage_name="prod",
    opts=pulumi.ResourceOptions(
        depends_on=[get_integration, post_integration]
    )
)

# Lambda Permissions
get_permission = aws.lambda_.Permission("get-api-permission",
    action="lambda:InvokeFunction",
    function=get_items.name,
    principal="apigateway.amazonaws.com",
    source_arn=api.execution_arn.apply(
        lambda arn: f"{arn}/*"
    )
)

post_permission = aws.lambda_.Permission("post-api-permission",
    action="lambda:InvokeFunction",
    function=create_item.name,
    principal="apigateway.amazonaws.com",
    source_arn=api.execution_arn.apply(
        lambda arn: f"{arn}/*"
    )
)

# Export Outputs
pulumi.export("api_url", deployment.invoke_url)
pulumi.export("table_name", table.name)
```

## 15. Troubleshooting & FAQ

### Common Issues & Solutions

**Resource Already Exists** — when an AWS resource already exists outside Pulumi:

```bash
pulumi import aws:ec2/instance:Instance name instance-id
pulumi import aws:s3/bucket:Bucket name bucket-name
pulumi import aws:rds/instance:Instance name db-id
```

**Insufficient Permissions** — ensure AWS credentials have the required permissions:

```bash
aws sts get-caller-identity
```

```json
// Add permissions to IAM policy
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "ec2:*",
    "Resource": "*"
  }]
}
```

**State Lock** — another deployment is in progress; wait or clear the lock:

```bash
pulumi stack
pulumi cancel
```

**Secrets Not Decrypting** — ensure the correct Pulumi backend and credentials are configured:

```bash
pulumi login
pulumi config show --all
pulumi refresh
```

### Frequently Asked Questions

**Q: Can I use Pulumi with Terraform state?**
A: Pulumi maintains its own state. Use `pulumi refresh` to reconcile with cloud resources.

**Q: How do I migrate from CloudFormation?**
A: Use Pulumi CrossWalk or manually rewrite resources. Both approaches work for different scenarios.

**Q: Can I manage multiple cloud providers?**
A: Yes! Pulumi supports AWS, GCP, Azure, Kubernetes, and more in a single program.

**Q: How do I handle sensitive data?**
A: Use `pulumi config set --secret` or external secret managers.

**Q: What's the difference between `pulumi up` and `deploy`?**
A: `pulumi up` is the correct command for creating/updating/deleting infrastructure.

### Performance Tips

- Use parallel deployments with the `--parallel` flag
- Minimize state file size by using cross-stack references
- Cache provider credentials to speed up operations
- Use resource targeting for partial updates
- Keep stack size manageable (< 500 resources)
- Separate concerns into multiple stacks
- Use data sources for read-only information
- Implement proper error handling in programs

### Debug Techniques

```bash
# Enable Debug Logging
PULUMI_DEBUG=true pulumi up

# Increase Verbosity
pulumi preview --logtostderr

# Export and Inspect State
pulumi stack export > state.json

# Check Resource Properties
pulumi stack --show-urns
pulumi preview --diff

# Validate Configuration
pulumi config show

# Test Program Syntax
python -m py_compile __main__.py
```

## 16. Cost Management & Advanced Topics

### Cost Analysis Tools

```python
# Cost Anomaly Detection
anomaly_monitor = aws.ce.AnomalyMonitor("spend",
    monitor_type="DIMENSIONAL",
    monitor_dimension="SERVICE"
)

# Monthly Budget Alert
budget = aws.budgets.Budget("monthly",
    budget_type="COST",
    limit_amount="5000",
    limit_unit="USD",
    time_period_start="2023-01-01",
    time_period_end="2099-12-31",
    time_unit="MONTHLY"
)

budget_notification = aws.budgets.BudgetNotification("alert",
    budget_name=budget.name,
    comparison_operator="GREATER_THAN",
    notification_type="FORECASTED",
    threshold=100,
    threshold_type="PERCENTAGE"
)

# Tag for Cost Tracking
instance = aws.ec2.Instance("app",
    ami="ami-0c55b159cbfafe1f0",
    instance_type="t3.micro",
    tags={
        "Cost-Center": "engineering",
        "Environment": "prod",
        "Project": "web-app"
    }
)
```

### Cost Optimization Strategies

- Use smaller instance types and scale horizontally
- Enable auto-scaling to match demand
- Use Spot instances for non-critical workloads
- Reserve capacity for predictable workloads
- Delete unused resources regularly
- Use managed services (Lambda, RDS) vs self-managed
- Implement lifecycle policies for S3 and EBS
- Monitor CloudWatch metrics for optimization
- Use VPC endpoints to reduce data transfer costs
- Consolidate workloads across accounts

### Advanced Topics: Automation API

```python
import pulumi.automation as auto

def create_stack_and_deploy():
    def program():
        bucket = aws.s3.Bucket("my-bucket")
        pulumi.export("bucket_name", bucket.id)

    # Create or select stack
    stack = auto.create_or_select_stack(
        project_name="my-project",
        stack_name="production",
        program=program
    )

    # Configure
    stack.set_config("aws:region",
        auto.ConfigValue(value="us-east-1")
    )

    # Deploy
    up_result = stack.up()
    print(f"Deployment: {up_result.summary.result}")
    return stack
```

### Importing Existing Resources

```python
import boto3

ec2 = boto3.client("ec2")
instances = ec2.describe_instances()

for reservation in instances["Reservations"]:
    for instance in reservation["Instances"]:
        instance_id = instance["InstanceId"]
        # pulumi import aws:ec2/instance:Instance name instance-id
```

---

*Source: adapted from the Pulumi cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
