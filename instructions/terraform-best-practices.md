# Terraform Best Practices

## Purpose
Guide AI assistants to write maintainable, scalable, secure Infrastructure as Code using Terraform. Apply to cloud infrastructure provisioning, infrastructure management, DevOps automation, and multi-cloud deployments. These practices reflect modern Terraform 1.7+ capabilities and industry standards.

## Core Principles

### 0. Version Requirements and Provider Configuration

**Always specify explicit version constraints**:

```hcl
# Good: Explicit version requirements with modern syntax
terraform {
  required_version = ">= 1.7.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.6"
    }
  }

  # Use cloud backend for team collaboration
  cloud {
    organization = "myorg"
    workspaces {
      name = "production"
    }
  }
}

# Good: Provider configuration with assume role and default tags
provider "aws" {
  region = var.aws_region

  default_tags {
    tags = local.common_tags
  }

  assume_role {
    role_arn     = var.assume_role_arn
    session_name = "terraform-${var.environment}"
  }
}

# Good: Provider aliases for multi-region deployments
provider "aws" {
  alias  = "us_east_1"
  region = "us-east-1"

  default_tags {
    tags = local.common_tags
  }
}

provider "aws" {
  alias  = "us_west_2"
  region = "us-west-2"

  default_tags {
    tags = local.common_tags
  }
}
```

```hcl
# Bad: No version constraints, missing default tags
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}
```

### 1. Module Structure and Organization

**Always organize Terraform code into modules**:

```hcl
# Good: Modern module structure with comprehensive organization
project/
├── .terraform.lock.hcl          # Lock file for provider versions
├── environments/
│   ├── dev/
│   │   ├── backend.tf           # Backend configuration
│   │   ├── main.tf              # Main resource definitions
│   │   ├── variables.tf         # Input variables
│   │   ├── outputs.tf           # Output values
│   │   ├── terraform.tfvars     # Variable values
│   │   ├── locals.tf            # Local values
│   │   └── versions.tf          # Terraform and provider versions
│   ├── staging/
│   └── production/
├── modules/
│   ├── networking/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   ├── versions.tf
│   │   ├── README.md            # Module documentation
│   │   └── examples/            # Usage examples
│   │       └── basic/
│   ├── compute/
│   ├── database/
│   └── security/
├── tests/                       # Terraform tests
│   ├── networking_test.tftest.hcl
│   └── integration_test.tftest.hcl
├── .tflint.hcl                  # TFLint configuration
├── .terraform-docs.yml          # Terraform-docs configuration
└── README.md
```

```hcl
# Good: Module usage with dependency management
# environments/dev/main.tf

module "networking" {
  source = "../../modules/networking"

  vpc_cidr           = var.vpc_cidr
  environment        = var.environment
  availability_zones = var.availability_zones

  tags = local.common_tags
}

module "security" {
  source = "../../modules/security"

  vpc_id      = module.networking.vpc_id
  environment = var.environment

  allowed_cidr_blocks = var.allowed_cidr_blocks

  tags = local.common_tags

  depends_on = [module.networking]
}

module "compute" {
  source = "../../modules/compute"

  vpc_id             = module.networking.vpc_id
  private_subnet_ids = module.networking.private_subnet_ids
  security_group_id  = module.security.instance_security_group_id
  instance_type      = var.instance_type

  tags = local.common_tags

  depends_on = [module.security]
}

# Good: Use moved blocks to handle resource refactoring
moved {
  from = aws_instance.old_name
  to   = module.compute.aws_instance.new_name
}

moved {
  from = module.old_module
  to   = module.new_module
}
```

### 2. Variable Definitions

Define variables with validation and clear descriptions:

