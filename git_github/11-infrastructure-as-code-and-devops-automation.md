# Infrastructure as Code (IaC) and DevOps Automation

**Created Date:** 2026-09-21
**Last Updated:** 2026-09-21
**Tags:** `#devops`, `#iac`, `#terraform`, `#cloud`, `#automation`
**References:** [Infrastructure as Code Patterns - Kief Morris]
**Status:** Published
**Difficulty:** Advanced

---

## 📌 Key Takeaways (TL;DR)
- Infrastructure as Code (IaC) replaces manual server provisioning with declarative configuration files, enabling version-controlled, repeatable, and automated cloud deployments.
- Declarative IaC (e.g., Terraform) specifies *what* the desired state should be, whereas imperative scripting specifies *how* to achieve it step by step.
- Integrating IaC with CI/CD pipelines ensures that infrastructure changes undergo code review, automated testing, and safe rolling updates.

## 🧠 Detailed Notes

### 1. Core Concepts
- **Declarative vs. Imperative:** Declarative tools (Terraform, Kubernetes YAML) define target state; imperative scripts (Bash scripts, AWS CLI commands) define procedural execution steps.
- **State Management:** Tracking the current state of infrastructure resources against configuration files (e.g., Terraform `.tfstate` stored securely in remote object storage like AWS S3 with DynamoDB locking).
- **Idempotency:** The property where running the same provisioning script multiple times yields the exact same system state without unintended modifications or duplicates.

### 2. How It Works (Mechanism)
- **Workflow:** Write `.tf` configuration files $\rightarrow$ `terraform init` (download providers) $\rightarrow$ `terraform plan` (dry run / diff calculation) $\rightarrow$ `terraform apply` (provision cloud resources).
- **Architecture Example (Terraform AWS VPC Provisioning):**

```hcl
# Example Terraform configuration for an AWS VPC and EC2 instance
provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main_vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  tags = {
    Name = "ai-research-vpc"
  }
}

resource "aws_subnet" "public_subnet" {
  vpc_id            = aws_vpc.main_vpc.id
  cidr_block        = "10.0.1.0/24"
  map_public_ip_on_launch = true
  tags = {
    Name = "ai-research-public-subnet"
  }
}
```

### 3. Practical Application & Trade-offs
- **Real-world Use Case:** Spinning up GPU instances (AWS EC2 `g4dn.xlarge`) and Kubernetes clusters on-demand for training and serving AI models.
- **Trade-offs:** 
  - *State File Locking & Corruption:* Mismanaged remote state files can lead to orphaned infrastructure or split-brain deployment issues.
  - *Learning Curve:* Requires rigorous understanding of cloud provider APIs and networking fundamentals.

### 4. Comparison Table
| Feature | Manual Provisioning (Console / CLI) | Infrastructure as Code (Terraform) |
|---|---|---|
| **Reproducibility** | Low (prone to human error / "Configuration Drift") | High (fully defined in version-controlled code) |
| **Auditability** | Difficult to track who changed what | Complete Git history (`git blame`) |
| **Speed** | Slow and tedious for large clusters | Fast, parallelized provisioning |

## 📝 Research Journey
- **Why:** Understanding how cloud infrastructure is programmatically provisioned is critical for deploying high-throughput AI systems.
- **Struggle:** Understanding how remote state locking prevents concurrent modifications by multiple developers.
- **Aha moment:** Realizing that infrastructure is just software code that happens to manage physical/virtual hardware states.
- **Career link:** Bridges the gap between software development, MLOps, and cloud architecture for an AI Engineer.

## 🔗 Related Topics
- [[git_github/10-workflows-cookbook.md]] — Advanced Git workflows and automation
- [[ai_ml/05-mlops-lifecycle-and-deployment-architecture.md]] — MLOps infrastructure deployment
- [SUGGESTED] kubernetes-container-orchestration — deploying scalable microservices and GPU pods.

---
*Feynman Technique: IaC is writing code to build your own servers and networks, so you can delete and recreate your entire data center with a single command.*
