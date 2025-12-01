# 🚀 Terraform — Complete Documentation & Best Practices

## 📘 Table of Contents
1. [Introduction](#introduction)
2. [Core Terraform Commands](#core-terraform-commands)
3. [Terraform Modules](#terraform-modules)
4. [Real-World Infrastructure Scenarios](#real-world-infrastructure-scenarios)
5. [Understanding Terraform Refresh](#understanding-terraform-refresh)
6. [Drift Detection & State Behavior](#drift-detection--state-behavior)
7. [Best Practices](#best-practices)
8. [Appendix: Useful Code Examples](#appendix-useful-code-examples)

---

# 🔰 Introduction
Terraform is a **declarative Infrastructure-as-Code (IaC)** tool that allows you to define cloud infrastructure using configuration files (HCL). Terraform uses a state file to track resources it manages, enabling consistent and predictable deployments.

---

# 🧰 Core Terraform Commands

## `terraform init`
Initializes the working directory by downloading providers, initializing backend, and preparing module dependencies.

## `terraform plan -out=tfplan`
Creates an execution plan showing what will change. Does not modify cloud resources.

## `terraform apply`
Applies the changes to reach the desired state.

## `terraform refresh` (Deprecated)
Previously updated state directly from cloud APIs. Deprecated due to unsafe behavior.

---

# ⭐ Refresh-Only Workflow (Modern Approach)

### Step 1 — Generate refresh-only plan

```
terraform plan -refresh-only -out=refresh.plan
```

✔ Detects drift  
✔ DOES *NOT* modify state  
✔ Creates a plan file only  

### Step 2 — Apply refresh-only changes

```
terraform apply refresh.plan
```

✔ Updates **state file only**  
✔ No changes to cloud resources  

> **Important: `terraform plan -refresh-only` does *not* change the state. The state is updated ONLY after `terraform apply refresh.plan`.**

---

# 📦 Terraform Modules

Modules are reusable components that group resources, variables, and outputs.

## Manual Module Example

```
modules/
  ec2/
    main.tf
    variables.tf
    outputs.tf

envs/
  prod/
    main.tf
```

**Using the module:**
```hcl
module "app_server" {
  source         = "../modules/ec2"
  ami            = data.aws_ami.ubuntu.id
  instance_type  = "t3.micro"
}
```

---

# 🌐 Real-World Infrastructure Scenarios

## 1️⃣ EKS Add‑On Created Manually → Not Detected

Terraform only refreshes resources *in its state*.  
A manually created addon is invisible.

Fix:
```
terraform import aws_eks_addon.example addon-name
```

## 2️⃣ Security Group Description Changed → No Drift Detected

Possible reasons:
- AWS API doesn’t return description  
- Provider doesn’t track field  
- Field may be ForceNew  

Fix:
```hcl
lifecycle {
  ignore_changes = [description]
}
```

## 3️⃣ Hardcoded AMI No Longer Exists → Apply Fails

Plan does not validate AMI existence. Apply fails during AWS API call.

Fix (dynamic AMI):
```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
}
```

## 4️⃣ Importing a Manually Created EC2 Instance

Steps:
1. Add resource block  
2. Import:

```
terraform import aws_instance.myvm i-1234abcd
```

3. Run `terraform plan`  
4. Reconcile differences  

---

# 🔍 Understanding Terraform Refresh

## ✅ Refresh CAN detect:
- Attribute changes on managed resources  
- Drift in instance type, tags, SG rules, scaling config  

## ❌ Refresh CANNOT detect:
- New manually created resources  
- Deleted resources (only plan catches deletion)  
- Attributes not returned by AWS API  
- Incorrect AMIs  
- Hardcoded ID failures  

---

# 🔄 Drift Detection & State Behavior

Drift = difference between:
- Terraform configuration  
- Actual cloud infrastructure  

You can detect drift using:
- `terraform plan`
- `terraform plan -refresh-only`
- CI/CD checks  
- AWS Config  

---

# 🏆 Best Practices

- Avoid console-created resources  
- Never hardcode AMI IDs  
- Use modules to enforce architecture standards  
- Use backend with locking (S3 + DynamoDB)  
- Version pin providers  
- Use PR-based plan review in GitHub  

---

# 📚 Appendix: Useful Code Examples

## Dynamic Ubuntu AMI Lookup
```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}
```

## EC2 Import
```
terraform import aws_instance.myvm i-0123456789abcdef0
```

---

# ✔ End of Documentation
This README is suitable for GitHub and contains detailed explanations, examples, and best practices.
