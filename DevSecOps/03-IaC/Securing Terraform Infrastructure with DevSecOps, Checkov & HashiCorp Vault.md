# 🔐 Securing Terraform Infrastructure with DevSecOps, Checkov & HashiCorp Vault

## 📌 Overview

Terraform is widely used for Infrastructure as Code (IaC), but insecure Terraform code can introduce serious security risks such as:

* Hardcoded AWS credentials
* Public S3 buckets
* Overly permissive IAM policies
* Exposed secrets
* Insecure security groups
* Long-lived CI/CD credentials
* Misconfigured cloud resources

To secure Terraform deployments, I follow a **DevSecOps approach** where security checks are integrated throughout the development and CI/CD lifecycle.

The key security controls covered here are:

1. **GitLeaks** → Detect exposed secrets
2. **Checkov** → Detect Terraform/IaC misconfigurations
3. **HashiCorp Vault** → Manage and generate short-lived credentials
4. **GitHub Actions OIDC** → Authenticate CI/CD securely without storing long-lived credentials
5. **Terraform** → Provision infrastructure through a controlled CI/CD pipeline

---

# 🏗️ High-Level DevSecOps Architecture

```text
                    Developer
                        |
                        v
                +----------------+
                | Git Repository  |
                +----------------+
                        |
                        v
              Pre-Commit Security
                        |
                 +------+------+
                 |             |
              GitLeaks      Terraform
                 |             |
          Secret Scanning   Validation
                 |             |
                 +------+------+
                        |
                        v
                GitHub Pull Request
                        |
                        v
              GitHub Actions CI/CD
                        |
             +----------+----------+
             |                     |
             v                     v
         GitLeaks                Checkov
       Secret Scan            IaC Security
             |                     |
             +----------+----------+
                        |
                        v
                 GitHub OIDC JWT
                        |
                        v
                +---------------+
                | HashiCorp     |
                | Vault         |
                +---------------+
                        |
                        v
             Temporary AWS Credentials
                        |
                        v
                  Terraform
                        |
                        v
                     AWS
```

---

# 1️⃣ Terraform DevSecOps Best Practices

Security should not be added only after infrastructure is deployed.

Security should be implemented throughout the complete development lifecycle.

```text
Developer
   ↓
Code
   ↓
Pre-Commit Security
   ↓
Pull Request
   ↓
CI Security Scanning
   ↓
Terraform Plan
   ↓
Approval
   ↓
Terraform Apply
   ↓
AWS Infrastructure
```

---

# 2️⃣ Never Hardcode Credentials

One of the most important Terraform security rules is:

> **Never hardcode cloud credentials inside Terraform code.**

### ❌ Bad Example

```hcl
provider "aws" {
  region     = "ap-south-1"
  access_key = "AKIAxxxxxxxxxxxx"
  secret_key = "xxxxxxxxxxxxxxxx"
}
```

This is dangerous because credentials can accidentally become visible through:

* Git repositories
* Pull requests
* Git history
* Terraform files
* CI/CD logs
* Developer machines

---

## ✅ Better Approach

Use:

* IAM Roles
* OIDC
* Vault
* Environment variables
* AWS credential providers
* Short-lived credentials

For CI/CD, prefer **OIDC-based authentication** and short-lived credentials instead of permanent access keys.

---

# 3️⃣ `.gitignore` for Terraform Security

Sensitive files should never be committed to Git.

Example `.gitignore`:

```gitignore
# Terraform
.terraform/
*.tfstate
*.tfstate.*
crash.log
crash.*.log
.terraform.lock.hcl

# Sensitive variables
*.tfvars
*.tfvars.json
.env
.env.*

# Credentials
*.pem
*.key
credentials
credentials.*

# Vault
.vault-token

# IDE
.vscode/
.idea/
```

### Important

Terraform state can contain sensitive information depending on the resources and configuration.

Therefore:

```text
terraform.tfstate
terraform.tfstate.backup
```

should generally **not be committed to Git**.

