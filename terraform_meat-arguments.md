
# Terraform Meta-arguments

Terraform meta-arguments are used to control how resources are created, managed, and destroyed. These are not specific to any single resource type — instead, they apply across all resources and help in creating flexible, scalable infrastructure code.

## List of Meta-arguments

- `count`
- `for_each`
- `depends_on`
- `provider`
- `lifecycle`

---

## Example 1: Using `count` to Create EC2 Instances

If you want to create multiple EC2 instances, one way is to copy the resource block repeatedly — but this approach becomes hard to maintain. Instead, you can use the `count` meta-argument for better scalability.

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "web_server" {
  count         = 3
  ami           = "ami-xxxxxxxxx" # Replace with a valid AMI ID from your region
  instance_type = "t2.micro"

  tags = {
    Name = "web-server-${count.index}"
  }
}
```

### ✅ What Happens Here?

- Terraform will create 3 EC2 instances.
- The tag will be different for each instance: `web-server-0`, `web-server-1`, and `web-server-2`.
- `${count.index}` gives a unique index starting from 0.

---

## Example 2: Using `for_each` for Custom Instance Types

If you want to assign **different instance types** to different EC2 instances, `count` won’t work. Use `for_each` with a map.

```hcl
provider "aws" {
  region = "us-east-1"
}

variable "instance_types" {
  default = {
    webserver1 = "t2.micro",
    webserver2 = "t2.small",
    webserver3 = "t2.medium"
  }
}

resource "aws_instance" "custom_instance" {
  for_each      = var.instance_types
  ami           = "ami-xxxxxxxxx" # Replace with valid AMI ID
  instance_type = each.value

  tags = {
    Name = each.key
  }
}
```

### ✅ Explanation

- `for_each` iterates over each key-value pair in the map.
- Instance type varies per instance.
- Instance names are `webserver1`, `webserver2`, etc.

---

## Example 3: Using `for_each` to Create Subnets in a VPC

Here’s how to use `for_each` to create multiple subnets dynamically.

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "main-vpc"
  }
}

variable "subnets" {
  default = {
    public_subnet1  = "10.0.1.0/24",
    public_subnet2  = "10.0.2.0/24",
    private_subnet1 = "10.0.3.0/24",
    private_subnet2 = "10.0.4.0/24"
  }
}

resource "aws_subnet" "subnets" {
  for_each   = var.subnets
  vpc_id     = aws_vpc.main.id
  cidr_block = each.value

  tags = {
    Name = each.key
  }
}
```

### ✅ Outcome

- One VPC and four subnets (2 public, 2 private) will be created.
- Subnet CIDRs and names come from the variable map.

---

## Example 4: Using `depends_on` to Control Resource Order

The `depends_on` meta-argument ensures that one resource is created **after** another.

```hcl
resource "aws_instance" "example" {
  ami           = "ami-xxxxxxxxx"
  instance_type = "t2.micro"

  depends_on = [aws_vpc.main]

  tags = {
    Name = "dependent-instance"
  }
}
```

### ✅ Use Case

Even though Terraform usually detects dependencies automatically, `depends_on` is useful when:

- The dependency is not obvious (e.g., script outputs, file rendering)
- You're using modules or provisioners

---

## Best Practices

- ✅ Use `count` for identical resources
- ✅ Use `for_each` when each resource is slightly different
- ✅ Use `depends_on` for explicit ordering
- ✅ Keep your AMI ID region-specific and valid
- ✅ Validate your code using `terraform validate`
- ✅ Format with `terraform fmt` for consistency

---

## Final Notes

- Meta-arguments help make your Terraform code modular, DRY, and scalable.
- Learn to use them properly to avoid repeating code and creating manual mistakes.

Happy Terraforming! 🚀
