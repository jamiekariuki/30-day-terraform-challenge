# Day 13: Managing Sensitive Data Securely in Terraform

## What You Will Accomplish Today

Every real infrastructure deployment involves secrets — database passwords, API keys, TLS certificates, tokens. The number one security mistake Terraform engineers make is letting those secrets end up in places they should never be: hardcoded in `.tf` files, committed to Git, or stored in plaintext inside state files. Today you will learn exactly how secrets leak in Terraform configurations and how to stop every one of those leak paths using AWS Secrets Manager and Terraform's built-in sensitive value handling. This knowledge is critical for any professional infrastructure role.

---

## Tasks

### 1. Read

**Book:** *Terraform: Up & Running* by Yevgeniy Brikman — **Chapter 6**, pages 191–221

Read the full chapter with focus on:
- *Secret Management Basics* — the categories of secrets and why each requires different handling
- *Secret Management Tools* — Vault, AWS Secrets Manager, and environment variables
- *Managing Sensitive Data in State and Code* — the specific ways secrets leak and how to prevent each one

Take careful notes on the state file problem. Even when you handle secrets correctly in your code, they can still end up in plaintext in `terraform.tfstate`. Understanding this is what separates security-aware engineers from everyone else.

---

### 2. Complete the Hands-On Labs

- **Lab 1:** [Module Versioning](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2004%20-%20Understand%20Terraform%20Basics/14%20-%20Module_Versioning.md)
- **Lab 2:** [Terraform Testing](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2004%20-%20Understand%20Terraform%20Basics/15%20-%20Terraform_Testing.md)

---

### 3. Understand the Three Secret Leak Paths

Before writing any code, internalise exactly where secrets leak in Terraform and why:

**Leak path 1 — Hardcoded in `.tf` files**
Any secret written directly into a resource argument is committed to version control the moment you run `git add`. Even if you delete it later, it exists in Git history permanently.

```hcl
# Never do this
resource "aws_db_instance" "example" {
  username = "admin"
  password = "super-secret-password"  # hardcoded — will end up in Git
}
```

**Leak path 2 — Passed as a variable with a default value**
`default` values are stored in your `.tf` files. Secrets must never have defaults.

```hcl
# Also wrong — default value is committed to source control
variable "db_password" {
  default = "super-secret-password"
}
```

**Leak path 3 — Stored in plaintext in state**
Even if you handle the first two correctly, Terraform stores the values of sensitive resource attributes in `terraform.tfstate` in plaintext. Anyone with read access to the state file can see all your secrets. This is why remote state with encryption and restricted access is non-negotiable.

---

### 4. Store Secrets in AWS Secrets Manager

Create your database credentials in Secrets Manager manually (never through Terraform for bootstrap secrets):

```bash
aws secretsmanager create-secret \
  --name "prod/db/credentials" \
  --secret-string '{"username":"dbadmin","password":"your-secure-password-here"}'
```

Then fetch them at apply time using a data source — they never touch your `.tf` files:

```hcl
data "aws_secretsmanager_secret" "db_credentials" {
  name = "prod/db/credentials"
}

data "aws_secretsmanager_secret_version" "db_credentials" {
  secret_id = data.aws_secretsmanager_secret.db_credentials.id
}

locals {
  db_credentials = jsondecode(
    data.aws_secretsmanager_secret_version.db_credentials.secret_string
  )
}

resource "aws_db_instance" "example" {
  engine         = "mysql"
  engine_version = "8.0"
  instance_class = "db.t3.micro"
  db_name        = "appdb"

  username = local.db_credentials["username"]
  password = local.db_credentials["password"]

  # Additional required fields
  allocated_storage = 10
  skip_final_snapshot = true
}
```

The secret is fetched at runtime. It never exists in your configuration files. The only risk is the state file — which you will address next.

---

### 5. Mark Outputs and Variables as Sensitive

Terraform's `sensitive = true` flag prevents values from being printed in plan and apply output. It does not prevent them from being stored in state, but it stops them appearing in terminal output and logs:

```hcl
variable "db_password" {
  description = "Database administrator password"
  type        = string
  sensitive   = true
  # No default — Terraform will prompt or require TF_VAR_db_password
}

output "db_connection_string" {
  value     = "mysql://${aws_db_instance.example.username}@${aws_db_instance.example.endpoint}"
  sensitive = true
}
```

Mark every variable and output that carries a secret as `sensitive = true`. Terraform will show `(sensitive value)` in plan output instead of the actual value.

---

### 6. Protect the State File

