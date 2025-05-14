# 🛠️ Cloud-Based Wazuh Monitoring Using Terraform and AWS

This project was created to launch a **Wazuh lab using the AWS sandbox**.
The goal was to host **Wazuh on an Ubuntu instance** and **scan a Windows Server 2019 instance**.
Terraform was used to automate the deployment, allowing future reuse and learning.

You’ll find the full sanitized code below, with clear explanations for each section.

---

## `main.tf`

*Sets up the VPC, subnet, internet gateway, and routing for the environment.*

```hcl
resource "aws_vpc" "main" {
  cidr_block = "<your-cidr-block>"
}

resource "aws_subnet" "public_subnet" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "<your-cidr-block>"
  map_public_ip_on_launch = true
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
}

resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
}

resource "aws_route_table_association" "public_rt_assoc" {
  subnet_id      = aws_subnet.public_subnet.id
  route_table_id = aws_route_table.public_rt.id
}

resource "aws_key_pair" "wazuh_lab_final_key" {
  key_name   = "<your-key-name>"
  public_key = file("~/.ssh/wazuh-lab-final-key.pub")
}
```

---

## `provider.tf`

*Defines the AWS provider and region. Required by Terraform to know where and how to deploy.*

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  required_version = ">= 1.3.0"
}

provider "aws" {
  region = var.aws_region
}
```

---

## `windows.tf`

*Launches a Windows Server 2019 EC2 instance.
The plan was to later install the Wazuh agent here to simulate a real monitored endpoint.*

```hcl
resource "aws_instance" "windows_ec2" {
  ami                    = "<your-ami-id>"
  instance_type          = "t3.micro"
  subnet_id              = aws_subnet.public_subnet.id
  vpc_security_group_ids = [aws_security_group.wazuh_sg.id]
  key_name               = "<your-key-name>"
  get_password_data      = true

  tags = {
    Name = "windows-server"
  }

  provisioner "local-exec" {
    command = "echo Windows instance deployed. RDP in after decrypting password in AWS Console."
  }
}
```

---

## `wazuh_server.tf`

*Provisions the Ubuntu instance intended to host Wazuh.
You can automate Wazuh installation via user data or cloud-init.*

```hcl
resource "aws_instance" "wazuh_server" {
  ami                         = "<your-ami-id>"
  instance_type               = "t2.medium"
  subnet_id                   = aws_subnet.public_subnet.id
  vpc_security_group_ids      = [aws_security_group.wazuh_sg.id]
  associate_public_ip_address = true
  key_name                    = "<your-key-name>"

  user_data = file("scripts/wazuh_user_data.sh")

  tags = {
    Name = "wazuh-server"
  }
}
```

---

## `security-groups.tf`

*Creates a security group allowing access to:*

* SSH for Ubuntu
* RDP for Windows
* Port 5601 for Wazuh Dashboard (Kibana)

```hcl
resource "aws_security_group" "wazuh_sg" {
  name        = "wazuh_sg"
  description = "Allow SSH, RDP, Wazuh ports"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 3389
    to_port     = 3389
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 5601
    to_port     = 5601
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

---

## `variables.tf`

*Stores configurable input values like region and key name for easier reusability.*

```hcl
variable "aws_region" {
  description = "AWS region to deploy into"
  type        = string
  default     = "us-east-1"
}

variable "key_name" {
  description = "Name of the SSH key pair"
  type        = string
}

variable "public_key_path" {
  description = "Path to your public key file"
  type        = string
}
```

---

## `outputs.tf`

*Terraform prints these outputs after deployment so you can easily connect to your EC2 instances.*

```hcl
output "wazuh_server_public_ip" {
  value = aws_instance.wazuh_server.public_ip
}

output "windows_instance_public_ip" {
  value = aws_instance.windows_ec2.public_ip
}
```

---

## ✅ To-Do & Expansion Plan

* [ ] Add a `terraform.tfvars` file with actual input values
* [ ] Create and share a script for Wazuh agent setup on Windows
* [ ] Add screenshots for AWS console, Wazuh dashboard, and EC2 setup
* [ ] Document commands used in Wazuh install and troubleshooting
* [ ] Explore log forwarding and event triggers inside Wazuh

---
