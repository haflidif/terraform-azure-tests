# Terraform Tests - Azure GitHub Action

[![Dependabot](https://badgen.net/badge/Dependabot/enabled/green?icon=dependabot)](https://dependabot.com/)

This GitHub Action runs tests against Terraform code/modules using an Azure Remote Backend.

## Inputs

| Name                      | Description                                                                                                      | Required | Type    | Default                |
|---------------------------|------------------------------------------------------------------------------------------------------------------|----------|---------|------------------------|
| `path`                    | (Optional) Path to the Terraform configuration files, defaults to the current working directory.                  | No       | string  | `.`                    |
| `test_type`               | (Required) Specify a test type, valid options are `plan`, `plan-apply`, `plan-apply-destroy`. Defaults to `plan`. | Yes      | string  | `plan`                 |
| `tf_version`              | (Optional) The version of Terraform to use. Defaults to the latest version.                                       | No       | string  | `latest`               |
| `tf_var_file`             | (Optional) The variable file to use. Defaults to `terraform.tfvars`.                                              | No       | string  | `terraform.tfvars`     |
| `tf_state_file`           | (Optional) The state file to use. Defaults to `terraform.tfstate`.                                                | No       | string  | `terraform.tfstate`    |
| `az_resource_group`       | (Required) The name of the Azure Resource Group to use for the remote backend.                                    | Yes      | string  |                        |
| `az_storage_account_name` | (Required) The name of the Azure Storage Account to use for the remote backend.                                   | Yes      | string  |                        |
| `az_storage_container_name`| (Required) The name of the Azure Storage Container to use for the remote backend.                                | Yes      | string  |                        |
| `az_storage_key`          | (Required) The key to use for the remote backend.                                                                 | Yes      | string  |                        |
| `arm_client_id`           | (Required) The client ID of the `Azure Service Principal` or `User Assigned Managed Identity` to use for authentication. | Yes      | string  |                        |
| `arm_use_oidc`            | (Optional) Whether to use OIDC for authentication. Defaults to `true`.                                            | No       | boolean | `true`                 |
| `arm_use_azuread`         | (Optional) Whether to use Azure AD for authentication towards Azure Remote Backend Storage Account and Containers. Requires RBAC permissions `Storage Blob Data Owner` or `Storage Blob Data Contributor` on the Storage Account/Container for the identity being used. Defaults to `true`. | No       | boolean | `true`                 |
| `arm_tenant_id`           | (Required) The tenant ID of the Azure Service Principal to use for authentication.                                | Yes      | string  |                        |
| `arm_subscription_id`     | (Required) The subscription ID of the Azure Service Principal to use for authentication.                          | Yes      | string  |                        |
| `github_token`            | (Optional) The GitHub token to use for posting comments on pull requests. Defaults to `${{ github.token }}`.      | No       | string  | `${{ github.token }}`  |

## Usage Examples

### Prerequisites

This test Github Actions assumes that User Assigned Managed Identity with Federated Credentials are being used, and that the identity has been assigned RBAC permissions on the Storage Account/Container.

- Create a User Assigned Managed Identity in Azure.
- Create a Federated Credential for the User Assigned Managed Identity - See here for more information: [Configure a federated identity credential on a user-assigned managed identity](https://learn.microsoft.com/en-us/entra/workload-id/workload-identity-federation-create-trust-user-assigned-managed-identity?pivots=identity-wif-mi-methods-azp#configure-a-federated-identity-credential-on-a-user-assigned-managed-identity)
- Assign RBAC permissions `Storage Blob Data Owner` or `Storage Blob Data Contributor` on the Storage Account/Container for the User Assigned Managed Identity.

Remember to set the right `jobs` permissions to use the id-token permission in your workflow. 

Example:
```yaml
permissions:
    id-token: write
    contents: read
```


### Installation

```yaml
name: Terraform Tests - Azure

on:
    push:
        branches:
            - main

jobs:
    terraform-tests:
        runs-on: ubuntu-latest
        environment: your-environment-name # Optional, if your are using environment scope in your Federated Credentials.
        permissions:
            id-token: write
            contents: read
            issues: write
            actions: read
        steps:
            - name: Checkout
                uses: actions/checkout@v4.2.0

            - name: Terraform Tests
                uses: ./
                with:
                    path: 'where-your-terraform-files-are-located'
                    test_type: 'plan'
                    tf_version: 'latest'
                    tf_var_file: 'terraform.tfvars'
                    tf_state_file: 'terraform.tfstate'
                    az_resource_group: 'your-resource-group'
                    az_storage_account_name: 'your-storage-account-name'
                    az_storage_container_name: 'your-container-name'
                    az_storage_key: 'your-storage-key'
                    arm_client_id: 'your-client-id'
                    arm_tenant_id: 'your-tenant-id'
                    arm_subscription_id: 'your-subscription-id'
                    github_token: ${{ secrets.GITHUB_TOKEN }}
```

### Testing using Plan Only mode.

This example runs a Terraform plan only test.

```yaml
name: Terraform Tests - Azure

on:
    push:
        branches:
            - main

jobs:
    terraform-tests:
        runs-on: ubuntu-latest
        environment: your-environment-name # Optional, if your are using environment scope in your Federated Credentials.
        permissions:
            id-token: write
            contents: read
            issues: write
            actions: read
        steps:
            - name: Checkout
                uses: actions/checkout@v4.2.0

            - name: Terraform Tests
                uses: haflidif/terraform-tests-azure@v1.0.0
                with:
                    test_type: 'plan'
                    az_resource_group: 'your-resource-group'
                    az_storage_account_name: 'your-storage-account-name'
                    az_storage_container_name: 'your-container-name'
                    az_storage_key: 'your-storage-key'
                    arm_client_id: 'your-client-id'
                    arm_tenant_id: 'your-tenant-id'
                    arm_subscription_id: 'your-subscription-id'
```

### Testing using Plan and Apply mode.

This code example will run a `terraform plan` and `terraform apply` in sequence, to deploy the infrastructure.

```yaml
name: Terraform Tests - Azure

on:
    push:
        branches:
            - main

jobs:
    terraform-tests:
        runs-on: ubuntu-latest
        environment: your-environment-name # Optional, if your are using environment scope in your Federated Credentials.
        permissions:
            id-token: write
            contents: read
            issues: write
            actions: read
        steps:
            - name: Checkout
                uses: actions/checkout@v4.2.0

            - name: Terraform Tests
                uses: haflidif/terraform-tests-azure@v1.0.0
                with:
                    test_type: 'plan-apply'
                    az_resource_group: 'your-resource-group'
                    az_storage_account_name: 'your-storage-account-name'
                    az_storage_container_name: 'your-container-name'
                    az_storage_key: 'your-storage-key'
                    arm_client_id: 'your-client-id'
                    arm_tenant_id: 'your-tenant-id'
                    arm_subscription_id: 'your-subscription-id'
```

### Testing using Plan, Apply, and Destroy mode.

This code example will run a `terraform plan`, `terraform apply`, and `terraform destroy` in sequence, to deploy and then tear down the infrastructure.

```yaml
name: Terraform Tests - Azure

on:
    push:
        branches:
            - main

jobs:
    terraform-tests:
        runs-on: ubuntu-latest
        environment: your-environment-name # Optional, if your are using environment scope in your Federated Credentials.
        permissions:
            id-token: write
            contents: read
            issues: write
            actions: read
        steps:
            - name: Checkout
                uses: actions/checkout@v4.2.0

            - name: Terraform Tests
                uses: haflidif/terraform-tests-azure@v1.0.0
                with:
                    test_type: 'plan-apply-destroy'
                    az_resource_group: 'your-resource-group'
                    az_storage_account_name: 'your-storage-account-name'
                    az_storage_container_name: 'your-container-name'
                    az_storage_key: 'your-storage-key'
                    arm_client_id: 'your-client-id'
                    arm_tenant_id: 'your-tenant-id'
                    arm_subscription_id: 'your-subscription-id'
```

## Credits
Inspired and based on GitHub Action by [Haflidi Fridthjofsson](https://github.com/haflidif)

## Authors
- Modified and refactored by [Haflidi Fridthjofsson](https://github.com/haflidif)

## Disclaimer
The code examples in this repository are provided “as-is” and can be used in production environments at your own responsibility. They come with no warranty of any kind. Use them at your own risk. I am not responsible for any issues, damages, or costs that may arise from using these code examples.

## License
This project is licensed under the GPL-3.0 License - see the [LICENSE](LICENSE) file for details.