```hcl
# Good: Well-documented variables with advanced validation
variable "environment" {
  description = "Environment name (dev, staging, production)"
  type        = string

  validation {
    condition     = contains(["dev", "staging", "production"], var.environment)
    error_message = "Environment must be dev, staging, or production."
  }
}

variable "instance_type" {
  description = "EC2 instance type for compute resources"
  type        = string
  default     = "t3.micro"

  validation {
    condition     = can(regex("^(t3|t3a|t4g)\\.(nano|micro|small|medium|large|xlarge|2xlarge)$", var.instance_type))
    error_message = "Instance type must be a valid t3, t3a, or t4g instance type."
  }
}

variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string

  validation {
    condition     = can(cidrhost(var.vpc_cidr, 0))
    error_message = "Must be a valid CIDR block."
  }

  validation {
    condition     = can(regex("^10\\.", var.vpc_cidr)) || can(regex("^172\\.(1[6-9]|2[0-9]|3[0-1])\\.", var.vpc_cidr)) || can(regex("^192\\.168\\.", var.vpc_cidr))
    error_message = "VPC CIDR must be within private IP ranges (10.0.0.0/8, 172.16.0.0/12, or 192.168.0.0/16)."
  }
}

variable "tags" {
  description = "Common tags to apply to all resources"
  type        = map(string)
  default     = {}

  validation {
    condition     = alltrue([for k, v in var.tags : can(regex("^[a-zA-Z0-9-_]+$", k))])
    error_message = "Tag keys must contain only alphanumeric characters, hyphens, and underscores."
  }
}

variable "enable_monitoring" {
  description = "Enable CloudWatch monitoring"
  type        = bool
  default     = true
  nullable    = false
}

# Good: Complex object types with defaults
variable "database_config" {
  description = "Database configuration settings"
  type = object({
    engine                = string
    engine_version        = string
    instance_class        = string
    allocated_storage     = number
    storage_encrypted     = optional(bool, true)
    backup_retention_days = optional(number, 7)
    multi_az              = optional(bool, false)
    skip_final_snapshot   = optional(bool, false)
  })

  validation {
    condition     = contains(["postgres", "mysql", "mariadb"], var.database_config.engine)
    error_message = "Database engine must be postgres, mysql, or mariadb."
  }

  validation {
    condition     = var.database_config.allocated_storage >= 20 && var.database_config.allocated_storage <= 65536
    error_message = "Allocated storage must be between 20 and 65536 GB."
  }
}

# Good: List of objects with validation
variable "ingress_rules" {
  description = "Security group ingress rules"
  type = list(object({
    description = string
    from_port   = number
    to_port     = number
    protocol    = string
    cidr_blocks = list(string)
  }))
  default = []

  validation {
    condition = alltrue([
      for rule in var.ingress_rules :
      rule.from_port >= 0 && rule.from_port <= 65535 &&
      rule.to_port >= 0 && rule.to_port <= 65535 &&
      rule.from_port <= rule.to_port
    ])
    error_message = "Port numbers must be between 0-65535 and from_port must be <= to_port."
  }

  validation {
    condition = alltrue([
      for rule in var.ingress_rules :
      contains(["tcp", "udp", "icmp", "-1"], rule.protocol)
    ])
    error_message = "Protocol must be tcp, udp, icmp, or -1 (all)."
  }
}

# Good: Sensitive variables
variable "database_password" {
  description = "Master password for database"
  type        = string
  sensitive   = true

  validation {
    condition     = length(var.database_password) >= 16
    error_message = "Database password must be at least 16 characters long."
  }

  validation {
    condition     = can(regex("[A-Z]", var.database_password)) && can(regex("[a-z]", var.database_password)) && can(regex("[0-9]", var.database_password))
    error_message = "Database password must contain uppercase, lowercase, and numeric characters."
  }
}
```

```hcl
# Bad: No description or validation
variable "env" {
  type = string
}

variable "size" {
  default = "small"
}
```

### 3. Outputs

Define comprehensive, useful outputs:

```hcl
# Good: Clear outputs with descriptions
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "private_subnet_ids" {
  description = "List of IDs of private subnets"
  value       = aws_subnet.private[*].id
}

output "security_group_id" {
  description = "ID of the security group for compute resources"
  value       = aws_security_group.compute.id
}

output "endpoint_url" {
  description = "URL endpoint for the application load balancer"
  value       = "https://${aws_lb.main.dns_name}"
  sensitive   = false
}

output "database_password" {
  description = "Database master password"
  value       = aws_db_instance.main.password
  sensitive   = true
}
```

### 4. Local Values

Use locals for computed values and to avoid repetition:

```hcl
# Good: Use locals for common tags and computed values
locals {
  common_tags = merge(
    var.tags,
    {
      Environment = var.environment
      ManagedBy   = "Terraform"
      Project     = var.project_name
      CostCenter  = var.cost_center
    }
  )

  name_prefix = "${var.project_name}-${var.environment}"

  vpc_cidr_parts = split(".", var.vpc_cidr)
  subnet_count   = length(var.availability_zones)

  # Compute subnet CIDRs
  private_subnet_cidrs = [
    for i in range(local.subnet_count) :
    cidrsubnet(var.vpc_cidr, 4, i)
  ]

  public_subnet_cidrs = [
    for i in range(local.subnet_count) :
    cidrsubnet(var.vpc_cidr, 4, i + local.subnet_count)
  ]
}

resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = merge(
    local.common_tags,
    {
      Name = "${local.name_prefix}-vpc"
    }
  )
}
```

### 5. Resource Naming and Tagging

Consistent naming and comprehensive tagging:

```hcl
# Good: Consistent naming convention
resource "aws_s3_bucket" "data" {
  bucket = "${local.name_prefix}-data-${data.aws_caller_identity.current.account_id}"

  tags = merge(
    local.common_tags,
    {
      Name        = "${local.name_prefix}-data"
      Purpose     = "Application data storage"
      Compliance  = var.compliance_level
    }
  )
}

resource "aws_s3_bucket_versioning" "data" {
  bucket = aws_s3_bucket.data.id

  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "data" {
  bucket = aws_s3_bucket.data.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}
```

```hcl
# Bad: Inconsistent naming, no tags
resource "aws_s3_bucket" "bucket1" {
  bucket = "my-bucket"
}
```

### 6. Import Blocks and State Management

**Use import blocks for declarative resource imports (Terraform 1.5+)**:

```hcl
# Good: Declarative import block
import {
  to = aws_instance.example
  id = "i-1234567890abcdef0"
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  tags = {
    Name = "Imported Instance"
  }
}

# Good: Import with for_each
import {
  for_each = toset(["i-abc123", "i-def456", "i-ghi789"])
  to       = aws_instance.servers[each.key]
  id       = each.key
}

resource "aws_instance" "servers" {
  for_each = toset(["i-abc123", "i-def456", "i-ghi789"])

  # Configuration...
}
```