Since secrets end up in state regardless, the state file itself must be secured. Verify all of the following are in place from Day 6:

```hcl
# S3 backend with encryption and restricted access
terraform {
  backend "s3" {
    bucket         = "your-terraform-state-bucket"
    key            = "production/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-locks"
    encrypt        = true  # AES-256 server-side encryption
  }
}
```

In addition, confirm your S3 bucket has:
- **Block all public access** enabled
- **Bucket versioning** enabled (to recover from accidental overwrites)
- **Bucket policy** restricting access to only the IAM roles that run Terraform
- **No `.terraform` directory committed** — add it to `.gitignore` along with `*.tfstate` and `*.tfstate.backup`

Your `.gitignore` for any Terraform project should always include:

```
# Terraform
.terraform/
.terraform.lock.hcl
*.tfstate
*.tfstate.backup
*.tfvars
override.tf
override.tf.json
```

---

### 7. Use Environment Variables for Provider Credentials

Never put AWS credentials in your Terraform configuration. Use environment variables instead — Terraform's AWS provider reads them automatically:

```bash
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="us-east-1"
```

In CI/CD systems, inject these from your secrets store (GitHub Actions secrets, AWS IAM roles, etc.) rather than hardcoding them in pipeline configuration files.

---

### 8. Bonus — Write an Advanced Secrets Management Guide

Write a comprehensive guide covering secrets management across multiple cloud environments. Structure it as a practical reference:

- The three leak paths and how to close each one
- AWS Secrets Manager integration pattern
- HashiCorp Vault integration pattern (for teams already using Vault)
- Environment variable handling for provider credentials
- State file security checklist
- `.gitignore` template for Terraform projects
- IAM policy for least-privilege state bucket access

Publish this on GitHub as a standalone repository or as a detailed blog post. This kind of reference guide gets bookmarked and shared widely in the DevOps community.

---

### 9. Write Your Blog Post

**Post title:** *How to Handle Sensitive Data Securely in Terraform*

Walk through all three leak paths with concrete before/after code examples. Show the AWS Secrets Manager integration in full. Cover `sensitive = true`. Include your state file security checklist. Make it the guide you wish you had on Day 1.

---

### 10. Post on Social Media

> 🔐 Day 13 of the 30-Day Terraform Challenge — secrets management deep dive. Three ways secrets leak in Terraform configurations, how to close every one of them, and why the state file is the last line of defence. Security is not optional in production infrastructure. #30DayTerraformChallenge #TerraformChallenge #Terraform #Security #DevOps #IaC #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL or your advanced guide GitHub repository URL.
2. **Live App Link field** — paste your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**The Three Leak Paths**
Explain each of the three secret leak paths in your own words. For each one, show the vulnerable code pattern and the secure alternative.

```hcl
# vulnerable pattern → secure alternative for each leak path
```

**AWS Secrets Manager Integration**
Paste your complete data source configuration and show how you referenced the secret in a resource. Confirm the secret value does not appear anywhere in your `.tf` files.

```hcl
# paste your secrets manager integration here
```

**Sensitive Variable and Output Declarations**
Paste your `sensitive = true` variable and output declarations. Show what Terraform displays in plan output when a sensitive value is involved.

**State File Security Audit**
Confirm your S3 backend has encryption, versioning, and restricted access. Paste the relevant parts of your backend configuration and describe the IAM policy restricting bucket access.

**`.gitignore` Contents**
Paste your complete Terraform `.gitignore`. Explain why each entry is there.

**Chapter 6 Learnings**
In your own words:
- Does `sensitive = true` prevent secrets from being stored in state? What does it actually do?
- What is the difference between HashiCorp Vault and AWS Secrets Manager and when would you use each?
- Why can you not fully prevent secrets from appearing in state for some resource types?

**Advanced Guide**
Paste the GitHub URL or blog URL of your advanced secrets management guide.

**Challenges and Fixes**
What issues came up with IAM permissions for Secrets Manager, `jsondecode` parsing, or sensitive output handling?

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [AWS Secrets Manager Documentation](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html)
- [Terraform Sensitive Values](https://developer.hashicorp.com/terraform/language/values/outputs#sensitive-suppressing-values-in-cli-output)
- [HashiCorp Vault Documentation](https://developer.hashicorp.com/vault/docs)
- [Terraform AWS Secrets Manager Data Source](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/secretsmanager_secret)
- [Protecting Sensitive Data in Terraform State](https://developer.hashicorp.com/terraform/language/state/sensitive-data)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
