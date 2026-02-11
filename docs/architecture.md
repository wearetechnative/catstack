# CatStack Architecture

This document provides a technical reference for CatStack's directory structure and conventions, designed for both human engineers and AI assistants.

## Overview

CatStack is a loosely coupled Terraform/OpenTofu framework that organizes infrastructure as code using **domains** (autonomous components with separate state) within **stacks** (container directories). This separation enables teams to work independently on different infrastructure components while maintaining clear dependency relationships.

## Directory Structure

```
project-root/
├── infra_environments/          # Environment-specific configuration
│   ├── nonprod/
│   │   ├── nonprod.tfvars.json  # Shared variables for non-production
│   │   └── nonprod.tfbackend    # Backend config for non-production
│   └── prod/
│       ├── prod.tfvars.json     # Shared variables for production
│       └── prod.tfbackend       # Backend config for production
├── stack/                       # Container for all domains
│   ├── 01_shared_kms/           # Low-level domain (foundational)
│   ├── 02_backend_config/       # Depends on 01_shared_kms
│   └── 03_sqs_dlq/              # Higher-level domain
├── modules/                     # (Optional) Project-specific shared modules
└── .gitignore
```

### Key Directories

| Directory | Purpose |
|-----------|---------|
| `infra_environments/` | Contains environment-specific configuration files (`.tfvars.json`, `.tfbackend`) |
| `stack/` | Contains all domain directories, each with its own Terraform state |
| `modules/` | (Optional) Shared modules used across multiple domains within the project |

## Domain File Patterns

Each domain within `stack/` follows a standard file structure:

```
stack/01_shared_kms/
├── main.tf           # Resource definitions and module calls
├── providers.tf      # AWS provider configuration
├── tf_backend.tf     # Partial S3 backend configuration
├── tf_settings.tf    # Terraform version and provider requirements
├── variables.tf      # Input variable definitions
└── output.tf         # Exported values for cross-domain consumption
```

### File Purposes

#### `main.tf`
Contains the primary resource definitions and module calls. This is where the domain's infrastructure is defined.

```hcl
module "kms" {
  source = "git@github.com:wearetechnative/terraform-aws-kms.git?ref=v1.0.0"
  
  name = "shared-key"
  # ... configuration
}
```

#### `providers.tf`
Configures the AWS provider with role assumption and account restrictions.

```hcl
provider "aws" {
  region = "eu-west-1"

  assume_role {
    role_arn = "arn:aws:iam::${var.aws_account_id}:role/TerraformAdmin"
  }

  allowed_account_ids = [var.aws_account_id]

  default_tags {
    tags = {
      Project = "project-name"
      Stack   = "01_shared_kms"
    }
  }
}
```

#### `tf_backend.tf`
Partial S3 backend configuration with the domain-specific state key. The remaining backend settings (bucket, role_arn, dynamodb_table) come from the `.tfbackend` file.

```hcl
terraform {
  backend "s3" {
    key = "01_shared_kms/terraform.tf"
  }
}
```

#### `tf_settings.tf`
Specifies Terraform version constraints and required providers.

```hcl
terraform {
  required_version = "~> 1.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

#### `variables.tf`
Defines input variables. Every domain includes common variables (`aws_account_id`, `infra_environment`) plus domain-specific ones.

```hcl
# Common variables (present in all domains)
variable "aws_account_id" {
  type        = string
  description = "AWS account ID for this environment"
}

variable "infra_environment" {
  type        = string
  description = "Infrastructure environment name (e.g., nonprod, prod)"
}

