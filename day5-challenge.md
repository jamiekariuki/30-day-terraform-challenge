# Day 5: Scaling Infrastructure and Understanding Terraform State

## What You Will Accomplish Today

Today has two equally important threads. First, you will complete your scaled infrastructure by deploying an Elastic Load Balancer in front of your cluster — making your app genuinely production-ready. Second, and just as critically, you will understand **Terraform state**: what it is, why it exists, what happens when you tamper with it, and why managing it correctly is one of the most important habits you can build as an infrastructure engineer.

---

## Tasks

### 1. Read

**Book:** *Terraform: Up & Running* by Yevgeniy Brikman

Finish **Chapter 2** and begin **Chapter 3**. Focus on these sections:

- *Deploying a Load Balancer*
- *What is Terraform State*
- *Shared Storage for State Files*
- *Limitations with Terraform State*

Pay close attention to what the state file actually contains, why it is the source of truth for Terraform, and what goes wrong when it gets out of sync with real infrastructure.

---

### 2. Complete the Hands-On Lab

- **Lab:** [Benefits of State](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2003%20-%20Understand%20The%20Purpose%20of%20Terraform/02%20-%20Benefits_of_State.md)

Work through the full lab. Observe what the state file looks like, what it tracks, and what Terraform does when state and real infrastructure diverge.

---

### 3. Scale Your Infrastructure with an Elastic Load Balancer

Extend your Day 4 cluster by deploying a fully configured **AWS Application Load Balancer** in front of your Auto Scaling Group.

Your deployment should include:

- An Application Load Balancer with a public-facing listener on port 80
- A Target Group configured for HTTP health checks
- An Auto Scaling Group attachment to the Target Group
- Security groups that allow inbound HTTP to the ALB and restrict direct access to instances
- Output values exposing the ALB DNS name

```hcl
output "alb_dns_name" {
  value       = aws_lb.example.dns_name
  description = "The domain name of the load balancer"
}
```

After applying, hit the ALB DNS name in your browser and confirm it routes to your instances. Test that if you stop one instance, the ALB continues serving traffic from the others.

---

### 4. Explore and Understand the State File

After your deployment, open the `terraform.tfstate` file and read through it. Then do the following experiments:

**Experiment 1 — Manual state tampering:**
Manually edit a value in `terraform.tfstate` (change an instance type or a tag value). Run `terraform plan` and observe what Terraform detects. Then restore the original value.

**Experiment 2 — State drift:**
In the AWS Console, manually change a tag on one of your EC2 instances. Run `terraform plan` without touching your code. Observe how Terraform detects the drift and what it proposes to do about it.

Document both experiments in your submission. Understanding drift and state reconciliation is tested directly in the Terraform Associate certification.

---

### 5. Bonus — Terraform Block Comparison Table

Create a comparison table covering every Terraform block type you have used so far. For each block, include what it does, when to use it, and a short code example. This makes an excellent study reference and a great section in your blog post.

| Block Type | Purpose | When to Use | Example |
|---|---|---|---|
| `provider` | Configures the cloud provider | Once per provider | `provider "aws" { region = "us-east-1" }` |
| `resource` | Defines infrastructure to create | Every piece of infrastructure | `resource "aws_instance" "web" { ... }` |
| `variable` | Declares an input variable | To avoid hardcoding values | `variable "instance_type" { default = "t2.micro" }` |
| `output` | Exposes values after apply | To surface IPs, DNS names, IDs | `output "alb_dns" { value = aws_lb.example.dns_name }` |
| `data` | Reads existing resources | To reference things not managed by this config | `data "aws_availability_zones" "all" {}` |

Extend this table with any additional blocks from your labs.

---

### 6. Write Your Blog Post

**Post title:** *Managing High Traffic Applications with AWS Elastic Load Balancer and Terraform*

Cover the full ALB setup, how it integrates with your ASG, and what the state file is doing behind the scenes. Include your block comparison table. A second angle you can take: *Best Practices for Managing Terraform State Files* — cover why you should never commit state to Git, what remote backends are, and why state locking matters.

---

### 7. Post on Social Media

> 🚀 Day 5 of the 30-Day Terraform Challenge — scaled my infrastructure with an AWS Elastic Load Balancer and got deep into Terraform state management today. Understanding state is what separates beginners from engineers who can be trusted with production. #30DayTerraformChallenge #TerraformChallenge #Terraform #AWS #ELB #IaC #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL or a link to your Terraform code.
2. **Live App Link field** — paste the ALB DNS name confirming your load-balanced cluster is running, or your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**ALB and Scaled Infrastructure Code**
Paste your complete Terraform configuration including the ALB, Target Group, Listener, and ASG attachment. Explain each resource and how they connect.

```hcl
# paste your infrastructure code here
```

**Output Values**
Paste the output of `terraform output` showing your ALB DNS name.

**State File Observations**
Describe what you saw inside `terraform.tfstate`. What information does it store about your resources? What surprised you?

**Experiment Results**
Document both state experiments. What did Terraform detect when you manually edited the state file? What did it detect when you changed a resource directly in the AWS Console?

**Block Comparison Table**
Include your completed comparison table covering all block types covered so far.

**Chapter 3 Learnings**
What is the purpose of remote state storage? Why should the state file never be committed to version control? What is state locking and why does it matter in a team environment?

**Challenges and Fixes**
What broke during the ALB setup? Target group health check failures, security group misconfigurations, and listener rule issues are all common — document what you hit and how you resolved it.

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [AWS Elastic Load Balancing Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)
- [Terraform State Documentation](https://developer.hashicorp.com/terraform/language/state)
- [Terraform aws_lb Resource](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb)
- [Terraform Output Values](https://developer.hashicorp.com/terraform/language/values/outputs)
- [Why You Should Not Store Terraform State in Git](https://developer.hashicorp.com/terraform/language/state/sensitive-data)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