For production, use a secure remote backend with appropriate access controls and encryption.

---

# 4️⃣ Why Terraform State Is Sensitive

Terraform state contains information about infrastructure managed by Terraform.

For example:

```text
terraform.tfstate
       |
       +-- Resource IDs
       +-- Network information
       +-- Configuration data
       +-- Resource attributes
       +-- Potentially sensitive values
```

Therefore production state should be:

* Stored remotely
* Encrypted
* Access-controlled
* Versioned where appropriate
* Protected from unauthorized access

Example AWS backend:

```hcl
terraform {
  backend "s3" {
    bucket = "company-terraform-state"
    key    = "prod/terraform.tfstate"
    region = "ap-south-1"
  }
}
```

A production design should also use appropriate locking/concurrency protection supported by the chosen backend architecture.

---

# 5️⃣ GitLeaks – Secret Detection

## What is GitLeaks?

**GitLeaks** is a secret scanning tool that detects credentials and sensitive information in Git repositories.

It can detect things such as:

* AWS access keys
* API tokens
* Passwords
* Private keys
* Authentication tokens
* Other high-entropy secrets

---

# 6️⃣ Why GitLeaks Is Required

Consider this developer mistake:

```bash
git add .
git commit -m "Add AWS configuration"
git push
```

If the repository contains:

```text
AWS_ACCESS_KEY_ID=AKIAxxxxxxxx
AWS_SECRET_ACCESS_KEY=xxxxxxxx
```

the credentials may enter Git history.

Even if the file is deleted later:

```bash
git rm .env
```

the secret may still exist in previous commits.

Therefore:

> **Removing a secret from the latest commit does not necessarily remove it from Git history.**

If a real credential is exposed, it should be treated as compromised and rotated/revoked.

---

# 7️⃣ Installing GitLeaks

Example installation:

```bash
brew install gitleaks
```

Linux installation can use the appropriate package/binary installation method for the environment.

Verify:

```bash
gitleaks version
```

Scan the repository:

```bash
gitleaks detect --source .
```

---

# 8️⃣ GitLeaks with Pre-Commit

Pre-commit hooks allow security checks to run before code reaches the repository.

Install:

```bash
pip install pre-commit
```

Create:

```text
.pre-commit-config.yaml
```

Example:

```yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.24.2
    hooks:
      - id: gitleaks
```

Install the hook:

```bash
pre-commit install
```

Run manually:

```bash
pre-commit run --all-files
```

Now when a developer commits:

```bash
git add .
git commit -m "Update Terraform"
```

the secret scanning hook executes automatically.

---

# 9️⃣ GitLeaks in GitHub Actions

Local scanning is useful, but it should not be the only security control.

Developers can:

* Skip hooks
* Forget to install hooks
* Use another machine
* Accidentally bypass local checks

Therefore security scanning should also run in CI/CD.

Example workflow:

```yaml
name: Security Scan

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  gitleaks:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Run GitLeaks
        uses: gitleaks/gitleaks-action@v2
```

The exact action version/configuration should be maintained according to the organization's approved version.

---

# 🔟 Checkov – Terraform Security Scanning

## What is Checkov?

**Checkov** is a static analysis tool used to scan Infrastructure as Code for security and compliance issues.

It supports technologies such as:

* Terraform
* Kubernetes
* CloudFormation
* ARM
* Docker
* Serverless
* Other IaC formats

---

# 1️⃣1️⃣ Why Terraform Validate Is Not Enough

Terraform validation primarily checks whether the configuration is structurally valid.

For example:

```bash
terraform validate
```

may confirm that the Terraform configuration is syntactically valid.

But a configuration can be:

```text
Valid Terraform
        +
Valid AWS resource
        +
Insecure configuration
```

Example:

```text
S3 Bucket
   |
   +-- Terraform syntax: VALID
   |
   +-- AWS resource: VALID
   |
   +-- Public access: SECURITY RISK
```

This is where Checkov helps.

---

# 1️⃣2️⃣ Example S3 Security Problem

