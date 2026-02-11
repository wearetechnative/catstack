# Change: Add CatStack Architecture Documentation

## Why
Engineers and AI assistants need a clear technical reference for CatStack's directory structure and conventions to bootstrap new projects correctly. Currently, this knowledge is scattered across external repos and implicit patterns.

## What Changes
- Add comprehensive architecture documentation explaining CatStack's directory structure
- Document the standard file patterns within domains (main.tf, providers.tf, tf_backend.tf, etc.)
- Explain the relationship between infra_environments, stacks, and domains
- Provide guidance on environment configuration patterns (.tfvars.json, .tfbackend)

## Impact
- Affected specs: `catstack-structure` (new capability)
- Affected code: None - documentation only
- Addresses: [GitHub Issue #1](https://github.com/wearetechnative/catstack/issues/1)
