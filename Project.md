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

