# Terraform

Presently, the `offer-service` [Terraform configuration](https://bitbucket.org/fetchrewards/offer-service/src/master/terraform/) is not integrated into BitBucket Pipelines and is maintained/deployed locally.
Refer to the config for resources managed, but the `offer-service` itself is not governed by Terraform. 

## Workspaces

Note the pattern: `ENV_offer_REGION_ACCT-ID`

- `default`: Unused, please be sure to select the correct/desired environment.
- `dev_offer_us-east-1_292095839402`
- `preprod_offer_us-east-1_292095839402`
- `prod_offer_us-east-1_292095839402`
- `stage_offer_us-east-1_292095839402`

## Usage

1. Ensure the `[profile codecommit]` entry exists in `~/.aws/config`:
    ```yaml
    [profile codecommit]
    region = us-east-1
    role_arn =  arn:aws:iam::292095839402:role/terraform-state-codecommit
    source_profile = admin
    ```
1. Ensure SSO credentials are up-to-date
1. Refer to the following example snippets:
    ```shell
    # Init, required for each workspace
    terraform init
    
    # Interact w/ workspace(s)
    terraform workspace list
    terraform workspace select ...
   
    # Plan/Apply
    terraform plan
    terraform apply
    ```
