# Day 28: Exam Preparation — Practice Exams 1 & 2

## What You Will Accomplish Today

Two full practice exams today. 57 questions each, timed, no looking things up mid-exam. This is not casual review — this is deliberate simulation of real exam conditions. Your goal is not just to get a score but to identify exactly which topics are still shaky so you can close those gaps before the actual exam. The way you use your wrong answers today determines how well you perform on exam day.

---

## Tasks

### 1. Take Practice Exam 1

Set a 60-minute timer. Work through 57 questions without pausing and without using any reference material. Flag questions you are uncertain about but do not stop to look anything up — keep moving.

Use these resources for your practice questions:

- [HashiCorp Official Sample Questions](https://learn.hashicorp.com/tutorials/terraform/associate-questions)
- [Terraform Associate Review Tutorial](https://developer.hashicorp.com/terraform/tutorials/certification-003/associate-review-003)
- [ExamTopics Terraform Associate](https://www.examtopics.com/exams/hashicorp/ta-003/)

After the timer stops, score yourself. The passing threshold is **70% (40/57)**. Record your score and which questions you got wrong before moving on to Exam 2.

---

### 2. Take Practice Exam 2

Take a 15-minute break, then set the timer again. Complete a second full set of 57 questions under the same conditions. Use a different question source from Exam 1 if possible to maximise coverage.

Score and record Exam 2 separately. Compare your two scores — if your second score is lower than your first, you are fatiguing. If it is higher, the warm-up effect is real and you should consider doing a short review session before your actual exam.

---

### 3. Conduct a Structured Wrong-Answer Review

For every question you got wrong on either exam, do this process — not just reading the correct answer, but actually understanding why each wrong answer was wrong:

```
Question topic: [e.g. terraform state rm behaviour]
What I answered: [my wrong answer]
Correct answer: [the correct answer]
Why I was wrong: [my actual reasoning error, not just "I didn't know"]
Doc reference: [link to the relevant Terraform documentation section]
Hands-on fix: [what I will run in my terminal to reinforce this]
```

Document every single wrong answer this way. This is the most valuable work of the day. A wrong answer you have analysed properly will not fool you on exam day.

---

### 4. Identify Your Weak Domains

After both exams, calculate your accuracy by domain:

| Domain | Questions Attempted | Correct | Accuracy |
|---|---|---|---|
| IaC concepts | | | |
| Terraform's purpose | | | |
| Terraform basics | | | |
| Terraform CLI | | | |
| Terraform modules | | | |
| Core workflow | | | |
| State management | | | |
| Configuration | | | |
| Terraform Cloud | | | |

Any domain below 70% needs targeted review before Day 30.

---

### 5. Hands-On Revision for Weak Areas

For each domain where your accuracy was below 70%, do a hands-on exercise — not just re-reading documentation. Run actual commands. If you got state management questions wrong, run `terraform state list`, `terraform state show`, `terraform state mv`, and `terraform state rm` against a test resource. If you got module questions wrong, write a small module and call it from a root configuration.

Learning the answer to an exam question by doing the thing is ten times more durable than learning it by reading about the thing.

---

### 6. Write Your Blog Post

**Post title:** *How I Prepared for the Terraform Associate Exam with Practice Questions*

Share your scores from both exams, the wrong-answer review process, which domains surprised you, and the specific hands-on exercises you did to close the gaps. Be honest about your scores — the most useful blog posts for other candidates are the ones that show the real process, not a highlight reel.

---

### 7. Post on Social Media

> 🚀 Day 28 of the 30-Day Terraform Challenge — two full practice exams done. Scored myself by domain, analysed every wrong answer, and ran hands-on exercises to close the gaps. Two days to go. #30DayTerraformChallenge #TerraformChallenge #Terraform #TerraformAssociate #CertificationPrep #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL.
2. **Live App Link field** — paste your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**Exam Scores**
Exam 1 score: X/57 (X%). Exam 2 score: X/57 (X%). Note whether your score improved between exams and what you think accounts for the difference.

**Domain Accuracy Table**
Paste your completed domain accuracy table. Circle or highlight any domain below 70%.

**Wrong-Answer Analysis**
Complete the structured wrong-answer review for every question you missed. Paste at least five entries in full using the template above.

**Hands-On Revision**
Describe the hands-on exercises you ran to address your weak domains. Paste the commands and their output where relevant.

**Pattern Recognition**
After two exams, what patterns do you notice in the questions you are consistently missing? Is it a particular command, a specific scenario type, or a conceptual area?

**Plan for Days 29 and 30**
Based on your scores today, what specific topics will you prioritise in the final two days?

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terraform Associate Study Guide](https://developer.hashicorp.com/terraform/tutorials/certification-003/associate-study-003)
- [Official Sample Questions](https://learn.hashicorp.com/tutorials/terraform/associate-questions)
- [Terraform CLI Commands Reference](https://developer.hashicorp.com/terraform/cli/commands)
- [Terraform State Documentation](https://developer.hashicorp.com/terraform/language/state)
- [Exam Registration](https://www.hashicorp.com/certification/terraform-associate)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
