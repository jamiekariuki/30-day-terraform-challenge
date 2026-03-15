# Day 6: Understanding and Managing Terraform State

## What You Will Accomplish Today

State is the heart of how Terraform works. Every `plan`, every `apply`, every `destroy` — all of it depends on Terraform's state being accurate. Today you will go deep on what state actually is, why local state breaks down the moment more than one person touches the same infrastructure, and how to configure remote state storage so your team can collaborate safely. This is one of the most certification-tested topics in the entire challenge.

---

## Tasks

### 1. Read

**Book:** *Terraform: Up & Running* by Yevgeniy Brikman — **Chapter 3**, pages 81–113

Read all three sections carefully:
- *What is Terraform State?*
- *Shared Storage for State Files*
- *Managing State Across Teams*

Take notes on the specific problems that arise when state is stored locally — concurrent runs, lost state, and secrets in plaintext. These come up directly in the certification exam.

---

### 2. Complete the Hands-On Labs

- **Lab 1:** [Output Values](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2004%20-%20Understand%20Terraform%20Basics/07%20-%20Intro_to_the_Output_Block.md)
- **Lab 2:** [State Management](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2003%20-%20Understand%20The%20Purpose%20of%20Terraform/02%20-%20Benefits_of_State.md)

Work through both fully. Pay attention to what outputs look like in the state file and how Terraform uses state to calculate diffs.

---

### 3. Deploy Infrastructure and Inspect the State File

Deploy any simple infrastructure — an S3 bucket, a security group, or reuse your server from previous days. After applying, open `terraform.tfstate` and read through it carefully.

Answer these questions from what you observe:
- What does Terraform store about each resource?
- Where are the resource attributes, IDs, and dependencies recorded?
- What happens to the state file after `terraform destroy`?

Run `terraform state list` to see a clean summary of what Terraform is tracking. Run `terraform state show <resource>` on one of your resources to inspect its full recorded attributes.

```
terraform state list
terraform state show aws_instance.example
```

---

### 4. Configure Remote State Storage with S3 and DynamoDB

This is the core activity for today. Move your state from local to remote using **AWS S3** as the backend and **DynamoDB** for state locking.

**Step 1 — Create the S3 bucket and DynamoDB table manually** (bootstrap problem: you cannot use Terraform to create the backend that Terraform itself needs):

```hcl
resource "aws_s3_bucket" "terraform_state" {
  bucket = "your-unique-terraform-state-bucket"

  lifecycle {
    prevent_destroy = true
  }
}

resource "aws_s3_bucket_versioning" "enabled" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "default" {
  bucket = aws_s3_bucket.terraform_state.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-state-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
```

**Step 2 — Configure the backend block** in your Terraform configuration:

```hcl
terraform {
  backend "s3" {
    bucket         = "your-unique-terraform-state-bucket"
    key            = "global/s3/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-locks"
    encrypt        = true
  }
}
```

**Step 3 — Migrate your state** by running `terraform init` again. Terraform will detect the new backend and offer to copy your existing local state to S3.

Confirm the migration worked by checking your S3 bucket in the AWS Console — your state file should be there, versioned and encrypted.

---

### 5. Test State Locking

Open two terminal windows pointing to the same Terraform configuration. In the first, run `terraform apply`. While it is running, attempt `terraform plan` in the second. Observe the lock error. Document what it says and why this protection matters in a team environment.

---

### 6. Write Your Blog Post

Choose one of these angles or combine both:

- **Option A:** *Managing Terraform State: Best Practices for DevOps* — cover what state is, why local state fails at scale, and how to set up S3 + DynamoDB correctly.
- **Option B:** *How to Securely Store Terraform State Files with Remote Backends* — focus on encryption, versioning, access control, and why state files can contain sensitive data.

Include your actual backend configuration code and explain every argument.

---

### 7. Post on Social Media

> 🗂️ Day 6 of the 30-Day Terraform Challenge — went deep on Terraform state today. Migrated from local state to a remote S3 backend with DynamoDB locking. If you are storing state locally in a team environment, you are one concurrent run away from disaster. #30DayTerraformChallenge #TerraformChallenge #Terraform #IaC #DevOps #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL or a link to your Terraform backend configuration code.
2. **Live App Link field** — paste your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**State File Observations**
Paste the output of `terraform state list` and `terraform state show` for one resource. Describe what you found inside the raw `terraform.tfstate` file — what surprised you about what Terraform stores?

**Remote Backend Configuration**
Paste your complete backend configuration including the S3 bucket, DynamoDB table, and the `terraform` block. Explain every argument and what it does.

```hcl
# paste your backend configuration here
```

**Migration Confirmation**
Describe the migration process. What did Terraform say when you ran `terraform init` with the new backend configured? Confirm the state file appeared in S3.

**State Locking Test**
Document your locking experiment. Paste the error message from the second terminal and explain why this protection is critical in team environments.

**Chapter 3 Learnings**
Answer the following in your own words:
- Why should `terraform.tfstate` never be committed to a Git repository?
- What is the bootstrap problem with remote backends and how do you solve it?
- What does state locking prevent?
- What does enabling versioning on the S3 bucket protect you from?

**Lab Takeaways**
What did the output values lab show you about how outputs are stored in state? How are they different from resource attributes?

**Challenges and Fixes**
What issues did you hit configuring the backend? IAM permission errors and bucket naming conflicts are common — document what happened and how you resolved it.

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terraform Remote State Documentation](https://developer.hashicorp.com/terraform/language/state/remote)
- [Terraform S3 Backend Configuration](https://developer.hashicorp.com/terraform/language/settings/backends/s3)
- [AWS S3 Versioning](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html)
- [AWS DynamoDB for Terraform State Locking](https://developer.hashicorp.com/terraform/language/settings/backends/s3#dynamodb-state-locking)
- [Terraform State CLI Commands](https://developer.hashicorp.com/terraform/cli/commands/state)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