### 7. Remote State Management

**Always use remote state with locking and encryption**:

```hcl
# Good: Remote state with S3 and DynamoDB locking
terraform {
  backend "s3" {
    bucket         = "mycompany-terraform-state"
    key            = "environments/production/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"

    # Enable state encryption with KMS
    kms_key_id = "arn:aws:kms:us-east-1:ACCOUNT:key/KEY-ID"

    # Additional security
    workspace_key_prefix = "workspaces"
  }
}

# Good: Terraform Cloud backend (recommended for teams)
terraform {
  cloud {
    organization = "mycompany"

    workspaces {
      name = "production-infrastructure"
    }
  }
}

# Good: Workspaces for environment management
# Use workspace names to differentiate environments
locals {
  environment = terraform.workspace

  # Workspace-specific configurations
  config = {
    dev = {
      instance_type = "t3.micro"
      instance_count = 1
    }
    staging = {
      instance_type = "t3.small"
      instance_count = 2
    }
    production = {
      instance_type = "t3.large"
      instance_count = 5
    }
  }

  current_config = local.config[local.environment]
}
```

```hcl
# Good: Reference remote state from other configurations
data "terraform_remote_state" "networking" {
  backend = "s3"

  config = {
    bucket = "mycompany-terraform-state"
    key    = "environments/production/networking/terraform.tfstate"
    region = "us-east-1"
  }
}

resource "aws_instance" "app" {
  subnet_id = data.terraform_remote_state.networking.outputs.private_subnet_ids[0]
  # ...
}
```

### 7. Dynamic Blocks and For Expressions

Use dynamic blocks and for expressions for flexibility:

```hcl
# Good: Dynamic block for variable ingress rules
variable "ingress_rules" {
  description = "List of ingress rules"
  type = list(object({
    from_port   = number
    to_port     = number
    protocol    = string
    cidr_blocks = list(string)
    description = string
  }))
}

resource "aws_security_group" "main" {
  name        = "${local.name_prefix}-sg"
  description = "Security group for ${var.environment}"
  vpc_id      = aws_vpc.main.id

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
      description = ingress.value.description
    }
  }

  tags = local.common_tags
}

# Good: For expression for creating multiple resources
resource "aws_subnet" "private" {
  for_each = toset(var.availability_zones)

  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 4, index(var.availability_zones, each.value))
  availability_zone = each.value

  tags = merge(
    local.common_tags,
    {
      Name = "${local.name_prefix}-private-${each.value}"
      Type = "private"
    }
  )
}
```

### 8. Data Sources

Use data sources to reference existing resources:

```hcl
# Good: Use data sources for existing resources
data "aws_ami" "amazon_linux_2" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

data "aws_caller_identity" "current" {}

data "aws_region" "current" {}

resource "aws_instance" "app" {
  ami           = data.aws_ami.amazon_linux_2.id
  instance_type = var.instance_type

  tags = merge(
    local.common_tags,
    {
      Name = "${local.name_prefix}-app-${data.aws_region.current.name}"
    }
  )
}
```

### 9. Conditional Resources

Use count and for_each for conditional resource creation:

```hcl
# Good: Conditional resource creation with count
resource "aws_cloudwatch_log_group" "app" {
  count = var.enable_logging ? 1 : 0

  name              = "/aws/app/${local.name_prefix}"
  retention_in_days = var.log_retention_days

  tags = local.common_tags
}

# Good: Conditional with for_each
resource "aws_db_instance" "replica" {
  for_each = var.enable_read_replicas ? toset(var.replica_regions) : []

  identifier     = "${local.name_prefix}-db-replica-${each.value}"
  replicate_source_db = aws_db_instance.main.arn
  instance_class = var.db_replica_instance_class

  tags = merge(
    local.common_tags,
    {
      Name   = "${local.name_prefix}-db-replica"
      Region = each.value
    }
  )
}
```

### 10. Lifecycle Management

Use lifecycle blocks to prevent resource disruption:

```hcl
# Good: Lifecycle rules for critical resources
resource "aws_db_instance" "main" {
  identifier     = "${local.name_prefix}-db"
  engine         = "postgres"
  engine_version = var.db_engine_version
  instance_class = var.db_instance_class

  lifecycle {
    prevent_destroy = true

    ignore_changes = [
      password,  # Managed externally
      snapshot_identifier
    ]

    create_before_destroy = true
  }

  tags = local.common_tags
}

resource "aws_launch_template" "app" {
  name_prefix = "${local.name_prefix}-"

  # Template configuration...

  lifecycle {
    create_before_destroy = true
  }
}
```

## Terraform Workflow

Follow a consistent workflow:

```bash
# 1. Initialize
terraform init

# 2. Format code
terraform fmt -recursive

# 3. Validate syntax
terraform validate

# 4. Plan changes
terraform plan -out=tfplan

# 5. Review plan
terraform show tfplan

# 6. Apply changes
terraform apply tfplan

# 7. Clean up plan file
rm tfplan
```

