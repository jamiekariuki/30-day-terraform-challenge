# Day 19: Adopting Infrastructure as Code in Your Team

## What You Will Accomplish Today

Writing Terraform code is the easy part. Getting a team, a department, or an organisation to adopt Infrastructure as Code is where most practitioners actually struggle. Today shifts from technical implementation to people, process, and strategy. You will read Chapter 10's guidance on IaC adoption, reflect honestly on the infrastructure culture in your own organisation, and build a concrete, incremental adoption plan that you could actually present to a team or manager. This is some of the most career-relevant content in the entire challenge.

---

## Tasks

### 1. Read

**Book:** *Terraform: Up & Running* by Yevgeniy Brikman — **Chapter 10**

Read the section *Adopting Infrastructure as Code in Your Team* in full. Focus on:
- The author's framework for convincing leadership to invest in IaC
- The case for working incrementally rather than attempting a full migration at once
- How to give your team the time and safety to learn without breaking production
- The cultural shift required alongside the technical one

---

### 2. Complete the Hands-On Labs

- **Lab 1:** [Terraform Cloud](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2006%20-%20Navigate%20Terraform%20Workflow/01%20-%20Terraform_Cloud.md)
- **Lab 2:** [Terraform Enterprise](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2006%20-%20Navigate%20Terraform%20Workflow/02%20-%20Terraform_Enterprise.md)

---

### 3. Reflect on Your Own Organisation

Before writing your adoption plan, do an honest audit of where your current team or organisation sits on the IaC maturity curve. Answer these questions in your documentation:

**Current state assessment**
- How is infrastructure currently provisioned — manual console clicks, scripts, existing IaC tools, or a mix?
- How many people are involved in infrastructure changes and what is the approval process?
- How often do infrastructure changes cause incidents or unexpected behaviour?
- Is there an existing drift between documented infrastructure and actual infrastructure?
- Are secrets managed properly or are credentials shared, hardcoded, or stored insecurely?

**Team readiness**
- What is the team's familiarity with version control for infrastructure (not just application code)?
- Is there executive or management appetite for a tooling change?
- What would it take for the team to trust automated deployments?

Be specific. Vague answers produce vague plans.

---

### 4. Build Your IaC Adoption Plan

Structure your plan in four phases. Each phase should be small enough to show results within 2–4 weeks and should not require stopping existing work to complete.

**Phase 1 — Start with something new**
Do not migrate existing infrastructure first. Pick one new piece of infrastructure — a new S3 bucket, a new IAM role, a monitoring dashboard — and provision it entirely with Terraform. This creates a success story with zero migration risk.

Deliverables for Phase 1:
- Terraform configuration for the chosen resource
- Remote state configured in S3
- Code reviewed and merged via pull request
- Team members able to run `terraform plan` and understand the output

**Phase 2 — Import existing infrastructure**
Once the team is comfortable with the workflow, begin importing critical existing resources into Terraform management. Use `terraform import` to bring them under state management without recreating them.

```bash
# Example: import an existing S3 bucket
terraform import aws_s3_bucket.existing_logs my-existing-logs-bucket

# Example: import an existing security group
terraform import aws_security_group.existing sg-0abc123def456789
```

Prioritise resources that change frequently or have caused incidents. Do not try to import everything at once.

**Phase 3 — Establish team practices**
Once multiple engineers are writing Terraform, establish the practices that prevent chaos:
- Module versioning and the internal module registry
- Code review requirements for all infrastructure changes
- `terraform plan` output as a required part of every PR
- Automated `terraform validate` and `terraform fmt` in CI
- State locking enforced via DynamoDB
- No manual console changes to Terraform-managed resources — ever

**Phase 4 — Automate deployments**
Connect Terraform to your CI/CD pipeline so that merges to main trigger `terraform apply` automatically. At this stage infrastructure changes go through the same review and deployment process as application code.

---

### 5. The Business Case for IaC

If you need to convince leadership, frame the argument around outcomes they care about — not technology:

| Business Problem | IaC Solution | Measurable Outcome |
|---|---|---|
| Infrastructure incidents from manual errors | Code review catches mistakes before apply | Fewer production incidents |
| Hours spent on repetitive environment setup | Reusable modules provision environments in minutes | Engineering time freed for product work |
| Inability to audit what changed and when | Every change is a git commit with author and timestamp | Full audit trail for compliance |
| Developer environments differ from production | Identical Terraform configs for all environments | Fewer "works on my machine" incidents |
| Difficulty onboarding new engineers to infra | Documented, version-controlled configurations | Faster onboarding |

Build a version of this table specific to your organisation using real numbers where you can estimate them.

---

### 6. Write Your Blog Post

**Post title:** *How to Convince Your Team to Adopt Infrastructure as Code*

Cover the business case, the incremental adoption strategy, the team practices that need to accompany the technical change, and the common failure modes (trying to migrate everything at once, underestimating the learning curve, not getting buy-in before starting). Make it honest — share what you have seen or experienced, not just what the textbook says.

---

### 7. Post on Social Media

> 🚀 Day 19 of the 30-Day Terraform Challenge — IaC adoption strategy. The technical part of Terraform is the easy part. Convincing a team to change how they work, building trust in automated deployments, migrating existing infrastructure incrementally — that is where the real challenge is. #30DayTerraformChallenge #TerraformChallenge #Terraform #IaC #DevOps #PlatformEngineering #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL.
2. **Live App Link field** — paste your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**Current State Assessment**
Answer the six assessment questions honestly for your own team or organisation. Be specific — vague answers are not useful for planning.

**Your Four-Phase Adoption Plan**
Write out your plan in full. For each phase include: what gets done, who does it, what the success criteria are, and approximately how long it should take.

**The Business Case Table**
Build your version of the business case table with problems and outcomes specific to your organisation. Include estimated numbers where you can.

**`terraform import` Practice**
Import at least one existing resource into a Terraform configuration. Paste the import command, the resource block you wrote to match it, and the output of `terraform plan` showing no changes after the import.

```hcl
# paste the resource block for your imported resource
```

**Terraform Cloud Lab Takeaways**
What did the Terraform Cloud lab demonstrate? What does Terraform Cloud provide that a plain S3 backend does not?

**Chapter 10 Learnings**
What does the author identify as the most common reason IaC adoption fails in organisations? Do you agree based on your own experience? What would you add?

**Challenges**
What is the hardest part of IaC adoption in your specific context — technical, organisational, or cultural? Be honest.

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terraform Cloud Documentation](https://developer.hashicorp.com/terraform/cloud-docs)
- [Terraform Import Command](https://developer.hashicorp.com/terraform/cli/commands/import)
- [IaC Maturity Model](https://developer.hashicorp.com/terraform/tutorials/cloud/cloud-sign-up)
- [HashiCorp State of Cloud Strategy Report](https://www.hashicorp.com/state-of-the-cloud)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
