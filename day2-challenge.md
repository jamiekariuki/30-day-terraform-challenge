# Day 2: Setting Up Your Terraform Environment

## What You Will Accomplish Today

Today is pure setup. By the end of this day your machine will be fully configured and ready to deploy real infrastructure. Every tool you install today will be used every single day for the rest of this challenge — invest the time to get this right.

If you completed the environment setup on Day 1, use today to verify everything is working correctly and deepen your understanding of how these tools connect to each other.

---

## Tasks

### 1. Read

**Book:** *Terraform: Up & Running* by Yevgeniy Brikman — **Chapter 2**

Focus on the sections covering AWS account setup and Terraform installation. Pay attention to how Terraform authenticates with AWS — this is a common source of confusion early on.

---

### 2. Complete the Hands-On Labs

Work through all three labs in order. Each one builds on the previous.

- **Lab 1:** [Set Up Your AWS Account](https://youtu.be/ne8LrbCzW0Q?si=RJToIfUApVTUW1ex) — skip if you already have an account, but watch it anyway for the IAM and billing setup tips.
- **Lab 2:** [Install and Configure the AWS CLI](https://www.youtube.com/watch?v=gx5XVAS-ZC8)
- **Lab 3:** [Install Terraform](https://www.youtube.com/watch?v=Cn6xYf0QJME) and [Connect Terraform to AWS](https://youtu.be/4ZCrRbPR3gc?si=hqrAdi9PurHQfDKh)

---

### 3. Set Up Your Environment

Complete every item below. Do not move to Day 3 with a broken or partial setup.

- **AWS Account** — Create one if you do not have it. Enable MFA on your root account immediately. Set up a billing alert so you are not surprised by charges.
- **IAM User** — Create a dedicated IAM user for Terraform with programmatic access. Do not use your root account credentials.
- **Terraform** — Install the latest version. Run `terraform version` to confirm.
- **AWS CLI** — Install and run `aws configure` with your IAM user credentials. Set your default region.
- **Visual Studio Code** — Install the **HashiCorp Terraform** extension and the **AWS Toolkit** plugin.
- **Full Validation** — Run all four commands below and confirm clean output:

```
terraform version
aws --version
aws sts get-caller-identity
aws configure list
```

---

### 4. Write Your Blog Post

**Post title:** *Step-by-Step Guide to Setting Up Terraform, AWS CLI, and Your AWS Environment*

Your post should walk through your exact setup process. Be specific — include the commands you ran, the decisions you made (which region, which installation method), and any issues you hit along the way. A detailed setup guide is one of the most searched pieces of content in the Terraform community.

Publish it and save the link for your submission.

---

### 5. Post on Social Media

> 💻 Day 2 of the 30-Day Terraform Challenge — full environment setup done. Terraform installed, AWS CLI configured, and ready to start deploying infrastructure. The journey continues! #30DayTerraformChallenge #TerraformChallenge #TerraformSetup #AWS #DevOps #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL here.
2. **Live App Link field** — paste your social media post URL here.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**Setup Validation**
Paste the output of all four validation commands as code blocks. This is your proof of a working environment:

```
terraform version        → paste output
aws --version            → paste output
aws sts get-caller-identity  → paste output (you may redact your Account ID)
aws configure list       → paste output
```

**VSCode Extensions**
List the extensions you installed. Name them exactly as they appear in VSCode.

**Setup Challenges**
Document any issues you ran into and exactly how you resolved them. This is some of the most valuable content you can write — others will hit the same issues.

**Chapter 2 Learnings**
What did you learn about how Terraform authenticates with AWS? What would happen if you used root credentials instead of an IAM user?

**Blog Post**
Paste the URL and write a short summary of what your post covers.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terraform Installation Docs](https://developer.hashicorp.com/terraform/install)
- [AWS CLI Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- [AWS Free Tier Overview](https://aws.amazon.com/free/)
- [Setting Up AWS Billing Alerts](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
