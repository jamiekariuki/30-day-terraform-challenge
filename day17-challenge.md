# Day 17: Manual Testing of Terraform Code

## What You Will Accomplish Today

Testing is what separates engineers who hope their infrastructure works from engineers who know it does. Today you start Chapter 9 and focus on the first layer of testing: manual testing. Before you can write automated tests, you need to know exactly what you are testing, why, and how to verify it. You will build a structured manual testing process for your existing infrastructure, document your findings, and establish the cleanup discipline that prevents runaway AWS costs.

---

## Tasks

### 1. Read

**Book:** *Terraform: Up & Running* by Yevgeniy Brikman — **Chapter 9**

Read the following sections:
- *Manual Tests*
- *Manual Testing Basics*
- *Cleaning Up After Tests*

Focus on the author's argument for why manual testing is a necessary precursor to automated testing, and what specific checks a manual test should cover that automated tests sometimes miss.

---

### 2. Complete the Hands-On Labs

- **Lab 1:** [State Migration](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2005%20-%20Use%20Terraform%20Outside%20of%20Core%20Workflow/02%20-%20Terraform_State_Migration.md)
- **Lab 2:** [Import Existing Infrastructure](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2005%20-%20Use%20Terraform%20Outside%20of%20Core%20Workflow/04%20-%20Terraform_Import.md)

---

### 3. Build a Manual Testing Checklist

Before running any tests, define exactly what you are verifying. A manual test without a checklist is just clicking around. Create a structured test plan for your webserver cluster that covers these categories:

**Provisioning verification**
- Does `terraform init` complete without errors?
- Does `terraform validate` pass cleanly?
- Does `terraform plan` show the expected number and type of resources?
- Does `terraform apply` complete without errors?

**Resource correctness**
- Are all expected resources visible in the AWS Console?
- Do resource names, tags, and regions match what your variables specify?
- Are security group rules exactly as defined — no extra rules, no missing rules?

**Functional verification**
- Does the ALB DNS name resolve?
- Does `curl http://<alb-dns>` return the expected response?
- Do all instances in the ASG pass health checks?
- If you stop one instance manually, does the ASG replace it automatically?

**State consistency**
- Does `terraform plan` return "No changes" immediately after a fresh apply?
- Does the state file accurately reflect what exists in AWS?

**Regression check**
- Make a small change to your configuration (add a tag, change a description). Does `terraform plan` show only that change and nothing unexpected?
- Apply it. Does `terraform plan` return clean afterward?

---

### 4. Execute the Tests and Document Everything

Work through your checklist against your webserver cluster from Days 3–16. For each test:

- Record the command you ran
- Record the actual output
- Record whether it passed or failed
- If it failed, document what was wrong and how you fixed it

Use this structure in your documentation:

```
Test: ALB DNS resolves and returns expected response
Command: curl -s http://my-app-alb-123456.us-east-1.elb.amazonaws.com
Expected: "Hello World v2"
Actual: "Hello World v2"
Result: PASS
```

```
Test: terraform plan returns clean after apply
Command: terraform plan
Expected: "No changes. Your infrastructure matches the configuration."
Actual: 1 resource change detected — missing tag on security group
Result: FAIL
Fix: Added missing tag to aws_security_group.instance resource, re-applied
```

This is not busywork — documenting failures is the most valuable output of manual testing. Every failure you find manually is a test case you can later automate.

---

### 5. Test Multiple Environments

Run your manual test suite against both your dev and production configurations. Document whether the behaviour differs between environments in any unexpected way. A common finding: something works in dev and fails in production because of a difference in instance type, region, or security group configuration that was not caught by `terraform validate`.

---

### 6. Clean Up Properly

After every manual test run, destroy all test resources immediately. Establish this as a habit — not an afterthought:

```bash
# Always confirm what you are destroying before running
terraform plan -destroy

# Destroy with explicit approval
terraform destroy
```

After destroying, verify cleanup in the AWS Console. Check EC2, security groups, load balancers, and target groups. Terraform occasionally leaves orphaned resources when a destroy fails partway through — find and manually delete anything left behind.

Document your cleanup process:

```bash
# Post-destroy verification commands
aws ec2 describe-instances --filters "Name=tag:ManagedBy,Values=terraform" --query "Reservations[*].Instances[*].InstanceId"
aws elbv2 describe-load-balancers --query "LoadBalancers[*].LoadBalancerArn"
```

Both commands should return empty results after a clean destroy.

---

### 7. Write Your Blog Post

**Post title:** *The Importance of Manual Testing in Terraform*

Cover why manual testing matters even when automated testing exists, how to build a structured test checklist, the difference between provisioning verification and functional verification, and why cleanup discipline is as important as the tests themselves. Include your actual test results — passes and failures both — as examples.

---

### 8. Post on Social Media

> 🔍 Day 17 of the 30-Day Terraform Challenge — manual testing deep dive. Built a structured test checklist, ran it against dev and production environments, documented every pass and failure. Manual testing is not optional — it is the foundation everything else is built on. #30DayTerraformChallenge #TerraformChallenge #Terraform #Testing #DevOps #IaC #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL.
2. **Live App Link field** — paste your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**Your Test Checklist**
Paste your complete manual test checklist organised by category. This should be something another engineer could pick up and run without asking you any questions.

**Test Execution Results**
Paste your test results using the structured format above — command, expected, actual, result. Include at least one failure and how you resolved it.

**Multi-Environment Comparison**
What differences did you find between dev and production during testing? Did anything behave unexpectedly in one environment but not the other?

**Cleanup Verification**
Paste the output of your post-destroy AWS CLI verification commands confirming all resources were removed.

**Chapter 9 Learnings**
In your own words — what does the author mean by "cleaning up after tests" and why does he say this is harder than it sounds? What is the risk of not cleaning up between test runs?

**Lab Takeaways**
What did the Import lab teach you? What problem does `terraform import` solve and what does it not solve?

**Challenges and Fixes**
What failed during your manual tests? Document every failure with a clear explanation of root cause and fix.

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terraform Testing Documentation](https://developer.hashicorp.com/terraform/language/tests)
- [Terraform Validate Command](https://developer.hashicorp.com/terraform/cli/commands/validate)
- [Terraform Import Command](https://developer.hashicorp.com/terraform/cli/commands/import)
- [AWS CLI EC2 Describe Instances](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instances.html)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
