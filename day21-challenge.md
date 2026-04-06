# Day 21: Workflow for Deploying Infrastructure Code

## What You Will Accomplish Today

Yesterday you mapped the seven-step application code deployment workflow in detail. Today you apply that same framework to infrastructure code — and discover exactly where the two workflows diverge and why those differences matter. Infrastructure code deployment has unique challenges that application code does not: state files, blast radius, approval gates, and the fact that a bad deploy can destroy production databases rather than just return a 500 error. By the end of today you will have run a complete infrastructure deployment workflow end-to-end and understood the specific safeguards that make it safe to do at scale.

---

## Tasks

### 1. Read

**Book:** *Terraform: Up & Running* by Yevgeniy Brikman — **Chapter 10**

Read the section *A Workflow for Deploying Infrastructure Code* in full. Map each of the seven steps against what you built yesterday and identify every place the author says the infrastructure workflow must handle something differently from the application workflow.

---

### 2. Complete the Hands-On Labs

- **Lab 1:** [Sentinel Policies](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2006%20-%20Navigate%20Terraform%20Workflow/03%20-%20Sentinel_Policies.md)
- **Lab 2:** [Cost Estimation](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2006%20-%20Navigate%20Terraform%20Workflow/04%20-%20Cost_Estimation.md)

---

### 3. The Infrastructure Code Workflow — Step by Step

Work through all seven steps for a real infrastructure change to your webserver cluster. This is not a simulation — deploy something real, review it properly, and apply it through the full workflow.

**Step 1 — Version control**
Your Terraform code is already in Git. Verify your branch protection rules are in place: main branch requires at least one reviewer approval, status checks must pass before merging, and no direct pushes are allowed.

**Step 2 — Run the code locally**
The infrastructure equivalent of running code locally is running `terraform plan` against your personal sandbox environment. You are not running a binary — you are generating a diff against your state file. Always run plan before any change, every time, without exception:

```bash
terraform workspace select dev
terraform plan -out=day21.tfplan
```

Review the plan output carefully. Count the resources to be created, modified, and destroyed. Any destruction of existing resources should trigger extra scrutiny.

**Step 3 — Make code changes**
Create a feature branch and make a meaningful infrastructure change. Suggested change: add a CloudWatch alarm to your cluster, or add a new output variable exposing the ASG name.

```bash
git checkout -b add-cloudwatch-alarms-day21
# make your change
terraform plan -out=day21.tfplan
# review the plan output one more time
git add .
git commit -m "Add CPU alarm for webserver cluster"
git push origin add-cloudwatch-alarms-day21
```

**Step 4 — Submit for review**
Open a pull request. In the PR description, paste the full output of `terraform plan`. This is the infrastructure equivalent of a code diff — your reviewer must be able to understand exactly what will change in production from the PR alone, without running Terraform themselves.

Your PR description template for infrastructure changes:

```
## What this changes
[Brief description of the infrastructure change]

## Terraform plan output
[paste terraform plan output here]

## Resources affected
- Created: X
- Modified: Y
- Destroyed: Z

## Blast radius
[What breaks if this apply fails partway through?]

## Rollback plan
[How do you revert this if it causes an incident?]
```

**Step 5 — Run automated tests**
Your GitHub Actions workflow should run `terraform validate`, `terraform fmt --check`, and your `terraform test` unit tests automatically on the PR. The workflow must be green before the PR is eligible for merge.

**Step 6 — Merge and release**
Once approved and tests pass, merge. For infrastructure modules, tag a new version:

```bash
git tag -a "v1.4.0" -m "Add CPU alarm for webserver cluster"
git push origin v1.4.0
```

Update any environment configurations that should consume the new module version.

**Step 7 — Deploy**
Apply the saved plan from Step 2. Using the saved plan file guarantees that exactly what was reviewed is what gets applied — no surprises:

```bash
terraform apply day21.tfplan
```

Verify the change in AWS. Confirm CloudWatch shows your new alarm. Run `terraform plan` again immediately after and confirm it returns clean.