## Security Best Practices

### Secrets Management

```hcl
# Good: Generate secure passwords
resource "random_password" "db_password" {
  length  = 32
  special = true

  lifecycle {
    ignore_changes = [
      length,
      special,
    ]
  }
}

# Good: Store secrets securely
resource "aws_secretsmanager_secret" "db_password" {
  name        = "${local.name_prefix}-db-password"
  description = "Database master password for ${var.environment}"

  recovery_window_in_days = 30

  tags = local.common_tags
}

resource "aws_secretsmanager_secret_version" "db_password" {
  secret_id = aws_secretsmanager_secret.db_password.id
  secret_string = jsonencode({
    username = "admin"
    password = random_password.db_password.result
    engine   = "postgres"
    host     = aws_db_instance.main.address
    port     = aws_db_instance.main.port
    dbname   = aws_db_instance.main.db_name
  })
}

# Good: Reference secrets, don't hardcode
data "aws_secretsmanager_secret_version" "api_key" {
  secret_id = "prod/api-key"
}

locals {
  api_credentials = jsondecode(data.aws_secretsmanager_secret_version.api_key.secret_string)
}
```

### Encryption

```hcl
# Good: Create KMS keys with proper policies
resource "aws_kms_key" "ebs" {
  description             = "KMS key for EBS volume encryption"
  deletion_window_in_days = 30
  enable_key_rotation     = true

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid    = "Enable IAM User Permissions"
        Effect = "Allow"
        Principal = {
          AWS = "arn:aws:iam::${data.aws_caller_identity.current.account_id}:root"
        }
        Action   = "kms:*"
        Resource = "*"
      },
      {
        Sid    = "Allow EC2 to use the key"
        Effect = "Allow"
        Principal = {
          Service = "ec2.amazonaws.com"
        }
        Action = [
          "kms:Decrypt",
          "kms:GenerateDataKey"
        ]
        Resource = "*"
      }
    ]
  })

  tags = local.common_tags
}

resource "aws_kms_alias" "ebs" {
  name          = "alias/${local.name_prefix}-ebs"
  target_key_id = aws_kms_key.ebs.key_id
}

# Good: Encrypt at rest
resource "aws_ebs_volume" "data" {
  availability_zone = var.availability_zone
  size              = var.volume_size
  encrypted         = true
  kms_key_id        = aws_kms_key.ebs.arn

  tags = local.common_tags
}

# Good: S3 encryption
resource "aws_s3_bucket" "data" {
  bucket = "${local.name_prefix}-data-${data.aws_caller_identity.current.account_id}"

  tags = local.common_tags
}

resource "aws_s3_bucket_server_side_encryption_configuration" "data" {
  bucket = aws_s3_bucket.data.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm     = "aws:kms"
      kms_master_key_id = aws_kms_key.s3.arn
    }
    bucket_key_enabled = true
  }
}

resource "aws_s3_bucket_versioning" "data" {
  bucket = aws_s3_bucket.data.id

  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_public_access_block" "data" {
  bucket = aws_s3_bucket.data.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

### IAM and RBAC

```hcl
# Good: Use IAM roles with OIDC for GitHub Actions
resource "aws_iam_openid_connect_provider" "github" {
  url = "https://token.actions.githubusercontent.com"

  client_id_list = ["sts.amazonaws.com"]

  thumbprint_list = [
    "6938fd4d98bab03faadb97b34396831e3780aea1"
  ]

  tags = local.common_tags
}

resource "aws_iam_role" "github_actions" {
  name               = "${local.name_prefix}-github-actions"
  assume_role_policy = data.aws_iam_policy_document.github_actions_assume.json

  tags = local.common_tags
}

data "aws_iam_policy_document" "github_actions_assume" {
  statement {
    actions = ["sts:AssumeRoleWithWebIdentity"]

    principals {
      type        = "Federated"
      identifiers = [aws_iam_openid_connect_provider.github.arn]
    }

    condition {
      test     = "StringEquals"
      variable = "token.actions.githubusercontent.com:aud"
      values   = ["sts.amazonaws.com"]
    }

    condition {
      test     = "StringLike"
      variable = "token.actions.githubusercontent.com:sub"
      values   = ["repo:myorg/myrepo:*"]
    }
  }
}

# Good: IAM role for EKS with IRSA (IAM Roles for Service Accounts)
resource "aws_iam_role" "eks_service_account" {
  name               = "${local.name_prefix}-eks-sa"
  assume_role_policy = data.aws_iam_policy_document.eks_assume.json

  tags = local.common_tags
}

data "aws_iam_policy_document" "eks_assume" {
  statement {
    actions = ["sts:AssumeRoleWithWebIdentity"]

    principals {
      type        = "Federated"
      identifiers = [aws_iam_openid_connect_provider.eks.arn]
    }

    condition {
      test     = "StringEquals"
      variable = "${replace(aws_iam_openid_connect_provider.eks.url, "https://", "")}:sub"
      values   = ["system:serviceaccount:${var.namespace}:${var.service_account_name}"]
    }

    condition {
      test     = "StringEquals"
      variable = "${replace(aws_iam_openid_connect_provider.eks.url, "https://", "")}:aud"
      values   = ["sts.amazonaws.com"]
    }
  }
}

