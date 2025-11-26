AWS instance(in terms of terraform)/ EC2 instance using ingress, egress, security groups:

----
provider "aws" {
  region = "us-east-1"
}

data "aws_vpc" "default" {
  default = true
}

data "aws_ssm_parameter" "my_instance" {
  name = "/aws/service/canonical/ubuntu/server/jammy/stable/current/amd64/hvm/ebs-gp2/ami-id"
}

resource "aws_instance" "example" {
  instance_type = "t3.micro"
  ami = data.aws_ssm_parameter.my_instance.value
  key_name = "new-aws-account"
    vpc_security_group_ids = [
    aws_security_group.app_sg.id]

  tags = {
    Name = "shellscript"
  }
}

  resource "aws_security_group" "app_sg" {
  name        = "app-sg"
  description = "Allow SSH and App traffic"
  vpc_id      = data.aws_vpc.default.id

  
  tags = {
    Name = "app-sg"
  }
  
}

resource "aws_vpc_security_group_ingress_rule" "allow_ssh" {
  security_group_id = aws_security_group.app_sg.id
  cidr_ipv4         = "0.0.0.0/0"   # Use your IP in real setups
  ip_protocol       = "tcp"
  from_port         = 22
  to_port           = 22
}

resource "aws_vpc_security_group_egress_rule" "allow_all_outbound" {
  security_group_id = aws_security_group.app_sg.id
  cidr_ipv4         = "0.0.0.0/0"
  ip_protocol       = "-1"
}

----
 
Explanation:

hcldata "aws_ami" "ubuntu" {

This tells Terraform: “Go fetch some information from AWS for me, but don’t create anything.”
data blocks are used to read/lookup existing resources or data in AWS.
aws_ami is a built-in data source that can find Amazon Machine Images (AMIs).
"ubuntu" is the name you give to this data source (you choose it, like a variable name). You’ll refer to it later as data.aws_ami.ubuntu.

hclmost_recent = true

Among all the matching Ubuntu AMIs, pick the newest one (based on creation date).
Without this, Terraform would error because it found multiple matches.

hclfilter {
    name = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }

This filter says: “Only look at AMIs whose name matches this pattern.”
ubuntu-jammy-22.04-amd64-server-* = official Ubuntu 22.04 LTS (Jammy Jellyfish) server images for 64-bit x86.
The * wildcard matches the date/build suffix (e.g., 20241001, 20241115, etc.).

hclfilter {
    name = "virtualization-type"
    values = ["hvm"]
  }

Only consider HVM (Hardware Virtual Machine) images.
Almost all modern EC2 instances use HVM (not the old “paravirtual” type).

hclowners = ["099720109477"] # Canonical

Only show AMIs published by Canonical (the official Ubuntu company).
Account ID 099720109477 belongs to Canonical. This prevents you from accidentally picking fake or modified Ubuntu images.

hcl}

End of the data "aws_ami" block.

hclresource "aws_instance" "example" {

Now we actually create something in AWS.
resource blocks create real infrastructure.
aws_instance tells Terraform to launch an EC2 virtual machine.
"example" is the logical name you give this instance inside Terraform.

hclami = data.aws_ami.ubuntu.id

Use the AMI ID that we found earlier with the data source.
data.aws_ami.ubuntu.id → automatically becomes something like ami-0abcdef1234567890.
This is the key line that connects the lookup with the actual instance.

hclinstance_type = "t3.micro"

Launch a t3.micro instance (cheap, 2 vCPU bursts, 1 GiB RAM, eligible for AWS free tier).

hcltags = {
    Name = "HelloWorld"
  }

Give the EC2 instance a friendly name tag that will appear in the AWS console as “HelloWorld”.

hcl}

End of the EC2 instance resource.

Summary – What this whole file does

Looks up the latest official Ubuntu 22.04 LTS AMI from Canonical.
Launches one t3.micro EC2 instance using that AMI.
Names the instance “HelloWorld” in the AWS console.

What you still need to add for it to actually work
This code alone is valid but incomplete. To run it you also need (at minimum):
hclterraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"   # or your preferred region
}
Plus a key pair, VPC/subnet, security group allowing SSH, etc., if you want to actually connect to it.
But the code you posted is the cleanest and most commonly used way to launch an up-to-date Ubuntu instance without hard-coding AMI IDs that go stale every few weeks. Great job!
