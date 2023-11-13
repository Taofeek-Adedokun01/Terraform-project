# I Created AWS Provider in my Main.tf file to specify the region and also configure my AWS CLI on my terminal.

`provider "aws" {
    region = ""
    access_key = ""
    secret_key = ""
}`

# I created aws_vpc and subnet inside the vpc with the code below

`resource "aws_vpc" "development-vpc" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "dev-subnet-1" {
    vpc_id = aws_vpc.development-vpc.id
    cidr_block = "10.0.10.0/24"
    availability_zone = "eu-west-2a"
}`

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
`

