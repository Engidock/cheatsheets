# Terraform Cheatsheet

Complete quick reference guide for Terraform and Infrastructure as Code — covering core commands, state management, variables, modules, data sources, provisioners, workspaces, backends, functions, and best practices.

## 🚀 Basic Commands

### Initialization & Setup

Initialize Terraform:

```bash
terraform init
```

Initialize with backend:

```bash
terraform init -backend-config="key=value"
```

Re-initialize backend:

```bash
terraform init -reconfigure
```

Validate configuration:

```bash
terraform validate
```

Format code:

```bash
terraform fmt
terraform fmt -recursive
```

### Plan & Apply

Create execution plan:

```bash
terraform plan
```

Save plan to file:

```bash
terraform plan -out=tfplan
```

Plan for specific resource:

```bash
terraform plan -target=aws_instance.example
```

Apply changes:

```bash
terraform apply
```

Apply saved plan:

```bash
terraform apply tfplan
```

Auto-approve apply:

```bash
terraform apply -auto-approve
```

Apply specific resource:

```bash
terraform apply -target=aws_instance.example
```

## 💾 State Management

### State Operations

List state resources:

```bash
terraform state list
```

Show resource state:

```bash
terraform state show aws_instance.example
```

Move resource in state:

```bash
terraform state mv aws_instance.old aws_instance.new
```

Remove resource from state:

```bash
terraform state rm aws_instance.example
```

Replace/recreate a resource on next apply:

```bash
terraform apply -replace="aws_instance.example"
```

Backup state:

```bash
terraform state pull > backup.tfstate
```

Force-unlock a stuck state lock:

```bash
terraform force-unlock LOCK_ID
```

## 📋 Variables & Configuration

### Input Variables

Define variable:

```hcl
variable "instance_type" {
  type        = string
  default     = "t2.micro"
  description = "EC2 instance type"
}
```

Set variables from CLI:

```bash
terraform apply -var="instance_type=t2.small"
```

Set variables from file:

```bash
terraform apply -var-file="prod.tfvars"
```

Use environment variables:

```bash
export TF_VAR_instance_type="t2.small"
terraform apply
```

### Output Variables

Define output:

```hcl
output "instance_ip" {
  value       = aws_instance.example.public_ip
  description = "Public IP of instance"
}
```

Display outputs:

```bash
terraform output
```

Get specific output:

```bash
terraform output instance_ip
```

## 🗑️ Destroy & Cleanup

### Destruction Operations

Destroy infrastructure:

```bash
terraform destroy
```

Auto-approve destroy:

```bash
terraform destroy -auto-approve
```

Destroy specific resource:

```bash
terraform destroy -target=aws_instance.example
```

Remove local state:

```bash
rm terraform.tfstate*
```

## 🔍 Debugging & Inspection

### Debugging Tools

Show graph:

```bash
terraform graph | dot -Tsvg > graph.svg
```

Show state:

```bash
terraform show
```

JSON output:

```bash
terraform show -json
```

Enable debug logging:

```bash
export TF_LOG=DEBUG
terraform apply
```

Show variable values (interactive console):

```bash
terraform console
```

## 📦 Modules

### Module Management

Call module:

```hcl
module "vpc" {
  source = "./modules/vpc"

  cidr_block = "10.0.0.0/16"
  name       = "my-vpc"
}
```

Module from registry:

```hcl
module "ec2" {
  source  = "terraform-aws-modules/ec2-instance/aws"
  version = "5.0.0"

  name = "my-instance"
}
```

Get module outputs:

```hcl
output "vpc_id" {
  value = module.vpc.vpc_id
}
```

Module from Git:

```hcl
module "network" {
  source = "git::https://github.com/user/repo.git"
}
```

## 🔗 Data Sources

### Data Source Examples

Fetch AMI:

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}
```

Get existing VPC:

```hcl
data "aws_vpc" "default" {
  default = true
}
```

Get security group:

```hcl
data "aws_security_group" "default" {
  name   = "default"
  vpc_id = data.aws_vpc.default.id
}
```

Read local file:

```hcl
data "local_file" "script" {
  filename = "${path.module}/scripts/init.sh"
}
```

## ⚙️ Provisioners

### Provisioner Types

Local-exec provisioner:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  provisioner "local-exec" {
    command = "echo ${self.public_ip} > ip_address.txt"
  }
}
```

Remote-exec provisioner:

```hcl
provisioner "remote-exec" {
  inline = [
    "sudo yum update -y",
    "sudo yum install -y nginx"
  ]

  connection {
    type        = "ssh"
    user        = "ec2-user"
    private_key = file("~/.ssh/id_rsa")
    host        = self.public_ip
  }
}
```

File provisioner:

