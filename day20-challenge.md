# Day 20: Workflow for Deploying Application Code

## What You Will Accomplish Today

Before you can understand how to deploy infrastructure code at scale, you need to understand the deployment workflow that engineering teams already trust for application code — the seven-step process that takes a change from a developer's laptop to production safely. Today you map that workflow in detail, simulate it end-to-end, and then integrate Terraform Cloud as the platform that makes the infrastructure equivalent of this workflow possible. You will also explore secure variable management and the Terraform Cloud private registry.

---

## Tasks

### 1. Read

**Book:** *Terraform: Up & Running* by Yevgeniy Brikman — **Chapter 10**

Read the section *A Workflow for Deploying Application Code* in full. The seven steps are:

1. Use version control
2. Run the code locally
3. Make code changes
4. Submit changes for review
5. Run automated tests
6. Merge and release
7. Deploy

As you read, think about where each step has a direct equivalent in a Terraform workflow and where the analogy breaks down. That gap is what Chapter 10 is ultimately about.

---

### 2. Complete the Hands-On Labs

- **Lab 1:** [Terraform Enterprise](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2006%20-%20Navigate%20Terraform%20Workflow/02%20-%20Terraform_Enterprise.md)
- **Lab 2:** [Sentinel Policies](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2006%20-%20Navigate%20Terraform%20Workflow/03%20-%20Sentinel_Policies.md)

---

### 3. Simulate the Seven-Step Application Deployment Workflow

Walk through all seven steps using your webserver cluster as the application. Document each step as you complete it.

**Step 1 — Version control**
Your Terraform code should already be in a Git repository. Confirm your `main` branch is protected — no direct pushes, only merges via pull request.

**Step 2 — Run locally**
Make a change to your cluster — update the HTML response in your user data script from whatever version it currently shows to a new version. Run `terraform plan` locally to verify the change is exactly what you intended.

```bash
terraform plan -out=day20.tfplan
```

Save the plan output. Never apply a plan you have not reviewed.

**Step 3 — Make the code change**
Create a feature branch:

```bash
git checkout -b update-app-version-day20
# make your change
git add .
git commit -m "Update app response to v3 for Day 20"
git push origin update-app-version-day20
```

**Step 4 — Submit for review**
Open a pull request. Include the output of `terraform plan` as a comment so the reviewer can see exactly what will change in production without running Terraform themselves. This is the infrastructure equivalent of a code diff.

**Step 5 — Run automated tests**
Your GitHub Actions workflow from Day 18 should trigger automatically on the pull request. Confirm the unit tests pass before merging.

**Step 6 — Merge and release**
Merge the pull request to main. Tag the merge commit with a version:

```bash
git tag -a "v1.3.0" -m "Update app response to v3"
git push origin v1.3.0
```

**Step 7 — Deploy**
Apply the plan from the merge commit. Verify the change is live:

```bash
terraform apply day20.tfplan
curl http://<your-alb-dns>
```

Confirm the response shows your new version.

---

### 4. Set Up Terraform Cloud

Connect your Terraform configuration to Terraform Cloud to give your workflow a proper backend with built-in plan storage, team access controls, and an audit log.

**Create a Terraform Cloud workspace:**

```hcl
terraform {
  cloud {
    organization = "your-org-name"

    workspaces {
      name = "webserver-cluster-dev"
    }
  }
}
```

Run `terraform login` to authenticate, then `terraform init` to migrate your state to Terraform Cloud.

**Secure your variables in Terraform Cloud:**
Move your AWS credentials and sensitive Terraform variables out of environment variables on your local machine and into Terraform Cloud workspace variables:

- Add `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` as **environment variables** marked sensitive
- Add your Terraform variables (instance type, cluster name, environment) as **Terraform variables**

Once configured, runs triggered from Terraform Cloud will use these variables automatically — no credentials on any developer's machine.

---

### 5. Explore the Terraform Cloud Private Registry

The private registry lets your team publish and consume internal modules in the same way they consume public Registry modules — with versioning, documentation, and a consistent source URL.

Publish your webserver cluster module to the private registry:

