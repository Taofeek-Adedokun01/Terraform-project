# I Created AWS Provider in my Main.tf file to specify the region and also configure my AWS CLI on my terminal.
# Project Overview

This Terraform project creates AWS infrastructure for a hypothetical application.
# Project Overview

This Terraform project creates AWS infrastructure for a hypothetical application.

## AWS Provider Configuration

```hcl
provider "aws" {
    region = ""
    access_key = ""
    secret_key = ""
}

# I created aws_vpc and subnet inside the vpc with the code below
```hcl
resource "aws_vpc" "development-vpc" {
  cidr_block = "10.0.0.0/16"
}
```hcl
resource "aws_subnet" "dev-subnet-1" {
    vpc_id = aws_vpc.development-vpc.id
    cidr_block = "10.0.10.0/24"
    availability_zone = "eu-west-2a"
}

# I created output for the aws_vpc and the aws_subnet associated with the vpc with the code below

`output "aws-dev-vpc" {
    value = aws_vpc.development-vpc.id
}

output "aws-dev-subnet01" {
    value = aws_subnet.dev-subnet-1.id
}`

# I created data from the existing default vpc with the underlying code

`Data “aws_vpc” “existing-vpc” {
Default = true
}
Resource “aws_subnet” “dev-subnet-2” {
vpc_id = data.aws_vpc.existing-vpc.id
Cidr = “10.0.0.1/24”`

# I created variable for my subnet cidr and my aws_vpc tag names with the code below and defined it in my terraform.tfvars with the code below

`resource "aws_vpc" "development-vpc" {
  cidr_block = "10.0.0.0/16"
  tags = {
    name: var.dev_environment,
  }
}
resource "aws_subnet" "dev-subnet-1" {
    vpc_id = aws_vpc.development-vpc.id
    cidr_block = var.subnet_cidr_block 
    availability_zone = "eu-west-2a"
    tags = {
    name: "dev-subnet-01"
  }

}
variable "dev_environment"
    description = "development environment"
}
variable "subnet_cidr_block" {
    description = "subnet cidr block"
}`

# I created variable for my subnet cidr and my aws_vpc tag names with the code below and defined it in my terraform.tfvars with the code below

`subnet_cidr_block = "10.0.50.0/24"
dev_environment = "development"`

# I created list of strings of cidr_blocks for aws_vpc and aws_subnet with a single variable getting the 1st and 2nd cidr_blocks defined in the terraform.tfvars file with the code below.

`variable "cidr_blocks" {
    description = "cidr blocks for vpcs and subnets"
    type = list(string)
}
cidr_blocks = ["10.0.0.0/16", "10.0.10.0/24"]
`

`resource "aws_vpc" "development-vpc" {
  cidr_block = var.cidr_blocks[0]
  tags = {
    name: "development",
  }
}
resource "aws_subnet" "dev-subnet-1" {
    vpc_id = aws_vpc.development-vpc.id
    cidr_block = var.cidr_blocks[1]
    tags = {
    name: "dev-subnet-01"
  }

}`

# Store your accesskeys and secret keys in aws home directory for credentials with the command below

`aws configure`

# Set variable using TF environment variable

# Set availability zone as a global variable using "export TF_VAR_avail_zone="eu-west-2a"

`variable avail_zone {}`

`resource "aws_subnet" "dev-subnet-1" {
    vpc_id = aws_vpc.development-vpc.id
    cidr_block = var.cidr_blocks[1].cidr_block
    availability_zone = var.avail_zone
    tags = {
    name: var.cidr_blocks[1].name
  }`

# SECTION 2, I created new infrastructure starting with the custom vpc , subnet and giving it names with tags with the codes below

`provider "aws" {
    region = "eu-west-2"
}
variable avail_zone {}
variable subnet_cidr_block {}
variable vpc_cidr_block {}
variable env_prefix {}


resource "aws_vpc" "myapp-vpc" {
  cidr_block = var.vpc_cidr_block
  tags = {
    name: "${var.env_prefix}-vpc",
  }
}
resource "aws_subnet" "myapp-subnet-1" {
    vpc_id = aws_vpc.myapp-vpc.id
    cidr_block = var.subnet_cidr_block
    availability_zone = var.avail_zone
    tags = {
    name: "${var.env_prefix}-subnet-1",
  }
}
avail_zone = "eu-west-2b"
subnet_cidr_block = "10.0.10.0/24"
vpc_cidr_block = "10.0.0.0/16"
env_prefix = "dev"

# Creating new rtb and igw with the code below 

`resource "aws_route_table" "myapp-route-table" {
    vpc_id = aws_vpc.myapp-vpc.id
route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.myapp-igw.id
}
   tags = {
    name: "${var.env_prefix}-rtb"
   }
}

resource "aws_internet_gateway" "myapp-igw" {
    vpc_id = aws_vpc.myapp-vpc.id
    tags = {
        name: "${var.env_prefix}-igw"
    }
}
`
# Associate subnet to route table with the code below

`resource "aws_internet_gateway" "myapp-igw" {
  vpc_id = aws_vpc.myapp-vpc.id
  tags = {
    Name : "${var.env_prefix}-igw"
  }
}

resource "aws_route_table" "myapp-rtb" {
  vpc_id = aws_vpc.myapp-vpc.id
  route {
    cidr_block = "0.0.0.0/16"
    gateway_id = aws_internet_gateway.myapp-igw.id
  }
  tags = {
    Name : "${var.env_prefix}-rtb"
  }
}
resource "aws_route_table_association" "a-rtb" {
    subnet_id = aws_subnet.myapp-subnet-1.id
    route_table_id = aws_route_table.myapp-rtb.id
}`

# Associate tag and internet gateway to default route table

'resource "aws_default_route_table" "main-myapp-rtb" {
    default_route_table_id = aws_vpc.myapp-vpc.default_route_table_id 
route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.myapp-igw.id
  }
  tags = {
    Name : "${var.env_prefix}-main-rtb"
  }
}

