# Terraform Data Sources

A **data source** allows Terraform to **query information** that can be used elsewhere in your configuration — without managing or modifying the source of that data.

---

## ✅ Common Use Cases

#### 1. Referencing Existing Infrastructure

Example:
You have a VPC or subnet created outside Terraform (manually or by another team), and you want to deploy a new EC2 instance inside it.

➡️ Use a data source to fetch the subnet ID.

#### 2. Fetching Outputs from Another Terraform Workspace/Module

Example:
A network is created in one module and you need its details in another module — data sources can help pull that information.

#### 3. Getting Dynamic Values from Cloud Providers

Example:
Get the **latest Amazon Linux AMI** ID in a given region.

---

## 📌 Example: Get Latest Amazon Linux AMI

```hcl
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

resource "aws_instance" "example" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t2.micro"
}
```

---

## 🧩 Example: Use Resource from One Project in Another

### Scenario:
- Create a VPC and subnet in **Project 1**
- Use that VPC/subnet in **Project 2** using Terraform **data sources** to launch an EC2 instance

> ⚠️ *This is for demo purposes. Real-world examples are shared below.*

---

## 📁 Project 1 – VPC & Subnet Creation

**File:** `project-network-main.tf`

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "project1-main-vpc"
  }
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true

  tags = {
    Name = "project1-public-subnet"
  }
}

output "vpc_id" {
  value = aws_vpc.main.id
}

output "subnet_id" {
  value = aws_subnet.public.id
}

output "vpc_name" {
  value = aws_vpc.main.tags["Name"]
}
```

---

## 🚀 Project 2 – Launch EC2 Using Remote State from Project 1

### 🔑 Prerequisites:

You must provide access to Project 1's state via one of the following:

- ✅ **Remote backend** (e.g., S3 shared bucket)
- ✅ **Manually pass output values** as input variables
- ✅ **Best Practice**: Use `terraform_remote_state` data source

---

### Terraform Configuration

```hcl
provider "aws" {
  region = "us-east-1"
}

# Fetch remote state from Project 1
data "terraform_remote_state" "network" {
  backend = "s3"
  config = {
    bucket = "my-terraform-states"
    key    = "project1-network/terraform.tfstate"
    region = "us-east-1"
  }
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0" # Replace with correct AMI ID
  instance_type = "t2.micro"
  subnet_id     = data.terraform_remote_state.network.outputs.subnet_id

  tags = {
    Name = "project2-web"
  }
}
```

---

## 🧠 Summary

- **Data sources** are a powerful tool for referencing existing infrastructure or outputs from other Terraform workspaces.
- Use them to keep projects **modular**, **reusable**, and **decoupled**.
- `terraform_remote_state` is ideal when working across multiple Terraform projects or teams.
