# Terraform Up And Running (1st ed.)

Modules sources can be versioned: `source = "git::git@github.com:foo/modules.git//frontend-app?ref=v0.0.3"`

Design: modules should be in a separate repository; configuration(s) in other(s)

Inline code should be avoided; use templates instead - note that the placeholder format (`${varname}`) used makes the templates valid shell scripts!

Automated testing should use a separate environments

The `aws_ami` data source supports automatic finding of the latest version!:

```
data "aws_ami" "ubuntu" {
  most_recent = true
  owners = ["099720109477"] # Canonical
  filter {
    name = "virtualization-type"
    values = ["hvm"]
  }
  filter {
    name = "architecture"
    values = ["x86_64"]
  }
  filter {
    name = "image-type"
    values = ["machine"]
  }
  filter {
    name = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-xenial-16.04-amd64-server-*"]
  }
}
```

Tools for automated testing to check out: `kitchen-terraform`, `serverspec`

The terraform vars file can also be in JSON format: `terraform.tfvars.json`

For the suggested structure, see pages 259 (unstructured) and 261 (structured)
