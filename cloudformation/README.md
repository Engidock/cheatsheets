# Cloud Formation Cheatsheet

## I. Fundamentals & CLI Operations

### 1. Setup & Configuration

Essential tools for authoring and deploying CloudFormation templates.

```bash
# 1. Install AWS CLI v2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip && sudo ./aws/install

# 2. Configure Profile
aws configure --profile dev
# AWS Access Key ID: AKIA...
# AWS Secret Access Key: wJalr...
# Default region: us-east-1

# 3. Install Linter (Python)
pip install cfn-lint
```

### 2. CLI Reference Table

| CLI Command | Description & Flags |
|---|---|
| `aws cloudformation create-stack` | Creates a new stack. Flags: `--stack-name`, `--template-body`, `--parameters`, `--capabilities` |
| `aws cloudformation update-stack` | Updates existing stack. Flags: `--use-previous-template`, `--parameters` |
| `aws cloudformation delete-stack` | Deletes stack and all resources. Flags: `--retain-resources` (for accidental deletion protection) |
| `aws cloudformation validate-template` | Checks JSON/YAML syntax. Example: `--template-body file://t.yaml` |
| `aws cloudformation describe-stack-events` | Debugging: Shows log of success/failure events. |
| `aws cloudformation get-template` | Retrieves the currently deployed code for a stack. |
| `aws cloudformation estimate-template-cost` | Returns a URL to the AWS Simple Monthly Calculator. |

### 3. The Package & Deploy Workflow

Required when your template references local artifacts (Lambda code, Nested templates). This automates the S3 upload process.

```bash
# Step 1: Package (Uploads artifacts to S3 & updates template)
aws cloudformation package \
  --template-file template.yaml \
  --s3-bucket my-deployment-bucket \
  --output-template-file packaged.yaml

# Step 2: Deploy (Creates ChangeSet & Executes)
aws cloudformation deploy \
  --template-file packaged.yaml \
  --stack-name my-app-stack \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides Env=Prod
```

## II. Template Logic & Functions

### 4. Template Anatomy (YAML)

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Standard VPC Template'
Parameters:
  Env:
    Type: String
    AllowedValues: [dev, prod]
Mappings:
  RegionMap:
    us-east-1: { AMI: "ami-123" }
Conditions:
  IsProd: !Equals [ !Ref Env, "prod" ]
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Condition: IsProd
    Properties:
      ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", AMI]
Outputs:
  InstanceIP:
    Value: !GetAtt MyInstance.PublicIp
    Export:
      Name: !Sub "${AWS::StackName}-IP"
```

### 5. Intrinsic Functions Cheat Sheet

| Function | Syntax | Use Case |
|---|---|---|
| `Fn::Ref` | `!Ref LogicalID` | Return Resource ID or Parameter Value. |
| `Fn::GetAtt` | `!GetAtt ID.Attribute` | Return specific attribute (PublicIP, Arn). |
| `Fn::Sub` | `!Sub "${Var}-suffix"` | String interpolation. Combines text with variables. |
| `Fn::Join` | `!Join [ delimiter, list ]` | Combine list into string. Useful for ARNs. |
| `Fn::Select` | `!Select [ index, list ]` | Get specific item from list. |
| `Fn::Split` | `!Split [ delimiter, string ]` | Turn string into list. Useful for comma-delimited parameters. |
| `Fn::ImportValue` | `!ImportValue ExportName` | Cross-stack reference. Read outputs from other stacks. |
| `Fn::Cidr` | `!Cidr [Block, Count, Bits]` | Calculate subnet ranges from a VPC block. |
| `Fn::Base64` | `!Base64 String` | Encode UserData scripts. |

### 6. Conditions & Control Flow

```yaml
Conditions:
  IsProduction: !Equals [ !Ref Environment, "prod" ]
  IsRegionUS: !Equals [ !Ref "AWS::Region", "us-east-1" ]
  ShouldCreateDB: !And [ !Condition IsProduction, !Condition IsRegionUS ]
Resources:
  MyDatabase:
    Type: AWS::RDS::DBInstance
    Condition: ShouldCreateDB
    Properties:
      AllocatedStorage: !If [ IsProduction, 100, 20 ]
```

## III. Common Resource Patterns

### S3 Bucket (Secure)

```yaml
MySecureBucket:
  Type: AWS::S3::Bucket
  DeletionPolicy: Retain
  Properties:
    BucketEncryption:
      ServerSideEncryptionConfiguration:
        - ServerSideEncryptionByDefault:
            SSEAlgorithm: AES256
    PublicAccessBlockConfiguration:
      BlockPublicAcls: true
      BlockPublicPolicy: true
      IgnorePublicAcls: true
      RestrictPublicBuckets: true
