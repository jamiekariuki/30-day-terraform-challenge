# Day 22: Putting It All Together — Completing the Book and Reflecting on the Journey

## What You Will Accomplish Today

Today you finish the book. This is the chapter that ties every concept together — version control, testing, CI/CD, immutable artifacts, Terraform Cloud, Sentinel policies, and the full deployment workflow running as one coherent system. It is also a moment to step back and reflect: you have covered more ground in three weeks than most engineers cover in months of self-study. By the end of today you will have combined your application and infrastructure workflows into a single integrated pipeline and written an honest reflection on where you are and where you are going.

---

## Tasks

### 1. Read

**Book:** *Terraform: Up & Running* by Yevgeniy Brikman — **Chapter 10**

Read the final section: *Putting It All Together*. Focus on the side-by-side comparison of the application and infrastructure code workflows and how the author shows them converging into one unified process. Pay close attention to the concept of promoting **immutable, versioned artifacts** across environments — this is the key architectural insight of the entire chapter.

---

### 2. Complete the Hands-On Labs

- **Lab 1:** [Cost Estimation](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2006%20-%20Navigate%20Terraform%20Workflow/04%20-%20Cost_Estimation.md)
- **Lab 2:** [Security Best Practices](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2006%20-%20Navigate%20Terraform%20Workflow/05%20-%20Security_Best_Practices.md)

---

### 3. Build the Integrated Workflow

Combine everything you have built over the last three weeks into one complete, end-to-end workflow. This means:

**Version control for everything**
- Application code in Git with branch protection
- Infrastructure code in Git with branch protection
- Modules versioned and tagged
- State stored remotely with locking

**Automated CI pipeline (GitHub Actions)**
Your pipeline should run on every pull request and include all of these checks in sequence:

```yaml
name: Infrastructure CI

on:
  pull_request:
    branches: [main]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3

      - name: Format check
        run: terraform fmt -check -recursive

      - name: Init
        run: terraform init -backend=false

      - name: Validate
        run: terraform validate

      - name: Unit tests
        run: terraform test

  plan:
    runs-on: ubuntu-latest
    needs: validate
    env:
      AWS_ACCESS_KEY_ID:     ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3

      - name: Init
        run: terraform init

      - name: Plan
        run: terraform plan -out=ci.tfplan

      - name: Upload plan
        uses: actions/upload-artifact@v4
        with:
          name: terraform-plan
          path: ci.tfplan
```

**Immutable versioned artifacts**
Every successful build in CI produces an immutable artifact — in this case, the saved `terraform plan` file — that is promoted through environments without being regenerated. The same plan that was reviewed in staging is the exact plan that gets applied in production.

**Sentinel enforcement**
Apply at least two Sentinel policies in Terraform Cloud:
- One that enforces allowed instance types across all workspaces
- One that requires all resources to have a `ManagedBy = "terraform"` tag

```python
# sentinel/require-terraform-tag.sentinel
import "tfplan/v2" as tfplan

main = rule {
  all tfplan.resource_changes as _, rc {
    rc.change.after.tags["ManagedBy"] is "terraform"
  }
}
```

**Cost estimation gate**
Enable cost estimation in Terraform Cloud. Configure a Sentinel policy that blocks applies if the monthly cost increase exceeds a threshold:

```python
# sentinel/cost-check.sentinel
import "tfrun"

maximum_monthly_increase = 50.0

main = rule {
  tfrun.cost_estimate.delta_monthly_cost < maximum_monthly_increase
}
```

---

### 4. The Side-by-Side Comparison

Fill in this complete comparison table documenting how both workflows look in your final integrated system:

| Component | Application Code | Infrastructure Code |
|---|---|---|
| Source of truth | Git repository | Git repository |
| Local run | `npm start` / `python app.py` | `terraform plan` |
| Artifact | Docker image / binary | Saved `.tfplan` file |
| Versioning | Semantic version tag | Semantic version tag |
| Automated tests | Unit + integration tests | `terraform test` + Terratest |
| Policy enforcement | Linting / SAST | Sentinel policies |
| Cost gate | N/A | Cost estimation policy |
| Promotion | Image promoted across envs | Plan promoted across envs |
| Deployment | CI/CD pipeline | `terraform apply <plan>` |
| Rollback | Redeploy previous image | `terraform apply <previous plan>` |

---

### 5. Reflect on Your Journey

Write an honest reflection covering the following:

**What you built** — List every significant piece of infrastructure you deployed over the last 22 days. VPCs, EC2 instances, load balancers, ASGs, S3 buckets, EKS clusters, multi-region deployments. The list is longer than most engineers build in their first year.

**What changed in how you think** — Identify one specific thing you now think about differently when approaching infrastructure. Not a tool — a mental model.

**What was harder than expected** — Be specific. State management? Module design? Testing? The gap between dev and production behaviour?

**What you would do differently** — If you started this challenge again from Day 1, knowing what you know now, what would you do differently in the first week?

**What comes next** — After the exam: what is the first real project you want to apply this to?

---

### 6. Write Your Blog Post

**Post title:** *Putting It All Together: Application and Infrastructure Workflows with Terraform*

Cover the integrated pipeline, the immutable artifact promotion pattern, your Sentinel policies, and the cost estimation gate. Then add a genuine reflection section — your journey, what clicked, what broke, and what surprised you. This is one of the posts people actually read all the way through.

---

### 7. Post on Social Media

> 🎉 Day 22 of the 30-Day Terraform Challenge — finished the book. Combined application and infrastructure deployment workflows into one integrated pipeline with CI, Sentinel policies, cost gates, and immutable plan promotion across environments. 22 days in and it is just getting interesting. #30DayTerraformChallenge #TerraformChallenge #Terraform #DevOps #IaC #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL or your GitHub Actions workflow file link.
2. **Live App Link field** — paste your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**Integrated CI Pipeline**
Paste your complete GitHub Actions workflow. Confirm all jobs ran successfully — paste a link to or description of a passing workflow run.

**Sentinel Policies**
Paste both Sentinel policies. Explain what each one blocks and why that enforcement matters for your organisation.

**Cost Estimation Gate**
Describe your cost threshold and what Terraform Cloud shows in the cost estimation section of a recent run.

**Side-by-Side Comparison Table**
Paste your completed comparison table with your own entries for each row.

**Journey Reflection**
Answer all five reflection questions honestly. This is the most important part of your submission today — make it real.

**Chapter 10 Final Learnings**
What is the single most important insight from Chapter 10 that you will carry into your next infrastructure project?

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terraform Associate Study Guide](https://developer.hashicorp.com/terraform/tutorials/certification-003/associate-study-003)
- [Sentinel Documentation](https://developer.hashicorp.com/sentinel/docs)
- [Terraform Cloud Cost Estimation](https://developer.hashicorp.com/terraform/cloud-docs/cost-estimation)
- [GitHub Actions Artifacts](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
