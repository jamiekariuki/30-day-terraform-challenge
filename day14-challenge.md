# Day 14: Working with Multiple Providers — Part 1

## What You Will Accomplish Today

Every real-world infrastructure spans more than one region, account, or even cloud platform. Today you begin Chapter 7 and learn how Terraform's provider system actually works under the hood — how providers are installed, versioned, and configured — and then apply that knowledge by deploying resources across multiple AWS regions using provider aliases. This is foundational for any multi-region or multi-account architecture.

---

## Tasks

### 1. Read

**Book:** *Terraform: Up & Running* by Yevgeniy Brikman — **Chapter 7**

Read these sections today:
- *Working with One Provider*
- *What Is a Provider?*
- *How Do You Install Providers?*
- *How Do You Use Providers?*
- *Working with Multiple Copies of the Same Provider*

Focus on understanding the provider installation process, the `.terraform.lock.hcl` file and what it does, and how provider aliases work. These are tested directly in the certification exam.

---

### 2. Complete the Hands-On Labs

- **Lab 1:** [Terraform Testing](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2004%20-%20Understand%20Terraform%20Basics/15%20-%20Terraform_Testing.md)
- **Lab 2:** [Terraform CI/CD Integration](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2005%20-%20Use%20Terraform%20Outside%20of%20Core%20Workflow/03%20-%20Terraform_CICD.md)

---

### 3. Understand How Providers Work

A provider is a plugin that translates Terraform resource declarations into API calls for a specific platform. When you run `terraform init`, Terraform reads your `required_providers` block and downloads the correct provider binary from the Terraform Registry.

Always pin your provider versions to avoid unexpected breaking changes:

```hcl
terraform {
  required_version = ">= 1.0.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

The `~> 5.0` constraint means: use any version `>= 5.0` but `< 6.0`. The `.terraform.lock.hcl` file records the exact version selected and should always be committed to version control so every team member and CI system uses the same provider version.

---

### 4. Deploy Resources Across Multiple AWS Regions

The default provider configuration applies to all resources in your configuration. To deploy resources in a second region, define an aliased provider:

```hcl
# Default provider — primary region
provider "aws" {
  region = "us-east-1"
}

# Aliased provider — secondary region
provider "aws" {
  alias  = "us_west"
  region = "us-west-2"
}
```

Reference the aliased provider explicitly in any resource that should deploy to the secondary region:

```hcl
# Deploys in us-east-1 (default provider)
resource "aws_s3_bucket" "primary" {
  bucket = "my-app-primary-bucket"
}

# Deploys in us-west-2 (aliased provider)
resource "aws_s3_bucket" "replica" {
  provider = aws.us_west
  bucket   = "my-app-replica-bucket"
}
```

Build a practical deployment that demonstrates multi-region infrastructure. A good exercise is S3 replication: create a primary bucket in `us-east-1` and a replica bucket in `us-west-2`, with replication configured between them.

```hcl
resource "aws_s3_bucket_replication_configuration" "replication" {
  role   = aws_iam_role.replication.arn
  bucket = aws_s3_bucket.primary.id

  rule {
    id     = "replicate-all"
    status = "Enabled"

    destination {
      bucket        = aws_s3_bucket.replica.arn
      storage_class = "STANDARD"
    }
  }
}
```

---

### 5. Deploy Across Multiple AWS Accounts

For multi-account deployments, use the `assume_role` argument to have each provider assume a different IAM role:

```hcl
provider "aws" {
  region = "us-east-1"
  alias  = "production"

  assume_role {
    role_arn = "arn:aws:iam::111111111111:role/TerraformDeployRole"
  }
}

provider "aws" {
  region = "us-east-1"
  alias  = "staging"

  assume_role {
    role_arn = "arn:aws:iam::222222222222:role/TerraformDeployRole"
  }
}
```

If you have access to multiple AWS accounts, deploy a simple resource (an S3 bucket or a security group) in each. If not, set up the provider configuration and document what the plan output shows.

---

### 6. Explore Provider Version Constraints

Spend 15 minutes reading the Terraform Registry page for the AWS provider. Look at the changelog for recent major versions and understand what kinds of changes cause a major version bump. Then check your `.terraform.lock.hcl` file after running `terraform init` and explain what each field records.

---

### 7. Prepare for Tomorrow

Tomorrow covers multi-cloud modules, Docker, and Kubernetes with Terraform. Review your module work from Days 8–9 — particularly how modules receive provider configurations from their callers. Tomorrow you will need to understand how to pass provider aliases into modules.

---

### 8. Write Your Blog Post

**Post title:** *Getting Started with Multiple Providers in Terraform*

Cover what a provider is, how installation and versioning work, the `.terraform.lock.hcl` file, and the provider alias pattern for multi-region deployments. Include your S3 replication example as a concrete demonstration. Make the provider versioning section practical — show the constraint syntax and explain what each operator does.

---

### 9. Post on Social Media

> 🔧 Day 14 of the 30-Day Terraform Challenge — provider deep dive. Multiple AWS regions, provider aliases, version pinning, and the lock file. Multi-region deployments are surprisingly clean once you understand how Terraform's provider system works. #30DayTerraformChallenge #TerraformChallenge #Terraform #AWS #MultiRegion #IaC #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL or a link to your multi-region code.
2. **Live App Link field** — paste your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**Provider Configuration**
Paste your `required_providers` block and both provider configurations (default and aliased). Explain what each argument does.

```hcl
# paste your provider configuration here
```

**Multi-Region Deployment Code**
Paste your multi-region resource code showing the default and aliased provider in use. Explain how Terraform decides which API endpoint to call for each resource.

```hcl
# paste your multi-region resources here
```

**`.terraform.lock.hcl` Explanation**
Paste the contents of your lock file and explain what `version`, `constraints`, and `hashes` each record. Why should this file be committed to version control?

**Multi-Account Setup**
Paste your `assume_role` provider configuration. If you deployed across multiple accounts, show the plan output. If not, explain what the configuration would do and what IAM permissions `TerraformDeployRole` would need.

**Chapter 7 Learnings**
In your own words:
- What happens during `terraform init` from a provider perspective?
- What is the difference between `version` and `~> version` in a provider constraint?
- Why does every resource need exactly one provider and how does Terraform determine which one to use when none is specified?

**Challenges and Fixes**
What issues came up — region mismatches, alias reference errors, IAM permission issues for cross-account roles?

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terraform Provider Documentation](https://developer.hashicorp.com/terraform/language/providers)
- [Terraform AWS Provider Registry Page](https://registry.terraform.io/providers/hashicorp/aws/latest)
- [Provider Version Constraints](https://developer.hashicorp.com/terraform/language/expressions/version-constraints)
- [Dependency Lock File](https://developer.hashicorp.com/terraform/language/files/dependency-lock)
- [AWS S3 Cross-Region Replication](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