'
# Create security group in your vpc with egress and ingress

`resource "aws_security_group" "myapp-sec-group" {
 name = "myapp-sg"
    vpc_id = aws_vpc.myapp-vpc.id
ingress {
    from_port = 22
    to_port = 22
    protocol = "tcp"
    cidr_blocks =  [var.my_ip]
}
ingress {
    from_port = 8080
    to_port = 8080
    protocol = "tcp"
    cidr_blocks =  ["0.0.0.0/0"]
}
egress {
    from_port = 0
    to_port = 0
    protocol = "-1"
    cidr_blocks =  ["0.0.0.0/0"]
    prefix_list_ids = []
}
tags = {
    Name = "${var.env_prefix}-sg"
}
}`

# Create name tags in default security group in your vpc with egress and ingress
'`resource "aws_default_security_group" "default-sg" {
    vpc_id = aws_vpc.myapp-vpc.id
ingress {
    from_port = 22
    to_port = 22
    protocol = "tcp"
    cidr_blocks =  [var.my_ip]
}
ingress {
    from_port = 8080
    to_port = 8080
    protocol = "tcp"
    cidr_blocks =  ["0.0.0.0/0"]
}
egress {
    from_port = 0
    to_port = 0
    protocol = "-1"
    cidr_blocks =  ["0.0.0.0/0"]
    prefix_list_ids = []
}
tags = {
    Name = "${var.env_prefix}-default-sg"
}
}
'
# Created ami, ssh key for instance, create ec2 instance

```hcl`provider "aws" {
  region = "eu-west-2"
}

variable "avail_zone" {}
variable "vpc_cidr_block" {}
variable "subnet_cidr_block" {}
variable "env_prefix" {}
variable "my_ip" {}
variable "instance_type" {}
variable "public_key_location" {}

resource "aws_vpc" "myapp-vpc" {
  cidr_block = var.vpc_cidr_block
  tags = {
    Name : "${var.env_prefix}-vpc",
  }
}

resource "aws_subnet" "myapp-subnet-1" {
  vpc_id            = aws_vpc.myapp-vpc.id
  cidr_block        = var.subnet_cidr_block
  availability_zone = var.avail_zone

  tags = {
    Name : "${var.env_prefix}-subnet-1"
  }
}

resource "aws_internet_gateway" "myapp-igw" {
  vpc_id = aws_vpc.myapp-vpc.id
  tags = {
    Name : "${var.env_prefix}-igw"
  }
}

/*resource "aws_route_table" "myapp-rtb" {
  vpc_id = aws_vpc.myapp-vpc.id
  route {
    cidr_block = "0.0.0.0/16"
    gateway_id = aws_internet_gateway.myapp-igw.id
  }
  tags = {
    Name : "${var.env_prefix}-rtb"
  }
}
resource "aws_route_table_association" "a-rtb" {
    subnet_id = aws_subnet.myapp-subnet-1.id
    route_table_id = aws_route_table.myapp-rtb.id
}*/

resource "aws_default_route_table" "main-myapp-rtb" {
  default_route_table_id = aws_vpc.myapp-vpc.default_route_table_id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.myapp-igw.id
  }
  tags = {
    Name : "${var.env_prefix}-main-rtb"
  }
}
resource "aws_security_group" "myapp-sg" {
  name   = "myapp-sg"
  vpc_id = aws_vpc.myapp-vpc.id
en  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [var.my_ip]
  }
ingress {
  from_port   = 8080
  to_port     = 8080
  protocol    = "tcp"
  cidr_blocks = ["0.0.0.0/0"]
}
egress {
  from_port       = 0
  to_port         = 0
  protocol        = "-1"
  cidr_blocks     = ["0.0.0.0/0"]
  prefix_list_ids = []
}
tags = {
  Name = "${var.env_prefix}-sg"
}
}
data "aws_ami" "latest-amazon-linux-image" {
  most_recent = true
  owners       = ["amazon"]
  filter {
    name   = "name"
    values = ["al2023-ami-*-kernel-6.1-x86_64"]
  }
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}
  output "aws_ami_id" {
 value = data.aws_ami.latest-amazon-linux-image.id
  }

  resource "aws_key_pair" "ssh-key" {
  key_name = "server-key"
  public_key = file(var.public_key_location)
  }

  resource "aws_instance" "myapp-server" {
 ami = data.aws_ami.latest-amazon-linux-image.id
 instance_type = var.instance_type
 subnet_id = aws_subnet.myapp-subnet-1.id
 vpc_security_group_ids = [aws_security_group.myapp-sg.id]
 availability_zone = var.avail_zone
 associate_public_ip_address = true
 key_name = aws_key_pair.ssh-key.key_name
'user_data = file ("entry-script.sh)'
 
 tags = {
    Name: "${var.env_prefix}-server"
 }
}
`
# Declaring the variable 
`vpc_cidr_block    = "cidr_block for vpc"
subnet_cidr_block = "subnet cidr block"
avail_zone        = "availability zone"
env_prefix        = "dev"
my_ip             = "my ip address"
instance_type = "instance type"
public_key_location = "path to my public key pair for key-gen"`
private_key_location = "path to my private key pair for key-gen"`