Suppose Terraform creates an S3 bucket.

```hcl
resource "aws_s3_bucket" "example" {
  bucket = "my-company-bucket"
}
```

The Terraform code may be valid.

But security requirements may require:

```text
Block Public ACLs
Block Public Policy
Ignore Public ACLs
Restrict Public Bucket Access
```

Checkov can identify security controls that are missing or incorrectly configured.

---

# 1️⃣3️⃣ Install Checkov

Using pip:

```bash
pip install checkov
```

Verify:

```bash
checkov --version
```

---

# 1️⃣4️⃣ Run Checkov

Scan the Terraform directory:

```bash
checkov -d .
```

Or:

```bash
checkov -d terraform/
```

Example output conceptually:

```text
Passed checks: 25
Failed checks: 3
Skipped checks: 1
```

A failed check may identify an issue such as:

```text
S3 bucket allows public access
```

The engineer then fixes the Terraform configuration and reruns Checkov.

---

# 1️⃣5️⃣ Terraform Security Workflow with Checkov

```text
Terraform Code
      |
      v
terraform fmt
      |
      v
terraform validate
      |
      v
Checkov
      |
      +---- PASS ----> Continue
      |
      +---- FAIL ----> Fix Security Issue
```

This provides an additional security gate before infrastructure deployment.

---

# 1️⃣6️⃣ Checkov in GitHub Actions

Example:

```yaml
name: Terraform Security

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  checkov:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Run Checkov
        uses: bridgecrewio/checkov-action@master
        with:
          directory: .
```

In an enterprise environment, pinning actions to approved versions or immutable references is preferable to tracking a moving branch.

---

# 1️⃣7️⃣ Checkov Suppression

Sometimes a Checkov finding may be intentionally accepted because of a documented business requirement.

Example:

```hcl
resource "aws_s3_bucket" "example" {
  #checkov:skip=CKV_AWS_18:Access logging is handled by the centralized logging platform

  bucket = "example"
}
```

However, suppression should not simply be used to make the pipeline green.

A proper process should include:

```text
Security Finding
      ↓
Review
      ↓
Business/Technical Justification
      ↓
Documented Exception
      ↓
Approved Suppression
```

---

# 1️⃣8️⃣ HashiCorp Vault

## What is HashiCorp Vault?

**HashiCorp Vault** is a secrets management platform used to securely store, control, and provide access to sensitive credentials.

Vault can manage:

* AWS credentials
* Database credentials
* API tokens
* Passwords
* Certificates
* Encryption keys
* Dynamic credentials

---

# 1️⃣9️⃣ Problem with Static CI/CD Credentials

A traditional CI/CD setup may look like:

```text
GitHub Actions
      |
      v
GitHub Secret
      |
      +-- AWS_ACCESS_KEY_ID
      +-- AWS_SECRET_ACCESS_KEY
      |
      v
AWS
```

The problem is that these credentials can be:

* Long-lived
* Difficult to rotate
* Shared between systems
* Accidentally exposed
* Difficult to attribute to an individual workflow
* High impact if compromised

---

# 2️⃣0️⃣ Vault Dynamic Credentials

Vault can provide temporary AWS credentials.

The conceptual flow is:

```text
GitHub Actions
      |
      | OIDC/JWT
      v
HashiCorp Vault
      |
      | Dynamic Credential Request
      v
AWS
      |
      v
Temporary Credentials
      |
      v
Terraform
```

The credentials have a limited lifetime.

For example:

```text
Credential created
       |
       v
TTL = 15 minutes
       |
       v
Terraform deployment
       |
       v
Credential expires
```

The exact TTL should be based on the organization's security and operational requirements.

---

# 2️⃣1️⃣ Vault + GitHub OIDC

GitHub Actions can issue an OIDC token for the workflow.

Conceptually:

```text
GitHub Actions
      |
      | OIDC JWT
      v
Vault JWT Auth
      |
      | Validate identity
      v
Vault Policy
      |
      | Authorize repository/workflow
      v
AWS Secrets Engine
      |
      v
Temporary AWS Credentials
```

