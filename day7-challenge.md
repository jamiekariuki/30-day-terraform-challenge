# Day 7: Terraform State Isolation — Workspaces vs File Layouts

## What You Will Accomplish Today

Yesterday you moved state to a remote backend. Today you solve the next problem: what happens when you need to manage multiple environments — dev, staging, and production — without them interfering with each other? Terraform gives you two approaches: **Workspaces** and **File Layouts**. You will implement both, understand exactly where each one shines, and develop a clear opinion on when to use which. This is a heavily tested topic in the Terraform Associate exam.

---

## Tasks

### 1. Read

**Book:** *Terraform: Up & Running* by Yevgeniy Brikman — **Chapter 3**, pages 81–113

Focus on these sections specifically:
- *State File Isolation*
- *Isolation via Workspaces*
- *Isolation via File Layouts*
- *The Remote State Data Source*

As you read, keep a mental note of the tradeoffs the author highlights between the two isolation approaches. You will need to articulate these clearly in your blog post and documentation.

---

### 2. Complete the Hands-On Labs

- **Lab 1:** [State Management](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2003%20-%20Understand%20The%20Purpose%20of%20Terraform/02%20-%20Benefits_of_State.md)
- **Lab 2:** [State Locking](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2003%20-%20Understand%20The%20Purpose%20of%20Terraform/02%20-%20Benefits_of_State.md)

---

### 3. State Isolation via Workspaces

Terraform Workspaces allow you to maintain multiple state files within the same backend and the same configuration directory. Each workspace is an isolated environment sharing the same code.

**Create and switch between workspaces:**

```
terraform workspace new dev
terraform workspace new staging
terraform workspace new production
terraform workspace list
terraform workspace select dev
```

Use `terraform.workspace` inside your configuration to make behaviour conditional on the environment:

```hcl
variable "instance_type" {
  description = "EC2 instance type per environment"
  type        = map(string)
  default = {
    dev        = "t2.micro"
    staging    = "t2.small"
    production = "t2.medium"
  }
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_type[terraform.workspace]

  tags = {
    Name        = "web-${terraform.workspace}"
    Environment = terraform.workspace
  }
}
```

Deploy to dev, then switch to staging and deploy there. Confirm that each workspace has a completely separate state file in your S3 bucket under different key paths.

---

### 4. State Isolation via File Layouts

File layout isolation uses a separate directory per environment, each with its own backend configuration pointing to a unique state file path. This is the approach recommended for production use cases.

Organise your project like this:

```
environments/
├── dev/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── backend.tf
├── staging/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── backend.tf
└── production/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    └── backend.tf
```

Each `backend.tf` points to a unique key in S3:

```hcl
# environments/dev/backend.tf
terraform {
  backend "s3" {
    bucket         = "your-terraform-state-bucket"
    key            = "environments/dev/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-locks"
    encrypt        = true
  }
}
```

```hcl
# environments/production/backend.tf
terraform {
  backend "s3" {
    bucket         = "your-terraform-state-bucket"
    key            = "environments/production/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-locks"
    encrypt        = true
  }
}
```

Run `terraform init` and `terraform apply` independently from each environment directory. Confirm that dev and production have completely separate state files and that changes in one directory have zero effect on the other.

---

### 5. Use the Remote State Data Source

Once you have separate state files per environment, use the `terraform_remote_state` data source to share outputs across configurations. For example, have your application layer read the VPC ID created by your networking layer:

```hcl
data "terraform_remote_state" "network" {
  backend = "s3"
  config = {
    bucket = "your-terraform-state-bucket"
    key    = "environments/dev/terraform.tfstate"
    region = "us-east-1"
  }
}

resource "aws_instance" "web" {
  subnet_id = data.terraform_remote_state.network.outputs.subnet_id
  # ...
}
```

---

### 6. Write Your Blog Post

**Post title:** *State Isolation: Workspaces vs File Layouts — When to Use Each*

Compare both approaches honestly. Cover the use cases where workspaces fall short (no code isolation, easy to accidentally apply to the wrong environment) and where file layouts add friction (more directory management, repeated backend config). Include a clear recommendation with reasoning. Your opinion matters — make it concrete.

---

### 7. Post on Social Media

> 🗂️ Day 7 of the 30-Day Terraform Challenge — state isolation deep dive. Implemented both Terraform Workspaces and File Layout isolation for multi-environment deployments. Knowing when to use each one is what separates good infrastructure from great infrastructure. #30DayTerraformChallenge #TerraformChallenge #Terraform #IaC #DevOps #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL or a link to your environment directory structure code.
2. **Live App Link field** — paste your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**Workspace Setup**
Paste the output of `terraform workspace list` showing all three environments. Show the different state file paths in your S3 bucket for each workspace.

**Workspace Configuration Code**
Paste the Terraform code where you used `terraform.workspace` to make instance type or naming conditional on the environment.

```hcl
# paste your workspace-aware configuration here
```

**File Layout Structure**
Show your directory tree and paste the `backend.tf` for at least two environments. Explain how the key paths differ and why that matters.

```hcl
# paste your file layout backend configs here
```

**Remote State Data Source**
Paste your `terraform_remote_state` data source configuration and show how you referenced an output from one state file in another configuration.

**Workspace vs File Layout Comparison**
Write a clear comparison in your own words. Use this as a guide:

- Which approach provides stronger environment isolation and why?
- Which is easier to accidentally misconfigure?
- Which scales better across a large team?
- Which would you use in a production environment and why?

**State Locking in Multi-Environment Context**
How does DynamoDB locking behave across workspaces? Is there a risk of two workspaces locking each other? What did you observe?

**Chapter 3 Learnings**
What is the Remote State data source and what problem does it solve? What are the limitations mentioned in the book?

**Challenges and Fixes**
What issues did you encounter switching between workspaces or initialising multiple environment directories?

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terraform Workspaces Documentation](https://developer.hashicorp.com/terraform/language/state/workspaces)
- [Terraform Remote State Data Source](https://developer.hashicorp.com/terraform/language/state/remote-state-data)
- [S3 Backend Configuration](https://developer.hashicorp.com/terraform/language/settings/backends/s3)
- [Terraform Recommended File Layout](https://developer.hashicorp.com/terraform/language/modules/develop/structure)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
