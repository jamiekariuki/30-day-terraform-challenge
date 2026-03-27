# Day 18: Automated Testing of Terraform Code

## What You Will Accomplish Today

Manual testing is essential but it does not scale. The moment your infrastructure grows beyond what one person can test in an afternoon, you need automated tests that run on every change, catch regressions before they reach production, and give your entire team confidence to move fast. Today you complete Chapter 9 and implement all three layers of Terraform automated testing: unit tests, integration tests, and end-to-end tests. You will also build a CI/CD pipeline that runs your tests automatically on every commit.

---

## Tasks

### 1. Read

**Book:** *Terraform: Up & Running* by Yevgeniy Brikman — **Chapter 9** (complete the chapter)

Read all remaining sections:
- *Automated Tests*
- *Unit Tests*
- *Integration Tests*
- *End-to-End Tests*
- *Other Testing Approaches*

As you read, map each test type to the infrastructure you have built. What would a unit test for your webserver module look like? What would an integration test assert? What does an end-to-end test verify that integration tests cannot?

---

### 2. Complete the Hands-On Labs

- **Lab 1:** [Import Existing Infrastructure](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2005%20-%20Use%20Terraform%20Outside%20of%20Core%20Workflow/04%20-%20Terraform_Import.md)
- **Lab 2:** [Terraform Cloud](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2006%20-%20Navigate%20Terraform%20Workflow/01%20-%20Terraform_Cloud.md)

---

### 3. Unit Tests with `terraform test`

Terraform 1.6+ ships with a native testing framework using `.tftest.hcl` files. This is the lightweight unit testing approach — no external dependencies, no real infrastructure deployed.

Create a test file for your webserver module:

```hcl
# modules/services/webserver-cluster/webserver_cluster_test.tftest.hcl

variables {
  cluster_name  = "test-cluster"
  instance_type = "t2.micro"
  min_size      = 1
  max_size      = 2
  environment   = "dev"
}

run "validate_cluster_name" {
  command = plan

  assert {
    condition     = aws_autoscaling_group.example.name_prefix == "test-cluster-"
    error_message = "ASG name prefix must match the cluster_name variable"
  }
}

run "validate_instance_type" {
  command = plan

  assert {
    condition     = aws_launch_configuration.example.instance_type == "t2.micro"
    error_message = "Instance type must match the instance_type variable"
  }
}

run "validate_security_group_port" {
  command = plan

  assert {
    condition     = aws_security_group.instance.ingress[0].from_port == 8080
    error_message = "Security group must allow traffic on port 8080"
  }
}
```

Run your tests with:

```bash
terraform test
```

The `command = plan` directive runs tests against the plan only — no real infrastructure is created. These are fast, cheap, and can run in seconds.

---

### 4. Integration Tests with Terratest

Integration tests deploy real infrastructure, run assertions against it, and then destroy it. Write your integration test in Go:

```go
// test/webserver_cluster_test.go
package test

import (
  "fmt"
  "testing"
  "time"

  "github.com/gruntwork-io/terratest/modules/http-helper"
  "github.com/gruntwork-io/terratest/modules/random"
  "github.com/gruntwork-io/terratest/modules/terraform"
  "github.com/stretchr/testify/assert"
)

func TestWebserverClusterIntegration(t *testing.T) {
  t.Parallel()

  uniqueID   := random.UniqueId()
  clusterName := fmt.Sprintf("test-cluster-%s", uniqueID)

  terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
    TerraformDir: "../modules/services/webserver-cluster",
    Vars: map[string]interface{}{
      "cluster_name":  clusterName,
      "instance_type": "t2.micro",
      "min_size":      1,
      "max_size":      2,
      "environment":   "dev",
    },
  })

  // Always destroy at the end, even if assertions fail
  defer terraform.Destroy(t, terraformOptions)

  terraform.InitAndApply(t, terraformOptions)

  albDnsName := terraform.Output(t, terraformOptions, "alb_dns_name")
  url        := fmt.Sprintf("http://%s", albDnsName)

  // Retry for up to 5 minutes — ALB takes time to register instances
  http_helper.HttpGetWithRetryWithCustomValidation(
    t,
    url,
    nil,
    30,
    10*time.Second,
    func(status int, body string) bool {
      return status == 200 && len(body) > 0
    },
  )

  // Assert the output is defined and non-empty
  assert.NotEmpty(t, albDnsName, "ALB DNS name should not be empty")
}
```

Install dependencies and run:

```bash
cd test
go mod init test
go get github.com/gruntwork-io/terratest/modules/terraform
go get github.com/gruntwork-io/terratest/modules/http-helper
go test -v -timeout 30m ./...
```

**Warning:** Integration tests deploy real AWS resources and take 5–15 minutes. They will incur costs. Always confirm they clean up after themselves via the `defer terraform.Destroy` call.

---

### 5. End-to-End Tests

End-to-end tests deploy your complete stack — networking, database, application — and verify the system works as a whole. Write an end-to-end test that deploys your VPC module, then your database module, then your webserver module, and asserts the full application path works:

