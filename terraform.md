# Terraform

- [Terraform](#terraform)
  - [Base configuration](#base-configuration)
  - [HCL](#hcl)
    - [Variables](#variables)
    - [Predefined variables](#predefined-variables)
    - [Strings](#strings)
    - [Functions](#functions)
  - [State operations](#state-operations)
    - [Move resources from one statefile to another](#move-resources-from-one-statefile-to-another)
  - [Resources](#resources)
    - [IAM](#iam)
      - [`aws_iam_group`/`aws_iam_group_policy_attachment`](#aws_iam_groupaws_iam_group_policy_attachment)
      - [`aws_iam_user`/`aws_iam_user_group_membership`](#aws_iam_useraws_iam_user_group_membership)
      - [`aws_iam_account_password_policy`](#aws_iam_account_password_policy)
      - [`aws_iam_role`/`aws_iam_role_policy_attachment`](#aws_iam_roleaws_iam_role_policy_attachment)
    - [Account-related](#account-related)
      - [`aws_budgets_budget`](#aws_budgets_budget)
    - [Networking](#networking)
      - [`aws_vpc`/`aws_subnet`](#aws_vpcaws_subnet)
      - [`aws_security_group`/`aws_security_group_rule`](#aws_security_groupaws_security_group_rule)
    - [EC2 and disks](#ec2-and-disks)
      - [`aws_instance`](#aws_instance)
      - [`aws_ebs_volume`/`aws_volume_attachment`/`aws_ebs_snapshot`](#aws_ebs_volumeaws_volume_attachmentaws_ebs_snapshot)
    - [Scaling](#scaling)
      - [`aws_lb`/`aws_lb_target_group`/`aws_lb_target_group_attachment`](#aws_lbaws_lb_target_groupaws_lb_target_group_attachment)
      - [`aws_launch_template`/`aws_autoscaling_group`](#aws_launch_templateaws_autoscaling_group)

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

## HCL

### Variables

Environment variables can be used by TF by prefixing them with `TF_VAR_`.

### Predefined variables

```hcl
path.module     # path of the current module
```

### Strings

```hcl
  # Indented heredoc (`<<-`). The closing token doesn't need to be at the beginning of the line;
  # the left margin is automatically computed from the string.
  #
  user_data = <<-EOF
    #!/bin/bash

    set -o errexit

    apt update
    apt install --yes apache2

    echo "<h1>Hello World from $(hostname -f)</h1>" > /var/www/html/index.html
  EOF
```

### Functions

```hcl
# String

base64encode(string)

# File

file(filename)       # read a file into a string
filebase64(filename) # read a file into a string, converting it to base64
```

## State operations

```sh
# List resources
state list

# Import a resource
import -state-out=terraform.tfstate aws_iam_user.saverio saverio

# Remove a resource from the statefile
state rm aws_iam_user.saverio saverio
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

The import format is be `terraform $resource_type.$local_name $resource_reference`; the standard for the reference is the resource_id, where otherwise, it's specified.

### IAM

#### `aws_iam_group`/`aws_iam_group_policy_attachment`

```hcl
# import ref.: name
#
resource "aws_iam_group" "developers" {
  name = "developers"
}

# import ref.: "group.name/policy_arn"
#
resource "aws_iam_group_policy_attachment" "test-attach" {
  group      = aws_iam_group.group.name
  policy_arn = aws_iam_policy.policy.arn
}
```

#### `aws_iam_user`/`aws_iam_user_group_membership`

The membership is non-exclusive - a user can have multiple resources.

```hcl
# import ref.: name
#
resource "aws_iam_user" "lb" {
  name = "loadbalancer"
}

# import ref.: "user.name/group.name{/groupN.name}"
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
# import ref.: `iam-account-password-policy`
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

#### `aws_iam_role`/`aws_iam_role_policy_attachment`

The attachment is non-exclusive.

```hcl
# import ref.: name
#
resource "aws_iam_role" "demo_role" {
  name = "DemoRoleForEC2"

  assume_role_policy = <<-EOF
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

# import ref.: "demo_role.name/policy_arn"
#
resource "aws_iam_role_policy_attachment" "demo-attach" {
  role       = aws_iam_role.demo_role.name
  policy_arn = "arn:aws:iam::aws:policy/IAMReadOnlyAccess"
}
```

### Account-related

#### `aws_budgets_budget`

```hcl
# import ref.: "account_id:name"
#
resource "aws_budgets_budget" "ec2" {
  name              = "Monthly EC2 budget"
  limit_amount      = "10.0"
  limit_unit        = "USD"
  time_unit         = "MONTHLY"
  time_period_start = "2020-07-01_00:00"
  time_period_end   = "2087-06-15_00:00"
  budget_type       = "COST"

  cost_filters = {
    Service = "Amazon Elastic Compute Cloud - Compute"
  }

  # Can be defined multiple times, e.g. actual and forecasted.
  #
  notification {
    comparison_operator = "GREATER_THAN"
    notification_type   = "FORECASTED" # or ACTUAL
    subscriber_email_addresses = [
      "user@email.com",
    ]
    subscriber_sns_topic_arns = []
    threshold                 = 80
    threshold_type            = "PERCENTAGE"
  }
}
```

### Networking

#### `aws_vpc`/`aws_subnet`

```hcl
resource "aws_vpc" "main" {
  cidr_block = "123.456.0.0/16"
}

resource "aws_subnet" "main" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "123.456.0.0/24"
  availability_zone = "eu-central-1a" # optional
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id

  # In the console, AWS forces this.
  #
  tags = {
    Name = "gateway"
  }
}

# There is an alternative route resource; the below is the inline version.
#
resource "aws_route_table" "routes" {
  vpc_id = aws_vpc.main.id

  # Multiple routes can be defined; the local route is implicit, and can't be specified.
  #
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
}
```

#### `aws_security_group`/`aws_security_group_rule`

```hcl
# import aws_security_group.main $sg_id

resource "aws_security_group" "main" {
  name        = "launch-wizard-1"
  description = "launch-wizard-1 created 2020-07-22T10:10:25.013+02:00"
  vpc_id      = aws_vpc.main.id

  # rules can be specified inline, as `ingress {}` and `egress {}`, however, they don't support the
  # local name, so, if one wants to directly map existing rule resources, they need to use specific
  # resources (see below).
}

# for ipv6, replace `cidr_blocks` with `ipv6_cidr_blocks`.
#
# import aws_security_group_rule.main-ingress-http $sgrule_id
#
resource "aws_security_group_rule" "main-ingress-http" {
  cidr_blocks = [
    "0.0.0.0/0"
  ]
  from_port         = 80
  protocol          = "tcp"
  security_group_id = aws_security_group.main.id
  to_port           = 80
  type              = "ingress"
}

resource "aws_security_group_rule" "main-egress" {
  cidr_blocks = [
    "0.0.0.0/0",
  ]
  from_port         = 0
  protocol          = "-1"
  security_group_id = aws_security_group.main.id
  to_port           = 0
  type              = "egress"
}
```

### EC2 and disks

#### `aws_instance`

```hcl
# smterraform import aws_instance.first_instance $instance_id
#
resource "aws_instance" "first_instance" {
  ami           = "ami-079024c517d22af5b" # Current Ubuntu 20.04 x86-64
  instance_type = "t2.micro"
  # availability_zone = ...

  # The "Auto-assign Public IP" is not captured by TF.
  #
  subnet_id = aws_subnet.main.id
  vpc_security_group_ids = [
    aws_security_group.main.id,
  ]
  associate_public_ip_address = true # if required, and not set in the SG

  # Changes force replacement.
  #
  key_name = "donald"

  # Attach a role.
  #
  iam_instance_profile = aws_iam_role.demo_role.id

  # Changes force replacement. Don't forget to add a terminating newline if/when creating the
  # instance via console.
  #
  user_data = <<-EOF
    #!/bin/bash

    set -o errexit

    apt update
    apt install --yes apache2

    echo "<h1>Hello World from $(hostname -f)</h1>" > /var/www/html/index.html
  EOF
}
```

#### `aws_ebs_volume`/`aws_volume_attachment`/`aws_ebs_snapshot`

```hcl
# import aws_ebs_volume.first_instance_root $volume_id
#
resource "aws_ebs_volume" "first_instance_extra" {
  availability_zone = aws_instance.first_instance.availability_zone
  size              = 2
}

# There are important gotchas when working with volumes:
#
# - root volumes are tied to the EC2 instance lifecycle;
# - importing partition attachments (like root volumes) requires the partition device, which is
#   invalid for creation;
# - a range of `/dev/sdN` devices can't be used (see https://git.io/JJlmj).
#
# import aws_volume_attachment.first_instance_extra $device_name:$volume_id:$instance_id
#
resource "aws_volume_attachment" "first_instance_extra" {
  device_name = "/dev/sdf"
  volume_id   = aws_ebs_volume.first_instance_extra.id
  instance_id = aws_instance.first_instance.id
}

# import aws_ebs_snapshot.first_instance_extra $snapshot_id
#
resource "aws_ebs_snapshot" "first_instance_extra" {
  volume_id = aws_ebs_volume.first_instance_extra.id
}
```

### Scaling

#### `aws_lb`/`aws_lb_target_group`/`aws_lb_target_group_attachment`

```hcl
# import ref.: arn
#
resource "aws_lb" "lb" {
  name               = "lb"
  internal           = false         # defaults to false
  load_balancer_type = "application" # "application" (default)  or "network"
  security_groups    = [aws_security_group.main.id]
  subnets = [
    aws_subnet.main.id,
    aws_subnet.main2.id,
  ]
}

# import ref: arn
#
resource "aws_lb_target_group" "lb-tg" {
  name     = "lb-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id
}

# Can't be imported! When creating, existing matching ones will be replaced.
#
resource "aws_lb_target_group_attachment" "first_instance" {
  target_group_arn = aws_lb_target_group.lb-tg.arn
  target_id        = aws_instance.first_instance.id
  port             = 80
}

resource "aws_lb_target_group_attachment" "second_instance" {
  target_group_arn = aws_lb_target_group.lb-tg.arn
  target_id        = aws_instance.second_instance.id
  port             = 80
}
```

#### `aws_launch_template`/`aws_autoscaling_group`

```hcl
resource "aws_launch_template" "asg-template" {
  name                   = "ASGTemplate"
  update_default_version = false # don't bother with versioning

  image_id      = "ami-079024c517d22af5b"
  instance_type = "t2.micro"

  vpc_security_group_ids = [
    aws_security_group.main.id
  ]
  network_interfaces {
    associate_public_ip_address = true
  }

  key_name = "saverio"

  iam_instance_profile {
    name = aws_iam_role.demo_role.name
  }

  # Must be base64 (!).
  #
  user_data = filebase64("${path.module}/ec2_instances_user_data.sh")
}

# import ref.: name
#
resource "aws_autoscaling_group" "asg" {
  name                      = "DemoASG"
  desired_capacity          = 2
  min_size                  = 1
  max_size                  = 4
  health_check_type         = "ELB"
  health_check_grace_period = 300
  launch_template {
    id      = aws_launch_template.asg-template.id
    version = "$Default" # or $Latest
  }
  vpc_zone_identifier = [
    aws_subnet.main.id,
    aws_subnet.main2.id,
  ]

  # TF will add the following, even if not specified (here and in the console).
  #
  force_delete              = false
  wait_for_capacity_timeout = "10m"
}
```