```

### DynamoDB Table

```yaml
MyTable:
  Type: AWS::DynamoDB::Table
  Properties:
    AttributeDefinitions:
      - AttributeName: PK
        AttributeType: S
    KeySchema:
      - AttributeName: PK
        KeyType: HASH
    BillingMode: PAY_PER_REQUEST
```

### Lambda Function (Inline)

```yaml
MyFunction:
  Type: AWS::Lambda::Function
  Properties:
    Handler: index.handler
    Role: !GetAtt LambdaExecutionRole.Arn
    Runtime: python3.9
    Code:
      ZipFile: !Sub |
        def handler(event, context):
            print("Hello from CloudFormation!")
            return "Success"
```

## IV. Advanced Architecture

### 10. Nested Stacks (Modularization)

Use for creating reusable modules (e.g., standard VPC) or overcoming the 500-resource limit.

```yaml
Resources:
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/vpc.yaml
      Parameters:
        CidrBlock: 10.0.0.0/16
  AppStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/app.yaml
      Parameters:
        VpcId: !GetAtt NetworkStack.Outputs.VpcId
```

### 11. Cross-Stack References

Locking Hazard: You cannot delete the Producer Stack if a Consumer Stack imports its output.

```yaml
# Producer Stack (Network)
Outputs:
  VpcId:
    Value: !Ref MyVPC
    Export:
      Name: Shared-VpcId

# Consumer Stack (App)
Resources:
  MySG:
    Type: AWS::EC2::SecurityGroup
    Properties:
      VpcId: !ImportValue Shared-VpcId
```

### 12. StackSets (Multi-Account)

Deploy to multiple accounts and regions from a central admin account.

```bash
# Create the Container
aws cloudformation create-stack-set \
  --stack-set-name SecurityBase \
  --template-url https://s3.../security.yaml \
  --permission-model SERVICE_MANAGED \
  --auto-deployment Enabled=true

# Deploy Instances
aws cloudformation create-stack-instances \
  --stack-set-name SecurityBase \
  --deployment-targets OrganizationalUnitIds=ou-1234 \
  --regions us-east-1 eu-west-1 \
  --operation-preferences FailureToleranceCount=1
```

## V. Security & Governance

### 13. IAM & Service Roles

Best Practice: Don't use your user permissions. Pass a Service Role to the stack.

PassRole: The user running this command needs `iam:PassRole` permission on the Service Role.

```bash
aws cloudformation create-stack \
  --stack-name prod-database \
  --template-body file://db.yaml \
  --role-arn arn:aws:iam::123456789012:role/CFNServiceRole
```

### 14. Stack Policies

JSON document that prevents accidental updates to specific resources during a stack update.

```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "Update:*",
      "Principal": "*",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": ["Update:Replace", "Update:Delete"],
      "Principal": "*",
      "Resource": "LogicalResourceId/MyProdDB"
    }
  ]
}
```

## VI. Troubleshooting & Limits

### 15. Critical Resource Attributes

| Attribute | Behavior |
|---|---|
| `DependsOn` | Wait for another resource to complete before starting this one. |
| `DeletionPolicy` | `Retain`: Keeps resource when stack is deleted. `Snapshot`: Takes backup before deletion (RDS/EBS). |
| `UpdateReplacePolicy` | Same as above, but triggers when a resource is replaced during update. |
| `CreationPolicy` | Pauses stack creation until `cfn-signal` is received (EC2/ASG). |

### 16. Troubleshooting Common Errors

Log File Location (EC2): `/var/log/cloud-init-output.log`

| Error Message | Solution |
|---|---|
| `UPDATE_ROLLBACK_FAILED` | A resource failed to delete during rollback (e.g., non-empty bucket). Use "Continue Update Rollback" in console and "Skip" that resource. |
| `WaitCondition Timed Out` | Instance failed to send signal. Check Internet Gateway, NAT Gateway, or UserData script syntax errors. |
| `Circular Dependency` | Resource A -> Resource B -> Resource A. Refactor by removing inline references (e.g. use separate `SecurityGroupIngress` resource). |
| `Export/Import Error` | Cannot delete a stack because another stack imports its output. You must update the Consumer stack first. |
| `Invalid Parameter Value` | Input does not match `AllowedValues` or `AllowedPattern`. Check regex in template. |
| `InsufficientCapabilities` | Template creates IAM resources. Add flag `--capabilities CAPABILITY_IAM`. |

### 17. Production Readiness Checklist

- [x] Termination Protection: Enabled for stateful stacks (DB, Storage).
- [x] DeletionPolicy: Set to `Retain` for critical data.
- [x] IAM Permissions: Stack uses a Least Privilege Service Role.
- [x] Drift: Detection run regularly.
- [x] Tags: `CostCenter` and `Owner` tags applied.
- [ ] Don't hardcode secrets (Use Secrets Manager).
- [ ] Don't put Lambda code inline (Use S3).

---

*Source: adapted from the Cloud Formation cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
