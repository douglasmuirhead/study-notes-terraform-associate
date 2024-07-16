# Terraform Providers

## What is it?

Providers are an essential part of Terraform.  They are what enables it to use one common language - HCL - to configure and deploy infrastructure across a multitude of Public Cloud providers and infrastructure platforms.

Providers can also provide utilities to Terraform configuration, such as generating numbers or resource names.

Terraform providers are distributed via the [Terraform Registry](https://registry.terraform.io/browse/providers).  At the time of writing, there are over 4000 individual providers available on the public regstry.  

The `provider` block is where providers are configured.  Some providers don't need any configuration, but some will require URLs or credentials.

## Provider Requirements

The Terraform `required_providers` is nested within the `terraform` block.  This is where provider versions are managed.

It's important to keep on top of provider versions and dependancies.  Most of the big public clouds have well maintained and regularly updated providers.  When upgrading, it's important to review the changelog to ensure there are no breaking changes introduced during the upgrade, and also that it is compatible with your current [Terraform version](/2-terraform-settings.md#terraform-versions).

## Local Names

Users of a Terraform provider can choose a name for it.  This can be dictated here:

```hcl
terraform {
    required providers {
        myprovidername = {
            source = "myprovider/mycloud"
        }
    }
}
```

Nearly every provider will have it's own _preferred local name_, which it prefixes all of it's resources with, for example, `hashicorp/azurerm`, all resources will be prefixed with `azurerm`.  Wherever possible, you should use the providers preferred local name, otherwise you will have to use the `provider` meta-argument for most resources which will make your code harder to understand.

## Source Addresses

Terraform provider source addresses follow this formatting:

`[HOSTNAME/]NAMESPACE/TYPE`

* Hostname: The hostname of the Terraform registry that distributes the provider.  If this is not specified, it will default to [registry.terraform.io](https://terraform.registry.io).
* Namespace: The organization that maintains & publishes this provider.
* Type: The name of the provider.  

Lets look at an example: `hashicorp/azurerm`. 

Looking at this, we can tell that this provider is distributed via [registry.terraform.io](https://terraform.registry.io), `hashicorp` is the organization which maintains & distributes the provider, and `azurerm` is the shortname of the provider.  

## Provider version constraints

Provider version constraints work in a very similar way to [Terraform version constraints](/2-terraform-settings.md#terraform-versions).  

### Best practices for provider versions

The best practice is to always specify a minimum version that your modules are known to work with, either by using `>=` or `~>`.

If you are defining provider version in your module configurations, you should not configure maximum versions.  This will create scenarios where updating a module's provider version could mean updating large amounts of resources and carry a high risk level.  Instead, you should only define a minimum version, and let the root module & the maintainers of it define the maximum version.
