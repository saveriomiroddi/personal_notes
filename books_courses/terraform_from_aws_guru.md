# Terraform From An Aws Guru

- [Terraform From An Aws Guru](#terraform-from-an-aws-guru)
  - [References](#references)
  - [8. First Terraform file](#8-first-terraform-file)
  - [12. CLI and Common Executions](#12-cli-and-common-executions)
  - [13. The Basics: Resources](#13-the-basics-resources)
  - [14. Providers](#14-providers)
  - [15. The Basics: Variables](#15-the-basics-variables)
  - [16. Outputting Attributes](#16-outputting-attributes)
  - [17. First Exercise](#17-first-exercise)
  - [20. Interpolation Expressions](#20-interpolation-expressions)
    - [Terraform console](#terraform-console)
  - [21. Locals](#21-locals)
  - [22. Data sources](#22-data-sources)
  - [23. Modules](#23-modules)
  - [24. Backends and Remote States](#24-backends-and-remote-states)
  - [25. Workspaces](#25-workspaces)
  - [26. Software Provisioning with Provisioners - Part 1](#26-software-provisioning-with-provisioners---part-1)
  - [27. Software Provisioning with Provisioners - Part 2](#27-software-provisioning-with-provisioners---part-2)
  - [29. Review](#29-review)
  - [30. Terraform with AWS: Overview](#30-terraform-with-aws-overview)
  - [31. AWS VPCs](#31-aws-vpcs)
  - [32. Elastic Block Store (EBS)](#32-elastic-block-store-ebs)
  - [33. Identity & Access Management (IAM)](#33-identity--access-management-iam)
  - [34. Route 53](#34-route-53)
  - [35. Autoscaling](#35-autoscaling)
  - [36. Amazon Relational Database Service (RDS)](#36-amazon-relational-database-service-rds)
  - [37. Amazon Elastic Load Balancing (ELB)](#37-amazon-elastic-load-balancing-elb)
  - [38. Lambda](#38-lambda)
  - [39. Terraform with AWS: Real-Life Examples](#39-terraform-with-aws-real-life-examples)

## References

Reference repository: git@github.com:/LevelUpEducation/terraform-tutorial

## 8. First Terraform file

Basic example:

```
provider "aws" {
  access_key = "Access Key"
  secret_key = "Secrey Key"
  region     = "us-east-1"
}
resource "aws_instance" "example" {
  ami           = "ami-2757f631"
  instance_type = "t2.micro"
}
```

```sh
terraform init    # Init terraform
terraform plan    # Show plan
terraform apply   # Execute plan (will create `terraform.tfstate`)
terraform show    # Show current state
terraform destroy # Destroy
```

## 12. CLI and Common Executions

```sh
terraform plan -out=<planfile.tfplan>         # output the plan to the file
terraform apply [--destroy] <planfile.tfplan> # apply the plan from the file; use `--destroy` for destruction
terraform refresh                             # ???
terraform validate                            # validate the the plan file
```

## 13. The Basics: Resources

Meta parameters

```ruby
# count
resource "aws_instance" "backend" {
  count         = 2
  ami           = "ami-abcdef"
  instance_type = "t2.micro"
}

# depends_on
resource "aws_instance" "frontend" {
  depends_on    = ["aws_instance.backend"]
  ami           = "ami-abcdef"
  instance_type = "t2.micro"
}

# life cycle: create_before_destroy (useful for replacement)
lifecycle {
  create_before_destroy = true
}

# prevent_destroy
lifecycle {
  prevent_destroy = true
}
```

Timeouts (cause errors and stop process):

```ruby
timeouts {
  create = "60m"
  delete = "2h"
}
```

## 14. Providers

Providers are implicitly created by resources, unless they're explicitly defined.

```ruby
provider "aws" {
  # ... east-1
}

provider "aws" {
  # ... west-1
  alias      = "us-west-1"
}

# no provider: uses the default; it can also be manually specified as `provider = "aws"`
resource "aws_instance" "us_east_example" {
  ami           = "ami-66506c1c"
  instance_type = "t2.micro"
}

# uses the aliased provider!
resource "aws_instance" "us_west_example" {
  provider      = "aws.us-west-1"
  ami           = "ami-07585467"
  instance_type = "t2.micro"
}
```

## 15. The Basics: Variables

Types: string, boolean, list, map.

Default and description are optional.

```ruby
variable "mylist" {
  type = "list"
  default = ["abc", "cde"]
  description = "the description"
}

variable "mymap" {
  type = "map"
  default = {
    "abc" = "cde"
    "def" = "xyz"
  }
  description = "the description"
}

variable mybool {
  type = "boolean"
  default = false
}
```

Setting variables runtime:

```sh
$ TF_VAR_mystring=abc terraform apply                         # sets the variable `mystring`
$ terraform apply -var-file=foo.tfvars -var-file=foo.2.tfvars # reads the variables from the file(s), in YAML format
$ terraform plan -var-file=foo.tfvars -var-file=foo.2.tfvars # reads the variables from the file(s), in YAML format
```

Usage:

```ruby
variable "zones" {
  default = ["us-east-1a", "us-east-1b"]
}

resource "aws_instance" "example" {
  count                 = 2
  availability_zone     = "${var.zones[count.index]}" # `index` will be 0 on the first iteration, and 1 on the second
  ami                   = "ami-07585467"
  instance_type         = "t2.micro"
}
```

Note the so-called "interpolation expression" (`${<expression>}`) in the `availability_zone` value.

Setting variables via env variables (will override the one defined in the file):

```sh
TF_VAR_zones='["us-east-1d", "us-east-1c"]' terraform plan
terraform apply -var 'zones=["us-east-1d", "us-east-1c"]'
```

## 16. Outputting Attributes

Outputs, at the end of the run, the desired values. General format:

```ruby
# Each output block defines one output value.
output NAME {
  value = VALUE # string. list or map
  # description [string] (optional)
  # depends_on [list of strings] (optional)
  # sensitive [boolean] (optional)
}
```

Example:

```ruby
output "backend_ips" {
  value = "${aws_instance.backend.*.public_ip}"
}
```

The above will print the `public_ip` value of all the `aws_instance.backend` resources.

## 17. First Exercise

```ruby
provider "aws" {
  access_key = "..."
  secret_key = "..."
  region     = "us-east-1"
  alias      = "east"
}

provider "aws" {
  access_key = "..."
  secret_key = "..."
  region     = "us-west-1"
  alias      = "west"
}

variable "east_zones" {
  default = ["us-east-1a", "us-east-1b"]
}

variable "west_zones" {
  default = ["us-west-1a", "us-west-1b"]
}

resource "aws_instance" "east_frontend" {
  provider              = "aws.east"
  count                 = 2
  availability_zone     = "${var.east_zones[count.index]}"
  ami                   = "ami-66506c1c"
  instance_type         = "t1.micro"

  depends_on    = ["aws_instance.east_backend"]

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_instance" "west_frontend" {
  provider              = "aws.west"
  count                 = 2
  availability_zone     = "${var.west_zones[count.index]}"
  ami                   = "ami-07585467"
  instance_type         = "t1.micro"

  depends_on    = ["aws_instance.west_backend"]

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_instance" "east_backend" {
  provider              = "aws.east"
  count                 = 2
  availability_zone     = "${var.east_zones[count.index]}"
  ami                   = "ami-66506c1c"
  instance_type         = "t1.micro"

  lifecycle {
    prevent_destroy = true
  }
}

resource "aws_instance" "west_backend" {
  provider              = "aws.west"
  count                 = 2
  availability_zone     = "${var.west_zones[count.index]}"
  ami                   = "ami-07585467"
  instance_type         = "t1.micro"

  lifecycle {
    prevent_destroy = true
  }
}

output "east_backend_ips" {
  value = "${aws_instance.east_backend.*.public_ip}"
}

output "west_backend_ips" {
  value = "${aws_instance.west_backend.*.public_ip}"
}
```

## 20. Interpolation Expressions

Supported expressions (to use inside `${}`)

```
self.<attribute>               # reference a resource's own attribute

<reference> ? <val1> : <val2>  # conditional (ternary operator)

concat(<list1>{, <listN>})     # concatenate lists
join(<string>, <list>)
contains(<list>, <value>)      # return true if a list contains a certain value
merge(<map1>{, <mapN>})        # merge maps
length(<collection>)

split(<string>, <separator>)
replace(<string>, <search>, <replace>)
```

Instantiate collections:

```
list("a", "b")
map("key1", "value1"{, "keyN", "valueN"})
```

See the [official reference](https://www.terraform.io/docs/configuration/interpolation.html).

See `3b_Interpolations_expressions/expressions.tf` for a realistic example.

Example:

```ruby
variable "multi-region-deployment" {
  default = true
}

resource "aws_instance" "west_frontend" {
  tags = {
    Name = "${join("-",list(var.environment-name, "frontend"))}"
  }

  count             = "${var.multi-region-deployment ? 1 : 0}"
  depends_on        = ["aws_instance.west_backend"]
  provider          = "aws.us-west-1"
  ami               = "ami-07585467"
  availability_zone = "${var.us-west-zones[count.index]}"
  instance_type     = "t2.micro"
}
```

Notable: count (+reference), ternary operator, implicit form list, join(), array reference

### Terraform console

```sh
terraform console [options] [dir] # also reads from stdin
```

## 21. Locals

Locals allow more complex expressions logic, as we can't use expressions in variables:

```ruby
locals {
  default_name_prefix = "${var.project_name}-web"
  name_prefix = "${var.name_prefix != "" ? var.name_prefix: default_name_prefix}"
}

resource "aws_s3_bucket" "files" {
  bucket = "${local.name_prefix}-files"
}
```

## 22. Data sources

Give access to data that is not defined in the Terraform configuration.

```ruby
data "aws_ip_ranges" "european_ec2" {
  regions = ["eu-west-1", "eu-central-1"]
  services = ["ec2"]
}

data "template_file" "setup" {
  template = "${file("config/setup.sh")}"
}
```

## 23. Modules

Modules are self-contained packages of configurations that are managed as a group.

They can be:

- local file paths;
- terraform registry (community modules);
- online source repositories (need to have `git` installed)
- http urls
- s3 buckets

Define a directory `<modulename>`, with the content of a conventional configuration group, eg:

```
aws_instances
├── main.tf
├── outputs.tf
├── providers.tf
└── variables.tf
```

then reference it in the parent, using the module definition:

```
module "backend" {
  source = "./aws_instances"
  total_instances = 2
  aws_access_key = "${var.aws_access_key}"
  aws_secret_key = "${var.aws_secret_key}"
}
```

making sure that the required variables are set.

## 24. Backends and Remote States

Backends: how (where) metadata is stored.

Remote state: (backend feature) store metadata in a remote location.

```ruby
backend "s3" {
  bucket = "ticketsolve-terraform"
  key    = "network/terraform.tfstate"
  region = "eu-west-1"
}
```

Credentials files! Create a `.aws` directory with the following files.

`config`:

```
[default]
region = us-east-1

[profile caylentProd]
region = us-east-1
```

`credentials`:

```
[caylentProd]
aws_access_key_id = my_access_key
aws_secret_access_key = my_secret_key

[default]
aws_access_key_id = my_access_key
aws_secret_access_key = my_secret_key
```

then set, if required, the env variable: `export AWS_PROFILE=caylentProd`.

## 25. Workspaces

Workspace: named container for Terraform states. There is always at least one, called `default`.

Operations:

```sh
terraform workspace new <workspace_name>
terraform workspace list
terraform workspace select <workspace_name>
terraform workspace delete <workspace_name>
```

Workspaces can be addressed via:

```ruby
${terraform.workspace}
```

and can be linked together (?) with `terraform_remote_state data source`

Their data is under `terraform.tfstate.d/<workspace_name>/`.

## 26. Software Provisioning with Provisioners - Part 1

```ruby
resource "aws_instance" "web" {
  # Configure the connection used by the provisioner
  connection {
    type = "ssh"
    port = "${var.ssh_port}"
    host = "${self.private_ip}"
    user = "ubuntu"
    timeout = "2m"
  }

  # Provisioners need to be inside a `resource` block.
  #
  provisioner "file" {
    source = "./scripts/EC2_APP_SERVER_STARTUP.sh"
    # content = "<script_content>" # alternative to `source`
    destination = "/tmp/STARTUP.sh"
  }

  provisioner "remote-exec" {
    inline = [
      "sudo chmod +x /tmp/STARTUP.sh",
      "sudo /tmp/STARTUP.sh ${self.instance_type} ${var.stack_number} web${count.index}-${var.stack_number}"
    ]
  }
}
```

If a script fails, the instance is marked as tainted (on the following terraform run, the instance will be destroyed and rebuilt).

## 27. Software Provisioning with Provisioners - Part 2

(Ansible - no notes taken)

Manually taint a resource:

```sh
terraform taint null_resource.ansible_main
```

will cause ansible to reexecute.

## 29. Review

(no notes)

## 30. Terraform with AWS: Overview

(no notes)

## 31. AWS VPCs

NACLs:

- Osi layer 2
- Stateless
- Control inbound/outbound traffic at subnet level
- Order list

Security groups:

- Osi layer 3+
- Act as a firewall for EC2 instances
- Control traffic by ports
- Control traffic by TCP/UDP

Sample of VPC configuration:

```ruby
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"

  name = "complete-example"

  cidr = "10.10.0.0/16"

  azs             = ["eu-west-1b", "eu-west-1c"]
  public_subnets  = ["10.10.1.0/24", "10.10.2.0/24"]
  private_subnets = ["10.10.3.0/24", "10.10.4.0/24"]

  enable_nat_gateway = true

  tags = {
    Owner       = "user"
    Environment = "terraform.workspace"
    Name        = "teffaform vpc"
  }
}
```

See registry.terraform.io for the `vpc` module.

## 32. Elastic Block Store (EBS)

```ruby
resource "aws_volume_attachment" "my_ebs_attachment" {
  device_name = "/dev/sdb"
  volume_id = "aws_ebs_volume.my_ebs_volume.id"
  instance_id = "${aws_instance.my_ec2_instance.id}"
}

resource "aws_ebs_volume" "my_ebs_volume" {
  availability_zone = "us-west-2a"
  size              = 100 # GB
}

resource "aws_instance" "my_ec2_instance" {
  # ...
}
```

## 33. Identity & Access Management (IAM)

IAM Roles also allow users to have temporary access to resources.

Policies are the concept used to assign permissions (to a principal (user or resource)).
- there different types: predefined and custom
- on each request, the policies are evaluated

IAM Groups aren't principals, and can't be identified in a permission policy (???).

```ruby
resource "aws_iam_user" "my_user" {
  name          = "my-user-name"
  path          = "/"
  force_destroy = "false" # destroy even if it has non-terraform managed IAM access key/login profile/MFA devices
}

resource "aws_iam_group_membership" "my_team" {
  name = "my-team-name"

  users = [
    "${aws_iam_user.my_user.name}",
  ]

  group = "${aws_iam_group.my_group.name}"
}

resource "aws_iam_group" "my_group" {
  name = "my-group-name"
}

resource "aws_iam_group_policy" "my_policy" {
  name  = "my-policy-name"
  group = "${aws_iam_group.my_group.id}"

  policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "ec2:Describe*"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
EOF
}
```

## 34. Route 53

- CNAME: map a name to another name; can't have a CNAME on zone apex
- Alias: map a name to a resource dynamically (not a fixed ip); can map zone apex to ELB

```ruby
resource "aws_route53_zone" "primary" {
  name = "my-example.com"
}

resource "aws_route53_record" "www" {
  zone_id = "${aws_route53_zone.primary.zone_id}"
  name    = "www.my-example.com"
  type    = "A"
  ttl     = "300"
  records = ["198.40.6.7"]
}
```

## 35. Autoscaling

Requires two resources:

- launch config: properties of the instances to launch
- policies: when/how instances are launched

Autoscaling group:

- can be of fixed size: if any instance fails, a new one will be launches
- of variable size

```ruby
module "example_asg" {
  source = "terraform-aws-modules/autoscaling/aws"

  name = "example-with-elb"

  # Launch configuration
  #
  # launch_configuration = "my-existing-launch-configuration" # Use the existing launch configuration
  # create_lc = false # disables creation of launch configuration
  lc_name = "example-lc"

  image_id        = "${data.aws_ami.amazon_linux.id}"
  instance_type   = "t2.micro"
  security_groups = ["${data.aws_security_group.default.id}"]
  load_balancers  = ["${module.elb.this_elb_id}"]

  ebs_block_device = [
    {
      device_name           = "/dev/xvdz"
      volume_type           = "gp2"
      volume_size           = "50"
      delete_on_termination = true
    },
  ]

  root_block_device = [
    {
      volume_size = "50"
      volume_type = "gp2"
    },
  ]

  # Auto scaling group
  asg_name                  = "example-asg"
  vpc_zone_identifier       = ["${data.aws_subnet_ids.all.ids}"]
  health_check_type         = "EC2"
  min_size                  = 0
  max_size                  = 1
  desired_capacity          = 1
  wait_for_capacity_timeout = 0

  tags = [
    {
      key                 = "Mykey"
      value               = "Myvalue"
      propagate_at_launch = true
    },
  ]
}
```

## 36. Amazon Relational Database Service (RDS)

During the maintenance window, the instance is not brought down - it just has a higher load.

## 37. Amazon Elastic Load Balancing (ELB)

We can balance the traffic at layer 7 (Application; HTTP protocol definition) and 4 (Transport; TCP)

Network:

- operates at connection level
- uses static IPS
- can handle millions req/s with low latencies

Application:

- operates at request level
- can provide TLS termination
- advanced routing

"Classic" LB is obsolete.
ELBs can be internal (!).

Example of Application LB:

```ruby
resource "aws_lb" "front_end" {
  name               = "front-end-lb-tf"
  internal           = false
  load_balancer_type = "application"
  security_groups    = ["${data.aws_security_group.default.id}"]

  subnets = ["${data.aws_subnet_ids.all.ids}"]

  enable_deletion_protection = false

  tags {
    Environment = "production"
  }
}

resource "aws_lb_target_group" "front_end" {
  name     = "tf-example-lb-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = "${data.aws_vpc.default.id}"
}

resource "aws_lb_target_group_attachment" "front_end" {
  target_group_arn = "${aws_lb_target_group.front_end.arn}"
  target_id        = "${element(module.ec2_instances.id, 0)}"
  port             = 80
}

resource "aws_lb_listener" "front_end" {
  load_balancer_arn = "${aws_lb.front_end.arn}"
  port              = "80"
  protocol          = "HTTP"

  default_action {
    target_group_arn = "${aws_lb_target_group.front_end.arn}"
    type             = "forward"
  }
}
```

## 38. Lambda

```ruby
provider "archive" {}

data "archive_file" "zip" {
  type        = "zip"
  source_file = "hello_lambda.py"
  output_path = "hello_lambda.zip"
}

data "aws_iam_policy_document" "policy" {
  statement {
    sid    = ""
    effect = "Allow"

    principals {
      identifiers = ["lambda.amazonaws.com"]
      type        = "Service"
    }

    actions = ["sts:AssumeRole"]
  }
}

resource "aws_iam_role" "iam_for_lambda" {
  name               = "iam_for_lambda"
  assume_role_policy = "${data.aws_iam_policy_document.policy.json}"
}

resource "aws_lambda_function" "lambda" {
  function_name = "hello_lambda"

  filename         = "${data.archive_file.zip.output_path}"
  source_code_hash = "${data.archive_file.zip.output_sha}"

  role    = "${aws_iam_role.iam_for_lambda.arn}"
  handler = "hello_lambda.lambda_handler"
  runtime = "python3.6"

  environment {
    variables = {
      greeting = "Hello"
    }
  }
}
```

The lambda function is:

```python
import os

def lambda_handler(event, context):
    return "{} from Lambda!".format(os.environ['greeting'])
```

## 39. Terraform with AWS: Real-Life Examples

(see directory for full infrastructure example)
