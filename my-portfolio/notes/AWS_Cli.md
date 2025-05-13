## 🔍 List All Running EC2 Instances

This command lists all EC2 instances in the specified AWS region. It extracts the **Instance ID** and the **Name tag** (if available), displaying the result in a table format.

```bash
aws ec2 describe-instances \
  --region <your-region> \
  --query 'Reservations[*].Instances[*].[InstanceId,Tags[?Key==`Name`].Value|[0]]' \
  --output table
````

---

## 🔑 Get Windows Server RDP Password

Use this command to retrieve the Windows **Administrator password** for an EC2 instance. You must provide the **instance ID**, your **private key file**, and the **region**.

```bash
aws ec2 get-password-data \
  --instance-id <your-instance-id> \
  --priv-launch-key ~/.ssh/<your-private-key>.pem \
  --region <your-region> \
  --query 'PasswordData' \
  --output text
```

> 🔐 Note: This only works after the EC2 instance has generated the password (usually several minutes after launch).

---

## 🌐 Describe Network Interfaces by Security Group

This lists all **network interfaces** attached to EC2 instances that are associated with a specific **security group**.

```bash
aws ec2 describe-network-interfaces \
  --filters "Name=group-id,Values=<your-security-group-id>" \
  --region <your-region> \
  --query "NetworkInterfaces[*].{ID:NetworkInterfaceId,AttachedTo:Attachment.InstanceId}" \
  --output table
```

---

## 🛠 Modify an Instance’s Security Groups

Replace all existing security groups attached to an EC2 instance with the specified one(s).

```bash
aws ec2 modify-instance-attribute \
  --instance-id <your-instance-id> \
  --groups <your-security-group-id> \
  --region <your-region>
```

> 💡 You can pass multiple group IDs separated by spaces if needed.

---

## ❌ Delete a Security Group

Use this to **delete a security group** that is no longer associated with any EC2 or other AWS resources.

```bash
aws ec2 delete-security-group \
  --group-id <your-security-group-id> \
  --region <your-region>
```

> ⚠️ Ensure no EC2 instance or ENI is using this security group, or the command will fail.



---

## 📌 Placeholder Key Reference

| Placeholder                | Description                              |
|----------------------------|------------------------------------------|
| `<your-region>`            | AWS region (e.g., `us-east-1`)           |
| `<your-instance-id>`       | EC2 instance ID (e.g., `i-xxxxxxxxxxxxx`)|
| `<your-security-group-id>` | Security group ID (e.g., `sg-xxxxxxxx`)  |
| `<your-private-key>`       | Filename of your `.pem` private key      |

---