# Day 24: Final Exam Review and Certification Focus

## What You Will Accomplish Today

This is the sharpest day of the challenge — no new concepts, no new deployments. Pure focused exam preparation. You will simulate real exam conditions, work through your remaining weak areas from yesterday's audit, and build the exam-day strategy that gives you the best chance of passing on your first attempt. The Terraform Associate exam rewards people who know the material precisely, not just conceptually. Today you close the gap between "I understand this" and "I can answer a question about this under time pressure."

---

## Tasks

### 1. Review the Official Documentation — Focused Pass

Return to the study guide and official docs. Today you are not reading broadly — you are drilling specifically on your yellow and red topics from yesterday's audit:

[Terraform Associate Certification Study Guide](https://developer.hashicorp.com/terraform/tutorials/certification-003/associate-study-003)

For each red topic, spend at least 30 minutes: read the documentation, find the relevant code example, run it if possible, and write a practice question about it. Do not move on until you can explain it without looking at notes.

---

### 2. Full Exam Simulation

Set a timer for 60 minutes. Work through 57 practice questions without pausing, without looking anything up, and without skipping questions. Flag ones you are uncertain about and return to them after completing the rest.

Use these sources for practice questions:
- [Official HashiCorp Sample Questions](https://learn.hashicorp.com/tutorials/terraform/associate-questions)
- [Terraform Associate Review Tutorial](https://developer.hashicorp.com/terraform/tutorials/certification-003/associate-review-003)
- Your five original questions from Day 23

After the timer stops, score yourself. The passing threshold is 70%. Record your score and the specific questions you got wrong.

---

### 3. Deep Drill on High-Weight Domains

The four heaviest domains account for 78% of the exam. If you are short on time, focus here:

**Terraform basics (24%)**

Know these cold:
- The difference between `terraform.tfstate` and `terraform.tfstate.backup`
- What `terraform refresh` does (syncs state with real infrastructure — note: deprecated in favour of `terraform apply -refresh-only`)
- The resource meta-arguments: `depends_on`, `count`, `for_each`, `lifecycle`, `provider`
- Built-in functions: `file()`, `templatefile()`, `lookup()`, `merge()`, `length()`, `toset()`, `tolist()`, `tomap()`
- The difference between `locals`, `variables`, and `outputs`
- What a `data` source is and how it differs from a `resource`

```hcl
# Know what each of these produces
locals {
  merged = merge(var.common_tags, { Name = "example" })
  upper_names = [for name in var.names : upper(name)]
  name_map = { for name in var.names : name => length(name) }
}
```

**Terraform CLI (26%)**

Know the exact behaviour of these scenarios — they are common exam traps:

- `terraform init` with `-upgrade` flag: forces provider version upgrades even when lock file pins them
- `terraform plan` with `-target`: limits plan to specific resources (useful for troubleshooting, not recommended for routine use)
- `terraform apply` with `-auto-approve`: skips the interactive approval prompt
- `terraform destroy` is equivalent to `terraform apply -destroy`
- `terraform state rm` removes from state only — does NOT destroy the real resource
- `terraform import` brings existing infrastructure under state management but does NOT generate the `.tf` configuration file (you must write the resource block yourself)
- `terraform workspace new` creates a new workspace AND switches to it
- `terraform output -json` returns outputs in JSON format

**IaC concepts (16%)**

- Declarative vs imperative IaC
- Idempotency — applying the same configuration multiple times produces the same result
- Immutable infrastructure — replacing rather than modifying
- Configuration drift — the gap between declared state and actual state
- Benefits of IaC: version control, code review, automation, documentation

**Terraform's purpose (20%)**

- Provider-agnostic: Terraform works with any provider that has an API
- State as the source of truth: Terraform depends on state to map configuration to real resources
- The core workflow: Write → Plan → Apply
- Terraform Cloud vs Terraform Enterprise: Cloud is SaaS, Enterprise is self-hosted

---

### 4. Exam-Day Strategy

**Time management**
57 questions in 60 minutes = just over 1 minute per question. Questions are not worth different amounts — a hard question counts the same as an easy one. Move through quickly, flag uncertain ones, and return to them after completing the rest.

**Elimination strategy**
Most questions have 4 answer choices. On any question where you are unsure, eliminate the clearly wrong answers first. Often you can get to 2 choices and then apply your knowledge to pick between them.

**Watch for these common traps:**
- "Terraform plan shows no changes" questions — always consider whether the state file might be stale
- `terraform destroy` vs `terraform state rm` — destroy removes the real resource, `state rm` does not
- Required vs optional variables — required variables with no default cause `terraform plan` to prompt for input
- Module source pinning — `?ref=main` is a branch (mutable), `?ref=v1.0.0` is a tag (immutable)
- `sensitive = true` does not prevent storage in state — it only masks output in terminal

**On multi-select questions:**
Read the question carefully — "select TWO" means exactly two. If you select one or three, the question is marked wrong regardless of which answers you chose.

---

### 5. Final Knowledge Check — Flash Card Style

Test yourself on each of these without looking anything up. Write your answer, then verify:

1. What file does `terraform init` create to record provider versions?
2. What is the difference between `terraform.workspace` and a Terraform Cloud workspace?
3. If you run `terraform state rm aws_instance.web`, what happens to the EC2 instance in AWS?
4. What does the `depends_on` meta-argument do and when should you use it?
5. What is the purpose of the `.terraform.lock.hcl` file?
6. How does `for_each` differ from `count` when items are removed from the middle of a collection?
7. What does `terraform apply -refresh-only` do?
8. What is the maximum number of items you can specify in a single `terraform import` command? (Answer: one)
9. What happens when you run `terraform plan` against a workspace that has never been applied?
10. What does the `prevent_destroy` lifecycle argument do and what does it NOT prevent?

---

### 6. Write Your Blog Post

**Post title:** *My Final Preparation for the Terraform Associate Exam*

Cover your exam simulation results, the specific topics you drilled, your exam-day strategy, and the resources that were most useful in your preparation. Make this the post you wish you had found when you started studying. Share your score from the simulation and what you did to close the gap.

---

### 7. Post on Social Media

> 🎓 Day 24 of the 30-Day Terraform Challenge — final exam prep. Full simulation under timed conditions, drilled the high-weight domains, built my exam-day strategy. Whatever the score is, I know this material better than I did 24 days ago. Let's go. #30DayTerraformChallenge #TerraformChallenge #Terraform #TerraformAssociate #CertificationPrep #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL.
2. **Live App Link field** — paste your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**Exam Simulation Score**
How many questions did you answer correctly out of 57? Which domains were you weakest in? Paste the topics of the questions you got wrong.

**Flash Card Answers**
Answer all ten flash card questions above in your own words. Do not look them up first — record your answers, then verify and note any corrections.

**High-Weight Domain Drill**
For each of the four high-weight domains, write three facts you now know precisely that you were fuzzy on before today's study session.

**Common Traps**
Add three more exam traps to the list above based on your own experience with the practice questions. Explain why each one is a trap.

**Exam-Day Strategy**
Write out your personal exam-day strategy in 5–7 bullet points. Be specific — not "manage your time" but "spend max 90 seconds on any single question before flagging and moving on."

**Remaining Red Topics**
Are there still any red topics from yesterday's audit? If yes, name them and describe exactly how you plan to address them before your exam date.

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terraform Associate Review Tutorial](https://developer.hashicorp.com/terraform/tutorials/certification-003/associate-review-003)
- [Terraform Built-in Functions Reference](https://developer.hashicorp.com/terraform/language/functions)
- [Terraform Resource Meta-Arguments](https://developer.hashicorp.com/terraform/language/meta-arguments/lifecycle)
- [Official Sample Questions](https://learn.hashicorp.com/tutorials/terraform/associate-questions)
- [Exam Registration — HashiCorp Certifications](https://www.hashicorp.com/certification/terraform-associate)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
