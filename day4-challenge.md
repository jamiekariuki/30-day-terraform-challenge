# Day 4: Mastering Basic Infrastructure with Terraform

## What You Will Accomplish Today

Yesterday you deployed your first server. Today you level up. You will learn one of the most important principles in Terraform — and in software engineering generally — which is **DRY: Don't Repeat Yourself**. You will apply it by deploying a configurable web server using input variables, then take it further by deploying a clustered web server capable of handling real traffic. By the end of today you will understand what separates a toy deployment from production-ready infrastructure.

---

## Tasks

### 1. Read

**Book:** *Terraform: Up & Running* by Yevgeniy Brikman — **Chapter 2**

Read pages **60 through 69**. Focus on:
- How input variables eliminate hardcoded values
- How to make infrastructure configurable without changing core logic
- The DRY principle as it applies to Terraform configurations
- How a clustered setup differs architecturally from a single server

---

### 2. Complete the Hands-On Labs

Both labs today are foundational. Variables and data sources appear in virtually every real-world Terraform configuration.

- **Lab 1:** [Intro to the Terraform Data Block](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2004%20-%20Understand%20Terraform%20Basics/08%20-%20Intro_to_the_Terraform_Data_Block.md)
- **Lab 2:** [Intro to Input Variables](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2004%20-%20Understand%20Terraform%20Basics/06%20-%20Intro_to_the_Input_Variables_Block.md)

---

### 3. Deploy a Configurable Web Server

Take the web server you built on Day 3 and refactor it using input variables. Nothing should be hardcoded. Instance type, region, port number, and any environment-specific values should all be driven by variables.

Your `variables.tf` should define at minimum:

```hcl
variable "server_port" {
  description = "The port the server will use for HTTP requests"
  type        = number
  default     = 8080
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}
```

Run `terraform plan` and verify that no hardcoded values remain in your `main.tf`. Deploy with `terraform apply` and confirm the server is reachable.

---

### 4. Deploy a Clustered Web Server

Now extend your deployment to a cluster. Use an **Auto Scaling Group (ASG)** backed by a **Launch Configuration** and put an **Application Load Balancer (ALB)** in front of it.

Your cluster should include:

- A Launch Configuration or Launch Template defining the instance spec
- An Auto Scaling Group with a minimum of 2 instances and a maximum of 5
- An Application Load Balancer distributing traffic across the instances
- A Target Group and Listener connecting the ALB to the ASG
- Security groups allowing HTTP traffic in and all traffic out

Use a data source to fetch the list of available availability zones dynamically:

```hcl
data "aws_availability_zones" "all" {}
```

Deploy, confirm the ALB DNS name resolves and returns your web page, then destroy everything.

---

### 5. Explore the Terraform Documentation

Spend 15–20 minutes reading the official docs on the resources you used today. Look up:
- [aws_autoscaling_group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/autoscaling_group)
- [aws_lb](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb)
- [Input Variables](https://developer.hashicorp.com/terraform/language/values/variables)

Understanding the docs is a core skill. The certification exam tests whether you can navigate them.

---

### 6. Write Your Blog Post

**Post title:** *Deploying a Highly Available Web App on AWS Using Terraform*

Cover the full journey from a single hardcoded server to a configurable, clustered, load-balanced deployment. Explain what input variables do and why they matter. Walk through the ASG and ALB setup. Include your actual Terraform code with commentary on each block.

---

### 7. Post on Social Media

> 🔥 Day 4 of the 30-Day Terraform Challenge — just deployed a highly available, load-balanced web app on AWS using Terraform. Auto Scaling Groups, Application Load Balancers, input variables — it's all coming together. #30DayTerraformChallenge #TerraformChallenge #Terraform #AWS #HighAvailability #IaC #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL or a link to your Terraform code.
2. **Live App Link field** — paste the ALB DNS name showing your clustered web server responding (capture this before you destroy), or your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**Configurable Web Server Code**
Paste your refactored `main.tf` and `variables.tf`. Explain what each variable does and why you chose the defaults you did.

```hcl
# paste your configurable server code here
```

**Clustered Web Server Code**
Paste your ASG, ALB, and related configuration. Walk through the key resources and how they connect to each other.

```hcl
# paste your clustered server code here
```

**Deployment Confirmation**
Paste the ALB DNS name and confirm it returned your web page. Include the output of `terraform output` if you defined any outputs.

**DRY Principle in Practice**
In your own words — what is the DRY principle and how did applying input variables demonstrate it? What would break in a large team if everyone hardcoded values?

**Difference Between Configurable and Clustered**
Explain the architectural difference between what you deployed today versus Day 3. What problems does clustering solve that a single server cannot?

**Lab Takeaways**
What did the data block lab teach you? How did you use a data source in your cluster deployment?

**Challenges and Fixes**
What went wrong during the ASG or ALB setup? Dependency issues, security group rules, listener configuration — document what tripped you up and how you solved it.

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terraform AWS Auto Scaling Group Docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/autoscaling_group)
- [Terraform AWS Load Balancer Docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb)
- [Input Variables — Terraform Language Docs](https://developer.hashicorp.com/terraform/language/values/variables)
- [Data Sources — Terraform Language Docs](https://developer.hashicorp.com/terraform/language/data-sources)
- [AWS Auto Scaling Overview](https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
