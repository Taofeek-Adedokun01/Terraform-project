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
variable "dev_environment"{
    description = "development environment"
}
variable "subnet_cidr_block" {
    description = "subnet cidr block"
}`

# I created variable for my subnet cidr and my aws_vpc tag names with the code below and defined it in my terraform.tfvars with the code below

`subnet_cidr_block = "10.0.50.0/24"
dev_environment = "development"`


