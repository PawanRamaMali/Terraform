# Terraform

[Download Terraform](https://www.terraform.io/downloads.html) 

[Terraform AWS Service Provider](https://registry.terraform.io/providers/hashicorp/aws/latest)


## Terraform Commands

```
terraform init
```

```
terraform plan
```

```
terraform apply
```

```
terraform destroy
```

## Terraform Code

###  Create a AWS instance 

```terraform
provider "aws" {
    region = "us-east-1"
    access_key = ""
    secret_key = ""
}

resource "aws_instance" "my-first-server" {
  ami = "ami-09e67e426f25ce0d7"
  instance_type = "t2.micro"
  tags = {
    #"Name" = "Ubuntu"
  }
}

```

### Build a AWS VPC

```terraform
provider "aws" {
    region = "us-east-1"
    access_key = ""
    secret_key = ""
}

resource "aws_vpc" "my-first-vpc-1" {
  cidr_block = "10.0.0.0/16"
  tags = {
    "Name" = "production"
  }
}

resource "aws_subnet" "subnet-2" {
  vpc_id = aws_vpc.my-first-vpc.id
  cidr_block = "10.0.1.0/24"
  tags = {
    "Name" = "prod-subnet"
  }
}


```


### 1. Create VPC

```
resource "aws_vpc" "prod-vpc" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "production"
  }
}

```

### 2. Create Internet Gateway

```

resource "aws_internet_gateway" "gw" {
  vpc_id = aws_vpc.prod-vpc.id
}
```

### 3. Create Custom Route Table

```
resource "aws_route_table" "prod-route-table" {
  vpc_id = aws_vpc.prod-vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.gw.id
  }

  route {
    ipv6_cidr_block = "::/0"
    gateway_id      = aws_internet_gateway.gw.id
  }

  tags = {
    Name = "Prod"
  }
}
```
