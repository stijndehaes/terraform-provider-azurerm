---
subcategory: "CDN"
layout: "azurerm"
page_title: "Azure Resource Manager: azurerm_cdn_frontdoor_profile_custom_domain"
description: |-
  Manages a CDN FrontDoor Custom Domain.
---

# azurerm_cdn_frontdoor_profile_custom_domain

Manages a CDN FrontDoor Custom Domain.

!>**IMPORTANT:** If you are using Terraform to manage your DNS Auth and DNS CNAME records for your Custom Domain you will need to add configuration blocks for both the `azurerm_dns_txt_record`(see the `Example DNS Auth TXT Record Usage` below) and the `azurerm_dns_cname_record`(see the `Example CNAME Record Usage` below) to your configuration file.

## Example Usage

```hcl
resource "azurerm_resource_group" "example" {
  name     = "example-cdn-frontdoor"
  location = "West Europe"
}

resource "azurerm_dns_zone" "example" {
  name                = "sub-domain.domain.com"
  resource_group_name = azurerm_resource_group.test.name
}

resource "azurerm_cdn_frontdoor_profile" "example" {
  name                = "example-profile"
  resource_group_name = azurerm_resource_group.example.name
  sku_name            = "Standard_AzureFrontDoor"
}

resource "azurerm_cdn_frontdoor_custom_domain" "example" {
  name                     = "example-customDomain"
  cdn_frontdoor_profile_id = azurerm_cdn_frontdoor_profile.example.id
  dns_zone_id              = azurerm_dns_zone.example.id
  host_name                = "contoso.com"

  associate_with_cdn_frontdoor_route_id = azurerm_cdn_frontdoor_route.example.id

  tls {
    certificate_type    = "ManagedCertificate"
    minimum_tls_version = "TLS12"
  }
}
```
## Example DNS Auth TXT Record Usage

```hcl
resource "azurerm_dns_txt_record" "example" {
  name                = join(".", ["_dnsauth", "contoso"])
  zone_name           = azurerm_dns_zone.example.name
  resource_group_name = azurerm_resource_group.example.name
  ttl                 = 3600

  record {
    value = azurerm_cdn_frontdoor_custom_domain.example.validation_token
  }
}
```

## Example CNAME Record Usage

!>**IMPORTANT:** You **must** include the `depends_on` meta-argument which references both the `azurerm_cdn_frontdoor_route` and the `azurerm_cdn_frontdoor_security_policy` that are associated with your Custom Domain. The reason for these `depends_on` meta-arguments is because all of the resources for the Custom Domain need to be associated within FrontDoor before the CNAME record can be written to the domains DNS, else the CNAME validation will fail and FrontDoor will not enable traffic to the Domain.

```hcl
resource "azurerm_dns_cname_record" "example" {
  depends_on = [azurerm_cdn_frontdoor_route.example, azurerm_cdn_frontdoor_security_policy.example]

  name                = "contoso"
  zone_name           = azurerm_dns_zone.example.name
  resource_group_name = azurerm_resource_group.example.name
  ttl                 = 3600
  record              = azurerm_cdn_frontdoor_endpoint.example.host_name
}
```

## Arguments Reference

The following arguments are supported:

* `name` - (Required) The name which should be used for this CDN FrontDoor Custom Domain. Possible values must be between 2 and 260 characters in length, must begin with a letter or number, end with a letter or number and contain only letters, numbers and hyphens. Changing this forces a new CDN FrontDoor Custom Domain to be created.

* `cdn_frontdoor_profile_id` - (Required) The ID of the Frontdoor Profile. Changing this forces a new Frontdoor Profile to be created.

* `host_name` - (Required) The host name of the domain. Changing this forces a new CDN FrontDoor Custom Domain to be created.

* `associate_with_cdn_frontdoor_route_id` (Optional) - The resource ID of the CDN FrontDoor Route this CDN FrontDoor Custom Domain should be associated with.

* `dns_zone_id` - (Optional) The ID of the DNS Zone which should be used for this FrontDoor Custom Domain.

<!-- * `pre_validated_cdn_frontdoor_custom_domain_id` - (Optional) The resource ID of the pre-validated CDN FrontDoor Custom Domain. This domain type is used when you wish to onboard a validated Azure service domain, and then configure the Azure service behind an Azure Front Door.

->**NOTE:** Currently `pre_validated_cdn_frontdoor_custom_domain_id` only supports domains validated by Static Web App. -->

* `tls` - (Required) A `tls` block as defined below.

---

A `tls` block supports the following:

* `certificate_type` - (Optional) Defines the source of the SSL certificate. Possible values include `CustomerCertificate` and `ManagedCertificate`. Defaults to `ManagedCertificate`.

->**NOTE:** It may take upto 15 minutes for the Frontdoor Service to validate the state and Domain ownership of the Custom Domain.

* `minimum_tls_version` - (Optional) TLS protocol version that will be used for Https. Possible values include `TLS10` and `TLS12`. Defaults to `TLS12`.

* `cdn_frontdoor_secret_id` - (Optional) Resource ID of the Frontdoor Secrect.

---

## Attributes Reference

In addition to the Arguments listed above - the following Attributes are exported:

* `id` - The ID of the CDN FrontDoor Custom Domain.

* `expiration_date` - The date time that the token expires.

* `validation_token` - Challenge used for DNS TXT record or file based validation.

## Timeouts

The `timeouts` block allows you to specify [timeouts](https://www.terraform.io/docs/configuration/resources.html#timeouts) for certain actions:

* `create` - (Defaults to 12 hours) Used when creating the CDN FrontDoor Custom Domain.
* `read` - (Defaults to 5 minutes) Used when retrieving the CDN FrontDoor Custom Domain.
* `update` - (Defaults to 24 hours) Used when updating the CDN FrontDoor Custom Domain.
* `delete` - (Defaults to 12 hours) Used when deleting the CDN FrontDoor Custom Domain.

## Import

CDN FrontDoor Custom Domains can be imported using the `resource id`, e.g.

```shell
terraform import azurerm_cdn_frontdoor_custom_domain.example /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/resourceGroup1/providers/Microsoft.Cdn/profiles/profile1/customDomains/customDomain1
```
