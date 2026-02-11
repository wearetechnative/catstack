# Project Context

## Purpose
CatStack is a loosely coupled Terraform/OpenTofu framework that makes complex infrastructure understandable and scalable. It solves crucial problems Cloud Engineers face during migrations and helps set up new projects faster.

**Main principles:**
- "Don't repeat yourself" (DRY)
- Domain-separated organization of infrastructure
- No additional software dependencies beyond Terraform/OpenTofu

## Tech Stack
- **IaC**: Terraform / OpenTofu (100% compatible)
- **Primary Cloud Provider**: AWS
- **Language**: HCL (HashiCorp Configuration Language)
- **State Backend**: S3 with DynamoDB for state locking
- **Supporting Tools**:
  - [RACE](https://github.com/wearetechnative/race) - Terraform Tools for CatStack
  - [Bootstrap-AWS-CatStack](https://github.com/wearetechnative/bootstrap-aws-catstack) - Bootstrap script

## Project Conventions

### Code Style
- Use `terraform fmt` for code formatting
- Use snake_case for resource names, variable names, and local values
- Prefix domain directories with numbers (e.g., `001-kms`, `002-vpc`) to indicate deployment order
- Lower numbers = lower-level domains (more dependants), higher numbers = higher-level domains

### Architecture Patterns
- **Domains**: Autonomous Terraform components with separate state files
- **Stacks**: Container directories holding all domains (like a kitchen drawer)
- **Modules**: Reusable groups of resources within domains
- **Shared Libraries**: Project-specific code reused across multiple domains
- **Multi-environment**: Separate production and non-production via different AWS accounts
- Domain modules for repeated resources within a domain (like functions in programming)

### Testing Strategy
- Manual testing via terraform plan reviews and apply to test environments
- Visual verification of plan output before applying changes
- Environment-based testing (non-production before production)

### Git Workflow
- Feature branches with pull request-based merging
- Descriptive commit messages explaining the infrastructure change
- Review terraform plan output as part of PR review process

## Domain Context
**Key terminology:**
- **infra_environments**: AWS accounts where resources are created and hosted
- **xxx.tfvars.json**: Terraform variables shared between domains and stacks
- **xxx.tfbackend**: Backend configuration for specific infrastructure environments
- **Domain**: Directory containing IaC code with numbered prefix indicating deployment order
- **Stack**: Directory containing all domains
- **Module**: Reusable group of resources providing value to a domain or stack

**Domain dependencies:**
- Low-level domains (e.g., KMS with prefix 001) have many dependants
- High-level domains (e.g., EC2 instances) depend on lower-level domains
- Use Terraform data blocks to read state from other domains

## Important Constraints
- Follow AWS Well-Architected Framework principles
- Multi-account strategy: separate AWS accounts per environment
- Domains must have their own isolated Terraform state
- Engineers should not work on the same domain simultaneously
- Understand Terraform and CatStack structure before using convenience tooling

## External Dependencies
- **State Storage**: AWS S3 bucket for Terraform state
- **State Locking**: AWS DynamoDB for state locking
- **Cloud Provider**: AWS (primary), but supports any Terraform provider
- **Module Registry**: Terraform Registry for community modules
- **Git**: For remote module references
