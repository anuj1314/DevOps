# Terraform — Practical, Complete Documentation

## 1. Introduction

Terraform is an Infrastructure-as-Code (IaC) tool that lets you define cloud resources using declarative configuration (HCL). Terraform maintains a state file that maps your configuration to actual resources in AWS, Azure, GCP, etc.

---

## 2. Basic Terraform Commands (Explained Simply)

### `terraform init`

Initializes the working directory by:
- Downloading necessary provider plugins
- Initializing backend
- Fetching modules

You run this whenever providers or modules change.

### `terraform plan -out=tfplan`

Compares the desired configuration with the current real-state and:
- Creates an execution plan
- Shows resources to be created, modified, or destroyed
- Does not make any actual changes

With `-out`, the plan is saved to a file so apply will run exactly that plan.

### `terraform apply`

Executes the changes required to reach the desired configuration.

You can either:
```bash
terraform apply         # Plan + apply
terraform apply tfplan  # Apply a saved plan
```

### `terraform refresh` (Deprecated)

#### Old Behavior

`terraform refresh` used to:
- Query cloud provider for the real resource attributes
- Update the state file accordingly
- Make no changes to cloud resources

#### Why Deprecated?

Because:
- Terraform automatically performs refresh during plan
- It caused accidental, silent overwrites of drift
- No visibility into changes unless followed by a plan

#### The Correct Modern Equivalent: Refresh-only plan

**Command:**
```bash
terraform plan -refresh-only -out=refresh.plan
```

**❗ Important – Your Statement (Integrated):**

`terraform plan -refresh-only -out=refresh.plan` will **NOT** change the state file at all.
- It only creates a plan.
- The state is updated **ONLY** after running:
  ```bash
  terraform apply refresh.plan
  ```

**Meaning:**
- The plan step detects drift
- But does not modify the state
- Only the apply step writes refreshed values into state
- And it never changes real cloud resources

This is the safest way to reconcile drift.

### Other useful commands:

#### `terraform fmt`
Formats the Terraform code properly.

#### `terraform validate`
Catches syntax errors and structural mistakes.

#### `terraform show`
Shows state contents or plan details.

#### `terraform state` (advanced)
Used for inspecting, moving, listing state resources.

#### `terraform import`
Brings an existing manually created cloud resource into Terraform state.

---

## 3. Terraform Modules

### What is a module?

A module is a folder containing `.tf` files that group resources logically. Terraform automatically treats the root folder as a module.

Modules allow:
- Reusability
- Cleaner structure
- Composable architecture

### Creating a Manual Module

**Folder layout:**
```
modules/
  ec2_module/
    main.tf
    variables.tf
    outputs.tf

envs/
  prod/
    main.tf
```

**Sample module:**
```hcl
# modules/ec2_module/main.tf
resource "aws_instance" "vm" {
  ami           = var.ami
  instance_type = var.type
}
```

**Using it:**
```hcl
module "app_vm" {
  source = "../modules/ec2_module"
  ami    = data.aws_ami.ubuntu.id
  type   = "t3.small"
}
```

### Predefined Modules (Terraform Registry)

Terraform Registry hosts official and community modules you can use directly:

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
}
```

---

## 4. Real-world Scenarios Explained

### Scenario 1: EKS cluster + manual add-on → refresh did NOT detect it

**Reason:**
- Terraform refresh only checks resources in its state.
- A manually created EKS add-on is not in the state → Terraform ignores it.
- Terraform cannot auto-discover external resources.

**Fix:**

If you want Terraform to manage the add-on:
1. Add the resource block in code
2. Run `terraform import` to capture it in state
3. Run plan to reconcile differences

### Scenario 2: Security Group description changed manually → refresh did NOT show drift

**Possible causes:**
- AWS API may not return updated description (inconsistent behavior)
- Terraform provider may not track the attribute
- Attribute may be "write-only" or "ForceNew"
- `lifecycle.ignore_changes` may be present

**Result:**
Terraform sees no change → no drift detected.

**Fix:**

If the SG description should be Terraform-managed — update config.

If you want Terraform to ignore description changes:
```hcl
lifecycle {
  ignore_changes = [description]
}
```

### Scenario 3: Hard-coded AMI no longer exists → plan succeeds, apply fails

**Reason:**
- Plan does NOT validate AMI availability
- Apply validates AMI at runtime via AWS API
- So the failure only appears during apply

**Correct Approach:**

Use AMI data sources, NOT hard-coded IDs:
```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}
```

### Scenario 4: Importing a manually created resource

**Is it good practice?**

Yes — importing existing infra is correct.

**Steps:**
1. Add matching resource block in code
2. Run:
   ```bash
   terraform import aws_instance.my_vm i-1234567890abcdef0
   ```
3. Run `terraform plan`
4. Fix differences if required

---

## 5. What Terraform Refresh Can and Cannot Detect

### Refresh CAN detect:
- ✅ Changes to managed resources
- ✅ SG rules updated
- ✅ Instance type changed
- ✅ Tag changes
- ✅ Drift in autoscaling configs

### Refresh CANNOT detect:
- ❌ New resources created manually
- ❌ Resources deleted manually (until plan)
- ❌ Attributes AWS does not expose
- ❌ Incorrect or missing configurations
- ❌ Invalid AMIs
- ❌ Unknown dependencies

---

## 6. Best Practices

1. **Never hard-code AMI IDs** → use data sources
2. **Review drift** through `plan -refresh-only`
3. **Avoid console changes** unless they are intentionally outside IaC scope
4. **Use modules** to enforce consistency
5. **Use `ignore_changes` sparingly**
6. **Always version-control provider versions**
7. **Use CI/CD** to run `terraform plan` for drift detection

---

## Quick Reference

| Command | Purpose |
|---------|---------|
| `terraform init` | Initialize working directory |
| `terraform plan` | Preview changes |
| `terraform apply` | Execute changes |
| `terraform plan -refresh-only` | Detect drift without changes |
| `terraform fmt` | Format code |
| `terraform validate` | Validate syntax |
| `terraform import` | Import existing resources |
| `terraform state` | Manage state |

---

**Draft for Review** - Last Updated: December 2025
