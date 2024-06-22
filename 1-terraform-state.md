# Terraform State

## History of Terraform State

Terraform actually started out without recording real-world state.  One of the early POCs used cloud-provider tags to discover which resources were managed by Terraform.  This was problematic, however, because not all cloud providers support tags, and this approach limited what can be deployed using Terraform.

## What is it?

Terraform state is a "database" which maps Terraform config to the real world.  

When you create resources in Terraform configuration, this is reflected in state, the following configuration would map the resource name `foo` to an instance id.

```hcl
resource "aws_instance" "foo" {}
```

Terraform expects that remote objects should only be bound to a single resource, so you must be careful when using `terraform import` commands to only import objects to a single resource.

## Remote vs local

Terraform state can either be stored locally, or remotely.  The local Terraform state file is called `terraform.tfstate`.  The Terraform state file *can* include credentials, so you should never commit this to version control.  You can use a [gitignore](https://github.com/github/gitignore/blob/main/Terraform.gitignore) file to ensure this doesn't occur.

When working with a team, its important to use remote state to ensure that:

* The state file is synced between multiple engineer workstations
* Only one engineer is writing changes at any one time

There are many different configurations for remote state storage, which are covered [here](./2-terraform-settings.md).

## State locking

When working with remote state, it is likely that you are working as part of a team.  Given the way that state works by comparing the real-world state to Terraform's knowledge, it's important to implement **state locking**.  This ensure that whilst Terraform configuration is being applied by one engineer, it cannot also be applied by another.

Services such as DynamoDB in AWS are often used for this purpose.  The Azurerm backend and HCP Terraform provides state locking using native capabilities.  Find out more [here](./2-terraform-settings.md).

## Credentials

At some point, you are likely to have to use or deploy a credential using Terraform.  It is important to understand that any credentials which are deployed using Terraform, **will** end up in Terraform state.  

All of the big cloud providers offer a key vault solution, or [Hashicorp Vault](https://www.vaultproject.io/) can be used.

## Terraform state commands

Terraform state commands are used to interact with the state file.  Whilst this is simply a JSON file, and can be viewed for troubleshooting, you should only ever interact with the state file using `terraform state` commands.

### List

* `terraform state list` - to view all the current resources that the state file knows about, including modules.
* `terraform state list aws_instance.foo` - used to view all resources for the given name.
* `terraform state list module.foo` - used to list all the resources in a given module.
* `terraform state list -id=i-2939a28d` - List a resource which is mapped to this specific id.

### Move

* `terraform state mv my_resource.foo my_resource.bar` - mostly used during refactoring.  When using this command, ensure you communicate with co-workers beforehand, to ensure people don't attempt to make changes and you end up with unintended results.

### Remove

* `terraform state rm my_resource.foo` - Used to remove a particular resource from state.  This might be used to remove a resource that was manually deleted already.

These commands can also be used to interact with state when the backend is hosted by HCP Terraform.