```go
func TestFullStackEndToEnd(t *testing.T) {
  t.Parallel()

  uniqueID := random.UniqueId()

  // Deploy VPC first
  vpcOptions := &terraform.Options{
    TerraformDir: "../modules/networking/vpc",
    Vars: map[string]interface{}{
      "vpc_name": fmt.Sprintf("test-vpc-%s", uniqueID),
    },
  }
  defer terraform.Destroy(t, vpcOptions)
  terraform.InitAndApply(t, vpcOptions)

  vpcID     := terraform.Output(t, vpcOptions, "vpc_id")
  subnetIDs := terraform.OutputList(t, vpcOptions, "private_subnet_ids")

  // Deploy app using VPC outputs
  appOptions := &terraform.Options{
    TerraformDir: "../modules/services/webserver-cluster",
    Vars: map[string]interface{}{
      "cluster_name": fmt.Sprintf("test-app-%s", uniqueID),
      "vpc_id":       vpcID,
      "subnet_ids":   subnetIDs,
      "environment":  "dev",
    },
  }
  defer terraform.Destroy(t, appOptions)
  terraform.InitAndApply(t, appOptions)

  albDnsName := terraform.Output(t, appOptions, "alb_dns_name")
  http_helper.HttpGetWithRetry(t, fmt.Sprintf("http://%s", albDnsName), nil, 200, "Hello", 30, 10*time.Second)
}
```

---

### 6. Build a CI/CD Pipeline

Set up a GitHub Actions workflow that runs your `terraform test` unit tests on every pull request and your integration tests on merges to main:

```yaml
# .github/workflows/terraform-test.yml
name: Terraform Tests

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  unit-tests:
    name: Unit Tests
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: "1.6.0"

      - name: Terraform Init
        run: terraform init
        working-directory: modules/services/webserver-cluster

      - name: Run Unit Tests
        run: terraform test
        working-directory: modules/services/webserver-cluster

  integration-tests:
    name: Integration Tests
    runs-on: ubuntu-latest
    if: github.event_name == 'push'
    needs: unit-tests

    env:
      AWS_ACCESS_KEY_ID:     ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      AWS_DEFAULT_REGION:    us-east-1

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-go@v4
        with:
          go-version: "1.21"

      - name: Run Integration Tests
        run: go test -v -timeout 30m ./...
        working-directory: test
```

Store your AWS credentials as GitHub Actions secrets — never hardcode them in the workflow file.

---

### 7. Write Your Blog Post

**Post title:** *Automating Terraform Testing: From Unit Tests to End-to-End Validation*

Cover all three test layers, the tools for each (`terraform test` for unit, Terratest for integration and E2E), and the CI/CD pipeline that ties them together. Be clear about the tradeoffs — unit tests are fast and free but test less; E2E tests are thorough but slow and costly. The right strategy uses all three.

---

### 8. Post on Social Media

> 🚀 Day 18 of the 30-Day Terraform Challenge — automated testing end to end. Native `terraform test` for unit tests, Terratest for integration tests, GitHub Actions for CI/CD. Infrastructure that is tested automatically on every commit is infrastructure you can deploy with confidence. #30DayTerraformChallenge #TerraformChallenge #Terraform #Testing #DevOps #CI/CD #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste the GitHub URL of your test repository or your blog post URL.
2. **Live App Link field** — paste a link to your GitHub Actions run showing tests passing, or your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**Unit Test File**
Paste your complete `.tftest.hcl` file. Explain what each `run` block tests and why that assertion matters.

```hcl
# paste your terraform test file here
```

**Integration Test**
Paste your Terratest Go function. Explain what `defer terraform.Destroy` guarantees and why it is critical.

```go
// paste your integration test here
```

**Test Execution Results**
Paste the terminal output of `terraform test` and `go test`. Show passing results or document failures and fixes.

**CI/CD Pipeline**
Paste your complete GitHub Actions workflow file. Explain the job dependency between unit tests and integration tests and why integration tests only run on push to main.

**Test Layer Comparison**
Fill in this table from your own experience today:

| Test Type | Tool | Deploys Real Infra | Time | Cost | What It Catches |
|---|---|---|---|---|---|
| Unit | `terraform test` | No | Seconds | Free | |
| Integration | Terratest | Yes | Minutes | Low | |
| End-to-End | Terratest | Yes | 15-30min | Medium | |

**Chapter 9 Learnings**
In your own words — what is the key difference between an integration test and an end-to-end test? Why does the author recommend running unit tests on every PR but E2E tests less frequently?

**Challenges and Fixes**
Go module errors, Terratest timeout issues, and AWS IAM permission failures in GitHub Actions are all common today — document everything.

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terraform Test Documentation](https://developer.hashicorp.com/terraform/language/tests)
- [Terratest Documentation](https://terratest.gruntwork.io/)
- [GitHub Actions — Terraform Setup](https://github.com/hashicorp/setup-terraform)
- [Go Testing Package](https://pkg.go.dev/testing)
- [AWS Credentials in GitHub Actions](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
