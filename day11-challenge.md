# Day 11: Mastering Terraform Conditionals — Smarter, More Flexible Deployments

## What You Will Accomplish Today

Yesterday you used conditionals briefly alongside loops. Today you go deep on them specifically. Conditionals are what make a single Terraform configuration behave differently across environments, regions, and use cases — without duplicating code. By the end of today you will have refactored your infrastructure to use conditional logic throughout, making your configurations genuinely environment-aware and eliminating the need to maintain separate codebases for dev versus production.

---

## Tasks

### 1. Read

**Book:** *Terraform: Up & Running* by Yevgeniy Brikman — **Chapter 5**, pages 160–169

Focus entirely on the *Conditionals with Terraform* section. Read the examples slowly and understand what Terraform evaluates at plan time versus apply time, and why some conditional patterns work cleanly while others produce confusing errors.

---

### 2. Complete the Hands-On Labs

- **Lab 1:** [Module Sources](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2004%20-%20Understand%20Terraform%20Basics/12%20-%20Module_Sources.md)
- **Lab 2:** [Module Composition](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2004%20-%20Understand%20Terraform%20Basics/13%20-%20Module_Composition.md)

---

### 3. Conditional Expressions — The Ternary Pattern

The core conditional in Terraform is the ternary expression. It works in any argument position that accepts an expression:

```hcl
variable "environment" {
  type    = string
  default = "dev"
}

locals {
  instance_type    = var.environment == "production" ? "t2.medium" : "t2.micro"
  min_cluster_size = var.environment == "production" ? 3 : 1
  max_cluster_size = var.environment == "production" ? 10 : 3
}

resource "aws_instance" "web" {
  instance_type = local.instance_type
}
```

Using `locals` to centralise conditional logic is the correct pattern. Scattering ternary operators directly into resource arguments makes configurations hard to read and harder to test. Move all conditional decisions into `locals` and reference those from resources.

---

### 4. Conditional Resource Creation with `count`

The `count = condition ? 1 : 0` pattern is how you make entire resources optional in Terraform. When count is 0, the resource is not created. When count is 1, it is created exactly once.

```hcl
variable "enable_detailed_monitoring" {
  description = "Enable CloudWatch detailed monitoring (incurs additional cost)"
  type        = bool
  default     = false
}

variable "create_dns_record" {
  description = "Whether to create a Route53 DNS record for the ALB"
  type        = bool
  default     = false
}

resource "aws_cloudwatch_metric_alarm" "high_cpu" {
  count = var.enable_detailed_monitoring ? 1 : 0

  alarm_name          = "${var.cluster_name}-high-cpu"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 120
  statistic           = "Average"
  threshold           = 80
  alarm_description   = "CPU utilization exceeded 80%"
}

resource "aws_route53_record" "alb" {
  count = var.create_dns_record ? 1 : 0

  zone_id = data.aws_route53_zone.primary.zone_id
  name    = var.domain_name
  type    = "A"

  alias {
    name                   = aws_lb.example.dns_name
    zone_id                = aws_lb.example.zone_id
    evaluate_target_health = true
  }
}
```

Apply these to your webserver cluster module. Test with both `true` and `false` for each toggle and confirm the plan output shows the correct resources being added or not.

---

### 5. Referencing Conditionally Created Resources

When a resource uses `count = condition ? 1 : 0`, you cannot reference it directly like a normal resource — you must use index notation and handle the zero-element case:

```hcl
# Wrong — will error if count is 0
output "alarm_arn" {
  value = aws_cloudwatch_metric_alarm.high_cpu.arn
}

# Correct — returns null when the resource does not exist
output "alarm_arn" {
  value = var.enable_detailed_monitoring ? aws_cloudwatch_metric_alarm.high_cpu[0].arn : null
}
```

Update all outputs that reference conditionally created resources to use this pattern.

---

### 6. Environment-Aware Module

Enhance your webserver cluster module to be fully environment-aware using a single `environment` variable that drives multiple conditional decisions:

```hcl
variable "environment" {
  description = "Deployment environment: dev, staging, or production"
  type        = string

  validation {
    condition     = contains(["dev", "staging", "production"], var.environment)
    error_message = "Environment must be dev, staging, or production."
  }
}

locals {
  is_production = var.environment == "production"

  instance_type      = local.is_production ? "t2.medium" : "t2.micro"
  min_size           = local.is_production ? 3 : 1
  max_size           = local.is_production ? 10 : 3
  enable_monitoring  = local.is_production
  deletion_policy    = local.is_production ? "Retain" : "Delete"
}
```

The `validation` block is new — it catches invalid environment values at plan time with a clear error message, before anything is deployed. This is one of the most practical Terraform features for modules used by multiple teams.

Deploy this module from dev and production calling configurations using only the `environment` variable to differentiate them. Confirm that the plan output for production shows larger instances and more resources than dev.

---

### 7. Conditional Data Source Lookups

Conditionals work with data sources too. Use them to optionally look up existing infrastructure:

```hcl
variable "use_existing_vpc" {
  type    = bool
  default = false
}

data "aws_vpc" "existing" {
  count = var.use_existing_vpc ? 1 : 0
  tags = {
    Name = "existing-vpc"
  }
}

locals {
  vpc_id = var.use_existing_vpc ? data.aws_vpc.existing[0].id : aws_vpc.new[0].id
}

resource "aws_vpc" "new" {
  count      = var.use_existing_vpc ? 0 : 1
  cidr_block = "10.0.0.0/16"
}
```

This pattern lets your module work both as a greenfield deployment (creates a new VPC) and as a brownfield deployment (uses an existing one) with a single boolean toggle.

---

### 8. Write Your Blog Post

**Post title:** *How Conditionals Make Terraform Infrastructure Dynamic and Efficient*

Cover the ternary expression, the `count = condition ? 1 : 0` pattern, referencing conditionally created resources safely, input validation blocks, and the environment-aware module pattern. Include the real problem each pattern solves and show before/after code where relevant.

---

### 9. Post on Social Media

> 💡 Day 11 of the 30-Day Terraform Challenge — conditionals deep dive. One Terraform configuration, multiple environments, zero code duplication. Environment-aware modules with input validation are genuinely powerful. #30DayTerraformChallenge #TerraformChallenge #Terraform #IaC #DevOps #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL or a link to your updated module code.
2. **Live App Link field** — paste your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**Locals-Centralised Conditional Logic**
Paste your `locals` block showing all conditional decisions in one place. Explain why this is better than scattering ternary operators across resource arguments.

```hcl
# paste your locals block here
```

**Conditional Resource Creation**
Paste your `count = condition ? 1 : 0` examples. Show the plan output for both the enabled and disabled state of at least one optional resource.

```hcl
# paste your conditional resource examples here
```

**Safe Output References**
Paste your output definitions for conditionally created resources. Explain what would happen without the ternary guard when count is 0.

```hcl
# paste your output definitions here
```

**Environment-Aware Module**
Paste your updated module with the `environment` variable driving multiple locals. Paste the plan output from both a dev and a production apply and highlight the differences in instance type and cluster size.

**Input Validation Block**
Paste your `validation` block and show what error message Terraform returns when you pass an invalid environment value.

**Conditional Data Source Pattern**
Paste your existing/new VPC pattern and explain the brownfield vs greenfield use case it enables.

**Chapter 5 Learnings**
In your own words — what is the difference between a conditional expression and conditional resource creation? Can you use a conditional to choose between two different resource types? Why or why not?

**Challenges and Fixes**
What went wrong? Index-out-of-range errors when referencing conditional resources, validation blocks rejecting valid values, and plan-time evaluation errors are all common today — document what you hit.

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terraform Conditional Expressions](https://developer.hashicorp.com/terraform/language/expressions/conditionals)
- [Terraform Input Variable Validation](https://developer.hashicorp.com/terraform/language/values/variables#custom-validation-rules)
- [Terraform `locals` Block](https://developer.hashicorp.com/terraform/language/values/locals)
- [Terraform `count` Meta-Argument](https://developer.hashicorp.com/terraform/language/meta-arguments/count)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