# Good: Least-privilege IAM policies
data "aws_iam_policy_document" "app_permissions" {
  # S3 access - scoped to specific bucket
  statement {
    sid = "S3Access"
    actions = [
      "s3:GetObject",
      "s3:PutObject",
      "s3:DeleteObject",
    ]
    resources = ["${aws_s3_bucket.data.arn}/*"]
  }

  statement {
    sid = "S3List"
    actions = [
      "s3:ListBucket",
    ]
    resources = [aws_s3_bucket.data.arn]
  }

  # Secrets Manager access - specific secret only
  statement {
    sid = "SecretsAccess"
    actions = [
      "secretsmanager:GetSecretValue",
    ]
    resources = [aws_secretsmanager_secret.db_password.arn]
  }

  # KMS access - decrypt only
  statement {
    sid = "KMSDecrypt"
    actions = [
      "kms:Decrypt",
      "kms:DescribeKey",
    ]
    resources = [aws_kms_key.ebs.arn]
  }
}

resource "aws_iam_policy" "app_permissions" {
  name        = "${local.name_prefix}-app-permissions"
  description = "Application permissions for ${var.environment}"
  policy      = data.aws_iam_policy_document.app_permissions.json

  tags = local.common_tags
}
```

### Network Security

```hcl
# Good: Security groups with egress restrictions
resource "aws_security_group" "app" {
  name        = "${local.name_prefix}-app-sg"
  description = "Security group for application servers"
  vpc_id      = aws_vpc.main.id

  tags = merge(
    local.common_tags,
    {
      Name = "${local.name_prefix}-app-sg"
    }
  )
}

# Define rules separately for better management
resource "aws_vpc_security_group_ingress_rule" "app_https" {
  security_group_id = aws_security_group.app.id
  description       = "Allow HTTPS from load balancer"

  from_port                    = 443
  to_port                      = 443
  ip_protocol                  = "tcp"
  referenced_security_group_id = aws_security_group.alb.id

  tags = {
    Name = "https-from-alb"
  }
}

resource "aws_vpc_security_group_egress_rule" "app_https" {
  security_group_id = aws_security_group.app.id
  description       = "Allow HTTPS to internet for API calls"

  from_port   = 443
  to_port     = 443
  ip_protocol = "tcp"
  cidr_ipv4   = "0.0.0.0/0"

  tags = {
    Name = "https-outbound"
  }
}

# Good: Enable VPC Flow Logs
resource "aws_flow_log" "main" {
  vpc_id          = aws_vpc.main.id
  traffic_type    = "ALL"
  iam_role_arn    = aws_iam_role.flow_logs.arn
  log_destination = aws_cloudwatch_log_group.flow_logs.arn

  tags = local.common_tags
}

resource "aws_cloudwatch_log_group" "flow_logs" {
  name              = "/aws/vpc/${local.name_prefix}"
  retention_in_days = var.log_retention_days
  kms_key_id        = aws_kms_key.logs.arn

  tags = local.common_tags
}
```

## Testing

### Native Terraform Tests (Terraform 1.6+)

```hcl
# tests/networking_test.tftest.hcl
# Good: Use native Terraform testing framework

variables {
  vpc_cidr    = "10.0.0.0/16"
  environment = "test"
}

run "validate_vpc_cidr" {
  command = plan

  assert {
    condition     = aws_vpc.main.cidr_block == "10.0.0.0/16"
    error_message = "VPC CIDR block did not match expected value"
  }
}

run "validate_subnet_count" {
  command = plan

  assert {
    condition     = length(aws_subnet.private) == 3
    error_message = "Expected 3 private subnets"
  }
}

run "validate_tags" {
  command = plan

  assert {
    condition     = aws_vpc.main.tags["Environment"] == "test"
    error_message = "VPC missing required Environment tag"
  }

  assert {
    condition     = aws_vpc.main.tags["ManagedBy"] == "Terraform"
    error_message = "VPC missing required ManagedBy tag"
  }
}

# Test apply and verify outputs
run "apply_and_verify" {
  command = apply

  assert {
    condition     = output.vpc_id != ""
    error_message = "VPC ID output is empty"
  }

  assert {
    condition     = length(output.private_subnet_ids) == 3
    error_message = "Expected 3 private subnet IDs in output"
  }
}
```

### Terratest (Go-based testing)

```go
// tests/terraform_aws_vpc_test.go
package test

import (
    "testing"
    "github.com/gruntwork-io/terratest/modules/terraform"
    "github.com/stretchr/testify/assert"
)