# Domain-specific variables
variable "kms_arn" {
  type        = string
  description = "ARN of the shared KMS key"
}
```

#### `output.tf`
Exports values that other domains may need. These can be consumed via `terraform_remote_state` data sources or shared through `.tfvars.json`.

```hcl
output "kms_key_arn" {
  value       = module.kms.key_arn
  description = "ARN of the shared KMS key"
}
```

## Environment Configuration

### `.tfvars.json` Files

JSON files containing variables shared across all domains in an environment. These are passed to Terraform with `-var-file`.

```json
{
  "aws_account_id": "123456789012",
  "infra_environment": "nonprod",
  "kms_arn": "arn:aws:kms:eu-west-1:123456789012:key/abc-123"
}
```

**Usage pattern:**
- `aws_account_id` and `infra_environment` are always present
- Cross-domain values (like `kms_arn`) are added after the source domain is deployed
- Update the file when domain outputs need to be shared

### `.tfbackend` Files

Partial backend configuration merged with `tf_backend.tf` at init time.

```hcl
role_arn       = "arn:aws:iam::123456789012:role/TerraformAdmin"
bucket         = "project-terraform-state"
dynamodb_table = "project-terraform-locks"
region         = "eu-west-1"
encrypt        = true
```

**Usage:** `terraform init -backend-config=../../infra_environments/nonprod/nonprod.tfbackend`

## Naming Conventions

### Domain Numeric Prefixes

Domains use numeric prefixes (`01_`, `02_`, etc.) to indicate deployment order and dependency hierarchy:

| Prefix Range | Category | Description |
|--------------|----------|-------------|
| `01_` - `09_` | Foundational | Core infrastructure (KMS, networking). Many dependants. |
| `10_` - `49_` | Platform | Shared services (databases, queues). Moderate dependencies. |
| `50_` - `99_` | Application | Application-specific resources. Few or no dependants. |

**Key principles:**
- **Lower numbers = more dependants** (deployed first, destroyed last)
- **Higher numbers = more dependencies** (deployed after prerequisites)
- Domains with the same prefix can be deployed in parallel

### Resource and Variable Naming

- Use `snake_case` for all Terraform identifiers
- Prefix resources with the domain purpose when clarity is needed
- Variable names should be descriptive and self-documenting

## Workflow

### Deploying a Domain

```bash
cd stack/01_shared_kms

# Initialize with environment-specific backend
terraform init -backend-config=../../infra_environments/nonprod/nonprod.tfbackend

# Plan with shared variables
terraform plan -var-file=../../infra_environments/nonprod/nonprod.tfvars.json

# Apply changes
terraform apply -var-file=../../infra_environments/nonprod/nonprod.tfvars.json
```

### Adding a New Domain

1. Determine the appropriate numeric prefix based on dependencies
2. Create the domain directory: `stack/XX_domain_name/`
3. Add all standard files (main.tf, providers.tf, tf_backend.tf, tf_settings.tf, variables.tf, output.tf)
4. Update `tf_backend.tf` with the domain-specific key
5. Add any domain-specific variables to `variables.tf`
6. Deploy in dependency order (lower numbers first)

### Sharing Values Between Domains

**Option 1: Via `.tfvars.json` (recommended)**
1. Deploy source domain and note output values
2. Add output values to `.tfvars.json`
3. Reference via variables in dependent domains

**Option 2: Via `terraform_remote_state`**
```hcl
data "terraform_remote_state" "kms" {
  backend = "s3"
  config = {
    bucket = "project-terraform-state"
    key    = "01_shared_kms/terraform.tf"
    region = "eu-west-1"
  }
}

# Use: data.terraform_remote_state.kms.outputs.kms_key_arn
```

## Recommended `.gitignore`

```gitignore
# Terraform
.terraform/
*.tfstate
*.tfstate.*
crash.log

# Lock file (versions pinned in tf_settings.tf)
.terraform.lock.hcl

# Secrets
secrets.auto.tfvars
*.auto.tfvars

# Local tools
.tool-versions
```

## Related Resources

- [RACE](https://github.com/wearetechnative/race) - Terraform tooling for CatStack
- [Bootstrap-AWS-CatStack](https://github.com/wearetechnative/bootstrap-aws-catstack) - Project bootstrapping
