# Day 29: Exam Preparation — Practice Exams 3 & 4

## What You Will Accomplish Today

Two more full practice exams — but today the work after the exams is as important as the exams themselves. You now have data from four practice runs. Your job today is to turn that data into a precise picture of exactly where you stand and exactly what you need to do before tomorrow. By the end of today you should know with confidence whether you are ready to book your exam, and if you are not, you should know precisely what the gaps are.

---

## Tasks

### 1. Take Practice Exam 3

Same conditions as Days 28 — 60-minute timer, 57 questions, no reference material during the exam. Use a fresh question source if possible.

After the exam, score immediately and record wrong answers before your memory of your reasoning fades. The sooner you do the wrong-answer analysis, the more accurately you can diagnose why you got each question wrong.

---

### 2. Take Practice Exam 2

15-minute break, then Exam 4. A fresh set of questions, same conditions, same discipline.

---

### 3. Four-Exam Score Trend Analysis

You now have four data points. Plot your scores:

| Exam | Score | % | Time Taken | Notes |
|---|---|---|---|---|
| Exam 1 (Day 28) | /57 | % | min | |
| Exam 2 (Day 28) | /57 | % | min | |
| Exam 3 (Today) | /57 | % | min | |
| Exam 4 (Today) | /57 | % | min | |

Analyse the trend honestly:

- Are your scores consistently above 70%? You are likely ready.
- Are your scores improving each time? Keep that momentum going into Day 30.
- Are your scores plateauing below 70%? You have a specific knowledge gap, not a practice gap — identify it precisely and address it hands-on today.
- Are your scores inconsistent (80%, 65%, 75%, 62%)? You have knowledge that is not yet solid. Inconsistency on practice exams predicts exam-day anxiety.

---

### 4. Deep Dive on Persistent Wrong Answers

Compare your wrong answers across all four exams. Any topic that has appeared in your wrong answers more than once is a genuine gap, not a one-off miss. Address those specifically today.

**Common persistent gaps for Terraform Associate candidates — check each one:**

*State management traps*
- `terraform state rm` removes from state only, does NOT destroy the resource
- `terraform import` brings a resource into state but does NOT generate the `.tf` configuration
- After `terraform import`, running `terraform plan` will show differences if your configuration does not match the actual resource
- State locking prevents concurrent `apply` operations but does NOT prevent concurrent `plan` operations

*Workspace behaviour*
- Each workspace has its own state file
- `terraform.workspace` returns the current workspace name as a string
- The default workspace cannot be deleted
- Workspaces in Terraform Cloud are not the same concept as CLI workspaces

*Provider and module versioning*
- `~> 1.0` means `>= 1.0.0` and `< 2.0.0`
- `~> 1.0.0` means `>= 1.0.0` and `< 1.1.0` — the last digit is the one that can increment
- `>= 1.0, < 2.0.0` is equivalent to `~> 1.0`
- Module `source` with no `version` always pulls latest — never do this in production

*Terraform workflow commands in the right order*
```
terraform init → terraform validate → terraform plan → terraform apply → terraform destroy
```
- `terraform fmt` can run at any point — it only reformats code
- `terraform validate` requires `terraform init` to have run first (providers must be installed)
- `terraform refresh` is now `terraform apply -refresh-only` in Terraform 0.15.4+

*Lifecycle rules*
- `create_before_destroy = true` creates the replacement before destroying the original
- `prevent_destroy = true` prevents `terraform destroy` from removing the resource — it does NOT prevent manual deletion from the cloud console
- `ignore_changes` accepts a list of attribute names that Terraform should not track for drift

---

### 5. Targeted Hands-On Revision

For any topic that appeared in your wrong answers more than once, run a hands-on exercise. Suggested exercises based on the most commonly missed topics:

**State commands practice:**
```bash
# Deploy a simple resource
echo 'resource "random_id" "test" { byte_length = 4 }' > main.tf
terraform init && terraform apply -auto-approve

# Practice state commands
terraform state list
terraform state show random_id.test

# Remove from state without destroying
terraform state rm random_id.test
terraform state list  # confirms it is gone from state
# The resource is gone from state but would still "exist" in a real scenario

# Clean up
terraform destroy -auto-approve
```

**Workspace practice:**
```bash
terraform workspace new dev
terraform workspace new staging
terraform workspace list
terraform workspace select dev
echo $TF_WORKSPACE  # or terraform workspace show
terraform workspace delete staging  # must not be on the workspace you are deleting
```

**Provider version constraint practice:**
Write three `required_providers` blocks using different constraint operators and describe in plain English what version range each one allows.

---

### 6. Write Your Blog Post

**Post title:** *Fine-tuning My Terraform Exam Prep with Practice Exams*

Cover your four-exam score trend, the persistent wrong-answer patterns you identified, and the specific hands-on exercises you ran to close those gaps. Include your domain accuracy breakdown across all four exams. This is the kind of detailed, honest exam prep content that is genuinely useful to the community.

---

### 7. Post on Social Media

> ⚡️ Day 29 of the 30-Day Terraform Challenge — four practice exams in two days. Identified my persistent gaps, ran targeted hands-on exercises to close them. One day left before the final push. #30DayTerraformChallenge #TerraformChallenge #Terraform #TerraformAssociate #CertificationPrep #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL.
2. **Live App Link field** — paste your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**Four-Exam Score Table**
Paste your completed score trend table. Write a brief analysis of the trend — are you improving, plateauing, or inconsistent?

**Readiness Assessment**
Based on four practice exams, give yourself an honest readiness rating: Ready / Nearly Ready / Need More Work. Explain your reasoning with specific evidence from your scores.

**Persistent Wrong-Answer Topics**
List every topic that appeared in your wrong answers more than once across the four exams. For each one, write a concise explanation of the concept in your own words — prove to yourself that you now understand it.

**Hands-On Exercises**
Paste the commands you ran and their output. Explain what each exercise reinforced.

**Provider Version Constraint Practice**
Paste your three `required_providers` examples with plain-English explanations of the version range each one allows.

**Final Study Priority List**
Based on all four exams, list your top five topics to review on Day 30 in priority order. Be specific — not "state management" but "the difference between `terraform state rm` and `terraform destroy` and what each one does to the actual infrastructure."

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terraform Workspace CLI Documentation](https://developer.hashicorp.com/terraform/cli/commands/workspace)
- [Terraform State CLI Documentation](https://developer.hashicorp.com/terraform/cli/commands/state)
- [Provider Version Constraints](https://developer.hashicorp.com/terraform/language/expressions/version-constraints)
- [Lifecycle Meta-Arguments](https://developer.hashicorp.com/terraform/language/meta-arguments/lifecycle)
- [Terraform Associate Review](https://developer.hashicorp.com/terraform/tutorials/certification-003/associate-review-003)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