func TestTerraformAwsVpc(t *testing.T) {
    t.Parallel()

    terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
        // Path to Terraform code
        TerraformDir: "../examples/complete",

        // Variables to pass to terraform
        Vars: map[string]interface{}{
            "vpc_cidr":    "10.0.0.0/16",
            "environment": "test",
            "availability_zones": []string{
                "us-east-1a",
                "us-east-1b",
                "us-east-1c",
            },
        },

        // Disable colors in output
        NoColor: true,
    })

    // Clean up resources at the end
    defer terraform.Destroy(t, terraformOptions)

    // Run terraform init and apply
    terraform.InitAndApply(t, terraformOptions)

    // Validate outputs
    vpcId := terraform.Output(t, terraformOptions, "vpc_id")
    assert.NotEmpty(t, vpcId)

    privateSubnetIds := terraform.OutputList(t, terraformOptions, "private_subnet_ids")
    assert.Equal(t, 3, len(privateSubnetIds))

    // Validate that VPC exists and has correct CIDR
    vpcCidr := terraform.Output(t, terraformOptions, "vpc_cidr")
    assert.Equal(t, "10.0.0.0/16", vpcCidr)
}
```

### Policy as Code (OPA/Sentinel)

```rego
# policies/terraform.rego
# Good: Use Open Policy Agent for policy enforcement

package terraform

import input as tfplan

deny[msg] {
    resource := tfplan.resource_changes[_]
    resource.type == "aws_s3_bucket"
    not has_encryption(resource)
    msg := sprintf("S3 bucket '%s' must have encryption enabled", [resource.address])
}

deny[msg] {
    resource := tfplan.resource_changes[_]
    resource.type == "aws_instance"
    not resource.change.after.monitoring
    msg := sprintf("EC2 instance '%s' must have detailed monitoring enabled", [resource.address])
}

deny[msg] {
    resource := tfplan.resource_changes[_]
    not has_required_tags(resource)
    msg := sprintf("Resource '%s' missing required tags", [resource.address])
}

has_encryption(resource) {
    resource.type == "aws_s3_bucket_server_side_encryption_configuration"
}

has_required_tags(resource) {
    required_tags := ["Environment", "ManagedBy", "Project"]
    resource_tags := object.keys(resource.change.after.tags)
    count([tag | tag := required_tags[_]; tag in resource_tags]) == count(required_tags)
}
```

### Compliance Testing (terraform-compliance)

```gherkin
# policies/security.feature
Feature: Security compliance requirements

  Scenario: Ensure all S3 buckets have encryption
    Given I have aws_s3_bucket defined
    When it has aws_s3_bucket_server_side_encryption_configuration
    Then it must have rule
    And it must have apply_server_side_encryption_by_default

  Scenario: Ensure all S3 buckets block public access
    Given I have aws_s3_bucket defined
    When it has aws_s3_bucket_public_access_block
    Then it must contain block_public_acls
    And its value must be true
    Then it must contain block_public_policy
    And its value must be true

  Scenario: Ensure all resources are properly tagged
    Given I have resource that supports tags defined
    Then it must contain tags
    And its tags must contain Environment
    And its tags must contain ManagedBy
    And its tags must contain Project

  Scenario: Ensure KMS keys have rotation enabled
    Given I have aws_kms_key defined
    Then it must contain enable_key_rotation
    And its value must be true

  Scenario: Ensure EBS volumes are encrypted
    Given I have aws_ebs_volume defined
    Then it must contain encrypted
    And its value must be true
```

### Static Analysis (TFLint)

```hcl
# .tflint.hcl
# Good: Comprehensive TFLint configuration

plugin "terraform" {
  enabled = true
  preset  = "recommended"
}

plugin "aws" {
  enabled = true
  version = "0.29.0"
  source  = "github.com/terraform-linters/tflint-ruleset-aws"
}

rule "terraform_required_version" {
  enabled = true
}

rule "terraform_required_providers" {
  enabled = true
}

rule "terraform_naming_convention" {
  enabled = true
  format  = "snake_case"
}

rule "terraform_documented_variables" {
  enabled = true
}

rule "terraform_documented_outputs" {
  enabled = true
}

rule "terraform_module_pinned_source" {
  enabled = true
}

rule "aws_instance_invalid_type" {
  enabled = true
}

rule "aws_s3_bucket_name" {
  enabled = true
  regex   = "^[a-z0-9][a-z0-9-]*[a-z0-9]$"
}
```

## CI/CD Integration

### GitHub Actions Workflow

```yaml
# .github/workflows/terraform.yml
name: Terraform CI/CD

on:
  pull_request:
    branches: [main]
    paths:
      - '**.tf'
      - '**.tfvars'
  push:
    branches: [main]
  workflow_dispatch:

env:
  TF_VERSION: "1.7.0"
  AWS_REGION: "us-east-1"

permissions:
  id-token: write  # Required for OIDC
  contents: read
  pull-requests: write