This removes the need to store a long-lived Vault authentication secret in GitHub when configured correctly.

---

# 2️⃣2️⃣ Why OIDC Is Important

Traditional approach:

```text
GitHub
   |
   +-- Static AWS Key
```

OIDC approach:

```text
GitHub
   |
   +-- Short-lived identity token
   |
   v
Trusted Identity Provider
   |
   v
Temporary Cloud Credentials
```

Benefits include:

* No long-lived cloud credentials in GitHub
* Short-lived authentication
* Identity-based access
* Better auditability
* Reduced credential exposure
* Fine-grained trust policies

---

# 2️⃣3️⃣ Vault AWS Secrets Engine

Vault can use the AWS Secrets Engine to generate AWS credentials dynamically.

Conceptually:

```text
Vault
 |
 +-- AWS Secrets Engine
       |
       +-- AWS Role
       |
       +-- IAM Permissions
       |
       +-- TTL
```

For example:

```text
Vault Role
     |
     +-- S3 permissions
     +-- Terraform deployment permissions
     +-- Short TTL
```

The generated credentials should follow the principle of least privilege.

---

# 2️⃣4️⃣ Vault Policies

Vault policies determine what an authenticated identity is allowed to access.

Example conceptual policy:

```hcl
path "aws/creds/terraform-role" {
  capabilities = ["read"]
}
```

This means the authenticated workflow can request credentials from the specified Vault path.

The actual policy should be restricted to only the paths and operations required by the pipeline.

---

# 2️⃣5️⃣ Vault Authentication Flow

```text
GitHub Workflow
      |
      | 1. Request OIDC token
      v
GitHub OIDC Provider
      |
      | 2. JWT
      v
Vault
      |
      | 3. Validate JWT
      |
      | 4. Check repository/branch claims
      |
      v
Vault Policy
      |
      | 5. Authorize
      v
AWS Secrets Engine
      |
      | 6. Generate temporary credentials
      v
GitHub Actions
      |
      | 7. Terraform
      v
AWS
```

---

# 2️⃣6️⃣ Example GitHub Actions Terraform Pipeline

A secure Terraform pipeline can follow this sequence:

```text
Checkout
   ↓
GitLeaks
   ↓
Terraform Format Check
   ↓
Terraform Init
   ↓
Terraform Validate
   ↓
Checkov
   ↓
Vault Authentication
   ↓
Temporary AWS Credentials
   ↓
Terraform Plan
   ↓
Approval
   ↓
Terraform Apply
```

---

# 2️⃣7️⃣ Example GitHub Actions Structure

```yaml
name: Terraform DevSecOps

on:
  pull_request:
  push:
    branches:
      - main

permissions:
  contents: read
  id-token: write

jobs:

  security:
    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: GitLeaks
        uses: gitleaks/gitleaks-action@v2

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Terraform Format
        run: terraform fmt -check -recursive

      - name: Terraform Init
        run: terraform init

      - name: Terraform Validate
        run: terraform validate

      - name: Checkov
        uses: bridgecrewio/checkov-action@master
        with:
          directory: .

  terraform:
    needs: security
    runs-on: ubuntu-latest

    permissions:
      id-token: write
      contents: read

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Authenticate to Vault
        # Vault authentication configuration
        # should be implemented according to the
        # organization's approved Vault setup.
        run: echo "Authenticate using GitHub OIDC"

      - name: Terraform Plan
        run: terraform plan
```

The Vault authentication step above is intentionally conceptual; the exact implementation depends on the Vault deployment, authentication configuration, and approved GitHub/Vault integration.

---

# 2️⃣8️⃣ Production-Grade Security Pipeline

For a production environment, I would design the pipeline like this:

