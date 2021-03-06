# Terraform Preboot

> :warning: the pratices promoted in this boilerplate are highly opinionated, and the examples require an [AWS](aws.amazon.com) account

## Features

* Multi-stages (TF Workspaces)
* Multi-regions (AWS)
* S3 Backend (+ Locking)
* Best-practices
* Tooling

## Setup

> `terraform > 0.10.0` is required, because `workspace` is used to control the available environments (`dev`, `test`, and `prod`), and to support hierarchies.
> `aws` cli installed, and configured
> be careful, the backend should be initialized only once

```
# --depth 1 removes all but one .git commit history
git clone --depth 1 https://github.com/katallaxie/tf-preboot.git

# Choose your prefered region, and setup remote state in prefered region
./utils/setup eu-west-1
```
> your ACCOUNT_ID is used to obfiscate the S3 bucket and DynamoDB Table

Please, either set `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`, or `AWS_SHARED_CREDENTIALS_FILE` to specify your [AWS](https://aws.amazon.com) credentials. You can find further information [here](https://www.terraform.io/docs/providers/aws/#environment-variables).

It is best practice to create a new [IAM](https://console.aws.amazon.com/iam) User (e.g. `tf-example`). You should use [AWS Profiles](https://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html) to stear the access.

An `~/.aws/credentials` would look like this

```
[default]
aws_access_key_id=YOUR_ACCESS_KEY
aws_secret_access_key=YOUR_ACCESS_SECRET

[tf-user]
aws_access_key_id=TF_ACCESS_KEY
aws_secret_access_key=TF_ACCESS_SECRET
```

The `~/.aws/config` would look like this

```
[default]
region=us-west-2
output=json

[profile tf-user]
output=json
```

You can then export the `tf-user` profile via `AWS_PROFILE` and use it in terraform.

There are three different environments defined (`dev`, `test`, and `prod`) to which you can deploy your infrastructure. Most will likely only use `prod`.

In `regions` you find an already defined region (`eu-west-1`). Of which env variables are located in `envs`. You use a region by requiring the module in `main.tf`.

We only use targets to deploy the infrastructure to the various regions and environments

Plan your [AWS VPC](https://aws.amazon.com/vpc/) in the `eu-west-1` region.

```
./utils/cli plan --env=dev --region=eu-west-1 --target=module.vpc
```

This writes the plan in the `plans` folder, which can be reviewd and is commited.

Apply your plan to the select region in the desired environment.

```
./utils/cli apply --env=dev --region=eu-west-1 --target=module.vpc
```

Voilà, you have deployed a `dev` VPC in the `eu-west-1` region.

> `./utils/cli destroy --env=dev --region=eu-west-1 --target=module.vpc` destroys this VPC again

## Example

```
# Setup Terraform in eu-west-1 region
./utils/setup eu-west-1

# Plan VPC in eu-west-1 region
./utils/cli plan --env=prod --region=eu-west-1 --target=module.vpc

# Apply VPC in eu-west-1 region
./utils/cli apply --env=prod --region=eu-west-1 --target=module.vpc
``` 

## CLI

There is a wrapper for `terraform` in `utils`. This enforces to save a plan and apply a plan in singular steps and to target the various modules, in the varios regions in the various enviroments.

The `cli` tool accepts various parameters `./utils/cli <subcommand> --env=<dev|test|prod> --region=<aws-region> --target=<module.*>`.

> the `target` is prefixed with the region selected

### `plan`

This saves a plan for later application in the `plans` folder.

### `apply`

This applies a saved plan from the `plans` folder.

### `destroy`

This destroyes a target

### Example
```
./utils/cli plan --env=dev --region=eu-west-1 --target=module.vpc
```

The region is an automatic prefix for the `-target`. So actually you are targeting `module.eu-west-1.module.vpc` in the above command.

# License
[MIT](/LICENSE)
