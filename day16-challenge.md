# Day 16: Building Production-Grade Infrastructure

## What You Will Accomplish Today

There is a significant gap between Terraform code that works and Terraform code that is production-ready. Production-grade infrastructure is modular, testable, version-controlled, composable, and maintainable by a team under pressure. Today you read the checklist that defines what production-grade actually means, then apply it by refactoring your existing infrastructure to meet that standard. This is the most important architectural exercise of the challenge.

---

## Tasks

### 1. Read

**Book:** *Terraform: Up & Running* by Yevgeniy Brikman — **Chapter 8**, pages 275–313

Read both sections in full:
- *The Production-Grade Infrastructure Checklist*
- *Building Testable and Composable Modules*

As you read the checklist, score your existing infrastructure against it honestly. Note every item your current code fails. That list becomes your work for today.

---

### 2. Complete the Hands-On Labs

- **Lab 1:** [Remote State](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2003%20-%20Understand%20The%20Purpose%20of%20Terraform/02%20-%20Benefits_of_State.md)
- **Lab 2:** [State Migration](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2005%20-%20Use%20Terraform%20Outside%20of%20Core%20Workflow/02%20-%20Terraform_State_Migration.md)

---

### 3. The Production-Grade Checklist

Use this checklist to audit your infrastructure. Every item you cannot check off is a gap to close today:

**Code structure**
- [ ] Configuration is broken into small, single-purpose modules — no monolithic `main.tf` files
- [ ] Modules have clear, minimal interfaces — inputs are well-named with descriptions and types
- [ ] All outputs are defined and documented
- [ ] No hardcoded values anywhere in resource blocks
- [ ] `locals` are used to centralise repeated expressions

**Reliability**
- [ ] Auto Scaling Groups have health checks pointing to ELB, not EC2
- [ ] `create_before_destroy` is set on any resource that cannot be modified in-place
- [ ] Resource names are either unique-by-construction or use `name_prefix`
- [ ] All critical resources have `prevent_destroy = true` in their lifecycle block

**Security**
- [ ] No secrets in `.tf` files, `.tfvars`, or state
- [ ] All sensitive variables and outputs are marked `sensitive = true`
- [ ] State is stored remotely with encryption and restricted IAM access
- [ ] IAM roles follow least-privilege — no `Action: "*"` policies
- [ ] Security groups have explicit ingress rules — no `0.0.0.0/0` on sensitive ports

**Observability**
- [ ] All resources have consistent tagging: at minimum `Name`, `Environment`, `ManagedBy = "terraform"`
- [ ] CloudWatch alarms exist for critical metrics (CPU, memory, error rates)
- [ ] Log groups are created and retention periods are set

**Maintainability**
- [ ] Every module has a `README.md` with usage examples
- [ ] Provider versions are pinned in `required_providers`
- [ ] Module sources reference versioned tags, not `main` branch
- [ ] `.terraform.lock.hcl` is committed to version control
- [ ] `.gitignore` excludes state files, `.terraform/`, and `*.tfvars`

---

### 4. Refactor Your Infrastructure

Take the webserver cluster configuration you have built over Days 3–15 and refactor it against the checklist above. Work through every gap you identified.

**Priority refactors to implement today:**

Add consistent tagging to every resource using a `locals` map:

```hcl
locals {
  common_tags = {
    Environment = var.environment
    ManagedBy   = "terraform"
    Project     = var.project_name
    Owner       = var.team_name
  }
}

resource "aws_instance" "web" {
  # ...
  tags = merge(local.common_tags, {
    Name = "${var.cluster_name}-instance"
  })
}
```

Add `prevent_destroy` to critical resources:

```hcl
resource "aws_s3_bucket" "state" {
  bucket = var.state_bucket_name

  lifecycle {
    prevent_destroy = true
  }
}
```

Add CloudWatch alarms for your cluster:

```hcl
resource "aws_cloudwatch_metric_alarm" "high_cpu" {
  alarm_name          = "${var.cluster_name}-high-cpu"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 120
  statistic           = "Average"
  threshold           = 80
  alarm_description   = "Triggers when CPU exceeds 80% for 4 minutes"

  dimensions = {
    AutoScalingGroupName = aws_autoscaling_group.example.name
  }

  alarm_actions = [aws_sns_topic.alerts.arn]
}

resource "aws_sns_topic" "alerts" {
  name = "${var.cluster_name}-alerts"
}
```

