# Day 15: Working with Multiple Providers — Part 2

## What You Will Accomplish Today

Today you complete Chapter 7 and tackle the most advanced provider scenarios: writing modules that accept provider configurations from their callers, deploying across multiple cloud providers in a single configuration, and using Terraform to manage Docker containers and a Kubernetes cluster on AWS EKS. By the end of today you will have deployed a containerised application on Kubernetes using nothing but Terraform code.

---

## Tasks

### 1. Read

**Book:** *Terraform: Up & Running* by Yevgeniy Brikman — **Chapter 7** (complete the chapter)

Read all remaining sections:
- *Working with Multiple AWS Regions*
- *Working with Multiple AWS Accounts*
- *Creating Modules That Can Work with Multiple Providers*
- *Working with Multiple Different Providers*
- *A Crash Course on Docker*
- *A Crash Course on Kubernetes*
- *Deploying Docker Containers in AWS Using Elastic Kubernetes Service*
- *Conclusion*

The section on passing providers into modules is critical — read it twice if needed.

---

### 2. Complete the Hands-On Labs

- **Lab 1:** [Terraform CI/CD Integration](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2005%20-%20Use%20Terraform%20Outside%20of%20Core%20Workflow/03%20-%20Terraform_CICD.md)
- **Lab 2:** [Remote State](https://github.com/btkrausen/hashicorp/blob/master/terraform/Hands-On%20Labs/Section%2003%20-%20Understand%20The%20Purpose%20of%20Terraform/02%20-%20Benefits_of_State.md)

---

### 3. Create Modules That Work with Multiple Providers

When a module needs to deploy resources in multiple regions or accounts, it must accept provider configurations from the calling root module — it cannot define its own providers internally. This is the correct pattern:

```hcl
# modules/multi-region-app/main.tf
# No provider blocks inside the module — providers are passed in by the caller

terraform {
  required_providers {
    aws = {
      source                = "hashicorp/aws"
      version               = "~> 5.0"
      configuration_aliases = [aws.primary, aws.replica]
    }
  }
}

resource "aws_s3_bucket" "primary" {
  provider = aws.primary
  bucket   = "${var.app_name}-primary"
}

resource "aws_s3_bucket" "replica" {
  provider = aws.replica
  bucket   = "${var.app_name}-replica"
}
```

```hcl
# live/main.tf — the calling root configuration
provider "aws" {
  alias  = "primary"
  region = "us-east-1"
}

provider "aws" {
  alias  = "replica"
  region = "us-west-2"
}

module "multi_region_app" {
  source   = "../../modules/multi-region-app"
  app_name = "my-app"

  providers = {
    aws.primary = aws.primary
    aws.replica = aws.replica
  }
}
```

The `configuration_aliases` declaration in the module's `required_providers` block tells Terraform which provider aliases the module expects to receive. The `providers` map in the module call wires the root module's providers to the module's expected aliases.

Build and deploy this pattern. Confirm resources appear in both regions.

---

### 4. Deploy Docker Containers with Terraform

Use the Docker provider to manage containers locally — this is an excellent way to test containerised application configurations before deploying to a cloud platform:

```hcl
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0"
    }
  }
}

provider "docker" {}

resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = false
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "terraform-nginx"

  ports {
    internal = 80
    external = 8080
  }
}
```

Run `terraform apply` and confirm nginx is serving on `http://localhost:8080`. This requires Docker to be running locally. Then run `terraform destroy` to remove the container.

---

### 5. Deploy an EKS Cluster with Terraform

This is the most complex deployment of the challenge so far. Deploy a production-ready EKS cluster using the official AWS EKS Terraform module:

```hcl
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 20.0"

  cluster_name    = "terraform-challenge-cluster"
  cluster_version = "1.29"

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  eks_managed_node_groups = {
    default = {
      min_size       = 1
      max_size       = 3
      desired_size   = 2
      instance_types = ["t3.small"]
    }
  }

  tags = {
    Environment = "dev"
    Challenge   = "30DayTerraform"
  }
}
```

After the cluster is running, configure the Kubernetes provider to deploy a workload onto it:

```hcl
provider "kubernetes" {
  host                   = module.eks.cluster_endpoint
  cluster_ca_certificate = base64decode(module.eks.cluster_certificate_authority_data)

  exec {
    api_version = "client.authentication.k8s.io/v1beta1"
    command     = "aws"
    args        = ["eks", "get-token", "--cluster-name", module.eks.cluster_name]
  }
}

resource "kubernetes_deployment" "nginx" {
  metadata {
    name = "nginx-deployment"
    labels = {
      app = "nginx"
    }
  }

  spec {
    replicas = 2

    selector {
      match_labels = {
        app = "nginx"
      }
    }

    template {
      metadata {
        labels = {
          app = "nginx"
        }
      }

      spec {
        container {
          image = "nginx:latest"
          name  = "nginx"

          port {
            container_port = 80
          }
        }
      }
    }
  }
}
```

**Important:** EKS clusters take 10–15 minutes to provision and will incur AWS charges. Destroy the cluster as soon as you have confirmed it works.

---

### 6. Write Your Blog Post

**Post title:** *Deploying Multi-Cloud Infrastructure with Terraform Modules*

Cover the provider alias pattern for modules, the `configuration_aliases` declaration, how to wire providers into modules with the `providers` map, and your EKS + Kubernetes deployment. The Docker provider section makes a good quick-start example before the more complex EKS content.

---

### 7. Post on Social Media

> 🌐 Day 15 of the 30-Day Terraform Challenge — multi-cloud modules, Docker containers, and a full EKS cluster all managed by Terraform. Two providers in one configuration, containers running on Kubernetes, zero manual console clicks. #30DayTerraformChallenge #TerraformChallenge #Terraform #EKS #Kubernetes #Docker #IaC #AWSUserGroupKenya #EveOps

---

## How to Submit

Use the **Workspace** tab to submit your work for today.

1. **Repository Link field** — paste your blog post URL or a link to your EKS or multi-provider module code.
2. **Live App Link field** — paste your social media post URL.
3. **Documentation editor** — write your full learning journal entry below.

### Your documentation must include:

**Multi-Provider Module Pattern**
Paste your module's `required_providers` block with `configuration_aliases` and the calling root configuration's `providers` map. Explain why modules cannot define their own providers and must receive them from the caller.

```hcl
# paste your module and calling configuration here
```

**Docker Deployment**
Paste your Docker provider configuration and container resource. Confirm the container was running — paste the output of `docker ps` or `terraform output`.

**EKS Cluster Configuration**
Paste your EKS module call and Kubernetes provider configuration. Explain how the Kubernetes provider authenticates to the cluster using the `exec` block.

```hcl
# paste your EKS and Kubernetes configuration here
```

**Kubernetes Deployment Confirmation**
Paste the output of `kubectl get pods` or `terraform show` confirming the nginx deployment is running on the cluster.

**Chapter 7 Learnings**
In your own words:
- Why can modules not contain their own provider blocks when the provider needs to be aliased?
- What does `configuration_aliases` do and why is it required?
- How does Terraform know which provider to use for Kubernetes resources after the EKS cluster is provisioned?

**EKS Cost Awareness**
What AWS resources does an EKS cluster create and approximately how much would it cost to leave it running for 24 hours? Why is `terraform destroy` critical after this exercise?

**Challenges and Fixes**
EKS provisioning errors, Kubernetes provider authentication issues, and Docker provider installation problems are all common today — document everything you hit.

**Blog Post**
Paste the URL and a short summary.

**Social Media**
Paste the URL to your post.

---

## Additional Resources

- [Terraform Docker Provider](https://registry.terraform.io/providers/kreuzwerker/docker/latest/docs)
- [Terraform Kubernetes Provider](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs)
- [AWS EKS Terraform Module](https://registry.terraform.io/modules/terraform-aws-modules/eks/aws/latest)
- [Module Provider Configuration](https://developer.hashicorp.com/terraform/language/modules/develop/providers)
- [AWS EKS Getting Started](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)

---

*This challenge is brought to you by **AWS AI/ML UserGroup Kenya**, **Meru HashiCorp User Group**, and **EveOps**.*
