terraform-provider-octopusdeploy
---
A Terraform provider for [Octopus Deploy](https://octopus.com).

Based on the [go-octopusdeploy](https://github.com/MattHodge/go-octopusdeploy) Octopus Deploy client.

[![Build status](https://ci.appveyor.com/api/projects/status/a5ejcududsoug94e/branch/master?svg=true)](https://ci.appveyor.com/project/MattHodge/terraform-provider-octopusdeploy/branch/master)

> :warning: This provider is in heavy development. It is not production ready yet.

<!-- TOC -->

- [Go Dependencies](#go-dependencies)
- [Downloading & Installing](#downloading--installing)
- [Configure the Provider](#configure-the-provider)
- [Data Sources](#data-sources)
- [Provider Resources](#provider-resources)
- [Provider Resources (To Be Moved To /docs)](#provider-resources-to-be-moved-to-docs)
    - [Variables](#variables)
        - [Example Usage](#example-usage-2)
        - [Argument Reference](#argument-reference-2)
        - [Attributes reference](#attributes-reference)
    - [Machine Policies](#machine-policies)
        - [Example Usage](#example-usage-3)
        - [Argument Reference](#argument-reference-3)
        - [Attributes Reference](#attributes-reference-2)
    - [Machines (Deployment Targets)](#machines-deployment-targets)
        - [Example Usage](#example-usage-4)
        - [Resource Argument Reference](#resource-argument-reference)
        - [Resource Attribute Reference](#resource-attribute-reference)
        - [Data Argument Reference](#data-argument-reference)
        - [Resource Attribute Reference](#resource-attribute-reference-1)

<!-- /TOC -->

# Go Dependencies
* Dependencies are managed using [dep](https://golang.github.io/dep/docs/new-project.html)

```bash
# Vendor new modules
dep ensure
```

# Downloading & Installing

As this provider is still under development, you will need to manually download it.

There are compiled binaries for most platforms in [Releases](https://github.com/MattHodge/terraform-provider-octopusdeploy/releases).

To use it, extract the binary for your platform into the same folder as your `.tf` file(s) will be located, then run `terraform init`.

# Configure the Provider

```hcl
# main.tf

provider "octopusdeploy" {
  address = "http://octopus.production.yolo"
  apikey  = "API-XXXXXXXXXXXXX"
}
```

# Data Sources

- [octopusdeploy_environment](docs/provider/data_sources/environment.md)
- [octopusdeploy_lifecycle](docs/provider/data_sources/lifecycle.md)

# Provider Resources

- [octopusdeploy_environment](docs/provider/resources/environment.md)
- [octopusdeploy_lifecycle](docs/provider/resources/lifecycle.md)
- [octopusdeploy_project](docs/provider/resources/project.md)
- [octopusdeploy_project_group](docs/provider/resources/project_group.md)

# Provider Resources (To Be Moved To /docs)

## Variables

[Variables](https://octopus.com/docs/deployment-process/variables) are values that change based on the
scope of the deployments (e.g. changing SQL Connection Strings between production and staging deployments).

### Example Usage

Basic usage:

```hcl
data "octopusdeploy_project" "finance" {
    name = "Finance"
}

resource "octopusdeploy_variable" "connection_string" {
  project_id = "${data.octopusdeploy_project.finance.id}"
  name       = "SQLConnectionString"
  type       = "String"
  value      = "Server=myServerAddress;Database=myDataBase;Trusted_Connection=True;"
}
```

More complex example (with environments and prompts)

```hcl
data "octopusdeploy_project" "finance" {
    name = "Finance"
}

data "octopusdeploy_environment" "staging" {
    name = "Staging"
}

resource "octopusdeploy_variable" "connection_string" {
  project_id = "${data.octopusdeploy_project.finance.id}"
  name       = "SQLConnectionString"
  type       = "String"
  value      = "Server=myServerAddress;Database=myDataBase;Trusted_Connection=True;"

  scope {
    environments = ["${data.octopusdeploy_environment.staging.id}"]
  }

  prompt{
    label    = "SQL Server Connection String"
    required = false
  }
}
```

Data usage (with scope):

```hcl
data "octopusdeploy_project" "finance" {
    name = "Finance"
}

data "octopusdeploy_environment" "staging" {
    name = "Staging"
}

data "octopusdeploy_variable" "connection_string" {
  project_id = "${data.octopusdeploy_project.finance.id}"
  name       = "SQLConnectionString"
  scope {
    environments = ["${data.octopusdeploy_environment.staging.id}"]
  }
}
```

### Argument Reference

* `project_id` (Required) ID of the Project to assign the variable against.
* `name` - (Required) Name of the variable
* `type` - (Required) Type of the variable. Must be one of `String`, `Certificate` or `AmazonWebServicesAccount` (`Sensitive` is not currently supported)
* `value` - (Required) The value of the variable
* `description` - (Optional) Description of the variable
* `scope` - (Optional) The scope to apply to this variable. Contains a list of arrays. All are optional:
    * (Optional) `environments`, `machines`, `actions`, `roles`, `channels`, `tenant_tags`
* `prompt` - (Optional) Prompt for value when a build is run
    * `label` - (Optional) The label for the prompt
    * `description` - (Optional) The description for the prompt
    * `required`- (Optional) Whether or not the value is required

### Attributes reference

* `id` - ID of the variable
* `name` - Name of the variable
* `type` - Type of the variable
* `value` - Value of the variable
* `description` - Description of the variable

## Machine Policies

[Machine policies](https://octopus.com/docs/infrastructure/machine-policies) are groups of settings that can be applied to Tentacle and SSH endpoints to modify their behavior.

Currently the Octopus terraform provider only provides machine policies as a data provider, as there are places elsewhere in
the provider that the IDs of machine policies need to be referenced.

### Example Usage

```hcl
data "octopusdeploy_machinepolicy" "default" {
  name = "Default Machine Policy"
}
```

### Argument Reference

* `name` - (Required) The name of the machine policy

### Attributes Reference

* `name` - The name of the machine policy
* `description` - The description of the machine policy
* `isdefault` - Whether or not this machine policy is the default policy

## Machines (Deployment Targets)

Octopus Deploy refers to Machines as [Deployment Targets](https://octopus.com/docs/infrastructure), however the API (and thus Terraform) refers to them as Machines.

### Example Usage

Basic Usage

```hcl
data "octopusdeploy_environment" "staging" {
  name = "Staging"
}

data "octopusdeploy_machinepolicy" "default" {
  name = "Default Machine Policy"
}

resource "octopusdeploy_machine" "testmachine" {
  name                            = "finance-web-01"
  environments                    = ["${data.octopusdeploy_environment.staging.id}"]
  isdisabled                      = false
  machinepolicy                   = "${data.octopusdeploy_machinepolicy.default.id}"
  roles                           = ["Staging"]
  tenanteddeploymentparticipation = "Untenanted"

  endpoint {
    communicationstyle = "TentaclePassive"
    thumbprint         = "81D0FF8B76FC"
    uri                = "https://finance-web-01:10933"
  }
}
```

Data Usage

```hcl
resource "octopusdeploy_machine" "testmachine" {
  name = "finance-web-01"
}
```

### Resource Argument Reference

* `name` - (Required) The name of the machine
* `endpoint` - (Required) The configuration of the machine endpoint
    * `communicationstyle` - (Required) Must be one of `None`, `TentaclePassive`, `TentacleActive`, `Ssh`, `OfflineDrop`, `AzureWebApp`, `Ftp`, `AzureCloudService`
    * `proxyid` - (Optional) ID of a defined proxy to use for communication with this machine
    * `thumbprint` - (Required) Thumbprint of the certificate this machine uses (if `communicationstyle` is `None` this should be blank)
    * `uri` - (Required) URI to access this machine (if `endpoint` is `None` this should be blank)
* `environments` - (Required) List of environment IDs to be assigned to this machine
* `isdisabled` - (Required) Whether or not this machine is disabled
* `machinepolicy` - (Required) The ID of the machine policy to be assigned to this machine
* `roles` - (Required) List of the roles to be assigned to this machine
* `tenanteddeploymentparticipation` - (Required) Must be one of `Untenanted`, `TenantedOrUntenanted`, `Tenanted`
* `tenantids` - (Optional) If tenanted, a list of the tenant IDs for this machine
* `tenanttags` - (Optional) If tenanted, a list of the tenant tags for this machine

### Resource Attribute Reference

* `environments` - List of environment IDs this machine is assigned to
* `haslatestcalamari` - Whether or not this machine has the latest Calamari version
* `isdisabled` - Whether or not this machine is disabled
* `isinprocess` - Whether or not this machine is being processed
* `machinepolicy` - The ID of the machine policy assigned to this machine
* `roles` - A list of the roles assigned to this machine
* `status` - The machine status code
* `statussummary` - Plain text description of the machine status
* `tenanteddeploymentparticipation` - One of `Untenanted`, `TenantedOrUntenanted`, `Tenanted`
* `tenantids` - If tenanted, a list of the tenant IDs for this machine
* `tenanttags` -  If tenanted, a list of the tenant tags for this machine

### Data Argument Reference

* `name` - (Required) The name of the machine

### Resource Attribute Reference

All items from the Resource Attribute Reference, and additionally:

* `endpoint_communicationstyle` - One of `None`, `TentaclePassive`, `TentacleActive`, `Ssh`, `OfflineDrop`, `AzureWebApp`, `Ftp`, `AzureCloudService`
* `endpoint_proxyid` - ID of a defined proxy to use for communication with this machine
* `endpoint_tentacleversiondetails_upgradelocked` - Whether or not this machine tentacle is upgrade locked
* `endpoint_tentacleversiondetails_upgraderequired` - Whether or not this machine tentacle required an upgrade
* `endpoint_tentacleversiondetails_upgradesuggested` - Whether or not this machine tentacle has a suggested ugrade
* `endpoint_tentacleversiondetails_version` - Version number of this machine tentacle
* `endpoint_thumbprint` - Thumbprint of the certificate this machine uses
* `endpoint_uri` - URI to access this machine
