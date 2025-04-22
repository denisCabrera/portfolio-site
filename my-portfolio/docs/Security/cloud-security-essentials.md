# ☁️ Cloud Security Essentials: Real-World Examples, Tools & Fixes

This page shares practical examples, tools, and techniques I’ve used to harden cloud environments, detect issues, and automate security. Whether you're just starting out or looking to compare notes, these are the essentials I rely on.

---

## 🔓 Common AWS Misconfigurations & Fixes

### 🚫 Public S3 Buckets
- **Issue:** Unrestricted access to sensitive data.
- **Detect:**  
  ```bash
  aws s3api get-bucket-acl --bucket your-bucket-name
  ```
- **Fix (Block Public Access):**  
  ```bash
  aws s3api put-public-access-block \
    --bucket your-bucket-name \
    --public-access-block-configuration 'BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true'
  ```

---

### 🔑 Overly Permissive IAM Policies
- **Issue:** Use of wildcards (`*`) in `Action` or `Resource`.
- **Fix:** Use scoped permissions and roles per function.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": ["arn:aws:s3:::example-bucket"]
    }
  ]
}
```

- **Audit tool:** [`policy_sentry`](https://github.com/salesforce/policy_sentry)

---

## 🔐 IAM Policy Best Practices

| Best Practice                | Description                                              |
|-----------------------------|----------------------------------------------------------|
| Least Privilege             | Grant only what is needed                                |
| Role Separation             | Separate roles by task (e.g., `EC2Admin`, `S3Reader`)    |
| Use Conditions              | Add conditions like source IP or MFA                     |
| Rotate Access Keys          | Periodically rotate or remove unused keys                |

Example with Conditions:
```json
"Condition": {
  "Bool": {
    "aws:MultiFactorAuthPresent": "true"
  }
}
```

---

## 🧰 Tools I Use for Cloud Security

| Tool        | Purpose                            |
|-------------|------------------------------------|
| AWS CLI     | Quick scripting and automation     |
| Terraform   | Infrastructure as Code + auditing  |
| tfsec       | Scans Terraform for misconfigs     |
| CloudSploit | CSP misconfiguration detection     |
| ScoutSuite  | Cloud posture assessment           |
| Pacu        | AWS exploitation framework         |
| TruffleHog  | Detect secrets in repos/logs       |

---

## ⚡ Security Scripts & Automation

### 🧪 Detect Public S3 Buckets (Python)
```python
import boto3
s3 = boto3.client('s3')
buckets = s3.list_buckets()['Buckets']
for bucket in buckets:
    acl = s3.get_bucket_acl(Bucket=bucket['Name'])
    for grant in acl['Grants']:
        if grant['Grantee'].get('URI') == 'http://acs.amazonaws.com/groups/global/AllUsers':
            print(f"[!] Public Bucket: {bucket['Name']}")
```

### 🧹 Terraform Security Audit (tfsec)
```bash
tfsec ./terraform-code/
```

---

## 🛡️ OWASP Top 10 in the Cloud (Mini Map)

| OWASP Risk                   | Cloud Example                                            |
|-----------------------------|----------------------------------------------------------|
| A01: Broken Access Control  | Misconfigured S3 bucket or API Gateway authorization     |
| A02: Cryptographic Failures | Disabled encryption-at-rest for EBS or S3                |
| A05: Security Misconfig     | Public EC2 ports, default VPC use                        |
| A07: IDOR                   | Direct object access in API Gateway                      |
| A09: Security Logging       | CloudTrail not enabled or misconfigured                  |

---

## ✍️ About This Page

This page is part of my ongoing effort to share what I’ve learned in cloud security, especially in AWS environments. I aim to add more automation, checklists, and projects as I grow.

---
