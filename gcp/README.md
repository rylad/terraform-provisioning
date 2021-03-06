# Lacework Terraform Provisioning for GCP
Terraform modules that create GCP resources required to integrate GCP Organizations or Projects
with the Lacework Cloud Security Platform.

## Use your Google Console

This couldn't be easier!

Follow [these instructions](GOOGLE_CLOUD_SHELL.md) to use the Google Cloud Shell to run these modules from
the comfort of your Google Console.

## Requirements
If you prefer to use these modules locally, you must meet the following requirements:

- [Terraform](terraform.io/downloads.html) `v0.12.x`
- [GCP Service Account](https://cloud.google.com/iam/docs/service-accounts)
- [Lacework API Key](https://support.lacework.com/hc/en-us/articles/360011403853-Generate-API-Access-Keys-and-Tokens) 

We also recommend that the [Lacework CLI](https://github.com/lacework/go-sdk/wiki/CLI-Documentation) be installed and the `[default]` profile is associated with the applicable Lacework Account `api_key` and `api_secret` in `~/.lacework.toml`

## GCP Organziation Level Integrations
The following section covers how to integrate GCP configuration assessment and Audit Log for
an entire Google Cloud Organization

### Setup GCP Service Account
To integrate Lacework with a Google Cloud Organization you will need a GCP service account with
the following permissions:
- Logs Configuration Writer
- Organziation Admin
- Project Owner

Download the service account `json` file to your workstation and move on to the next section.

More information on how to download a GCP service account key can be found [here](https://cloud.google.com/iam/docs/creating-managing-service-account-keys)

### Usage

**IMPORTANT:** We use the `master` branch in source just as an example. In your code, **do NOT pin to master** because there may
be breaking changes between releases. Instead we recommend to pin to the release tag (e.g. `?ref=tags/v0.1.0`) of one of
our [latest releases](https://github.com/lacework/terraform-provisioning/releases).

#### Enable New GCP Organization
```hcl
provider "google" {}

provider "lacework" {}

module "gcp_organization_config" {
	source          = "git::https://github.com/lacework/terraform-provisioning.git//gcp/modules/config?ref=master"
	org_integration = true
	organization_id = "my-organization-id"
}

module "gcp_organization_audit_log" {
	source                       = "git::https://github.com/lacework/terraform-provisioning.git//gcp/modules/audit_log?ref=master"
	bucket_force_destroy         = true
	org_integration              = true
	use_existing_service_account = true
	service_account_name         = module.gcp_organization_config.service_account_name
	service_account_private_key  = module.gcp_organization_config.service_account_private_key
	organization_id              = "my-organization-id"
}
```

More information on adding GCP credentials for Terraform can be found [here](https://www.terraform.io/docs/providers/google/guides/getting_started.html#adding-credentials)

## GCP Project Level Integration
The following section covers how to integrate GCP configuration assessment and Audit Log on a per
project basis. 

### Setup GCP Service Account
In order to integrate Lacework with a GCP Project you will need a GCP service account in each project you intend to integrate with the following permissions:
- Project Owner

Download the service account `json` file to your workstation and move on to the next section.

More information on how to download a GCP service account key can be found [here](https://cloud.google.com/iam/docs/creating-managing-service-account-keys)

### Usage

**IMPORTANT:** We use the `master` branch in source just as an example. In your code, *do NOT pin to master* because there may
be breaking changes between releases. Instead we recommend to pin to the release tag (e.g. `?ref=tags/v0.1.0`) of one of
our [latest releases](https://github.com/lacework/terraform-provisioning/releases).

#### Enable New GCP Project
```hcl
provider "google" {}

provider "lacework" {}

module "gcp_project_config" {
	source = "git::https://github.com/lacework/terraform-provisioning.git//gcp/modules/config?ref=master"
}

module "gcp_project_audit_log" {
	source                       = "git::https://github.com/lacework/terraform-provisioning.git//gcp/modules/audit_log?ref=master"
	bucket_force_destroy         = true
	use_existing_service_account = true
	service_account_name         = module.gcp_project_config.service_account_name
	service_account_private_key  = module.gcp_project_config.service_account_private_key
}
```
More information on adding GCP credentials for Terraform can be found [here](https://www.terraform.io/docs/providers/google/guides/getting_started.html#adding-credentials)

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| prefix | The prefix that will be use at the beginning of every generated resource | `string` | lw-at | no |
| org_integration | Set this to `true` to configure an organization level integration. When set to `true` you must provide the `organization_id` | `bool` | `false` | no |
| organization_id | The organization ID | `string` | "" | no |
| project_id | A project ID different from the default defined inside the `google` provider. | `string` | "" | no |
| use_existing_service_account | Set this to `true` to use an existing Service Account. When set to `true` you must provide both the `service_account_name` and `service_account_private_key` | `bool` | `false` | no |
| service_account_name | The Service Account name . | `string` | "" | no |
| service_account_private_key | The private key in JSON format, base64 encoded. | `string` | "" | no |
| bucket_force_destroy | Force destroy storage bucket (Required when bucket not empty) | `bool` | false | no |
| existing_bucket_name | The name of an existing bucket you want to send the logs to | `string` | "" | no |
| lacework_integration_name | The name of the integration in Lacework. This input is available in both the config, and the audit_log module | `string` | TF config | no |
| wait_time | Define a custom delay between cloud resource provision and Lacework external integration to avoid errors while things settle down. Use `10s` for 10 seconds, `5m` for 5 minutes. | `string` | `10s` | no |

## Outputs

| Name | Description |
|------|-------------|
| service_account_name | The Service Account name |
| service_account_private_key | The private key in JSON format, base64 encoded |
| bucket_name | The storage bucket name |
| pubsub_topic_name | The PubSub topic name |
