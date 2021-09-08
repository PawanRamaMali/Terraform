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

```
```