```hcl
provisioner "file" {
  source      = "app/"
  destination = "/home/ec2-user/app"

  connection {
    type        = "ssh"
    user        = "ec2-user"
    private_key = file("~/.ssh/id_rsa")
    host        = self.public_ip
  }
}
```

## 🏢 Workspaces

### Workspace Operations

List workspaces:

```bash
terraform workspace list
```

Create workspace:

```bash
terraform workspace new prod
```

Switch workspace:

```bash
terraform workspace select prod
```

Delete workspace:

```bash
terraform workspace delete prod
```

Current workspace in code:

```hcl
${terraform.workspace}
```

## ☁️ Backends

### Backend Configuration

S3 backend:

```hcl
terraform {
  backend "s3" {
    bucket         = "my-bucket"
    key            = "terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
  }
}
```

Azure backend:

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-name"
    storage_account_name = "storage-name"
    container_name       = "container-name"
    key                  = "terraform.tfstate"
  }
}
```

GCS backend:

```hcl
terraform {
  backend "gcs" {
    bucket = "my-bucket"
    prefix = "terraform/state"
  }
}
```

Migrate backend:

```bash
terraform init -migrate-state
```

## 🔢 Functions & Expressions

### Common Functions

String functions:

```hcl
upper("hello")              # HELLO
lower("HELLO")              # hello
join(",", ["a", "b"])       # a,b
split(",", "a,b,c")         # ["a", "b", "c"]
```

Collection functions:

```hcl
length([1, 2, 3])           # 3
contains([1, 2, 3], 2)      # true
merge({a = 1}, {b = 2})     # {a = 1, b = 2}
```

Numeric functions:

```hcl
max(1, 2, 3)                # 3
min(1, 2, 3)                # 1
ceil(4.3)                   # 5
```

Date & time functions:

```hcl
timestamp()                                # Current time
formatdate("YYYY-MM-DD", timestamp())      # Formatted date
```

## ✅ Best Practices & Patterns

- **Use remote state** — Store state in S3, Azure Storage, or GCS
- **Enable state locking** — Use DynamoDB with S3 to prevent race conditions
- **Use workspaces** — Separate dev, staging, and production environments
- **Organize with modules** — Break infrastructure into reusable components
- **Use variables** — Make configurations reusable and environment-specific
- **Version control everything** — Commit `.tf` files to Git (exclude `.tfstate`)
- **Add comments** — Document complex resources and logic
- **Use data sources** — Reference existing resources instead of hardcoding

### Troubleshooting Checklist

**🔍 State conflicts?**

1. Check state lock status when a command fails with a lock error
2. Pull remote state: `terraform state pull`
3. Force unlock: `terraform force-unlock LOCK_ID`

**📊 Resources not found?**

1. Check state: `terraform state list`
2. Refresh state: `terraform refresh`
3. Import resource: `terraform import aws_instance.example i-1234567890abcdef0`

**⚡ Plan shows unexpected changes?**

1. Check variables: `terraform plan -var-file="prod.tfvars"`
2. Refresh state: `terraform refresh`
3. Target specific resource: `terraform plan -target=aws_instance.example`

### 💡 Environment Variables

| Variable | Purpose |
|---|---|
| `TF_VAR_*` | Set input variables |
| `TF_LOG` | Enable logging (`DEBUG`, `INFO`, `WARN`) |
| `TF_INPUT` | Set to `false` to disable input prompts |

### 📋 Quick Reference Flags

| Command | Purpose | Common Options |
|---|---|---|
| `terraform init` | Initialize workspace | `-backend-config`, `-reconfigure` |
| `terraform plan` | Create execution plan | `-out=file`, `-target=resource` |
| `terraform apply` | Apply changes | `-auto-approve`, `-target=resource` |
| `terraform destroy` | Destroy infrastructure | `-auto-approve`, `-target=resource` |
| `terraform validate` | Validate configuration | `-json` |
| `terraform fmt` | Format code | `-recursive`, `-write=false` |
| `terraform state` | Manage state | `list`, `show`, `mv`, `rm` |
| `terraform output` | Display outputs | `-json`, `[output_name]` |

## 📚 Resource Types

### AWS Resources

- `aws_instance`
- `aws_s3_bucket`
- `aws_vpc`
- `aws_security_group`
- `aws_rds_instance`
- `aws_lambda_function`

### Azure Resources

- `azurerm_virtual_machine`
- `azurerm_storage_account`
- `azurerm_virtual_network`
- `azurerm_sql_database`
- `azurerm_app_service`
- `azurerm_container_registry`

### GCP Resources

- `google_compute_instance`
- `google_storage_bucket`
- `google_compute_network`
- `google_sql_database_instance`
- `google_cloud_run_service`
- `google_container_cluster`

### Other Resources

- `kubernetes_deployment`
- `docker_image`
- `github_repository`
- `helm_release`
- `vault_generic_secret`
- `http_resource`

---

*Source: adapted from the Terraform cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
