# Day 8: Building Reusable Infrastructure with Terraform Modules

## What You Will Accomplish Today

You have been writing Terraform code for a week. By now you have probably noticed patterns repeating — the same security group structure, the same instance configuration, the same load balancer setup appearing across environments. Today you learn how to stop repeating yourself at scale. **Modules** are how professional Terraform engineers package reusable infrastructure components and share them across projects, teams, and organisations. By the end of today you will have built your own module, deployed it in a real environment, and understood why modules are the single most important abstraction Terraform offers.

---

## Tasks

### 1. Read

**Book:** *Terraform: Up & Running* by Yevgeniy Brikman — **Chapter 4**, pages 115–139

Read these sections carefully:
- *Module Basics*
- *Module Inputs*
- *Module Outputs*

Focus on how modules encapsulate infrastructure logic, how inputs make them configurable, and how outputs expose the values consumers need. Pay attention to the directory structure conventions — the exam tests this.

---

### 2. Complete the Hands-On Labs

- **Lab 1:** [Terraform Workspaces](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2005%20-%20Use%20Terraform%20Outside%20of%20Core%20Workflow/01%20-%20Terraform_Workspaces.md)
- **Lab 2:** [Terraform Modules](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2004%20-%20Understand%20Terraform%20Basics/11%20-%20Intro_to_Modules.md)

---

### 3. Build Your First Terraform Module

Create a reusable module for a common infrastructure component. A **web server cluster** is the recommended choice since you have been building one all week — now you will package it properly.

Your module directory structure should follow this convention:

```
modules/
└── services/
    └── webserver-cluster/
        ├── main.tf
        ├── variables.tf
        ├── outputs.tf
        └── README.md
```

**`variables.tf`** — define every configurable input with a description and type:

```hcl
variable "cluster_name" {
  description = "The name to use for all cluster resources"
  type        = string
}

variable "instance_type" {
  description = "EC2 instance type for the cluster"
  type        = string
  default     = "t2.micro"
}

variable "min_size" {
  description = "Minimum number of EC2 instances in the ASG"
  type        = number
}

variable "max_size" {
  description = "Maximum number of EC2 instances in the ASG"
  type        = number
}

variable "server_port" {
  description = "Port the server uses for HTTP"
  type        = number
  default     = 8080
}
```

**`outputs.tf`** — expose everything a consumer might need:

```hcl
output "alb_dns_name" {
  value       = aws_lb.example.dns_name
  description = "The domain name of the load balancer"
}

output "asg_name" {
  value       = aws_autoscaling_group.example.name
  description = "The name of the Auto Scaling Group"
}
```

Keep all your actual resource definitions in `main.tf`. No hardcoded values — everything configurable should flow through a variable.

---

### 4. Deploy Infrastructure Using Your Module

Create a root configuration that calls your module. This is the pattern you will use for every real project:

```
live/
├── dev/
│   └── services/
│       └── webserver-cluster/
│           └── main.tf
└── production/
    └── services/
        └── webserver-cluster/
            └── main.tf
```

In each environment's `main.tf`, call the module with environment-appropriate values:

```hcl
# live/dev/services/webserver-cluster/main.tf

module "webserver_cluster" {
  source = "../../../../modules/services/webserver-cluster"

  cluster_name  = "webservers-dev"
  instance_type = "t2.micro"
  min_size      = 2
  max_size      = 4
}

output "alb_dns_name" {
  value = module.webserver_cluster.alb_dns_name
}
```

```hcl
# live/production/services/webserver-cluster/main.tf

module "webserver_cluster" {
  source = "../../../../modules/services/webserver-cluster"

  cluster_name  = "webservers-production"
  instance_type = "t2.medium"
  min_size      = 4
  max_size      = 10
}
```

Run `terraform init` and `terraform apply` from the dev directory. Confirm the cluster is running, then look at the production configuration — notice it is the same module, different inputs, zero code duplication.

Run `terraform destroy` on dev when done.

---

### 5. Refactor Your Previous Code into a Module

Take the infrastructure you deployed in Days 3–5 and refactor it into the module structure above. Move hardcoded values to variables, ensure all useful outputs are defined, and confirm the module can be called from multiple calling configurations without modification.

This is the bonus activity — it is also the most valuable exercise of the day. Refactoring existing code into a module teaches you the decisions you need to make about what to expose as an input versus what to keep internal.

---

### 6. Write Your Blog Post

**Post title:** *Building Reusable Infrastructure with Terraform Modules*

Cover the module directory structure, how inputs and outputs work, the calling pattern from a root configuration, and the difference between a module that is easy to use versus one that is a pain. Include your actual module code with explanations. A second strong angle: *Best Practices for Structuring Terraform Modules* — cover naming conventions, when to split a large module into smaller ones, and how to write a useful module README.

---

### 7. Post on Social Media

> 🔄 Day 8 of the 30-Day Terraform Challenge — built my first reusable Terraform module today. Packaged the entire web server cluster into a module and deployed it across dev and production with different inputs and zero code duplication. This is how real infrastructure teams work. #30DayTerraformChallenge #TerraformChallenge #Terraform #IaC #DevOps #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL or a link to your module code.
2. **Live App Link field** — paste the ALB DNS name from your dev deployment or your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**Module Directory Structure**
Show your full directory tree — the module source and both calling configurations.

**Module Code**
Paste your complete `variables.tf`, `outputs.tf`, and `main.tf` from inside the module. Explain every input variable — why it exists, what it controls, and why you chose the default you did (if any).

```hcl
# paste your module code here
```

**Calling Configuration**
Paste the root `main.tf` from your dev and production environments. Show how the same module produces different infrastructure based on different inputs.

```hcl
# paste your calling configurations here
```

**Deployment Confirmation**
Paste the output of `terraform output` from your dev deployment showing the ALB DNS name. Confirm the cluster was reachable.

**Module Design Decisions**
Answer these in your own words:
- What did you choose to expose as input variables versus keep hardcoded inside the module?
- What outputs did you define and why would a caller need them?
- What would break if someone called your module without passing a required variable?

**Refactoring Observations**
What did you learn refactoring your previous code into a module? What had to change and what stayed the same?

**Chapter 4 Learnings**
What is the difference between a root module and a child module? What does `terraform init` do when you add a new module source? What happens to module outputs in the state file?

**Challenges and Fixes**
What tripped you up — relative source paths, missing outputs, variable type mismatches? Document what happened and how you resolved it.

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terraform Module Documentation](https://developer.hashicorp.com/terraform/language/modules)
- [Terraform Module Structure Best Practices](https://developer.hashicorp.com/terraform/language/modules/develop/structure)
- [Terraform Registry — Public Modules](https://registry.terraform.io/browse/modules)
- [Module Sources](https://developer.hashicorp.com/terraform/language/modules/sources)
- [Writing a Good Module README](https://developer.hashicorp.com/terraform/language/modules/develop/documentation)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
