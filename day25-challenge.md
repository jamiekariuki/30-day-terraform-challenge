# Day 25: Deploy a Static Website on AWS S3 with Terraform

## What You Will Accomplish Today

The exam prep continues in the background — but today you build something real and complete. Deploying a static website on AWS using S3 and CloudFront is one of the most practical, portfolio-worthy projects in cloud infrastructure. More importantly, today's project requires you to apply every best practice from the last 24 days simultaneously: modular code, remote state, DRY configuration, environment isolation, version control, and proper tagging. By the end of today you will have a live, globally distributed website deployed entirely through Terraform.

---

## Tasks

### 1. Project Structure

Set up your project following this exact directory structure before writing a single resource block:

```
day25-static-website/
├── modules/
│   └── s3-static-website/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
├── envs/
│   └── dev/
│       ├── main.tf
│       ├── variables.tf
│       ├── outputs.tf
│       └── terraform.tfvars
├── backend.tf
└── provider.tf
```

The `modules/s3-static-website` directory contains your reusable module. The `envs/dev` directory is your calling configuration. Nothing is hardcoded — everything configurable flows through variables.

---

### 2. Build the S3 Static Website Module

Create `modules/s3-static-website/variables.tf`:

```hcl
variable "bucket_name" {
  description = "Globally unique name for the S3 bucket"
  type        = string
}

variable "environment" {
  description = "Deployment environment"
  type        = string

  validation {
    condition     = contains(["dev", "staging", "production"], var.environment)
    error_message = "Environment must be dev, staging, or production."
  }
}

variable "tags" {
  description = "Additional tags to apply to all resources"
  type        = map(string)
  default     = {}
}

variable "index_document" {
  description = "The index document for the website"
  type        = string
  default     = "index.html"
}

variable "error_document" {
  description = "The error document for the website"
  type        = string
  default     = "error.html"
}
```

Create `modules/s3-static-website/main.tf`:

```hcl
locals {
  common_tags = merge(var.tags, {
    Environment = var.environment
    ManagedBy   = "terraform"
    Project     = "static-website"
  })
}

resource "aws_s3_bucket" "website" {
  bucket = var.bucket_name
  tags   = local.common_tags
}

resource "aws_s3_bucket_website_configuration" "website" {
  bucket = aws_s3_bucket.website.id

  index_document {
    suffix = var.index_document
  }

  error_document {
    key = var.error_document
  }
}

resource "aws_s3_bucket_public_access_block" "website" {
  bucket = aws_s3_bucket.website.id

  block_public_acls       = false
  block_public_policy     = false
  ignore_public_acls      = false
  restrict_public_buckets = false
}

resource "aws_s3_bucket_policy" "website" {
  bucket = aws_s3_bucket.website.id
  policy = data.aws_iam_policy_document.website.json

  depends_on = [aws_s3_bucket_public_access_block.website]
}

data "aws_iam_policy_document" "website" {
  statement {
    principals {
      type        = "*"
      identifiers = ["*"]
    }
    actions   = ["s3:GetObject"]
    resources = ["${aws_s3_bucket.website.arn}/*"]
  }
}

resource "aws_cloudfront_distribution" "website" {
  enabled             = true
  default_root_object = var.index_document
  price_class         = "PriceClass_100"
  tags                = local.common_tags

  origin {
    domain_name = aws_s3_bucket_website_configuration.website.website_endpoint
    origin_id   = "s3-website"

    custom_origin_config {
      http_port              = 80
      https_port             = 443
      origin_protocol_policy = "http-only"
      origin_ssl_protocols   = ["TLSv1.2"]
    }
  }

  default_cache_behavior {
    allowed_methods        = ["GET", "HEAD"]
    cached_methods         = ["GET", "HEAD"]
    target_origin_id       = "s3-website"
    viewer_protocol_policy = "redirect-to-https"

    forwarded_values {
      query_string = false
      cookies {
        forward = "none"
      }
    }

    min_ttl     = 0
    default_ttl = 3600
    max_ttl     = 86400
  }

  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }

  viewer_certificate {
    cloudfront_default_certificate = true
  }
}

resource "aws_s3_object" "index" {
  bucket       = aws_s3_bucket.website.id
  key          = "index.html"
  content_type = "text/html"

  content = <<-HTML
    <!DOCTYPE html>
    <html>
    <head><title>Terraform Static Website</title></head>
    <body>
      <h1>Deployed with Terraform</h1>
      <p>Environment: ${var.environment}</p>
      <p>Bucket: ${var.bucket_name}</p>
    </body>
    </html>
  HTML
}

resource "aws_s3_object" "error" {
  bucket       = aws_s3_bucket.website.id
  key          = "error.html"
  content_type = "text/html"

  content = <<-HTML
    <!DOCTYPE html>
    <html>
    <head><title>404 - Not Found</title></head>
    <body>
      <h1>404 - Page Not Found</h1>
    </body>
    </html>
  HTML
}
```

Create `modules/s3-static-website/outputs.tf`:

