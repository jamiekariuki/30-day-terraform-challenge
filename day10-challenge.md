# Day 10: Terraform Loops and Conditionals — Dynamic Infrastructure at Scale

## What You Will Accomplish Today

Until now every resource in your configurations has been declared individually. That works for small setups but breaks down fast when you need ten IAM users, five S3 buckets, or a security group rule per environment. Today you learn the tools that eliminate that repetition entirely: `count`, `for_each`, `for` expressions, and conditional logic. These are the features that make Terraform feel like a real programming language — and they are tested extensively in the Terraform Associate exam.

---

## Tasks

### 1. Read

**Book:** *Terraform: Up & Running* by Yevgeniy Brikman — **Chapter 5**, pages 141–160

Read these sections:
- *Loops with `count` and `for_each`*
- *Conditionals with Terraform*

Pay close attention to the author's guidance on when to use `count` versus `for_each`, and why `count` has a subtle limitation when dealing with lists that can change. This distinction comes up directly in the exam.

---

### 2. Complete the Hands-On Labs

- **Lab 1:** [Terraform Modules](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2004%20-%20Understand%20Terraform%20Basics/11%20-%20Intro_to_Modules.md)
- **Lab 2:** [Module Sources](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2004%20-%20Understand%20Terraform%20Basics/12%20-%20Module_Sources.md)

---

### 3. Loops with `count`

`count` is the simplest loop — use it when you need N identical copies of a resource.

```hcl
resource "aws_iam_user" "example" {
  count = 3
  name  = "user-${count.index}"
}
```

The problem with `count` and lists: if you use `count = length(var.user_names)` and then remove an item from the middle of the list, Terraform renumbers all subsequent resources and recreates them. This is destructive behaviour that surprises engineers who do not know about it.

```hcl
# Fragile — removing "alice" from position 0 causes bob and charlie to be recreated
variable "user_names" {
  type    = list(string)
  default = ["alice", "bob", "charlie"]
}

resource "aws_iam_user" "example" {
  count = length(var.user_names)
  name  = var.user_names[count.index]
}
```

---

### 4. Loops with `for_each`

`for_each` solves the ordering problem by keying resources on a map or set value rather than an index. Removing one entry only affects that specific resource.

```hcl
variable "user_names" {
  type    = set(string)
  default = ["alice", "bob", "charlie"]
}

resource "aws_iam_user" "example" {
  for_each = var.user_names
  name     = each.value
}
```

With a map, you can carry additional configuration per item:

```hcl
variable "users" {
  type = map(object({
    department = string
    admin      = bool
  }))
  default = {
    alice = { department = "engineering", admin = true }
    bob   = { department = "marketing",   admin = false }
  }
}

resource "aws_iam_user" "example" {
  for_each = var.users
  name     = each.key
  tags = {
    Department = each.value.department
  }
}
```

Refactor your existing infrastructure to use `for_each` wherever you have repeated resources of the same type.

---

### 5. `for` Expressions

`for` expressions transform collections inline — inside resource arguments, outputs, or locals. They do not create resources; they reshape data.

```hcl
# Produce a list of uppercase names
output "upper_names" {
  value = [for name in var.user_names : upper(name)]
}

# Produce a map of name → ARN from the for_each users resource
output "user_arns" {
  value = { for name, user in aws_iam_user.example : name => user.arn }
}
```

Use a `for` expression to output a clean map of all your IAM user ARNs keyed by username.

---

### 6. Conditionals

Terraform conditionals use the ternary operator: `condition ? true_value : false_value`. Combine them with `count` to make resources optional:

```hcl
variable "enable_autoscaling" {
  description = "Enable autoscaling for the cluster"
  type        = bool
  default     = true
}

resource "aws_autoscaling_policy" "scale_out" {
  count = var.enable_autoscaling ? 1 : 0
  # ...
}
```

Apply this pattern to your webserver cluster module — add an `enable_autoscaling` variable that controls whether the scaling policy is created. Test it with both `true` and `false` and confirm the correct resources are created or skipped.

Also use conditionals to vary instance sizing between environments:

```hcl
variable "environment" {
  type    = string
  default = "dev"
}

locals {
  instance_type = var.environment == "production" ? "t2.medium" : "t2.micro"
}

resource "aws_instance" "web" {
  instance_type = local.instance_type
  # ...
}
```

---

### 7. Refactor Your Existing Infrastructure

Go back to your webserver cluster code from Days 3–9 and apply everything you learned today:

- Replace any repeated resource blocks with `for_each`
- Use `count = var.some_bool ? 1 : 0` to make optional resources toggleable
- Use a `for` expression in at least one output to produce a useful map
- Use `locals` to centralise conditional logic rather than scattering ternary operators through resource arguments

Deploy, confirm everything works, and destroy.

---

### 8. Write Your Blog Post

**Post title:** *Mastering Loops and Conditionals in Terraform*

Cover all four tools: `count`, `for_each`, `for` expressions, and the ternary conditional. Explain the `count` index problem with a concrete example of what breaks and why `for_each` fixes it. Show real code for each. Make it the definitive reference post you wish you had before today.

---

### 9. Post on Social Media

> 💡 Day 10 of the 30-Day Terraform Challenge — loops and conditionals unlocked. `count`, `for_each`, `for` expressions, and ternary conditionals turn static configs into dynamic infrastructure. No more copy-pasting resource blocks. #30DayTerraformChallenge #TerraformChallenge #Terraform #IaC #DevOps #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL or a link to your refactored code.
2. **Live App Link field** — paste your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**`count` Example**
Paste a working example using `count` and explain what `count.index` gives you. Then show the breakage scenario — what happens when you remove an item from the middle of a list.

```hcl
# paste your count example here
```

**`for_each` Example**
Paste your `for_each` implementation using both a set and a map. Explain why this is safer than `count` for lists that change.

```hcl
# paste your for_each example here
```

**`for` Expression**
Paste the `for` expression you used in an output. Explain what it transforms and why that output is useful.

```hcl
# paste your for expression here
```

**Conditional Logic**
Paste your `enable_autoscaling` implementation and your environment-based instance sizing. Show the plan output for both `true` and `false` toggle states.

```hcl
# paste your conditional code here
```

**Refactored Infrastructure**
Describe what you changed in your existing code. Which repeated resource blocks did you collapse with `for_each`? Which optional resources did you make toggleable with `count`?

**`count` vs `for_each` — Your Verdict**
In your own words, when would you choose `count` over `for_each`? Are there cases where `count` is actually the better choice?

**Chapter 5 Learnings**
What does Terraform do when you use `count` to create a list of resources and then reference them? How do you access a specific resource from a `for_each` collection in an output?

**Challenges and Fixes**
What broke? Type constraint errors, `each.key` vs `each.value` confusion, and conditional count conflicts are all common today — document what you hit.

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terraform `for_each` Documentation](https://developer.hashicorp.com/terraform/language/meta-arguments/for_each)
- [Terraform `count` Documentation](https://developer.hashicorp.com/terraform/language/meta-arguments/count)
- [Terraform `for` Expressions](https://developer.hashicorp.com/terraform/language/expressions/for)
- [Terraform Conditional Expressions](https://developer.hashicorp.com/terraform/language/expressions/conditionals)
- [Terraform `locals` Block](https://developer.hashicorp.com/terraform/language/values/locals)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
