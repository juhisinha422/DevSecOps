# 🔐 Checkov Notes

## 🔹 What is Checkov?

Checkov is an open-source Infrastructure as Code (IaC) security scanner that analyzes cloud infrastructure configurations to detect security and compliance issues before deployment.

---

## 🔹 Why Use Checkov?

* Detects security misconfigurations early
* Prevents insecure cloud deployments
* Automates security checks in CI/CD pipelines
* Ensures compliance with best practices

---

## 🔹 Key Features

* Scans multiple IaC frameworks:

  * Terraform
  * CloudFormation
  * Kubernetes
  * Dockerfile
  * ARM Templates
* Detects security and compliance violations
* Provides detailed reports with fixes
* Integrates easily with CI/CD tools
* Supports custom policies

---

## 🔹 How Checkov Works

1. Reads IaC files (e.g., `.tf`, `.yaml`, `Dockerfile`)
2. Applies security policies and rules
3. Identifies misconfigurations
4. Outputs results with severity and remediation steps

---

## 🔹 Installation

### Using pip

```bash
pip install checkov
```

### Using Docker

```bash
docker pull bridgecrew/checkov
```

---

## 🔹 Basic Commands

### Scan current directory

```bash
checkov -d .
```

### Scan specific file

```bash
checkov -f main.tf
```

### Output as JSON

```bash
checkov -d . -o json
```

### Skip a check

```bash
checkov -d . --skip-check CKV_AWS_20
```

---

## 🔹 Example Use Case

A Terraform file defines an S3 bucket without encryption.

Checkov:

* Detects the misconfiguration
* Flags it as a security issue
* Suggests enabling encryption
* Helps fix before deployment

---

## 🔹 Supported Cloud Providers

* AWS
* Azure
* Google Cloud (GCP)

---

## 🔹 Best Practices

* Run Checkov before every deployment
* Integrate with CI/CD pipelines
* Fix high and critical issues first
* Use custom policies for organization standards
* Combine with other DevSecOps tools

---

## 🔹 Integration with CI/CD (Example: GitHub Actions)

```yaml
name: Checkov Scan

on: [push]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Checkov
        run: |
          pip install checkov
          checkov -d .
```

---

## 🔹 Interview Questions

**1. What is Checkov?**
Checkov is an IaC security scanning tool used to detect misconfigurations in infrastructure code.

**2. Which files can Checkov scan?**
Terraform, Kubernetes YAML, Dockerfile, CloudFormation, and more.

**3. How does Checkov help in DevOps?**
It ensures secure infrastructure by catching issues before deployment.

**4. Can Checkov be integrated into CI/CD?**
Yes, it can be integrated into pipelines like GitHub Actions, Jenkins, etc.

---

## 🔹 One-Line Definition

Checkov is an open-source tool that scans Infrastructure as Code to detect and fix security misconfigurations before deployment.