jobs:
  terraform-validate:
    name: Validate and Lint
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Terraform Format
        run: terraform fmt -check -recursive

      - name: Setup TFLint
        uses: terraform-linters/setup-tflint@v4

      - name: Run TFLint
        run: |
          tflint --init
          tflint --recursive

      - name: Terraform Init
        run: terraform init -backend=false

      - name: Terraform Validate
        run: terraform validate

  terraform-security:
    name: Security Scan
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Run tfsec
        uses: aquasecurity/tfsec-action@v1.0.3
        with:
          soft_fail: false

      - name: Run Checkov
        uses: bridgecrewio/checkov-action@v12
        with:
          directory: .
          framework: terraform
          soft_fail: false

  terraform-plan:
    name: Plan
    runs-on: ubuntu-latest
    needs: [terraform-validate, terraform-security]
    environment: ${{ github.event_name == 'push' && 'production' || 'development' }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Configure AWS Credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Terraform Init
        run: terraform init

      - name: Terraform Plan
        id: plan
        run: |
          terraform plan -out=tfplan -no-color
          terraform show -no-color tfplan > plan.txt

      - name: Save Plan
        uses: actions/upload-artifact@v4
        with:
          name: tfplan
          path: tfplan

      - name: Comment PR
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            const fs = require('fs');
            const plan = fs.readFileSync('plan.txt', 'utf8');
            const output = `#### Terraform Plan
            \`\`\`
            ${plan}
            \`\`\`
            `;
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: output
            });

  terraform-apply:
    name: Apply
    runs-on: ubuntu-latest
    needs: [terraform-plan]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    environment: production
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Configure AWS Credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Terraform Init
        run: terraform init

      - name: Download Plan
        uses: actions/download-artifact@v4
        with:
          name: tfplan

      - name: Terraform Apply
        run: terraform apply -auto-approve tfplan
```

### GitLab CI/CD

```yaml
# .gitlab-ci.yml
variables:
  TF_VERSION: "1.7.0"
  TF_ROOT: ${CI_PROJECT_DIR}
  TF_STATE_NAME: ${CI_COMMIT_REF_SLUG}

image:
  name: hashicorp/terraform:$TF_VERSION
  entrypoint: [""]

stages:
  - validate
  - plan
  - apply

cache:
  paths:
    - ${TF_ROOT}/.terraform

before_script:
  - cd ${TF_ROOT}
  - terraform init

validate:
  stage: validate
  script:
    - terraform fmt -check -recursive
    - terraform validate
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

security:
  stage: validate
  image: aquasec/tfsec:latest
  script:
    - tfsec . --soft-fail
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

plan:
  stage: plan
  script:
    - terraform plan -out=tfplan
    - terraform show -json tfplan > tfplan.json
  artifacts:
    paths:
      - ${TF_ROOT}/tfplan
      - ${TF_ROOT}/tfplan.json
    expire_in: 1 week
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

apply:
  stage: apply
  script:
    - terraform apply -auto-approve tfplan
  dependencies:
    - plan
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
      when: manual
  only:
    - main
```

## Cost Optimization

### Resource Tagging for Cost Tracking

```hcl
# Good: Comprehensive tagging for cost allocation
locals {
  cost_tags = {
    CostCenter  = var.cost_center
    Project     = var.project_name
    Environment = var.environment
    Owner       = var.owner
    ManagedBy   = "Terraform"
  }

  common_tags = merge(var.tags, local.cost_tags)
}

# Enable cost allocation tags in AWS
resource "aws_ce_cost_category" "environment" {
  name         = "Environment"
  rule_version = "CostCategoryExpression.v1"

  rule {
    value = "Production"
    rule {
      tags {
        key    = "Environment"
        values = ["production", "prod"]
      }
    }
  }

  rule {
    value = "Development"
    rule {
      tags {
        key    = "Environment"
        values = ["development", "dev", "staging"]
      }
    }
  }
}
```

### Instance Right-Sizing

```hcl
# Good: Use data sources to validate instance types
data "aws_ec2_instance_types" "valid" {
  filter {
    name   = "instance-type"
    values = ["t3.*", "t3a.*", "t4g.*"]
  }

  filter {
    name   = "current-generation"
    values = ["true"]
  }
}

# Good: Auto Scaling with scheduled actions for cost savings
resource "aws_autoscaling_schedule" "scale_down_evening" {
  count                  = var.environment == "dev" ? 1 : 0
  scheduled_action_name  = "scale-down-evening"
  min_size               = 0
  max_size               = 0
  desired_capacity       = 0
  recurrence             = "0 19 * * MON-FRI"  # 7 PM weekdays
  autoscaling_group_name = aws_autoscaling_group.app.name
}

resource "aws_autoscaling_schedule" "scale_up_morning" {
  count                  = var.environment == "dev" ? 1 : 0
  scheduled_action_name  = "scale-up-morning"
  min_size               = 1
  max_size               = 3
  desired_capacity       = 1
  recurrence             = "0 7 * * MON-FRI"  # 7 AM weekdays
  autoscaling_group_name = aws_autoscaling_group.app.name
}
```

### Spot Instances and Savings Plans

```hcl
# Good: Use Spot Instances for non-critical workloads
resource "aws_autoscaling_group" "spot" {
  name                = "${local.name_prefix}-spot-asg"
  vpc_zone_identifier = aws_subnet.private[*].id
  min_size            = 1
  max_size            = 10
  desired_capacity    = 3

  mixed_instances_policy {
    instances_distribution {
      on_demand_base_capacity                  = 1
      on_demand_percentage_above_base_capacity = 20
      spot_allocation_strategy                 = "price-capacity-optimized"
      spot_instance_pools                      = 4
    }

    launch_template {
      launch_template_specification {
        launch_template_id = aws_launch_template.app.id
        version            = "$Latest"
      }

      # Multiple instance types for better spot availability
      override {
        instance_type = "t3.medium"
      }
      override {
        instance_type = "t3a.medium"
      }
      override {
        instance_type = "t4g.medium"
      }
    }
  }

  tag {
    key                 = "Name"
    value               = "${local.name_prefix}-spot-instance"
    propagate_at_launch = true
  }

  dynamic "tag" {
    for_each = local.common_tags
    content {
      key                 = tag.key
      value               = tag.value
      propagate_at_launch = true
    }
  }
}
```

### Storage Optimization

```hcl
# Good: S3 lifecycle policies for cost optimization
resource "aws_s3_bucket_lifecycle_configuration" "data" {
  bucket = aws_s3_bucket.data.id

  rule {
    id     = "archive-old-data"
    status = "Enabled"

    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }

    transition {
      days          = 90
      storage_class = "GLACIER_IR"
    }

    transition {
      days          = 180
      storage_class = "DEEP_ARCHIVE"
    }

    expiration {
      days = 365
    }

    noncurrent_version_transition {
      noncurrent_days = 30
      storage_class   = "GLACIER"
    }

    noncurrent_version_expiration {
      noncurrent_days = 90
    }
  }

  rule {
    id     = "abort-incomplete-multipart"
    status = "Enabled"

    abort_incomplete_multipart_upload {
      days_after_initiation = 7
    }
  }
}