```hcl
output "bucket_name" {
  value       = aws_s3_bucket.website.id
  description = "Name of the S3 bucket"
}

output "website_endpoint" {
  value       = aws_s3_bucket_website_configuration.website.website_endpoint
  description = "S3 website endpoint"
}

output "cloudfront_domain_name" {
  value       = aws_cloudfront_distribution.website.domain_name
  description = "CloudFront distribution domain name — use this URL to access your website"
}

output "cloudfront_distribution_id" {
  value       = aws_cloudfront_distribution.website.id
  description = "CloudFront distribution ID"
}
```

---

### 3. Configure the Dev Environment

Create `envs/dev/main.tf`:

```hcl
module "static_website" {
  source = "../../modules/s3-static-website"

  bucket_name    = var.bucket_name
  environment    = var.environment
  index_document = var.index_document
  error_document = var.error_document

  tags = {
    Owner   = "terraform-challenge"
    Day     = "25"
  }
}
```

Create `envs/dev/terraform.tfvars`:

```hcl
bucket_name    = "my-terraform-challenge-website-dev-unique123"
environment    = "dev"
index_document = "index.html"
error_document = "error.html"
```

Create `backend.tf` in the root:

```hcl
terraform {
  backend "s3" {
    bucket         = "your-terraform-state-bucket"
    key            = "day25/static-website/dev/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-locks"
    encrypt        = true
  }
}
```

---

### 4. Deploy and Verify

```bash
cd envs/dev
terraform init
terraform validate
terraform plan
terraform apply
```

After apply completes, retrieve your CloudFront URL:

```bash
terraform output cloudfront_domain_name
```

Open it in your browser. You should see your "Deployed with Terraform" page served over HTTPS. CloudFront distributions take 5–15 minutes to fully propagate globally after creation.

---

### 5. Bonus — Route53 Custom Domain

If you have a custom domain registered in Route53, point it to your CloudFront distribution:

```hcl
data "aws_route53_zone" "primary" {
  name = var.domain_name
}

resource "aws_route53_record" "website" {
  zone_id = data.aws_route53_zone.primary.zone_id
  name    = var.domain_name
  type    = "A"

  alias {
    name                   = aws_cloudfront_distribution.website.domain_name
    zone_id                = aws_cloudfront_distribution.website.hosted_zone_id
    evaluate_target_health = false
  }
}
```

You will also need to add an ACM certificate and update the CloudFront viewer certificate configuration. Document the full setup in your submission.

---

### 6. Clean Up

CloudFront distributions accumulate charges even when serving no traffic. Destroy the distribution after verifying it works:

```bash
terraform destroy
```

S3 buckets with objects cannot be destroyed by Terraform by default. Enable force destroy for dev environments:

```hcl
resource "aws_s3_bucket" "website" {
  bucket        = var.bucket_name
  force_destroy = var.environment != "production"
}
```

---

### 7. Write Your Blog Post

**Post title:** *Deploying a Static Website on AWS S3 with Terraform: A Beginner's Guide*

Walk through the full project structure, the module design decisions, the S3 and CloudFront configuration, and the deployment process. Show the live website URL. Explain why you used a module rather than putting everything in one file, how remote state protects your infrastructure, and what the DRY principle looks like in practice across the `envs/dev` and the module itself.

---

### 8. Post on Social Media

> 🚀 Day 25 of the 30-Day Terraform Challenge — deployed a fully modular, globally distributed static website on AWS S3 + CloudFront using Terraform. Remote state, DRY modules, environment isolation, consistent tagging. Everything from the last 24 days in one project. #30DayTerraformChallenge #TerraformChallenge #Terraform #AWS #CloudFront #IaC #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL or GitHub repository URL for the project.
2. **Live App Link field** — paste your CloudFront domain name URL showing the live website.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**Project Directory Tree**
Paste the output of `tree day25-static-website/` showing your full project structure.

**Module Code**
Paste your complete module — `variables.tf`, `main.tf`, and `outputs.tf`. Explain every variable and why it has (or does not have) a default value.

```hcl
# paste your module code here
```

**Calling Configuration**
Paste your `envs/dev/main.tf` and `terraform.tfvars`. Explain how the calling configuration stays clean because all the complexity lives in the module.

**Deployment Output**
Paste the output of `terraform apply` and `terraform output`. Confirm the CloudFront URL is present.

**Live Website Confirmation**
Paste the CloudFront URL and confirm you accessed it successfully in a browser. Describe what you saw.

**DRY Principle in Practice**
How did using a module enforce the DRY principle in this project? What would the configuration look like if you had not used a module and instead written everything in a single flat file?

**Bonus — Route53**
If you completed the Route53 bonus, paste your Route53 and ACM certificate configuration and show the custom domain resolving to your CloudFront distribution.

**Cleanup Confirmation**
Paste the output of `terraform destroy` confirming all resources were removed.

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [AWS S3 Static Website Hosting](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)
- [AWS CloudFront with S3](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/DownloadDistS3AndCustomOrigins.html)
- [Terraform AWS S3 Website Configuration](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_website_configuration)
- [Terraform AWS CloudFront Distribution](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/cloudfront_distribution)
- [ACM Certificate for CloudFront](https://docs.aws.amazon.com/acm/latest/userguide/acm-regions.html)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
