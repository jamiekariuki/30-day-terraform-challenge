# Day 23: Exam Preparation — Brushing Up on Key Terraform Concepts

## What You Will Accomplish Today

The book is done. The builds are running. Now it is time to shift focus entirely toward the certification exam. Today you audit yourself honestly against the official exam objectives, identify exactly where your gaps are, and build a structured study plan for the remaining days. You will also review specific Terraform features — provider aliases, autocomplete, and non-cloud providers — that appear in the exam but may not have come up prominently in your hands-on work so far.

---

## Tasks

### 1. Review the Official Exam Documentation

Start here and work through the entire page before doing anything else:

[Terraform Associate Certification Study Guide](https://developer.hashicorp.com/terraform/tutorials/certification-003/associate-study-003)

This page is the authoritative source of what is tested. Print it or keep it open. Every topic listed there is fair game on the exam.

The exam covers six domains:

| Domain | Weight |
|---|---|
| Understand Infrastructure as Code concepts | 16% |
| Understand Terraform's purpose | 20% |
| Understand Terraform basics | 24% |
| Use the Terraform CLI | 26% |
| Interact with Terraform modules | 12% |
| Navigate the core Terraform workflow | 8% |
| Implement and maintain state | 8% |
| Read, generate, and modify configuration | 8% |
| Understand Terraform Cloud capabilities | 4% |

Work through each domain. For every topic, honestly rate yourself:
- **Green** — I can explain this and have done it hands-on
- **Yellow** — I understand it conceptually but have not done it hands-on
- **Red** — I am not confident about this

---

### 2. Review Key Exam Topics You May Have Missed

Several topics are heavily tested but do not appear prominently in the book's hands-on exercises. Review all of these today:

**Terraform CLI commands — know every flag**

```bash
# Commands you must know cold
terraform init        # Downloads providers, configures backend
terraform validate    # Checks syntax and internal consistency
terraform fmt         # Formats code to canonical style
terraform plan        # Shows proposed changes
terraform apply       # Creates or updates infrastructure
terraform destroy     # Removes infrastructure
terraform output      # Reads output values from state
terraform state list  # Lists resources in state
terraform state show  # Shows attributes of a resource in state
terraform state mv    # Moves a resource within or between states
terraform state rm    # Removes a resource from state without destroying it
terraform import      # Imports existing infrastructure into state
terraform taint       # (deprecated) Marks resource for recreation
terraform workspace   # Manages workspaces
terraform providers   # Shows providers used in config
terraform login       # Authenticates to Terraform Cloud
terraform graph       # Outputs dependency graph
```

Know what each command does, when you would use it, and its key flags. The exam tests this at the level of "what does `terraform state rm` do to the real infrastructure?" (Answer: nothing — it only removes the resource from state, leaving the real resource untouched.)

**Provider aliases — review the syntax**

```hcl
provider "aws" {
  region = "us-east-1"
}

provider "aws" {
  alias  = "west"
  region = "us-west-2"
}

resource "aws_s3_bucket" "west_bucket" {
  provider = aws.west
  bucket   = "my-west-bucket"
}
```

**Non-cloud providers — Terraform manages more than just cloud resources**

Terraform has providers for DNS, TLS certificates, random values, local files, and more. The `random` and `local` providers appear frequently in exam questions:

```hcl
resource "random_id" "example" {
  byte_length = 8
}

resource "random_password" "db" {
  length  = 16
  special = true
}

resource "local_file" "example" {
  content  = "Hello from Terraform"
  filename = "${path.module}/output.txt"
}
```

**Terraform Cloud capabilities**
- Remote runs vs local runs
- Workspaces and their relationship to environments
- Sentinel policies and when they run in the workflow
- Variable types: Terraform variables vs environment variables, sensitive vs non-sensitive
- The private registry and how module versioning works there
- Cost estimation and its limitations

**`terraform.tfstate` internals**
Know what the state file contains, why it exists, and specifically what happens to it during each command. The exam frequently asks scenario questions: "A team member manually deleted an S3 bucket that Terraform manages. What happens when you run `terraform plan`?" (Answer: Terraform detects the drift and shows the bucket as a resource to be created.)

---

### 3. Build Your Personal Study Plan

Based on your green/yellow/red audit, create a specific study plan for Days 24 through the exam. Be concrete — do not write "review state management." Write "re-read the state commands section of the docs, run `terraform state mv` and `terraform state rm` against a test resource, and write three practice questions about state."

**Template for each study session:**

| Topic | Current confidence | Study method | Time needed |
|---|---|---|---|
| `terraform state` commands | Yellow | Run each command against test infra | 45 min |
| Sentinel policy syntax | Red | Read docs + write two policies | 60 min |
| Workspace vs file layout isolation | Green | Write one practice question | 15 min |

---

### 4. Run Through Practice Questions

The official HashiCorp sample questions are the best proxy for exam difficulty:
[Official Sample Questions](https://learn.hashicorp.com/tutorials/terraform/associate-questions)

Work through all of them. For every question you get wrong, add it to your study plan.

Additionally, write five practice questions of your own based on the infrastructure you have built in this challenge. Writing questions is one of the most effective study techniques — it forces you to understand the material well enough to construct a plausible wrong answer.

---

### 5. Write Your Blog Post

**Post title:** *Preparing for the Terraform Associate Exam — Key Resources and Tips*

Share your self-audit approach, the domains you found most challenging, your study plan structure, and specific tips for the CLI commands section which most people underestimate. Include a link to the official study guide.

---

### 6. Post on Social Media

> 🎯 Day 23 of the 30-Day Terraform Challenge — full exam prep mode. Audited every objective domain, built a structured study plan, ran through the official practice questions. The CLI commands section is more detailed than most people expect. #30DayTerraformChallenge #TerraformChallenge #Terraform #TerraformAssociate #CertificationPrep #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL.
2. **Live App Link field** — paste your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**Domain Audit**
List all exam domains and your green/yellow/red rating for each. Be honest — yellow and red are where your remaining study time goes.

**Your Study Plan**
Paste your completed study plan table covering Days 24 through your exam date. Include specific study methods and time estimates.

**CLI Commands Self-Test**
For each of the 15 CLI commands listed above, write one sentence describing what it does and one sentence describing a scenario where you would use it. No copy-pasting from docs — your own words only.

**Non-Cloud Provider Code**
Paste a working example using the `random` or `local` provider. Explain where these providers are useful in real Terraform configurations.

**Practice Questions**
Write your five original practice questions with answer choices and explanations. Mark the correct answer and explain why each wrong answer is wrong.

**Official Practice Question Results**
How many of the official sample questions did you get right on your first attempt? Which topics did you miss and what did you learn from reviewing the correct answers?

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terraform Associate Study Guide](https://developer.hashicorp.com/terraform/tutorials/certification-003/associate-study-003)
- [Official Sample Questions](https://learn.hashicorp.com/tutorials/terraform/associate-questions)
- [Terraform CLI Commands Reference](https://developer.hashicorp.com/terraform/cli/commands)
- [Terraform Random Provider](https://registry.terraform.io/providers/hashicorp/random/latest/docs)
- [Terraform Associate Exam Review](https://developer.hashicorp.com/terraform/tutorials/certification-003/associate-review-003)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