# Good: EBS volume optimization with GP3
resource "aws_ebs_volume" "optimized" {
  availability_zone = var.availability_zone
  size              = var.volume_size
  type              = "gp3"  # GP3 is more cost-effective than GP2
  iops              = 3000   # Baseline included
  throughput        = 125    # MB/s baseline included
  encrypted         = true
  kms_key_id        = aws_kms_key.ebs.arn

  tags = local.common_tags
}
```

### Budget Alerts

```hcl
# Good: Set up budget alerts
resource "aws_budgets_budget" "monthly" {
  name              = "${local.name_prefix}-monthly-budget"
  budget_type       = "COST"
  limit_amount      = var.monthly_budget
  limit_unit        = "USD"
  time_period_start = "2024-01-01_00:00"
  time_unit         = "MONTHLY"

  cost_filter {
    name = "TagKeyValue"
    values = [
      "Project$${var.project_name}",
    ]
  }

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 80
    threshold_type             = "PERCENTAGE"
    notification_type          = "ACTUAL"
    subscriber_email_addresses = var.budget_alert_emails
  }

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 100
    threshold_type             = "PERCENTAGE"
    notification_type          = "ACTUAL"
    subscriber_email_addresses = var.budget_alert_emails
  }

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 90
    threshold_type             = "PERCENTAGE"
    notification_type          = "FORECASTED"
    subscriber_email_addresses = var.budget_alert_emails
  }
}
```

## Key Points

- **Explicit version constraints** - Pin Terraform and provider versions
- **Provider default tags** - Simplify tagging strategy
- **Module-based organization** - Reusable, maintainable infrastructure
- **Remote state with locking** - Prevent state corruption with S3/DynamoDB or Terraform Cloud
- **Import blocks** - Declaratively import existing resources (Terraform 1.5+)
- **Moved blocks** - Handle resource refactoring without destruction
- **Variable validation** - Multiple validations with comprehensive rules
- **Complex type constraints** - Use `optional()` for flexible object types
- **Comprehensive outputs** - Enable module composition and integration
- **Local values** - Avoid repetition, compute values once
- **Dynamic blocks** - Flexible resource configuration
- **Consistent naming** - Use name prefixes with environment and account ID
- **Comprehensive tagging** - Track resources, costs, ownership
- **Data sources** - Reference existing resources
- **Lifecycle rules** - Prevent accidental deletion and manage replacements
- **Conditional resources** - Use count and for_each for flexibility
- **Security by default** - OIDC authentication, IRSA, encryption, least privilege
- **Secrets management** - Use Secrets Manager/Parameter Store, never hardcode
- **IAM best practices** - Least privilege, OIDC for CI/CD, IRSA for Kubernetes
- **Network security** - Separate security group rules, VPC Flow Logs, egress restrictions
- **Testing** - Native Terraform tests, Terratest, Policy as Code (OPA), terraform-compliance
- **Static analysis** - TFLint, tfsec, Checkov in CI/CD pipeline
- **CI/CD integration** - Automated validation, security scanning, plan/apply workflows
- **Cost optimization** - Spot instances, lifecycle policies, budget alerts, right-sizing
- **Version pinning** - Pin Terraform, provider, and module versions
- **Documentation** - README for each module with examples

## When to Apply

Apply these practices to:
- **All cloud infrastructure provisioning** - AWS, Azure, GCP, multi-cloud
- **Multi-environment setups** - Dev, staging, production with proper isolation
- **Module development** - Reusable, versioned infrastructure components
- **Infrastructure refactoring** - Use moved blocks to avoid resource recreation
- **Compliance requirements** - Security policies, audit logging, encryption standards
- **Team collaboration** - Remote state, code reviews, automated testing
- **Cost-sensitive projects** - Budget alerts, spot instances, lifecycle policies
- **Enterprise deployments** - RBAC, OIDC authentication, comprehensive auditing
- **Kubernetes infrastructure** - EKS/AKS/GKE with IRSA/Workload Identity
- **CI/CD pipelines** - GitHub Actions, GitLab CI, automated deployment workflows
