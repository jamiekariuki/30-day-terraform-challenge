# Day 12: Zero-Downtime Deployments with Terraform

## What You Will Accomplish Today

Deploying infrastructure updates without taking your application offline is one of the hardest problems in operations — and one of the most valuable skills you can demonstrate. Today you will learn how Terraform handles this using `create_before_destroy` lifecycle rules, and how to implement a proper **blue/green deployment** strategy using Auto Scaling Groups and a load balancer. By the end of today you will have deployed a live application update with zero service interruption and understood exactly why it worked.

---

## Tasks

### 1. Read

**Book:** *Terraform: Up & Running* by Yevgeniy Brikman — **Chapter 5**, pages 169–189

Read the entire *Zero-Downtime Deployment Techniques* section. Pay close attention to:
- Why the default Terraform behaviour causes downtime during updates
- How `create_before_destroy` changes the order of operations
- The specific problem with ASG names and how to solve it with `random_id`
- The limitations the author identifies — these come up in real deployments and in the exam

---

### 2. Complete the Hands-On Labs

- **Lab 1:** [Module Composition](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2004%20-%20Understand%20Terraform%20Basics/13%20-%20Module_Composition.md)
- **Lab 2:** [Module Versioning](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2004%20-%20Understand%20Terraform%20Basics/14%20-%20Module_Versioning.md)

---

### 3. Understand Why Default Terraform Causes Downtime

When you update a resource that cannot be modified in-place — like a Launch Configuration or Launch Template — Terraform's default behaviour is to **destroy first, then create**. For an ASG, this means:

1. Old ASG is destroyed → your instances are terminated → your app goes down
2. New ASG is created → new instances spin up → your app comes back

The downtime window between steps 1 and 2 can be minutes. For any production service this is unacceptable.

The fix is the `create_before_destroy` lifecycle rule, which reverses the order:

1. New ASG is created with new instances → new instances pass health checks
2. Old ASG is destroyed → traffic has already shifted to the new instances

---

### 4. Implement `create_before_destroy`

Add the lifecycle block to your Launch Configuration and ASG:

```hcl
resource "aws_launch_configuration" "example" {
  image_id        = var.ami
  instance_type   = var.instance_type
  security_groups = [aws_security_group.instance.id]

  user_data = templatefile("${path.module}/user-data.sh", {
    server_port = var.server_port
    db_address  = var.db_address
    db_port     = var.db_port
  })

  lifecycle {
    create_before_destroy = true
  }
}
```

**The ASG naming problem:** When `create_before_destroy` is true, the new resource must exist alongside the old one before the old is destroyed. But AWS does not allow two ASGs with the same name to exist simultaneously. If your ASG name is hardcoded, the apply will fail.

The solution is to make the ASG name unique per deployment using a `random_id` resource or a name prefix instead of a fixed name:

```hcl
resource "random_id" "server" {
  keepers = {
    # A new random ID is generated when the launch configuration changes
    ami_id = var.ami
  }
  byte_length = 8
}

resource "aws_autoscaling_group" "example" {
  # Use name_prefix instead of name to let AWS generate a unique name
  name_prefix          = "${var.cluster_name}-"
  launch_configuration = aws_launch_configuration.example.name
  vpc_zone_identifier  = data.aws_subnets.default.ids

  target_group_arns = [aws_lb_target_group.asg.arn]
  health_check_type = "ELB"

  min_size = var.min_size
  max_size = var.max_size

  lifecycle {
    create_before_destroy = true
  }

  tag {
    key                 = "Name"
    value               = var.cluster_name
    propagate_at_launch = true
  }
}
```

---

### 5. Deploy and Verify Zero-Downtime

**Step 1 — Initial deployment:**
Deploy your cluster with version 1 of your user data script. Confirm the ALB DNS name returns your v1 response.

**Step 2 — Simulate a continuous traffic check:**
While you make changes, open a second terminal and run a loop that hits your ALB every two seconds:

```bash
while true; do
  curl -s http://<your-alb-dns-name>
  sleep 2
done
```

Watch the output carefully throughout the next step.

**Step 3 — Deploy version 2:**
Change something visible in your user data — update the HTML response text from "Hello World v1" to "Hello World v2". Run `terraform apply`.

