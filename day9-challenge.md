# Day 9: Advanced Terraform Modules — Versioning, Gotchas, and Multi-Environment Reuse

## What You Will Accomplish Today

Yesterday you built your first module and called it from a root configuration. Today you go deeper. You will learn the rough edges that catch engineers off guard — the **module gotchas** that cause subtle bugs in real deployments — and then solve the most important problem in module management: **versioning**. By the end of today your module will be version-pinned, deployed differently across environments, and ready to be shared with a team. This is the pattern used by every serious Terraform codebase at scale.

---

## Tasks

### 1. Read

**Book:** *Terraform: Up & Running* by Yevgeniy Brikman — **Chapter 4**, pages 115–139

Focus specifically on:
- *Module Gotchas* — file paths, inline blocks, and the dangers of modules that create resources with side effects
- *Module Versioning* — how to pin modules to specific versions using Git tags or the Terraform Registry

Read slowly. The gotchas section covers mistakes that are easy to make and hard to debug without knowing they exist.

---

### 2. Complete the Hands-On Labs

- **Lab 1:** [Terraform Workspaces](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2005%20-%20Use%20Terraform%20Outside%20of%20Core%20Workflow/01%20-%20Terraform_Workspaces.md)
- **Lab 2:** [Terraform Modules](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2004%20-%20Understand%20Terraform%20Basics/11%20-%20Intro_to_Modules.md)

---

### 3. Learn the Module Gotchas

Before enhancing your module, understand the three most common module mistakes covered in Chapter 4:

**Gotcha 1 — File paths inside modules**
If your module references a file using a relative path like `./user-data.sh`, that path is resolved relative to where Terraform is run — not relative to the module itself. Always use the `path.module` expression for files inside a module:

```hcl
user_data = templatefile("${path.module}/user-data.sh", {
  server_port = var.server_port
})
```

**Gotcha 2 — Inline blocks vs separate resources**
Some resources support both an inline block and a separate resource for the same configuration (e.g. `ingress` blocks inside `aws_security_group` vs standalone `aws_security_group_rule`). Mixing both in a module causes conflicts. Pick one pattern and stick to it — separate resources are more flexible for modules because callers can add rules without modifying the module.

**Gotcha 3 — Module output dependencies**
If your root configuration references a module output in a `depends_on`, Terraform evaluates the entire module as a dependency, not just the specific resource. This can cause unnecessary resource recreation. Structure your modules to expose granular outputs rather than forcing callers to depend on the whole module.

Document each of these in your submission with a concrete example of what goes wrong and how to avoid it.

---

### 4. Add Versioning to Your Module

Push your module from Day 8 to a GitHub repository and tag it with a version number:

```
git init
git add .
git commit -m "Initial module release"
git tag -a "v0.0.1" -m "First release of webserver-cluster module"
git remote add origin https://github.com/your-username/terraform-aws-webserver-cluster
git push origin main --tags
```

Now update your calling configurations to reference the versioned source instead of a local path:

```hcl
# live/dev/services/webserver-cluster/main.tf
module "webserver_cluster" {
  source = "github.com/your-username/terraform-aws-webserver-cluster?ref=v0.0.1"

  cluster_name  = "webservers-dev"
  instance_type = "t2.micro"
  min_size      = 2
  max_size      = 4
}
```

---

### 5. Deploy Multiple Versions Across Environments

Make a meaningful change to your module — add a new input variable, change a default, or add a new output. Commit and tag it as `v0.0.2`.

Now configure your environments to use different versions intentionally:

```hcl
# live/dev/services/webserver-cluster/main.tf
module "webserver_cluster" {
  source = "github.com/your-username/terraform-aws-webserver-cluster?ref=v0.0.2"
  # dev uses the latest version for testing
  cluster_name  = "webservers-dev"
  instance_type = "t2.micro"
  min_size      = 2
  max_size      = 4
}
```

```hcl
# live/production/services/webserver-cluster/main.tf
module "webserver_cluster" {
  source = "github.com/your-username/terraform-aws-webserver-cluster?ref=v0.0.1"
  # production stays pinned to the stable version
  cluster_name  = "webservers-production"
  instance_type = "t2.medium"
  min_size      = 4
  max_size      = 10
}
```

This is the core pattern: dev tests new module versions, production stays pinned until the new version is validated. Run `terraform init` in each environment — Terraform will pull the correct version for each.

---

### 6. Write a Module README

Every shared module needs a README. Create `README.md` inside your module with:

- What the module does in one paragraph
- All input variables with their types, descriptions, and defaults
- All outputs with descriptions
- A usage example showing the minimum required inputs
- Any known limitations or gotchas specific to your module

This is not optional for real teams — an undocumented module is an unusable module.

---

### 7. Write Your Blog Post

**Post title:** *Advanced Terraform Module Usage: Versioning, Gotchas, and Reuse Across Environments*

Cover the three module gotchas with concrete code examples, walk through the versioning workflow from tagging to pinning, and explain the multi-environment deployment pattern. Make the versioning section practical — show exactly what the source URL looks like for local, Git, and Registry sources.

---

### 8. Post on Social Media

> 🔄 Day 9 of the 30-Day Terraform Challenge — went deep on advanced Terraform modules today. Module versioning, file path gotchas, and deploying different module versions across dev and production. This is the pattern that keeps large infrastructure codebases manageable. #30DayTerraformChallenge #TerraformChallenge #Terraform #IaC #DevOps #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste the GitHub URL of your versioned module repository.
2. **Live App Link field** — paste your blog post URL or social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**Module Gotchas**
Explain each of the three gotchas from Chapter 4 in your own words. For each one, show a broken code example and the corrected version.

```hcl
# broken example
# corrected example
```

**Versioned Module Repository**
Paste the GitHub URL of your module. Show the output of `git tag -l` confirming both versions exist. Explain what changed between v0.0.1 and v0.0.2.

**Multi-Environment Calling Configurations**
Paste the root `main.tf` from dev (using v0.0.2) and production (using v0.0.1). Explain why pinning production to an older version is the correct practice.

```hcl
# dev calling configuration
# production calling configuration
```

**`terraform init` Output**
Paste the output of `terraform init` from one of your environments showing Terraform downloading the versioned module source.

**Module README**
Paste the content of your module README. It should be complete enough that a stranger could use your module without asking you any questions.

**Version Pinning Strategy**
In your own words — why is it dangerous to reference a module without a version pin? What could happen in a team environment if two engineers run `terraform apply` and the module source has changed between their runs?

**Chapter 4 Gotchas Learnings**
Which gotcha do you think is the most dangerous in a production environment and why? Have you hit any of these already in your work this week?

**Challenges and Fixes**
What issues came up with GitHub source URLs, `terraform init` caching, or tagging? Document everything.

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terraform Module Sources — Git](https://developer.hashicorp.com/terraform/language/modules/sources#github)
- [Terraform Registry — Publishing Modules](https://developer.hashicorp.com/terraform/registry/modules/publish)
- [Module Versioning Best Practices](https://developer.hashicorp.com/terraform/language/modules/develop/versions)
- [Semantic Versioning](https://semver.org/)
- [Terraform path.module Expression](https://developer.hashicorp.com/terraform/language/expressions/references#path-module)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
