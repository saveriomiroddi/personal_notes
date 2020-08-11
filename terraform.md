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
    - [Key pair](#key-pair)
    - [IAM](#iam)
      - [`aws_iam_group`/`aws_iam_group_policy_attachment`](#aws_iam_groupaws_iam_group_policy_attachment)
      - [`aws_iam_user`/`aws_iam_user_group_membership`](#aws_iam_useraws_iam_user_group_membership)
      - [`aws_iam_account_password_policy`](#aws_iam_account_password_policy)
      - [`aws_iam_role`/`aws_iam_role_policy_attachment`/`aws_iam_policy`](#aws_iam_roleaws_iam_role_policy_attachmentaws_iam_policy)
    - [KMS/Secrets Manager](#kmssecrets-manager)
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
    - [S3](#s3)
      - [Bucket/properties/policies (many)](#bucketpropertiespolicies-many)
      - [Objects](#objects)
    - [Lambda](#lambda)
    - [Lightsail](#lightsail)
    - [DNS: `aws_route53_zone`](#dns-aws_route53_zone)
    - [Cloudfront: `aws_cloudfront_distribution`, `aws_cloudfront_origin_access_identity`](#cloudfront-aws_cloudfront_distribution-aws_cloudfront_origin_access_identity)
    - [SNS](#sns)
    - [Cloudwatch](#cloudwatch)
      - [Alarms](#alarms)
      - [Logs](#logs)
      - [Events](#events)
    - [CloudTrail](#cloudtrail)
    - [RDS](#rds)

## Base configuration

Provider configuration (e.g. `terraform.tf`):

```hcl
terraform {
  required_version = "0.12.28"
}

provider "aws" {
  region     = var.aws_default_region
  access_key = var.aws_access_key_id
  secret_key = var.aws_secret_access_key
}

# Alternate provider, for other region. Don't forget to set the keys.
#
provider "aws" {
  alias  = "eu-west-2"
  region = "eu-west-2"
  access_key = var.aws_access_key_id
  secret_key = var.aws_secret_access_key
}

# Conveniences for referencing the region. Reference as: `data.aws_region.eu-west-2.name`
#
data "aws_region" "eu-central-1" {
  provider = aws
}

data "aws_region" "eu-west-2" {
  provider = aws.eu-west-2
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

Create a resource in the alternate region:

```hcl
resource "aws_instance" "instance_london" {
  provider = aws.eu-west-2
  // [...]
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
# Encodings

base64encode(string)
jsonencode(object)   # can encode maps!
jsondecode(string)

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

### Key pair

Key pairs are per-region!

```hcl
resource "aws_key_pair" "deployer" {
  key_name   = "deployer-key"
  public_key = "ssh-rsa [...] email@example.com"
}
```

### IAM

#### `aws_iam_group`/`aws_iam_group_policy_attachment`

```hcl
# Import ref.: name
#
resource "aws_iam_group" "developers" {
  name = "developers"
}

# Import ref.: "group.name/policy_arn"
#
resource "aws_iam_group_policy_attachment" "test-attach" {
  group      = aws_iam_group.group.name
  policy_arn = aws_iam_policy.policy.arn
}
```

#### `aws_iam_user`/`aws_iam_user_group_membership`

The membership is non-exclusive - a user can have multiple resources.

```hcl
# Import ref.: name
#
resource "aws_iam_user" "myuser" {
  name = "loadbalancer"
}

# Import ref.: "user.name/group.name{/groupN.name}"
#
resource "aws_iam_user_group_membership" "example" {
  user = aws_iam_user.myuser.name

  groups = [
    aws_iam_group.group1.name,
    aws_iam_group.group2.name,
  ]
}
```

#### `aws_iam_account_password_policy`

```hcl
# Import ref.: `iam-account-password-policy`
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

#### `aws_iam_role`/`aws_iam_role_policy_attachment`/`aws_iam_policy`

The attachment is non-exclusive.

Example, using predefined policy:

```hcl
# Import ref.: name
#
resource "aws_iam_role" "demo_role" {
  name = "DemoRoleForEC2"

  force_detach_policies = true # convenient; default: false

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

# Import ref.: "role.name/policy_arn"
#
resource "aws_iam_role_policy_attachment" "demo-attach" {
  role       = aws_iam_role.demo_role.name
  policy_arn = "arn:aws:iam::aws:policy/IAMReadOnlyAccess"
}
```

Custom policy:

```hcl
# Import ref.: arn
#
resource "aws_iam_policy" "lambda_example_logging" {
  name = "AWSLambdaBasicExecutionRole-6d128aa8-189b-42d2-aa07-9bb2e622c1cd"
  path = "/service-role/"

  # On automatic creation, AWS also adds this permission:
  #
  #   {
  #       "Effect": "Allow",
  #       "Action": "logs:CreateLogGroup",
  #       "Resource": "arn:aws:logs:<region>:<account_number>:*"
  #   },
  #
  # which in this case is not needed, since the log group is created by TF.
  #
  policy = <<-JSON
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "logs:CreateLogStream",
                    "logs:PutLogEvents"
                ],
                "Resource": [
                    "${aws_cloudwatch_log_group.lambda_example_log_group.arn}"
                ]
            }
        ]
    }
  JSON
}
```

### KMS/Secrets Manager

Customer managed key:

```hcl
# Import ref.: key_id
#
resource "aws_kms_key" "ebs-test" {}
```

Secrets manager secret:

```
locals {
  # These can be, for example, passed from the environment.
  #
  my_secret_values = {
    key1 = "value1",
    key2 = "value2",
  }
}

# Import ref.: arn (same as id)
#
resource "aws_secretsmanager_secret" "my_secret" {
  name = "my_secret" # optional

  recovery_window_in_days = 30 # Default

  # Optional; if not specified, the account's default CMK is used.
  #
  # kms_key_id = "..."
}

# Import: <secret_id>|<secret_version>
# Use CLI for finding the version.
#
resource "aws_secretsmanager_secret_version" "my_secret" {
  secret_id     = aws_secretsmanager_secret.my_secret.id
  secret_string = jsonencode(local.my_secret_values)
}
```

### Account-related

#### `aws_budgets_budget`

```hcl
# Import ref.: "account_id:name"
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

Networks are per-region!

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

# WATCH OUT! Since each VPC comes with a default routing table set as main, it's necessary to use
# the main association resource (below).
#
resource "aws_route_table" "internet" {
  vpc_id = aws_vpc.main.id

  # Inline version of the routes (alternative: use `aws_route` resources).
  # Multiple routes can be defined; the local route is implicit, and can't be specified.
  #
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
}

# Import not available.
#
resource "aws_main_route_table_association" "internet" {
  vpc_id         = aws_vpc.main.id
  route_table_id = aws_route_table.internet.id
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
  ami           = "ami-079024c517d22af5b" # Current Ubuntu 20.04 x86-64; per-region!
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
  encrypted         = true # required, if the key (below) is set
  kms_key_id        = aws_kms_key.ebs-test.arn
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
# Import ref.: arn
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

# Import ref: arn
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

# Import ref.: name
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

### S3

#### Bucket/properties/policies (many)

See the notes - there's much going on.

```hcl
locals {
  base_bucket_name = "sav986" # used to avoid cycles
}

# Current account id (`account_id`)
#
data "aws_caller_identity" "current" {}

# Current user id.
#
data "aws_canonical_user_id" "current" {}

# https://www.terraform.io/docs/providers/aws/r/s3_bucket.html
#
# Import ref.: bucket
#
resource "aws_s3_bucket" "sav-test" {
  bucket = local.base_bucket_name

  # TF forces these.
  #
  acl           = "private"
  force_destroy = true # default: false

  versioning {
    enabled = true
  }

  # Endpoint: http://sav986.s3-website.eu-central-1.amazonaws.com.
  # See note in the index object!
  #
  website {
    index_document = "index.html"
  }

  logging {
    bucket = "${aws_s3_bucket.sav-test.bucket}-logging"
    target_prefix = "log/"
  }

  lifecycle_rule {
    enabled = true # mandatory

    transition {
      # Classes: STANDARD, STANDARD_IA, ONEZONE_IA, INTELLIGENT_TIERING, GLACIER, DEEP_ARCHIVE
      #
      storage_class = "INTELLIGENT_TIERING"
      # days          = 30
    }
  }
}

resource "aws_s3_bucket" "sav-test-logging" {
  bucket = "${local.base_bucket_name}-logging"

  versioning {
    enabled = true
  }

  # Bucket ACLs required for setting a bucket for logging. Such grants are automatically set by AWS
  # when setting up logging via console.
  # Canonical id info: https://docs.aws.amazon.com/general/latest/gr/acct-identifiers.html.
  #
  grant {
    permissions = [
      "READ_ACP",
      "WRITE",
    ]
    type = "Group"
    uri  = "http://acs.amazonaws.com/groups/s3/LogDelivery"
  }
  grant {
    id = data.aws_canonical_user_id.current.id
    permissions = [
      "FULL_CONTROL",
    ]
    type = "CanonicalUser"
  }
}

# Import ref.: bucket
#
resource "aws_s3_bucket_public_access_block" "sav-test" {
  bucket = aws_s3_bucket.sav-test.bucket

  # AWS defaults all to true
  #
  block_public_acls       = false
  block_public_policy     = false
  ignore_public_acls      = false
  restrict_public_buckets = false
}

resource "aws_s3_bucket_policy" "sav-test-allow-get-objects" {
  bucket = aws_s3_bucket.sav-test.bucket

  # As of v0.12.28, TF doesn't correctly handle the resources:
  #
  #   aws_s3_bucket.sav-test: Creating...
  #   aws_s3_bucket.sav-test: Creation complete after 2s [id=sav986]
  #   aws_s3_bucket_policy.sav-test-allow-get-objects: Creating...
  #   [...]
  #   Error: Error putting S3 policy: OperationAborted: A conflicting conditional operation is currently in progress against this resource.
  #
  # see https://git.io/JJ8Tj.
  #
  depends_on = [aws_s3_bucket_public_access_block.sav-test]

  policy = <<-POLICY
    {
      "Id": "Policy1595497452733",
      "Version": "2012-10-17",
      "Statement": [
        {
          "Sid": "Stmt1595497451256",
          "Action": [
            "s3:GetObject"
          ],
          "Effect": "Allow",
          "Resource": "${aws_s3_bucket.sav-test.arn}/*",
          "Principal": "*"
        }
      ]
    }
  POLICY
}
```

#### Objects

```hcl
# Import currently not supported (see https://git.io/JJlhX)
#
resource "aws_s3_bucket_object" "coffee" {
  bucket = aws_s3_bucket.sav-test.bucket
  key    = "coffee.jpg"
  source = "s3/coffee.jpg"

  # This is the default.
  #
  # content_type = "binary/octet-stream"
}

# Important! Since this is the index, the `content_type` must be specified! Otherwise, the index
# will be downloaded instead of opened.
#
resource "aws_s3_bucket_object" "index" {
  bucket       = aws_s3_bucket.sav-test.bucket
  key          = "index.html"
  source       = "s3/index.html"
  content_type = "text/html"
}

resource "aws_s3_bucket_object" "images-dir" {
  bucket = aws_s3_bucket.sav-test.bucket
  key    = "images/"
  source = "/dev/null"
}

resource "aws_s3_bucket_object" "beach" {
  bucket = aws_s3_bucket.sav-test.bucket
  key    = "images/beach.jpg"
  source = "s3/images/beach.jpg"

  storage_class = "INTELLIGENT_TIERING"
}
```

### Lambda

```hcl
# Required role.
#
resource "aws_iam_role" "lambda-example" {
  name = "example-role-32t7sco4"
  path = "/service-role/"

  force_detach_policies = true

  assume_role_policy = <<-JSON
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Action": "sts:AssumeRole",
          "Principal": {
            "Service": "lambda.amazonaws.com"
          },
          "Effect": "Allow"
        }
      ]
    }
  JSON
}

# Import ref.: function_name
#
resource "aws_lambda_function" "example" {
  function_name = "example"
  role          = aws_iam_role.lambda-example.arn

  runtime          = "python3.7"
  filename         = "lambda/example.zip"
  handler          = "lambda_function.lambda_handler" # <file-name>.<method>
  source_code_hash = filebase64sha256("lambda/example.zip")

  # Optional params
  #
  timeout     = 3   # seconds
  memory_size = 128 # MB

  # Forced by TF
  #
  publish = false # default
}
```

The zip contains a `/lambda_function.py` file, which contains:

```python
def lambda_handler(event, context):
    print("value1 = " + event['key1'])
    return event['key1']  # Echo back the first key value
```

### Lightsail

```hcl
# Import ref: name (watch out the case!)
#
resource "aws_lightsail_instance" "wordpress-test" {
  name              = "WordPress"
  availability_zone = "eu-central-1a"
  blueprint_id      = "wordpress"
  bundle_id         = "nano_2_0"

  # Defaults given by Lightsail (for this blueprint)
  #
  key_pair_name = "LightsailDefaultKeyPair"
  username      = "bitnami"
}
```

### DNS: `aws_route53_zone`

When the DNS is managed by a 3rd party, create the zone, then set the DNS nameservers associated to the zone.

```hcl
resource "aws_route53_zone" "demo" {
  name = "64kram.systems"

  # Forced by TF.
  #
  comment       = "Managed by Terraform"
  force_destroy = false
}

# Import ref.: <zone_id>_<name>_<type>_<set_identifier>; example: `000000000000000000000_www.domain.com_A_eu-central-1`
#
resource "aws_route53_record" "www-eu-central-1" {
  zone_id        = aws_route53_zone.demo.id
  name           = "www.64kram.systems"
  set_identifier = "eu-central-1"
  type           = "A"
  ttl            = "300" # default
  latency_routing_policy {
    region = "eu-central-1"
  }
  records = [
    aws_instance.first_instance.public_ip
  ]
}
```

### Cloudfront: `aws_cloudfront_distribution`, `aws_cloudfront_origin_access_identity`

Cloudfront distribution + OAI:

```hcl
# Workaround to avoid self-reference. See https://www.terraform.io/docs/providers/aws/r/cloudfront_origin_access_identity.html#using-with-cloudfront.
#
resource "aws_cloudfront_origin_access_identity" "sav-test" {}

# In order to update/destroy the distribution, it needs to be disabled first, AND the operation
# needs to complete on the AWS side.
#
resource "aws_cloudfront_distribution" "sav-test" {
  # See the corresponding bucket policy.
  #
  origin {
    # Append the region to `s3`, in order to send requests to the (selected) closer regional
    # endpoint, in order to have a faster first propagation, e.g. `s3-eu-central-1`.
    #
    domain_name = "${aws_s3_bucket.sav-test-private.bucket}.s3.amazonaws.com"
    origin_id   = "S3-${aws_s3_bucket.sav-test-private.bucket}"

    s3_origin_config {
      # See resource-based workaround above
      #
      origin_access_identity = aws_cloudfront_origin_access_identity.sav-test.cloudfront_access_identity_path
    }
  }

  enabled         = true
  is_ipv6_enabled = true

  default_cache_behavior {
    allowed_methods  = ["GET", "HEAD"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "S3-${aws_s3_bucket.sav-test-private.bucket}"

    forwarded_values {
      query_string = false # default

      cookies {
        forward = "none" # default
      }
    }

    viewer_protocol_policy = "allow-all"
    default_ttl            = 0 # Default: 86400
    max_ttl                = 0 # Default: 31536000
  }

  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }

  viewer_certificate {
    cloudfront_default_certificate = true
  }
}
```

Bucket policy:

```hcl
# See https://www.terraform.io/docs/providers/aws/r/cloudfront_origin_access_identity.html#updating-your-bucket-policy.
#
# Import ref.: bucket
#
resource "aws_s3_bucket_policy" "sav-test-private-allow-cloudfront" {
  bucket = aws_s3_bucket.sav-test-private.bucket

  depends_on = [aws_s3_bucket_public_access_block.sav-test-private]

  policy = <<-POLICY
    {
        "Version": "2008-10-17",
        "Id": "PolicyForCloudFrontPrivateContent",
        "Statement": [
            {
                "Sid": "1",
                "Effect": "Allow",
                "Principal": {
                    "AWS": "${aws_cloudfront_origin_access_identity.sav-test.iam_arn}"
                },
                "Action": "s3:GetObject",
                "Resource": "${aws_s3_bucket.sav-test-private.arn}/*"
            }
        ]
    }
  POLICY
}
```

### SNS

```hcl
# Import ref.: arn
#
resource "aws_sns_topic" "system_alarms" {
  name = "SystemAlarms"
}

# Email subscriptions (`protocol = "email"`) are not supported (see https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/sns_topic_subscription#protocols-supported).
#
# Import ref.: arn
#
resource "aws_sns_topic_subscription" "system_alarms_email" {
  topic_arn = aws_sns_topic.system_alarms.arn

  protocol = "email"
  endpoint = "test-aws-system-alarms@mailinator.com"
}
```

### Cloudwatch

#### Alarms

```hcl
# Import ref.: <alarm_name>
#
resource "aws_cloudwatch_metric_alarm" "ec2_first_instance_cpu_utilization_alarm" {
  alarm_name = "EC2_First_Instance_CPUUtilizationAlarm"

  metric_name = "CPUUtilization"

  namespace = "AWS/EC2"

  dimensions = {
    "InstanceId" = aws_instance.first_instance.id
  }

  # `GreaterThanOrEqualToThreshold`, etc.
  #
  comparison_operator = "GreaterThanThreshold"

  threshold = 80

  alarm_actions = [
    aws_sns_topic.system_alarms.arn,
  ]

  evaluation_periods  = 1
  datapoints_to_alarm = 1
  period              = 300
  statistic           = "Average"
}

# Created via EC2 instances Monitoring tab.
#
resource "aws_cloudwatch_metric_alarm" "ec2_first_instance_system_status_check_failed" {
  alarm_name = "EC2_First_Instance_System_Status_Check_Failed"

  metric_name = "StatusCheckFailed_System"

  namespace = "AWS/EC2"

  dimensions = {
    "InstanceId" = aws_instance.first_instance.id
  }

  comparison_operator = "GreaterThanOrEqualToThreshold"

  threshold = 1

  alarm_actions = [
    "arn:aws:automate:eu-central-1:ec2:recover", # Recover the instance
    aws_sns_topic.system_alarms.arn,
  ]

  evaluation_periods = 2
  period             = 60
  statistic          = "Maximum"
  treat_missing_data = "missing"

  # Interestingly, 0 is the default when created via Monitoring tab, which is invalid for TF, which
  # requires at least 1.
  #
  datapoints_to_alarm = 1

  alarm_description = "Created from EC2 Console"
}

# Special billing alarms - needs to be in `us-east-1`.
#
resource "aws_cloudwatch_metric_alarm" "billing_alarm" {
  alarm_name = "BillingAlarm"
  provider   = aws.us-east-1

  metric_name = "EstimatedCharges"

  namespace = "AWS/Billing"

  dimensions = {
    "Currency" = "USD"
  }

  comparison_operator = "GreaterThanThreshold"

  threshold = 10

  alarm_actions = [
    aws_sns_topic.billing_alarms.arn,
  ]

  evaluation_periods = 1
  period             = 21600
  statistic          = "Maximum"
  treat_missing_data = "missing"

  # Interestingly, 0 is the default when created via Monitoring tab, which is invalid for TF, which
  # requires at least 1.
  #
  datapoints_to_alarm = 1
}
```

#### Logs

Log group:

```hcl
# Import ref.: name
#
resource "aws_cloudwatch_log_group" "lambda_example_log_group" {
  name = "/aws/lambda/${aws_lambda_function.example.function_name}"

  # Options: 1, 3, 5, 7, 14, 30, 60, 90, 120, 150, 180, 365, 400, 545, 731, 1827, 3653
  #
  retention_in_days = 7
}
```

#### Events

Scheduled:

```hcl
# Import ref.: name
#
resource "aws_cloudwatch_event_rule" "execute_example_lambda_every_hour" {
  name = "ExecuteExampleLambdaEveryHour"

  # Cron format: `cron(0 20 * * ? *)`
  #
  schedule_expression = "rate(1 hour)"
}

# Import ref.: rule.name/target_id (target_id requires awscli)
#
resource "aws_cloudwatch_event_target" "execute_lambda" {
  rule = aws_cloudwatch_event_rule.execute_example_lambda_every_hour.name
  arn  = aws_lambda_function.example.arn
}
```

With event pattern:

```hcl
resource "aws_cloudwatch_event_rule" "send_email_on_user_login" {
  name = "SendEmailOnUserLogin"

  event_pattern = <<-JSON
    {
        "detail-type": [
            "AWS Console Sign In via CloudTrail"
        ]
    }
  JSON
}

resource "aws_cloudwatch_event_target" "notify_on_login" {
  rule = aws_cloudwatch_event_rule.send_email_on_user_login.name
  arn  = aws_sns_topic.system_alarms.arn
}
```

### CloudTrail

Example. For the bucket, see the related project.

```hcl
locals {
  cloudtrail_bucket_name = "cloudtrail-demo-s999"
}

resource "aws_cloudtrail" "demo_trail" {
  name                       = "demo-trail"
  s3_bucket_name             = aws_s3_bucket.cloudtrail_demo.bucket
  is_multi_region_trail      = true
  enable_log_file_validation = true
}
```

### RDS

```hcl
# When an instance is created, the default (readonly) option and parameter groups are created for
# the major version, if they don't exist:
#
# - option g.: `default:mysql-8-0`
# - parameter g.: `default.mysql8.0`
#
# A subnet is group is created as well, and a subnet if requested.

# Import ref.: identifier
#
resource "aws_db_instance" "demo" {
  identifier = "database-1"

  engine         = "mysql"
  engine_version = "8.0.20"
  instance_class = "db.t2.micro"

  copy_tags_to_snapshot = true
  skip_final_snapshot   = true

  # The password is not required for an imported resource, however, if it's subsequently specified,
  # it will be marked as changed even if it matches.
  # Note the pwd is stored in the statefile!
  #
  # password = "password"
}

# Import ref.: name
#
resource "aws_db_subnet_group" "default" {
  name        = "default-vpc-0c9209e70c0813162"
  description = "Created from the RDS Management Console" # Console default

  subnet_ids = [
    aws_subnet.main.id,
    aws_subnet.main2.id
  ]
}

# AWS is inconsistent - the default PG has dots, which are actually disallowed.
# This PG can't be deleted, and can't be kept also if imported (due to naming), so it's left
# commented here only for reference.
#
# Import ref.: name
#
# resource "aws_db_parameter_group" "default" {
#   name = "default.mysql.80"
#
#   family = "mysql8.0"
#
#   parameter {
#     name  = "character_set_server"
#     value = "utf8"
#   }
# }

# Inconsistency here as well - colon not allowed.
#
# Import ref.: name
#
# resource "aws_db_option_group" "default" {
#   name                 = "defaultmysql-8-0"
#   engine_name          = "mysql"
#   major_engine_version = "8.0"
#
#   option {
#     option_name = "Timezone"
#
#     option_settings {
#       name  = "TIME_ZONE"
#       value = "UTC"
#     }
#   }
# }
```
