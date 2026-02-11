# catstack-structure Specification

## Purpose
TBD - created by archiving change add-architecture-docs. Update Purpose after archive.
## Requirements
### Requirement: Directory Structure Documentation
The CatStack documentation SHALL provide a clear reference of the standard directory structure for both humans and AI assistants.

#### Scenario: New user understands top-level structure
- **WHEN** a user reads the architecture documentation
- **THEN** they understand that `infra_environments/` contains environment-specific configuration
- **AND** they understand that `stack/` contains all domain directories

#### Scenario: AI assistant bootstraps new project
- **WHEN** an AI assistant is asked to create a new CatStack project
- **THEN** it can reference the documentation to scaffold the correct directory structure

### Requirement: Domain File Pattern Documentation
The documentation SHALL explain the standard files within each domain directory and their purposes.

#### Scenario: User creates a new domain
- **WHEN** a user needs to create a new domain
- **THEN** they can reference documentation for required files:
  - `main.tf` - Resource definitions and module calls
  - `providers.tf` - AWS provider configuration with assume_role
  - `tf_backend.tf` - Partial S3 backend with domain-specific key
  - `tf_settings.tf` - Terraform version and provider requirements
  - `variables.tf` - Input variables including common and domain-specific
  - `output.tf` - Exported values for cross-domain consumption

#### Scenario: AI assistant generates domain boilerplate
- **WHEN** an AI assistant is asked to create a new domain
- **THEN** it can generate all required files following documented patterns

### Requirement: Environment Configuration Documentation
The documentation SHALL explain the environment configuration file patterns and their usage.

#### Scenario: User configures new environment
- **WHEN** a user needs to add a new infrastructure environment (e.g., production)
- **THEN** they can reference documentation to create:
  - `<env>.tfvars.json` - Shared variables (aws_account_id, infra_environment, cross-domain values)
  - `<env>.tfbackend` - Backend configuration (role_arn, bucket, dynamodb_table)

#### Scenario: User understands variable sharing
- **WHEN** a user needs to share values between domains
- **THEN** documentation explains using `.tfvars.json` for environment-wide values like `kms_arn`

### Requirement: Domain Naming Convention Documentation
The documentation SHALL explain the numeric prefix convention for domain directories.

#### Scenario: User understands deployment order
- **WHEN** a user sees domain directories like `01_shared_kms`, `02_backend_config`
- **THEN** documentation explains that lower numbers indicate foundational domains with more dependants
- **AND** higher numbers indicate domains that depend on lower-level resources

#### Scenario: User adds new domain at correct position
- **WHEN** a user needs to add a domain that depends on existing domains
- **THEN** they can determine appropriate numeric prefix based on dependencies