```text
                     Developer
                         |
                         v
                  Git Pull Request
                         |
          +--------------+--------------+
          |                             |
          v                             v
      GitLeaks                        Checkov
    Secret Scan                   IaC Security Scan
          |                             |
          +--------------+--------------+
                         |
                         v
                Terraform Validate
                         |
                         v
                  Security Gate
                         |
                         v
                    Terraform Plan
                         |
                         v
                  Manual Approval
                         |
                         v
                GitHub OIDC Authentication
                         |
                         v
                  HashiCorp Vault
                         |
                         v
              Temporary AWS Credentials
                         |
                         v
                  Terraform Apply
                         |
                         v
                       AWS
```

---

# 2️⃣9️⃣ Least Privilege

The CI/CD pipeline should not have unrestricted AWS access.

Instead of:

```text
AdministratorAccess
```

use only the permissions required by Terraform.

For example:

```text
Terraform Pipeline Role
        |
        +-- S3
        +-- IAM
        +-- VPC
        +-- EKS
        +-- CloudWatch
```

Permissions should be reviewed based on the actual infrastructure being managed.

---

# 3️⃣0️⃣ Secret Lifecycle

A secure secret lifecycle should look like:

```text
Create
  ↓
Store Securely
  ↓
Authenticate
  ↓
Use
  ↓
Rotate / Expire
  ↓
Revoke
```

With dynamic credentials:

```text
Request
   ↓
Generate
   ↓
Use
   ↓
Expire
```

This reduces the lifetime of credentials and therefore reduces the window in which a leaked credential can be abused.

---

# 3️⃣1️⃣ GitLeaks vs Checkov vs Vault

| Tool        | Primary Purpose             | Example                          |
| ----------- | --------------------------- | -------------------------------- |
| GitLeaks    | Secret detection            | Detect AWS keys committed to Git |
| Checkov     | IaC security scanning       | Detect insecure S3 configuration |
| Vault       | Secret management           | Generate temporary credentials   |
| GitHub OIDC | Workload authentication     | Authenticate GitHub workflow     |
| Terraform   | Infrastructure provisioning | Create AWS resources             |

These tools solve different problems and complement each other.

---

# 3️⃣2️⃣ Security Controls

| Security Risk                   | Control                         |
| ------------------------------- | ------------------------------- |
| Hardcoded credentials           | GitLeaks                        |
| Secrets committed to Git        | Pre-commit + CI scanning        |
| Public S3 bucket                | Checkov                         |
| Excessive IAM permissions       | Checkov + IAM review            |
| Long-lived AWS credentials      | Vault/OIDC                      |
| Unauthorized CI/CD access       | OIDC trust policies             |
| Terraform state exposure        | Secure remote backend           |
| Insecure infrastructure changes | Terraform plan + security gates |
| Unreviewed production changes   | Pull request + approval         |
| Credential lifetime             | Dynamic credentials/TTL         |

---

# 3️⃣3️⃣ Real-World Example

Suppose a developer creates:

```hcl
resource "aws_s3_bucket" "data" {
  bucket = "company-production-data"
}
```

The developer also accidentally adds:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

to a configuration file.

The security pipeline works like this:

```text
Developer Push
      |
      v
GitLeaks
      |
      +---- Secret Found
      |
      X
Pipeline Stops
```

If the secret is not detected locally but reaches CI:

```text
GitHub Actions
      |
      v
GitLeaks
      |
      +---- Secret Found
      |
      X
Pipeline Stops
```

If the secret scan passes:

```text
Checkov
   |
   +---- Insecure Terraform Configuration
   |
   X
Pipeline Stops
```

If all security checks pass:

```text
GitHub OIDC
     |
     v
Vault
     |
     v
Temporary AWS Credentials
     |
     v
Terraform Plan
     |
     v
Approval
     |
     v
Terraform Apply
```

This creates multiple security layers instead of relying on a single tool.

---

# 3️⃣4️⃣ Defense-in-Depth Model

Terraform security should follow a defense-in-depth strategy.