1. Create a GitHub repository named `terraform-aws-webserver-cluster` (the naming convention is `terraform-<provider>-<name>`)
2. Tag a release: `git tag v1.0.0 && git push origin v1.0.0`
3. In Terraform Cloud, navigate to Registry → Publish → Module and connect the repository

Once published, your team can reference it like any public module:

```hcl
module "webserver_cluster" {
  source  = "app.terraform.io/your-org/webserver-cluster/aws"
  version = "1.0.0"

  cluster_name  = "prod-cluster"
  instance_type = "t2.medium"
  min_size      = 3
  max_size      = 10
  environment   = "production"
}
```

---

### 6. Map the Workflow Comparison

After completing the seven steps, fill in this comparison table documenting how each application code step maps to infrastructure code:

| Step | Application Code | Infrastructure Code | Key Difference |
|---|---|---|---|
| 1. Version control | Git for source code | Git for `.tf` files | State file is NOT in Git |
| 2. Run locally | `npm start` / `python app.py` | `terraform plan` | Plan shows what will change, not a running app |
| 3. Make changes | Edit source files | Edit `.tf` files | Changes affect real cloud resources |
| 4. Review | Code diff in PR | Plan output in PR | Reviewer must understand cloud resource implications |
| 5. Automated tests | Unit tests, linting | `terraform test`, Terratest | Infra tests deploy real resources and cost money |
| 6. Merge and release | Merge + tag | Merge + tag | Module consumers must pin to versions |
| 7. Deploy | CI/CD pipeline | `terraform apply` | Apply must be run from a trusted, locked environment |

---

### 7. Write Your Blog Post

**Post title:** *A Workflow for Deploying Application Code with Terraform*

Walk through all seven steps with your concrete examples. Cover where the application code and infrastructure code workflows align and where they diverge. Include the Terraform Cloud variable management and private registry sections — these are features many teams do not know exist and they solve real pain points.

---

### 8. Post on Social Media

> 🚀 Day 20 of the 30-Day Terraform Challenge — application deployment workflow mapped to Terraform. Seven steps from local change to production, Terraform Cloud for state and variable management, private registry for internal module sharing. Infrastructure as Code done properly looks exactly like good software engineering. #30DayTerraformChallenge #TerraformChallenge #Terraform #TerraformCloud #DevOps #IaC #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL or your Terraform Cloud workspace URL.
2. **Live App Link field** — paste the ALB DNS name showing your updated v3 response, or your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**Seven-Step Walkthrough**
Document each of the seven steps as you executed them. For each step paste the relevant command output, git log, or pull request screenshot description.

**Terraform Plan Output**
Paste the `terraform plan` output from Step 2 showing exactly what changed and confirming it matched your intent.

**Terraform Cloud Setup**
Paste your `terraform` block with the `cloud` configuration. Confirm state migrated successfully — describe what you see in the Terraform Cloud UI.

**Variable Configuration**
List the variables you configured in Terraform Cloud and which ones are marked sensitive. Explain why sensitive variables must never appear in `.tf` files or CI logs.

**Private Registry**
Describe your private registry setup. Paste the `source` URL your team would use to reference the module. What advantages does the private registry give over referencing a GitHub URL directly?

**Workflow Comparison Table**
Paste your completed comparison table. Which step has the biggest difference between application code and infrastructure code workflows and why?

**Chapter 10 Learnings**
In your own words — what is the most important insight from the application code deployment workflow that applies directly to how Terraform should be used? What breaks when teams skip any of the seven steps?

**Challenges and Fixes**
What issues came up with Terraform Cloud login, state migration, or workspace variable configuration?

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terraform Cloud Documentation](https://developer.hashicorp.com/terraform/cloud-docs)
- [Terraform Cloud Variable Management](https://developer.hashicorp.com/terraform/cloud-docs/workspaces/variables)
- [Terraform Cloud Private Registry](https://developer.hashicorp.com/terraform/cloud-docs/registry)
- [Terraform Login Command](https://developer.hashicorp.com/terraform/cli/commands/login)
- [Terraform Cloud Getting Started](https://developer.hashicorp.com/terraform/tutorials/cloud/cloud-sign-up)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
