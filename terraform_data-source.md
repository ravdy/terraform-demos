# Terraform Data Sources

A **data source** allows Terraform to **query information** that can be used elsewhere in your configuration — without managing or modifying the source of that data.

---

## ✅ Common Use Cases

#### 1. Referencing Existing Infrastructure

#### 2. Fetching Outputs from Another Terraform Workspace/Module

#### 3. Getting Dynamic Values from Cloud Providers

---

Lets Demonistarte 1st use case `Referencing Existing Infrastructure`
- Create a security group manually with name Demo-SecurityGroup
- Create terraform code to use this as data source
  ```sh
  provider "aws" {
  region = "us-east-1"
}

# Fetch existing security group by Name tag
data "aws_security_group" "existing_sg" {
  filter {
    name   = "tag:Name"
    values = ["Demo-SecurityGroup"]
  }

# Launch EC2 instance using the existing SG
resource "aws_instance" "example" {
  ami           = "ami-xxxxxxxxxxx"
  instance_type = "t2.micro"
  vpc_security_group_ids = [data.aws_security_group.existing_sg.id]

  tags = {
    Name = "Refering-existing-SG"
  }
}
```

Now lets see 2nd Use case


## Realtime Use cases
1. Fetches existing VPC peering connections  
```sh
data "aws_vpc_peering_connections" "pending_acceptance" {

  filter {
    name   = "requester-vpc-info.owner-id"
    values = var.requester_owner_ids
  }
}
```
2. Fetches existing Amazon EKS cluster from AWS 
```sh
data "aws_eks_cluster" "cluster" {
  name = module.eks.cluster_name
}
```
3. Fetches existing AWS Secret 
```sh
data "aws_secretsmanager_secret_version" "datadog_api_key" {
  secret_id = "eksdatadogkey"
}
```
---

## 🧠 Summary

- **Data sources** are a powerful tool for referencing existing infrastructure or outputs from other Terraform workspaces.
- Use them to keep projects **modular**, **reusable**, and **decoupled**.
- `terraform_remote_state` is ideal when working across multiple Terraform projects or teams.
