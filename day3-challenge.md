# Day 3: Deploying Your First Server with Terraform

## What You Will Accomplish Today

Today is where things get real. You will stop reading about Terraform and start using it. By the end of today you will have a running server on the cloud — provisioned entirely through code. You will also learn the two most fundamental building blocks of every Terraform configuration: the **provider block** and the **resource block**.

---

## Tasks

### 1. Read

**Book:** *Terraform: Up & Running* by Yevgeniy Brikman — **Chapter 2**

Read the following sections specifically:
- *Deploying a Single Server*
- *Deploying a Web Server*

Stop at page 59. Focus on understanding what a provider block is, what a resource block is, and how they work together to describe real infrastructure.

---

### 2. Complete the Hands-On Labs

Work through both labs fully. Write the code yourself — do not copy-paste. Typing it out builds muscle memory and helps you catch errors early.

- **Lab 1:** [Intro to the Terraform Provider Block](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2004%20-%20Understand%20Terraform%20Basics/04%20-%20Intro_to_the_Terraform_Provider_Block.md)
- **Lab 2:** [Intro to the Terraform Resource Block](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2004%20-%20Understand%20Terraform%20Basics/05%20-%20Intro_to_the_Terraform_Resource_Block.md)

---

### 3. Deploy Your First Server

Use Terraform to deploy a basic web server on your cloud platform of choice — **AWS**, **GCP**, or **Azure**. AWS is recommended since most labs in this challenge are AWS-focused.

Your deployment should include at minimum:

- A provider block configured for your chosen cloud
- A resource block for a compute instance (EC2, Compute Engine, or Virtual Machine)
- A security group or firewall rule allowing HTTP traffic on port 80
- A simple user data script that serves a basic HTML page

Run the full Terraform workflow:

```
terraform init
terraform plan
terraform apply
```

Confirm your server is reachable — paste the public IP in your browser and verify the page loads. Then clean up:

```
terraform destroy
```

Get into the habit of destroying resources after each day. Unused cloud resources cost money.

---

### 4. Create an Architecture Diagram

Using [draw.io](https://app.diagrams.net/) or any diagramming tool you prefer, create a simple architecture diagram of what you deployed today.

Your diagram should show:
- The cloud provider and region
- Your compute instance
- The security group and which ports are open
- The internet gateway or public access path

Export it as a PNG or PDF. You will upload or link it in your workspace submission.

---

### 5. Write Your Blog Post

**Post title:** *Deploying Your First Server with Terraform: A Beginner's Guide*

Walk your readers through your deployment step by step. Include your actual Terraform code with explanations of each block. Explain what `terraform init`, `terraform plan`, and `terraform apply` each do. Share what broke and how you fixed it — that is the content people actually search for.

Publish it and save the link.

---

### 6. Post on Social Media

> 🔥 Day 3 of the 30-Day Terraform Challenge — just deployed my first server using Terraform! Infrastructure as Code makes this feel like magic. Huge shoutout to AWS AI/ML UserGroup Kenya, Meru HashiCorp User Group, and EveOps for putting this together. #30DayTerraformChallenge #TerraformChallenge #Terraform #IaC #AWS #DevOps #AWSUserGroupKenya #EveOps

---

### 7. Bonus — Record a Video Walkthrough

Record a short screen capture of your deployment process and share it on YouTube or LinkedIn. Walk through your Terraform code, run the apply, and show the server responding in the browser. Teaching accelerates learning — and it earns community points.

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste the link to your blog post or a GitHub gist with your Terraform code.
2. **Live App Link field** — paste the public IP or URL of your deployed server (before you destroy it), or your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**Your Terraform Code**
Paste your complete `main.tf` (and any other files you created) as a code block. Explain every block — what it does and why you wrote it that way.

```hcl
# paste your terraform code here
```

**Deployment Output**
Paste the output of `terraform apply` showing the resources created and the public IP assigned.

**Server Confirmation**
Describe how you confirmed the server was running. Paste the URL or IP you tested, and what the browser returned.

**Architecture Diagram**
Describe your diagram or link to it. Explain the components you included and why.

**Chapter 2 Learnings**
What is the difference between a provider block and a resource block? What does the `terraform plan` step actually do before applying changes?

**Challenges and Fixes**
What went wrong? What error messages did you hit and how did you resolve them?

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terraform AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [AWS EC2 User Data Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html)
- [draw.io Architecture Diagrams](https://app.diagrams.net/)
- [Terraform CLI Commands Reference](https://developer.hashicorp.com/terraform/cli/commands)
- [AWS Free Tier Instance Types](https://aws.amazon.com/free/)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
