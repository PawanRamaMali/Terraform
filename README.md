# Terraform

Terraform is an infrastructure as code (IaC) tool that allows you to build, change, and version infrastructure safely and efficiently. This includes low-level components such as compute instances, storage, and networking, as well as high-level components such as DNS entries, SaaS features, etc. Terraform can manage both existing service providers and custom in-house solutions.

Terraform is distributed as a single binary. Install Terraform by unzipping it and moving it to a directory included in your system's PATH .
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

```terraform
resource "aws_vpc" "prod-vpc" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "production"
  }
}

```

### 2. Create Internet Gateway

```terraform

resource "aws_internet_gateway" "gw" {
  vpc_id = aws_vpc.prod-vpc.id
}
```

### 3. Create Custom Route Table

```terraform
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

### 4. Create a Subnet 

```terraform
resource "aws_subnet" "subnet-1" {
  vpc_id            = aws_vpc.prod-vpc.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"

  tags = {
    Name = "prod-subnet"
  }
}
```

### 5. Associate subnet with Route Table

```terraform
resource "aws_route_table_association" "a" {
  subnet_id      = aws_subnet.subnet-1.id
  route_table_id = aws_route_table.prod-route-table.id
}
```



### 6. Create Security Group to allow port 22,80,443

```terraform
resource "aws_security_group" "allow_web" {
  name        = "allow_web_traffic"
  description = "Allow Web inbound traffic"
  vpc_id      = aws_vpc.prod-vpc.id

  ingress {
    description = "HTTPS"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "allow_web"
  }
}

```

### 7. Create a network interface with an ip in the subnet that was created in step 4

```terraform

resource "aws_network_interface" "web-server-nic" {
   subnet_id       = aws_subnet.subnet-1.id
   private_ips     = ["10.0.1.50"]
   security_groups = [aws_security_group.allow_web.id]
   
```


### 8. Assign an elastic IP to the network interface created in step 7

```terraform

 resource "aws_eip" "one" {
   vpc                       = true
   network_interface         = aws_network_interface.web-server-nic.id
   associate_with_private_ip = "10.0.1.50"
   depends_on                = [aws_internet_gateway.gw]
 }
 
 output "server_public_ip" {
   value = aws_eip.one.public_ip
 }
 
 ```
 
### 9. Create Ubuntu server and install/enable apache2

```terraform
 resource "aws_instance" "web-server-instance" {
   ami               = "ami-085925f297f89fce1"
   instance_type     = "t2.micro"
   availability_zone = "us-east-1a"
   key_name          = "main-key"

   network_interface {
     device_index         = 0
     network_interface_id = aws_network_interface.web-server-nic.id
   }

   user_data = <<-EOF
                 #!/bin/bash
                 sudo apt update -y
                 sudo apt install apache2 -y
                 sudo systemctl start apache2
                 sudo bash -c 'echo your very first web server > /var/www/html/index.html'
                 EOF
   tags = {
     Name = "web-server"
   }
 }



 output "server_private_ip" {
   value = aws_instance.web-server-instance.private_ip

 }

 output "server_id" {
   value = aws_instance.web-server-instance.id
 }


 resource "<provider>_<resource_type>" "name" {
     config options.....
     key = "value"
     key2 = "another value"
 }
```
