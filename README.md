# CatStack

Loosely coupled Terraform framework making complex infrastructure
understandable and scalable.

<img src="catstacklogo.png" style="width:200px" />

## What is CatStack?

CatStack is a Terraform/OpenTofu framework that originated within TechNative. The
framework solves several crucial problems that Cloud Engineers face when
performing migrations and helps them set up completely new projects much faster.
CatStack draws inspiration from frameworks in the software development world
without trying to resemble them.

The main principles of CatStack are:

- "Don't repeat yourself" (DRY)
- Domain-separated organization of infrastructure

CatStack is designed to be used without having to install additional software
except for Terraform/OpenTofu itself.

## Is CatStack magic?

No, we're not making it prettier than it is. CatStack is simply a well-considered
collection of conventions, best practices, and a directory structure that
bundles multiple Terraform projects together. All of this combined delivers a
comprehensible and scalable IaC project that increases productivity for any cloud team,
whether small or large.

## The advantages of CatStack

These are the advantages of building a terraform project with CatStack.

### Collaboration in a team environment

By dividing infrastructure into logical domains, multiple engineers can
collaborate on a larger project. CatStack has best practices, but in principle,
a team is free to determine its own domains. A domain has its own
terraform state, so as long as engineers don't work on a domain simultaneously,
they can use `terraform apply` whenever they want.

### DRY and multi-environments

CatStack helps engineers write efficient source code that is reusable.
First, CatStack implements multi-environments in a clear and
scalable way. An environment can be deployed in multiple ways but
the most obvious application is to separate production and non-production
cloud accounts from each other. Heavier resources can be configured
for the production account. The source code that defines the resources
is shared across all environments.

### DRY and data blocks

Through Terraform's data blocks, resources from one domain can read necessary
information from the state of another domain.

### DRY and shared code libraries

CatStack offers the possibility to place project-specific terraform code that
appears in multiple domains in a shared library.

### DRY and Domain modules

This is a programming pattern that CatStack encourages, but that every good
Terraform programmer naturally knows and applies. As soon as you define
multiple nearly identical resources within a domain, you create a module. You do this
within the domain. You can compare this to creating a function in a
generic programming language.

### DRY and Terraform remote modules

Within a domain, you can of course make use of Terraform's powerful
module system. Use any of the thousands of modules from the
Terraform Registry, or reference previously written modules in a git repo.
Don't limit yourself because Terraform modules are the building blocks of the
Cloud.

### Flexible

CatStack is incredibly flexible. Any application can be captured within the
CatStack framework. And this while maintaining the ability to make as much use
as possible of existing building blocks. This combination of qualities
makes CatStack ideal for teams that execute many projects. Each project
is tailor-made, and at the same time, each project contributes to the projects that
are yet to come.

### Cloud Agnostic

Just like Terraform is suitable for AWS, Azure, or any other provider...
CatStack is just as versatile. It's even possible within a CatStack
project to have multiple providers working together. As a fun experiment,
we built a several projects in MineCraft with CatStack and the Terraform MineCraft
provider.

### Optional tooling

As stated earlier, CatStack is a framework that can be used with
Terraform as its only dependency. However, because CatStack is
based on smart conventions, it offers excellent opportunities to make tasks
even easier. TechNative has created quite a few scripts and small programs for AWS
that make our engineers even more productive.

At TechNative, we work with the rule: feel free to use all those handy scripts
that allow you to work faster. But not before you understand how Terraform
works and how CatStack is structured.

Here's a list of CatStack tooling we use withing TechNative:

- [RACE](https://github.com/wearetechnative/race) - Terraform Tools for TechNative CatStack 
- [Bootstrap-AWS-CatStack](https://github.com/wearetechnative/bootstrap-aws-catstack) - Bootstrap-AWS-CatStack Script

## FAQ

> What does the name CatStack mean?

CatStack doesn't mean that much. A cat is a nice animal. You can read it as:
Cloud AWS Terraform Stack.

> Does CatStack also work with OpenTofu instead of Terraform?

CatStack is 100% compatible with OpenTofu.

## Definition of Terms

infra_environments: infra_environments short for infrastructure environments
are the AWS accounts where resources will be created and hosted.

xxx.tfvars.json: This file hosts Terraform variables that are shared between
the domains and stacks.

xxx.tfbackend: This file hosts the details of the Terraform backend
configuration for that specific infrastructure environment.

Domain: Domains are directories which contains the IaC code of the resources
deployed. Domains have numbers infront of them which indicates in which order
of the Stack they are applied. For example, Domains starting with 001 means
that the resources in those domains are the first that should be deployed in an
environment.

Stack: The Stack directory contains the directory of all domains. Think of a
Stack as a kitchen drawer and the contents in that drawer are known as domains.

Module: A module is a group of resources that together provide a certain value
for a domain or stack.

## Domains

A CatStack is a composition of _domains_. A domain is an autonomous Terraform
component with its own seperated Terraform state. You can have low level
domains and higher level domains, E.g. the KMS domain is a low level domain as
it has many dependants, while an EC2 instance domain is high level domain. Like
any other vanilla Terraform project, a CatStack domain can use terraform
modules.

A CatStack domain is not distributed as an installable module, but its
more like glue code for one or more resources to define an infrastruture
component in a larger system or solution.