Observe: the traffic loop in your second terminal should continue returning responses throughout the entire apply. At some point the responses will switch from v1 to v2 — but there should be no connection errors or timeouts during the transition.

**Step 4 — Document the transition:**
Note the exact moment the responses switched versions. Paste the terminal output showing the clean v1 → v2 transition with no errors.

---

### 6. Implement a Blue/Green Deployment

Blue/green deployment takes zero-downtime further by maintaining two complete, separate environments — blue (current live) and green (new version) — and shifting traffic between them atomically at the load balancer level.

```hcl
variable "active_environment" {
  description = "Which environment is currently active: blue or green"
  type        = string
  default     = "blue"
}

resource "aws_lb_listener_rule" "blue_green" {
  listener_arn = aws_lb_listener.http.arn
  priority     = 100

  action {
    type             = "forward"
    target_group_arn = var.active_environment == "blue" ? aws_lb_target_group.blue.arn : aws_lb_target_group.green.arn
  }

  condition {
    path_pattern {
      values = ["/*"]
    }
  }
}
```

To switch from blue to green, change `active_environment = "green"` and run `terraform apply`. The listener rule updates atomically — traffic shifts in a single API call with no downtime window.

Deploy both target groups with different versions of your app. Test the switch, confirm it works cleanly, and then switch back.

---

### 7. Bonus — Record a Screencast

Record a short screen capture walking through your zero-downtime deployment. Show the traffic loop running in one terminal, your `terraform apply` running in another, and the moment traffic switches from v1 to v2 without any errors. Share it on YouTube or LinkedIn. This is some of the most compelling content you can create for your portfolio — live proof that your infrastructure works under real conditions.

---

### 8. Write Your Blog Post

**Post title:** *Mastering Zero-Downtime Deployments with Terraform*

Cover why default Terraform causes downtime, how `create_before_destroy` fixes it, the ASG naming problem and solution, and the blue/green pattern. Include your terminal output showing the clean version transition. This is a post that engineers actively search for when they are about to do their first production deployment.

---

### 9. Post on Social Media

> 🚀 Day 12 of the 30-Day Terraform Challenge — zero-downtime deployments with Terraform. Deployed a live update with `create_before_destroy`, watched traffic shift from v1 to v2 with zero errors, then implemented full blue/green switching at the load balancer level. Production-ready infrastructure. #30DayTerraformChallenge #TerraformChallenge #Terraform #DevOps #ZeroDowntime #IaC #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL or a link to your deployment code.
2. **Live App Link field** — paste the ALB DNS name showing your v2 deployment, your screencast URL, or your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**The Problem — Default Downtime**
Explain in your own words why Terraform's default destroy-then-create behaviour causes downtime. What is the sequence of events and where does the outage window occur?

**`create_before_destroy` Implementation**
Paste your Launch Configuration and ASG code with the lifecycle block. Explain the ASG naming problem and show how you solved it with `name_prefix`.

```hcl
# paste your lifecycle implementation here
```

**Zero-Downtime Deployment Evidence**
Paste the terminal output from your traffic loop showing the clean v1 → v2 transition. No errors, no timeouts — just the moment responses switch versions.

**Blue/Green Configuration**
Paste your listener rule and target group configuration. Explain how the `active_environment` variable controls which target group receives traffic and why the switch is instantaneous.

```hcl
# paste your blue/green configuration here
```

**Blue/Green Switch Evidence**
Describe the switch from blue to green. How long did `terraform apply` take? Was there any observable interruption in the traffic loop?

**Limitations**
Chapter 5 identifies specific limitations of the `create_before_destroy` approach. What are they? How does blue/green address some of them while introducing its own tradeoffs?

**Screencast**
Paste the YouTube or LinkedIn URL if you recorded one. If not, describe in detail what you would show in a recording.

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terraform Lifecycle Meta-Arguments](https://developer.hashicorp.com/terraform/language/meta-arguments/lifecycle)
- [AWS Auto Scaling — Instance Refresh](https://docs.aws.amazon.com/autoscaling/ec2/userguide/asg-instance-refresh.html)
- [Blue/Green Deployments on AWS](https://docs.aws.amazon.com/whitepapers/latest/blue-green-deployments/introduction.html)
- [Terraform `random_id` Resource](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/id)
- [AWS ALB Listener Rules](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/listener-update-rules.html)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
