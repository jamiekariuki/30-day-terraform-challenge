# Day 30: Final Practice Exam, Fill-in-the-Blank, and Challenge Complete

## What You Will Accomplish Today

This is Day 30. You started this challenge as someone learning Terraform. You are finishing it as someone who has deployed servers, clusters, databases, static websites, and Kubernetes workloads, written automated tests, built CI/CD pipelines, managed state across environments, and understood how infrastructure deployment works at a professional level. Today you complete your final exam preparation and celebrate what you have built. Then you go pass that certification.

---

## Tasks

### 1. Take Practice Exam 5 — Final Simulated Exam

This is your last full simulation before the real exam. Treat it with the same discipline as the previous four: 60-minute timer, 57 questions, no reference material.

After this exam, do not spend more time on topics you still feel shaky on — at this point you know what you know. Your job now is to consolidate, not to learn new material. Review your wrong answers briefly, but resist the temptation to go deep into new documentation today. That work is done.

---

### 2. Complete the Fill-in-the-Blank Questions

Work through 20 fill-in-the-blank questions. These test precise knowledge of command syntax, argument names, and specific Terraform behaviour — the kind of precision that the multiple-choice format sometimes lets you get away without. They are excellent preparation for the exam's scenario-based questions.

Write out your answers before checking them. The act of retrieving the answer from memory rather than recognising it among choices is the highest-quality practice you can do at this stage.

**Sample fill-in-the-blank questions to self-test with:**

1. The command to check the formatting of Terraform code without making changes is `terraform ___`.
2. The meta-argument that prevents a resource from being destroyed is `___ = true` inside a `lifecycle` block.
3. To reference the current workspace name inside a configuration, you use `terraform.___`.
4. The backend block for storing state in S3 requires a `___` argument to enable server-side encryption.
5. The `for_each` meta-argument requires the value to be a `map` or a `___`.
6. The command that removes a resource from Terraform state without destroying the real infrastructure is `terraform state ___`.
7. Provider version constraint `~> 2.0` allows versions `>= 2.0` and `< ___`.
8. A `data` block reads ___ infrastructure; a `resource` block manages ___ infrastructure.
9. The `terraform init -upgrade` flag forces Terraform to update provider versions even when they are pinned in the `___` file.
10. To apply a previously saved plan file named `myplan.tfplan`, the command is `terraform apply ___`.

Write your answers first, then verify against the official documentation. Record every question you got wrong or were uncertain about.

---

### 3. Final Readiness Check

Before you book or walk into your exam, answer these ten questions cold — no notes, no docs:

1. What does `terraform init` do to your `.terraform` directory?
2. What is the difference between `terraform.tfstate` and `terraform.tfstate.backup`?
3. Why should you never commit `terraform.tfstate` to version control?
4. What does `depends_on` do and when should you use it?
5. What is the difference between a `variable` block and a `locals` block?
6. What happens if you run `terraform apply` and the state file has been modified by another team member since your last `terraform plan`?
7. What does the `terraform graph` command output and what is it used for?
8. What is the Terraform Registry and what are the three types of things published there?
9. What is the difference between Terraform Cloud and Terraform Enterprise?
10. When a module uses `configuration_aliases`, what problem does it solve?

If you can answer all ten confidently, you are ready for the exam.

---

### 4. Exam Day Logistics

Confirm all of the following before your exam:

- You have registered at [hashicorp.com/certification](https://www.hashicorp.com/certification/terraform-associate) and have your exam time booked
- You know whether your exam is online proctored or at a testing centre
- For online proctored: your room is quiet, your desk is clear, your webcam works, and you have your government ID ready
- You have read the exam policies and know what is and is not permitted
- You know your exam score will appear in your Credly account within 24 hours of completion

---

### 5. Reflect on 30 Days

You have completed one of the most intensive self-directed learning programs in the cloud engineering community. Take 20 minutes to write a genuine reflection. Not a list of technologies. A reflection.

**What changed?** Not what you learned — that is a list. What actually changed about how you think about infrastructure, about automation, about what it means to manage systems professionally?

**What are you most proud of?** Pick one specific thing you built or figured out. Not the easiest thing — the thing that was genuinely hard and that you pushed through anyway.

**What comes next?** You have the certification prep. What is the first real project, team, or system where you will apply everything from this challenge?

---

### 6. Write Your Final Blog Post

**Post title:** *Ready for Terraform Certification: My Final Exam Prep and 30-Day Reflection*

This is your capstone post. Cover your final practice exam, the fill-in-the-blank results, your readiness assessment, and then write the reflection. This post should be the one someone reads when they are considering doing this challenge. Make it honest, make it specific, and make it worth reading.

---

### 7. Post on Social Media

> 🎉 Day 30 of the 30-Day Terraform Challenge — complete. Five practice exams, 30 days of builds, modules, state management, testing, CI/CD, and a full certification prep programme. Thank you to AWS AI/ML UserGroup Kenya, Meru HashiCorp User Group, and EveOps for making this happen. Now let's go pass that exam. #30DayTerraformChallenge #TerraformChallenge #Terraform #TerraformAssociate #IaC #DevOps #AWSUserGroupKenya #MeruHashiCorp #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your final day's work.

1. **Repository Link field** — paste your final blog post URL.
2. **Live App Link field** — paste your social media post URL.
3. **Documentation editor** — write your complete final submission below.

### Your documentation must include:

**Practice Exam 5 Score**
X/57 (X%). How does this compare to your Exam 1 score from Day 28? What does the improvement (or stability) tell you about your preparation?

**Fill-in-the-Blank Results**
Paste your answers to the ten sample questions above before you checked them, then show the corrections. For any question you got wrong, write a clear explanation of the correct answer.

**Final Readiness Check**
Answer all ten readiness questions in full. If you cannot answer any of them confidently, note it and describe how you will address it before your exam.

**Five-Exam Score Summary**
| Exam | Score | % |
|---|---|---|
| Exam 1 | | |
| Exam 2 | | |
| Exam 3 | | |
| Exam 4 | | |
| Exam 5 | | |

**30-Day Reflection**
Answer all three reflection questions. Write at least a paragraph for each. This is not a checklist — write something genuine.

**Exam Logistics Confirmation**
Confirm your exam is booked. If it is not yet booked, describe your plan for when you will take it.

**Closing Message to the Community**
Write a short message to future participants of this challenge — something you wish someone had told you on Day 1.

**Blog Post**
Paste the URL.

**Social Media**
Paste the URL to your post.

---

## Fill-in-the-Blank Answers (verify after attempting)

1. `fmt` — `terraform fmt` checks and reformats code
2. `prevent_destroy` — `lifecycle { prevent_destroy = true }`
3. `workspace` — `terraform.workspace`
4. `encrypt` — `encrypt = true`
5. `set` — `for_each` accepts a `map` or a `set`
6. `rm` — `terraform state rm`
7. `3.0` — `~> 2.0` allows `>= 2.0, < 3.0`
8. `existing` / `managed` — data reads existing; resource manages
9. `.terraform.lock.hcl` — the dependency lock file
10. `myplan.tfplan` — `terraform apply myplan.tfplan`

---

## Additional Resources

- [Exam Registration](https://www.hashicorp.com/certification/terraform-associate)
- [Terraform Associate Study Guide](https://developer.hashicorp.com/terraform/tutorials/certification-003/associate-study-003)
- [Credly — HashiCorp Badges](https://www.credly.com/organizations/hashicorp/badges)
- [Terraform Associate Review](https://developer.hashicorp.com/terraform/tutorials/certification-003/associate-review-003)

---

*Congratulations on completing the 30-Day Terraform Challenge. This programme was brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**. Now go pass that exam. 🚀*