```text
Layer 1
Developer Security
       |
       v
Pre-Commit + GitLeaks
       |
       v
Layer 2
Repository Security
       |
       v
Pull Request Review
       |
       v
Layer 3
CI Security
       |
       +-- GitLeaks
       +-- Checkov
       +-- Terraform Validate
       |
       v
Layer 4
Identity Security
       |
       +-- GitHub OIDC
       +-- Vault
       +-- Least Privilege
       |
       v
Layer 5
Infrastructure Security
       |
       +-- AWS IAM
       +-- Network Controls
       +-- Encryption
       +-- Monitoring
```

---

# 3️⃣5️⃣ What Happens If Credentials Are Accidentally Committed?

If an AWS credential is accidentally committed:

### Step 1 – Stop further exposure

Do not continue using the credential.

### Step 2 – Revoke/rotate the credential

Disable or rotate the compromised credential in AWS.

### Step 3 – Investigate

Check:

```text
CloudTrail
Git history
CI/CD logs
Repository access
```

### Step 4 – Remove the secret from the repository/history where appropriate

Use an approved Git history rewriting procedure.

### Step 5 – Run secret scanning again

```bash
gitleaks detect --source .
```

### Step 6 – Identify the root cause

For example:

```text
Why was the secret committed?
Why did local scanning not detect it?
Why did CI allow it?
```

### Step 7 – Improve controls

Implement:

```text
Pre-commit
+
CI GitLeaks
+
OIDC
+
Vault
+
Least Privilege
```

---

# 3️⃣6️⃣ Terraform Security Checklist

## Before Commit

```text
[ ] No hardcoded credentials
[ ] .gitignore configured
[ ] GitLeaks scan passed
[ ] Terraform formatted
[ ] Sensitive variables protected
```

## Pull Request

```text
[ ] GitLeaks passed
[ ] Checkov passed
[ ] Terraform validate passed
[ ] Terraform plan reviewed
[ ] Security findings addressed
```

## Before Production

```text
[ ] Least privilege verified
[ ] Remote Terraform state secured
[ ] Encryption enabled
[ ] CI/CD identity secured
[ ] OIDC configured
[ ] Vault policies reviewed
[ ] Temporary credentials configured
[ ] Approval completed
```

---

# 3️⃣7️⃣ Key Interview Questions

## Q1. Why do you use GitLeaks with Terraform?

GitLeaks scans the repository for exposed credentials such as AWS access keys, passwords, API tokens, and private keys. I use it both as a pre-commit control and as a CI/CD security gate. This provides protection even if a developer bypasses or misses the local pre-commit hook.

---

## Q2. What is the difference between Terraform Validate and Checkov?

`terraform validate` primarily validates the Terraform configuration and its structure.

Checkov performs security and compliance analysis against the Infrastructure as Code.

For example, Terraform may accept an S3 bucket configuration as valid while Checkov can identify security risks such as inappropriate public access configuration.

So I use both:

```text
terraform validate → Configuration validation

Checkov → Security/compliance validation
```

---

## Q3. Why should Terraform state not be committed to Git?

Terraform state can contain infrastructure details and potentially sensitive resource attributes. Committing state to Git increases the risk of unauthorized exposure and creates state-management problems for teams.

For production, I use a secure remote backend with appropriate encryption, access control, and concurrency protection.

---

## Q4. Why is Vault better than storing AWS credentials in GitHub Secrets?

The key advantage is reducing dependence on long-lived static credentials.

With Vault and OIDC:

```text
GitHub
  ↓
OIDC Identity
  ↓
Vault
  ↓
Temporary AWS Credentials
  ↓
Terraform
```

The credentials can have a short TTL and are automatically invalidated when they expire.

---

## Q5. What is the role of OIDC?

OIDC allows GitHub Actions to prove its workload identity to a trusted identity provider without requiring a long-lived secret to authenticate the workflow.

In a Vault-based setup, Vault validates the GitHub OIDC/JWT claims and applies the appropriate policy before issuing credentials.

---

## Q6. How does Vault provide dynamic AWS credentials?

Vault's AWS Secrets Engine can generate temporary AWS credentials based on a configured Vault role and AWS permissions.