---

### 4. Infrastructure-Specific Safeguards

The following safeguards have no equivalent in application code deployment. Implement all of them:

**Approval gates for destructive changes**
If `terraform plan` shows any resource destructions, require a second explicit approval before applying — separate from the PR review. Configure this in Terraform Cloud as a mandatory apply approval step.

**Plan file pinning**
Always apply from a saved plan file, never from a fresh plan. The gap between `terraform plan` and `terraform apply` can introduce drift if infrastructure changes between the two:

```bash
# Correct — apply exactly what was reviewed
terraform plan -out=reviewed.tfplan
terraform apply reviewed.tfplan

# Risky — the plan may differ from what was reviewed
terraform apply
```

**State backup before apply**
Before any significant apply, verify your S3 state bucket has versioning enabled. Know how to restore a previous state version if an apply corrupts state:

```bash
# List available state versions in S3
aws s3api list-object-versions \
  --bucket your-terraform-state-bucket \
  --prefix production/terraform.tfstate
```

**Blast radius documentation**
Every PR that touches shared infrastructure (VPCs, security groups, IAM roles) must document what other resources depend on it and what breaks if the apply fails midway.

---

### 5. Explore Sentinel Policies

Sentinel is Terraform Cloud's policy-as-code framework. It enforces rules on every plan before apply is permitted. Write a Sentinel policy that prevents deployments to production unless the plan has been reviewed:

```python
# sentinel/require-instance-type.sentinel
import "tfplan/v2" as tfplan

allowed_instance_types = ["t2.micro", "t2.small", "t2.medium", "t3.micro", "t3.small"]

main = rule {
  all tfplan.resource_changes as _, rc {
    rc.type is not "aws_instance" or
    rc.change.after.instance_type in allowed_instance_types
  }
}
```

Document what Sentinel enforces in your submission and how it differs from `terraform validate`.

---

### 6. Write Your Blog Post

**Post title:** *A Workflow for Deploying Infrastructure Code with Terraform*

Cover all seven steps with your concrete examples, the infrastructure-specific safeguards that have no application code equivalent, and Sentinel policies as the enforcement layer. Make the blast radius and rollback plan sections prominent — this is where most engineers underinvest.

---

### 7. Post on Social Media

> 🔧 Day 21 of the 30-Day Terraform Challenge — infrastructure deployment workflow. Plan files, approval gates, blast radius documentation, Sentinel policies. Deploying infrastructure code safely requires discipline that goes well beyond what application deployments need. #30DayTerraformChallenge #TerraformChallenge #Terraform #DevOps #IaC #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL or PR link.
2. **Live App Link field** — paste your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**The Full Seven-Step Walkthrough**
Document each step as you executed it. For Steps 2, 4, and 7 paste the relevant plan output, PR description, and apply confirmation.

**PR Description**
Paste the full PR description you wrote using the template above including the plan output and blast radius section.

**Safeguards Implemented**
Confirm which of the four infrastructure-specific safeguards you implemented. For plan file pinning, paste the commands showing you applied from a saved plan file.

**Sentinel Policy**
Paste your Sentinel policy and explain in plain English what it enforces and what it would block.

**Infrastructure vs Application Workflow — Key Differences**
List the three most significant differences between the two workflows in your own words and explain why each one exists.

**Chapter 10 Learnings**
What does the author identify as the most dangerous step in the infrastructure deployment workflow and why? What safeguard does he recommend that most teams skip?

**Challenges and Fixes**
What issues came up with plan file handling, Sentinel configuration, or approval gate setup?

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terraform Cloud Run Approvals](https://developer.hashicorp.com/terraform/cloud-docs/run/manage)
- [Sentinel Policy Language](https://developer.hashicorp.com/sentinel/docs/language)
- [Terraform Plan Command](https://developer.hashicorp.com/terraform/cli/commands/plan)
- [S3 Object Versioning for State Recovery](https://developer.hashicorp.com/terraform/language/settings/backends/s3)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
