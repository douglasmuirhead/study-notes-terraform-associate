# Terraform Settings

## What is it?

The `terraform` block is for configuring Terraform itself.  This block can be used to specify some of the following configurations:

* HCP Terraform Cloud
* Backend configuration
* Terraform executable required version
* Required providers & their versions
* Enable experimental features
* Provider metadata

## HCP Terraform / Terraform Cloud

In order to have access to the CLI-Driven workflow, you must configure Terraform to do so.  This workflow is unique to users of HCP Terraform Cloud, and enables users to run commands such as `terraform plan` and `terraform apply` (dangerous) from the files in your local directory.  

>*opinion: I really like this workflow.  It's super convenient and makes working with Terraform configuration MUCH easier, resulting in more PRs being approved first time.*

### The `cloud` block

This Terraform setting configures Terraform to work with HCP Terraform.  It looks something like this:

```hcl
terraform {
    cloud {
        organization = "my_org"
        workspaces {
            name = "my_workspace"
        }
    }
}
```

Once the `cloud` block is specified in your Terraform configuration, you can use `terraform init` to initialize your configuration.  If you do not have a workspace created with the name specified, HCP Terraform cloud will automatically create one.  If you also specify a **project** in the workspaces block, HCP will also create this automatically if it does not exist.

The `cloud` block can also be configured with multiple workspaces using the `prefix` argument.  For example, if you have a `my_workspace_prod` and `my_workspace_stag`, you could use `prefix = "my_workspace_"` the use the `terraform workspace select` command to switch between workspaces.  It's useful when working with multiple environments as part of your development lifecycle.

If your configuration includes a `cloud` block, it cannot include a `backend` block.

## Backend configuration

### The `backend` block

If you are not using HCP Terraform, the, `backend` block can be used to configure the Terraform state backend.  There are many different types of backend available:

* local
* remote
* artifactory
* azurerm
* consul
* cos
* etcd
* etcdv3
* gcs
* http
* kubernetes
* manta
* oss
* pg
* oss
* s3
* swift

The most common backend types are **s3, azurerm and remote**, and the configuration of these is what I'll focus on.

### s3

When working with AWS, an s3 bucket can be used for remote state.  This is often combined with DynamoDB to provide state locking.

S3 buckets can be secured by using either IAM policies attached to users, either via the user or groups, or a resource policy applied directly to the bucket.  DynamoDB will require an IAM policy.

Example S3 bucket IAM policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::mybucket"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject", "s3:DeleteObject"],
      "Resource": "arn:aws:s3:::mybucket/path/to/my/key"
    }
  ]
}
```

Example DynamoDB IAM policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:DeleteItem"
      ],
      "Resource": "arn:aws:dynamodb:*:*:table/mytable"
    }
  ]
}
```

Example s3 with DynamoDB state locking Terraform configuration:

```hcl
terraform {
  backend "s3" {
    bucket            = "mybucket"
    key               = "path/to/my/key"
    region            = "us-east-1"
    dynamodb_table    = "terraform_state
  }
}
```

### AzureRM

State is stored in the Azure backend as a blob in a container.  The Azure backend benefits from state locking & consistency checks with no additional configuration required.

When using Entra ID Authentication, the user must have the `Storage Blob Data Owner` role.

Terraform can authenticate with Azure in several different ways including:

* Azure CLI
* Service Principal
* Managed System Identity
* Azure AD (Entra ID)
* Access Key

Example backend configuration with Azure AD Authentication:

```hcl
terraform {
  backend "azurerm" {
    storage_account_name = "example_storage_account"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
    use_azuread_auth     = true
    subscription_id      = "00000000-0000-0000-0000-000000000000"
    tenant_id            = "00000000-0000-0000-0000-000000000000"
  }
}
```

### Remote

Using the `remote` backend makes use of the HCP Terraform (Terraform Cloud) backend to store state, provides a Terraform-based CD platform, and also grants access to the **CLI-Driven workflow**.

> **Note!**
>
> As of Terraform version 1.1.0, the [cloud](#the-cloud-block) backend was introduced, and it's recommended to use this rather than the `remote` backend.

The `remote` backend can also be used for other Terraform backend integrations.  One example of this is [spacelift.io](spacelift.io).

```hcl
terraform {
  backend "remote" {
    hostname     = "spacelift.io
    organization = "example_org"

    workspaces {
      name = "my_spacelift_stack"
    }
  }
}
```

Typically Spacelift doesn't recommend that you utilise the CLI commands, as Spacelift maintains a lock on the Terraform state file, to quote Spacelift:

> Unfortunately, since the state is locked by Spacelift, you cannot run the Terraform Console against the remote state unless you pull it locally. I say “unfortunately” because you may wish to use it, but just remember that running commands that could possibly modify your remote state from the CLI is actually extremely disadvised. Hence, it’s probably best that this lock stays in place.

For testing and troubleshooting Terraform code, Spacelift has their own CLI modules and GUI driven workflows.

## Terraform Versions

Terraform, just like any other software product, it updated regularly.  With each new update, new features are added, bugs resolved, and security issues fixed, so it's important to stay up to date with the latest versions.

The `terraform` top-level block is used to manage the Terraform `required_version`.  

```hcl
terraform {
  required_version = "1.1.10"
}
```

The code above will require that Terraform uses exactly version `1.1.10`.  If you attempt to initialize Terraform with any other version, you will recieve an error.

Some other version constraints that can be used are as follows:

* `>=` - Any version equal to or greater than the version specified.  This has no upper limit.
* `~>` - Any version of that major & minor branch, but no later.  Minor version upgrades are meant to be non-distruptive and require less thought.
* `>= 1.7.5, < 1.9.5` - Above one version, but lower than another.  Useful for avoiding specific versions.

It's possible to inspect the state file to discover what version of Terraform is being used.  The following command can be used:

`grep -e '"version"' -e '"terraform_version"' terraform.tfstate`