The general process is:

```text
GitHub OIDC
     ↓
Vault Authentication
     ↓
Vault Policy
     ↓
AWS Secrets Engine
     ↓
Temporary Credentials
```

The credentials have a defined lifetime and expire after the TTL.

---

## Q7. What happens when Checkov fails?

I don't simply ignore the failure.

I first identify:

```text
Check ID
Resource
Security Risk
Recommended Remediation
```

Then I fix the Terraform configuration and rerun Checkov.

If the finding is intentionally acceptable, I use a documented and approved exception process rather than blindly suppressing the finding.

---

## Q8. What would you do if GitLeaks detects an AWS key?

I would treat the key as compromised.

My process would be:

```text
Stop using credential
       ↓
Rotate/Revoke credential
       ↓
Investigate usage
       ↓
Remove secret from repository/history where required
       ↓
Run GitLeaks
       ↓
Identify root cause
       ↓
Strengthen CI/CD controls
```

---

# 3️⃣8️⃣ Interview Scenario

### Interviewer:

**"You have Terraform running through GitHub Actions. How would you secure the pipeline?"**

### Answer:

I would implement security at multiple layers.

First, I would make sure credentials are never hardcoded in Terraform or committed to Git. I would use `.gitignore` for sensitive files and configure GitLeaks as both a pre-commit hook and a CI security check.

For Terraform security, I would run `terraform fmt`, `terraform validate`, and Checkov to detect IaC security misconfigurations such as public S3 access, insecure security groups, or excessive permissions.

For authentication, instead of storing long-lived AWS access keys in GitHub, I would use GitHub OIDC with a trusted authentication flow. In a Vault-based architecture, GitHub would authenticate to Vault using its OIDC/JWT identity, Vault would validate the claims and policies, and the AWS Secrets Engine would provide short-lived credentials with least-privilege permissions.

The pipeline would then run Terraform plan and, after the required approval, Terraform apply.

The overall flow would be:

```text
Git
 ↓
GitLeaks
 ↓
Terraform Validate
 ↓
Checkov
 ↓
GitHub OIDC
 ↓
Vault
 ↓
Temporary AWS Credentials
 ↓
Terraform Plan
 ↓
Approval
 ↓
Terraform Apply
```

This gives us secret scanning, IaC security scanning, identity-based authentication, short-lived credentials, least privilege, and an auditable deployment process.

---

# 3️⃣9️⃣ Final DevSecOps Model

```text
                         TERRAFORM
                            |
          +-----------------+-----------------+
          |                 |                 |
          v                 v                 v
       GitLeaks          Checkov          Terraform
     Secret Scan        IaC Security      Validation
          |                 |                 |
          +-----------------+-----------------+
                            |
                            v
                    GitHub Actions
                            |
                            v
                       GitHub OIDC
                            |
                            v
                     HashiCorp Vault
                            |
                            v
                  Temporary AWS Credentials
                            |
                            v
                     Terraform Plan
                            |
                            v
                       Approval
                            |
                            v
                    Terraform Apply
                            |
                            v
                           AWS
```

# 🎯 Key Takeaways

### GitLeaks

> Detect secrets before they reach or remain in the repository.

### Checkov

> Detect security and compliance issues in Terraform/IaC before deployment.

### HashiCorp Vault

> Securely manage secrets and, where configured, generate short-lived dynamic credentials.

### GitHub OIDC

> Provide workload identity without relying on long-lived static cloud credentials.

### Terraform

> Provision infrastructure through a controlled, reviewed, and security-gated process.

### DevSecOps

> Shift security left and make security part of the entire Infrastructure as Code lifecycle.

---

# 🔥 One-Line Interview Summary

> **"For Terraform security, I use GitLeaks for secret detection, Checkov for IaC security scanning, Terraform validation for configuration correctness, and GitHub OIDC with HashiCorp Vault for secure, short-lived credentials, combined with least privilege, remote state protection, PR reviews, and CI/CD security gates."**