Add input validation to every module variable that has a constrained set of valid values:

```hcl
variable "environment" {
  type        = string
  description = "Deployment environment"

  validation {
    condition     = contains(["dev", "staging", "production"], var.environment)
    error_message = "Environment must be dev, staging, or production."
  }
}

variable "instance_type" {
  type        = string
  description = "EC2 instance type"

  validation {
    condition     = can(regex("^t[23]\\.", var.instance_type))
    error_message = "Instance type must be a t2 or t3 family type."
  }
}
```

---

### 5. Write Tests for Your Module

Write at least one automated test using **Terratest** — the Go-based testing framework for Terraform:

```go
package test

import (
  "testing"
  "github.com/gruntwork-io/terratest/modules/terraform"
  "github.com/gruntwork-io/terratest/modules/http-helper"
  "time"
)

func TestWebserverCluster(t *testing.T) {
  t.Parallel()

  terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
    TerraformDir: "../modules/services/webserver-cluster",
    Vars: map[string]interface{}{
      "cluster_name":  "test-cluster",
      "instance_type": "t2.micro",
      "min_size":      1,
      "max_size":      2,
      "environment":   "dev",
    },
  })

  defer terraform.Destroy(t, terraformOptions)
  terraform.InitAndApply(t, terraformOptions)

  albDnsName := terraform.Output(t, terraformOptions, "alb_dns_name")
  url := "http://" + albDnsName

  http_helper.HttpGetWithRetry(t, url, nil, 200, "Hello World", 30, 10*time.Second)
}
```

If you are not familiar with Go, you do not need to run this test today — writing it and understanding what it tests is the learning objective. Install Go and Terratest if you want to run it, but the conceptual understanding is what matters for the exam and for your blog post.

---

### 6. Write Your Blog Post

**Post title:** *Creating Production-Grade Infrastructure with Terraform*

Structure it around the checklist. Explain what each category — structure, reliability, security, observability, maintainability — means in practice. Show before/after code for your most impactful refactors. Include the Terratest example and explain what automated infrastructure testing gives you that manual testing cannot.

---

### 7. Post on Social Media

> 🚀 Day 16 of the 30-Day Terraform Challenge — production-grade infrastructure deep dive. Audited my existing code against the full production checklist: consistent tagging, lifecycle rules, CloudWatch alarms, input validation, Terratest. The gap between "it works" and "it's production-ready" is significant. #30DayTerraformChallenge #TerraformChallenge #Terraform #DevOps #IaC #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL or a link to your refactored infrastructure code.
2. **Live App Link field** — paste your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**Checklist Audit**
Go through the full production checklist above and mark each item. For every item you cannot check off, explain why and what it would take to fix it.

**Top 3 Refactors**
Pick the three most impactful changes you made today. For each one paste the before and after code and explain what problem the change solves.

```hcl
# before
# after
```

**Tagging Implementation**
Paste your `common_tags` locals block and show it applied to at least two different resource types using `merge`.

**Lifecycle Rules**
Paste your `prevent_destroy` and `create_before_destroy` implementations. Explain what would happen without each one in a production environment.

**CloudWatch Alarms**
Paste your alarm configuration. What threshold did you choose and why? What happens when the alarm fires?

**Input Validation**
Paste at least two `validation` blocks from your variables. Show what error message Terraform returns when you pass an invalid value.

**Terratest**
Paste your test function. Explain in your own words what it deploys, what it asserts, and why `defer terraform.Destroy` is important.

**Chapter 8 Learnings**
What is the most important item on the production checklist that you had not thought about before today? What surprised you most about the gap between your existing code and production-grade standards?

**Challenges and Fixes**
What broke during the refactor — tag propagation issues, lifecycle conflicts, validation regex syntax?

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terratest Documentation](https://terratest.gruntwork.io/)
- [Terraform Lifecycle Meta-Arguments](https://developer.hashicorp.com/terraform/language/meta-arguments/lifecycle)
- [Terraform Input Variable Validation](https://developer.hashicorp.com/terraform/language/values/variables#custom-validation-rules)
- [AWS CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)
- [Terraform Resource Tagging Best Practices](https://developer.hashicorp.com/terraform/tutorials/aws/aws-tagging)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
