# Day 1: Introduction to Terraform and Infrastructure as Code (IaC)

## What You Will Accomplish Today

Today is about foundations. Before writing a single line of Terraform, you need to understand *why* it exists, what problem it solves, and how it fits into the modern infrastructure landscape. You will also set up your complete local environment so every day from here runs smoothly.

---

## Tasks

### 1. Read

**Book:** *Terraform: Up & Running* by Yevgeniy Brikman — **Chapter 1**

Read with intention. Focus on what Terraform is, why infrastructure-as-code matters, and how declarative tooling differs from manual provisioning. Take notes — they will feed directly into your blog post today.

---

### 2. Complete the Hands-On Lab

Work through the lab below fully. Do not just read it — run the commands, observe the output, and understand what is happening at each step.

[Benefits of Infrastructure as Code](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2002%20-%20Understand%20IAC%20Concepts/02%20-%20Benefits_of_Infrastructure_as_Code.md)

---

### 3. Set Up Your Environment

Complete every item on this list before moving on. Skipping setup now will slow you down for the rest of the challenge.

- **AWS Account** — Create one if you do not already have it. The free tier is sufficient for this entire challenge.
- **Terraform** — Install the latest version locally. Confirm it works by running `terraform version` in your terminal.
- **AWS CLI** — Install and configure with your credentials by running `aws configure`.
- **Visual Studio Code** — Install VSCode, then add the **HashiCorp Terraform** extension and the **AWS Toolkit** plugin.
- **Verify everything** — Run `terraform version` and `aws sts get-caller-identity`. Both should return clean output with no errors.

---

### 4. Launch Your Blog and Write Your First Post

If you do not have a blog yet, set one up today. Recommended platforms: **Hashnode**, **Dev.to**, **Medium**, or **GitHub Pages**. Pick one and commit to it — you will be publishing weekly throughout this challenge.

**Post title:** *What is Infrastructure as Code and Why It's Transforming DevOps*

Your post should cover:

- What IaC is and the problem it solves
- The difference between declarative and imperative approaches
- Why Terraform is worth learning
- Your personal goals for this 30-day challenge

Publish it before you submit your workspace today.

---

### 5. Post on Social Media

Share your Day 1 progress on **LinkedIn**, **X (Twitter)**, or both. Use this as a starting point and make it yours:

> 🚀 Day 1 of the 30-Day Terraform Challenge done! Just set up my full Terraform environment and started learning Infrastructure as Code. Excited to be part of this challenge alongside the AWS AI/ML UserGroup Kenya, Meru HashiCorp User Group, and EveOps communities. Let's build! #30DayTerraformChallenge #TerraformChallenge #IaC #HashiCorp #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL here.
2. **Live App Link field** — paste your social media post URL here.
3. **Documentation editor** — this is your learning journal. Write clearly and in detail. The community reads your submissions and votes on them — thorough documentation earns more points.

### Your documentation must include:

**Environment Setup**
Confirm what you installed. Paste your terminal output as code blocks:

```
terraform version
aws sts get-caller-identity
```

Include any issues you ran into and how you resolved them. This is genuinely useful for others hitting the same wall.

**Key Learnings from Chapter 1**
In your own words — what is Terraform? What problem does it solve? What surprised or challenged your thinking?

**Lab Takeaways**
What did the Benefits of IaC lab demonstrate? What clicked for you that reading alone did not?

**Blog Post**
Paste the URL to your published post and write a short summary of what you covered.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Official Terraform Documentation](https://www.terraform.io/docs)
- [HashiCorp Learn — Getting Started with Terraform](https://learn.hashicorp.com/terraform)
- [TechWorld with Nana — Terraform Full Course](https://www.youtube.com/playlist?list=PLy7NrYWoggjxCF3avZvc8Zf0_CdNCozpn)
- [Introduction to Terraform by Armon Dadgar, HashiCorp Co-founder](https://www.youtube.com/watch?v=h970ZBgKINg)
- [Martin Fowler on Infrastructure as Code](https://martinfowler.com/bliki/InfrastructureAsCode.html)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
