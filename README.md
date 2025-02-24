# CatStack

Loosely coupled Terraform framework making complex infrastructure understandable and scalable.

## Domains

A CatStack is a composition of _domains_. A domain is an autonomous Terraform component with its own seperated Terraform state. You can have low level domains and higher level domains, E.g. the KMS domain is a low level domain as it has many dependants, while an EC2 instance domain is high level domain. Like any other vanilla Terraform project, a CatStack domain can use terraform modules. A CatStack domain is not distributed as an installable module, but its more like glue code for one or more resources to define an infrastruture component in a larger system or solution. 

### Example Domains

- https://github.com/wearetechnative/catsdom-tanix
