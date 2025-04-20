# 📦 Terraform S3 Bucket + IAM User Policy Automation

This project demonstrates how to use Terraform to securely automate the creation of an AWS S3 bucket and a custom IAM user with scoped access.

---

## 🔧 What This Project Does

- Creates an **S3 bucket** with public access blocked and ownership enforced
- Creates an **IAM user**
- Defines an IAM **policy template** in a separate `.tpl.json` file with dynamic bucket name injection
- Attaches the custom policy to the IAM user

---

## ✅ Real-World Use Case

This setup mirrors real-world scenarios where scoped S3 access is required. It's especially useful for:

- Backup scripts or Lambda functions writing to a bucket
- Granting temporary access to a developer or third-party service
- Creating a limited-access CI/CD pipeline role

By using Terraform and dynamic policy templates, this project allows teams to securely and consistently provision access in a repeatable way.

---

## 📁 Project Structure

```
s3-user-policy/
├── main.tf
├── provider.tf
├── variables.tf
├── outputs.tf
├── terraform.tfvars.example
└── policies/
    └── s3-access-policy.tpl.json
```

---

## 🔐 Security Practices Followed

- No hardcoded names or secrets
- `terraform.tfvars` excluded from version control
- `.gitignore` used to avoid committing state files and sensitive data
- IAM policies are templated and dynamically filled at deploy time
- S3 bucket blocks public access and enforces owner control

---

## 🔠 Input Variables (`variables.tf`)
```hcl
variable "region" {
  description = "AWS region to deploy resources"
  type        = string
  default     = "us-west-2"
}

variable "bucket_name" {
  description = "Globally unique S3 bucket name"
  type        = string
}

variable "iam_user_name" {
  description = "IAM user who will access the bucket"
  type        = string
  default     = "placeholder-user"
}
```

---

## ⚙️ AWS Provider (`provider.tf`)
```hcl
provider "aws" {
  region = var.region
}
```

---

## 📜 Policy Template (`policies/s3-access-policy.tpl.json`)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": [
        "arn:aws:s3:::${bucket}",
        "arn:aws:s3:::${bucket}/*"
      ]
    }
  ]
}
```

---

## 🚀 Main Configuration (`main.tf`)
```hcl
data "template_file" "s3_policy" {
  template = file("${path.module}/policies/s3-access-policy.tpl.json")
  vars = {
    bucket = var.bucket_name
  }
}

resource "aws_s3_bucket" "my_bucket" {
  bucket        = var.bucket_name
  force_destroy = true
}

resource "aws_s3_bucket_ownership_controls" "bucket_ownership" {
  bucket = aws_s3_bucket.my_bucket.id

  rule {
    object_ownership = "BucketOwnerEnforced"
  }
}

resource "aws_s3_bucket_public_access_block" "public_access" {
  bucket = aws_s3_bucket.my_bucket.id

  block_public_acls       = true
  ignore_public_acls      = true
  block_public_policy     = true
  restrict_public_buckets = true
}

resource "aws_iam_user" "s3_user" {
  name = var.iam_user_name
}

resource "aws_iam_policy" "s3_access_policy" {
  name        = "s3-access-policy"
  description = "Allow user to access the S3 bucket"
  policy      = data.template_file.s3_policy.rendered
}

resource "aws_iam_user_policy_attachment" "attach_policy" {
  user       = aws_iam_user.s3_user.name
  policy_arn = aws_iam_policy.s3_access_policy.arn
}
```

---

## 📤 Outputs (`outputs.tf`)
```hcl
output "bucket_name" {
  description = "The name of the S3 bucket"
  value       = aws_s3_bucket.my_bucket.bucket
}

output "iam_user_name" {
  description = "The IAM user's name"
  value       = aws_iam_user.s3_user.name
}
```

---

## 🧪 Sample Inputs (`terraform.tfvars.example`)
```hcl
bucket_name   = "placeholder-bucket-name"
region        = "us-west-2"
iam_user_name = "placeholder-user"
```

---

## 🔄 Terraform Workflow Commands

These are the commands used to manage the lifecycle of the project:

```bash
terraform init       # Initialize Terraform working directory
terraform plan       # Preview the resources to be created/modified
terraform apply      # Deploy infrastructure
terraform destroy    # Tear down all resources created
```

---

## 🧠 Challenges & Improvements

- Learned how to use `template_file` to inject Terraform variables into JSON IAM policies
- Cleanly separated IAM policy logic from Terraform code
- Handled S3 security requirements using `aws_s3_bucket_public_access_block` and `ownership_controls`
- In future iterations, this could be turned into a module and expanded for group/user-role access models

---

## 📚 Further Reading

- [Terraform AWS Provider Docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [AWS IAM Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html)
- [S3 Bucket Security Best Practices](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-best-practices.html)

---