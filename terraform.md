# Terraform

- [Terraform](#terraform)
  - [Base configuration](#base-configuration)
  - [Variables](#variables)
  - [State operations](#state-operations)
    - [Move resources from one statefile to another](#move-resources-from-one-statefile-to-another)
  - [Resources](#resources)
    - [IAM](#iam)
      - [`aws_iam_user`](#aws_iam_user)
      - [`aws_iam_group`](#aws_iam_group)
      - [`aws_iam_group_policy_attachment`](#aws_iam_group_policy_attachment)
      - [`aws_iam_user_group_membership`](#aws_iam_user_group_membership)
      - [`aws_iam_account_password_policy`](#aws_iam_account_password_policy)
      - [`aws_iam_role`](#aws_iam_role)
      - [`aws_iam_role_policy_attachment`](#aws_iam_role_policy_attachment)

## Base configuration

Provider configuration (e.g. `terraform.tf`):

```hcl
terraform {
  required_version = "0.12.24"
}

provider "aws" {
  access_key = var.aws_access_key_id
  secret_key = var.aws_secret_access_key
  region     = var.aws_default_region
}
```

Input variables (e.g. `inputs.tf`):

```hcl
variable "aws_access_key_id" {
  type = string
}

variable "aws_secret_access_key" {
  type = string
}

variable "aws_default_region" {
  type = string
}
```

See [Variables](#variables) for setting env vars.

## Variables

Environment variables can be used by TF by prefixing them with `TF_VAR_`.

## State operations

```sh
# Import a resource
#
terraform import -state-out=terraform.tfstate aws_iam_user.saverio saverio

# List resources
#
terraform state list
```

### Move resources from one statefile to another

```sh
resources=(res1 res2)

cd "$project_dest"
terraform state pull > remote.tfstate

cd "$project_src"
for resource in "${resources[@]}"; do
  terraform state mv -state-out="$project_dest/remote.tfstate" "$resource" "$resource"
done

cd "$project_dest"
terraform state push remote.tfstate
```

## Resources

### IAM

#### `aws_iam_user`

```hcl
# import aws_iam_user.lb loadbalancer
#
resource "aws_iam_user" "lb" {
  name = "loadbalancer"
}
```

#### `aws_iam_group`

```hcl
# import aws_iam_group.developers developers
#
resource "aws_iam_group" "developers" {
  name = "developers"
}
```

#### `aws_iam_group_policy_attachment`

```hcl
# import aws_iam_group_policy_attachment.test-attach test-group/arn:aws:iam::xxxxxxxxxxxx:policy/test-policy
#
resource "aws_iam_group_policy_attachment" "test-attach" {
  group      = aws_iam_group.group.name
  policy_arn = aws_iam_policy.policy.arn
}
```

#### `aws_iam_user_group_membership`

Non-exclusive - a user can have multiple resources.

```hcl
# import aws_iam_user_group_membership.example1 user1/group1/group2
#
resource "aws_iam_user_group_membership" "example1" {
  user = aws_iam_user.user1.name

  groups = [
    aws_iam_group.group1.name,
    aws_iam_group.group2.name,
  ]
}
```

#### `aws_iam_account_password_policy`

```hcl
# import aws_iam_account_password_policy.strict iam-account-password-policy
#
resource "aws_iam_account_password_policy" "strict" {
  minimum_password_length        = 8
  require_lowercase_characters   = true
  require_numbers                = true
  require_uppercase_characters   = true
  require_symbols                = true
  allow_users_to_change_password = true
}
```

#### `aws_iam_role`

```hcl
# import aws_iam_role.demo_role DemoRoleForEC2
#
resource "aws_iam_role" "demo_role" {
  name = "DemoRoleForEC2"

  assume_role_policy = <<EOF
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Action": "sts:AssumeRole",
          "Principal": {
            "Service": "ec2.amazonaws.com"
          },
          "Effect": "Allow"
        }
      ]
    }
EOF
}
```

#### `aws_iam_role_policy_attachment`

The attachment is non-exclusive.

```hcl
# import aws_iam_role_policy_attachment.test-attach DemoRoleForEC2/arn:aws:iam::aws:policy/IAMReadOnlyAccess
#
resource "aws_iam_role_policy_attachment" "test-attach" {
  role       = aws_iam_role.demo_role.name
  policy_arn = "arn:aws:iam::aws:policy/IAMReadOnlyAccess"
}
```
