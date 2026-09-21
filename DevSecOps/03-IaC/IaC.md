# Securing Terraform Infrastructure with DevSecOps Best Practices, Checkov, and HashiCorp Vault

This demonstration discusses **how to secure Terraform infrastructure code** using practical DevSecOps best practices, automated security scanning with Checkov, and secret management leveraging HashiCorp Vault integrated with GitHub Actions CI/CD. The focus is on preventing misconfigurations, protecting sensitive credentials, and enabling secure, auditable Terraform deployments in a production environment.

## 1. DevSecOps Best Practices for Terraform Security

Security best practices form the foundation for maintaining secure Terraform workflows, especially when collaborating on infrastructure-as-code repositories.

- **Never hardcode credentials:** Avoid embedding AWS or cloud provider credentials in Terraform files or environment variables tracked in Git.
- **Use `.gitignore` files:** 
  - Add entries for sensitive files like `.env`, Terraform state files (`terraform.tfstate`), private keys/pem files.
  - Commit `.gitignore` to the repository so all team members can avoid accidental commit of secrets.
- **Implement pre-commit hooks with GitLeaks:** 
  - Use the **pre-commit framework** to automate secret scanning locally before commits.
  - GitLeaks scans staged files and blocks commits containing passwords, API keys, or tokens.
  - Installation example: `brew install pre-commit` (Mac), `pip install pre-commit` (Linux).
  - Commit a `.pre-commit-config.yaml` specifying GitLeaks and run `pre-commit install`.
- **Enforce secret scanning in CI/CD pipelines:**
  - Even if local hooks are missed, configure GitHub Actions workflows to run GitLeaks on commits and pull requests.
  - This blocks push or merge operations containing secrets visible to the public or unauthorized users.
- **Rationale:** 
  - Git history preserves committed secrets permanently.
  - Accidental exposure risks organization-wide compromise.
  - Automated scanning and pipeline enforcement reduce human error and increase security posture.

## 2. Detecting Insecure Terraform Configurations Using Checkov

Checkov is a static code analysis tool used to detect **Terraform misconfigurations and security issues** before deployment.

- **Problem:** Terraform native validation or syntax checks do not catch security misconfigurations, e.g., creating a publicly accessible S3 bucket.
- **Scenario:** 
  - A junior engineer writes Terraform code to create an S3 bucket.
  - The bucket is inadvertently made **public** due to disabled blocking of public ACLs/policies.
  - Such misconfigurations may go unnoticed in code reviews or pre-commit checks.
- **Using Checkov:** 
  - Run `checkov -d <directory>` pointing to the Terraform code directory.
  - Checkov automatically executes relevant policies depending on resource types (e.g., S3, EC2).
  - It provides a detailed report listing:
    - Passed security checks.
    - Failed security checks specifying risks such as "S3 bucket has public ACLs enabled".
  - Enables remedial action by updating Terraform code to enforce secure settings (block public ACLs/policies).
- **Integration:** 
  - Run Checkov locally before commit.
  - Integrate into CI/CD pipelines to enforce secure Terraform configurations organization-wide.
- **Benefit:** 
  - Automates security testing as internal “test cases” for infrastructure code.
  - Reduces manual review overhead.
  - Helps establish Terraform security best practices for teams.

## 3. HashiCorp Vault: Secure Secrets Management for Terraform CI/CD

HashiCorp Vault addresses **the inherent risks in managing long-lived static credentials** commonly used in CI/CD workflows, especially in organizations.

### 3.1 Limitations of Traditional Credential Management

- In hobby/local projects, developers store AWS credentials in `.env` and execute Terraform locally.
- In production, Terraform should be executed only via CI/CD to enforce:
  - **Accountability:** Changes go through version control and approved pipelines.
  - **Auditing:** Track which engineer committed what and when.
  - **Revertibility:** Easy rollbacks in case of infrastructure failures.

- Problem: Storing AWS service account credentials (bot accounts) as static secrets inside GitHub repository secrets exposes risks:
  - Potential credential leakage among dozens/hundreds of users with repository access.
  - Lack of accountability: Hard to trace which person leaked shared credentials.
  - Long-lived credentials, often not rotated frequently, increase exposure window.

### 3.2 Vault as a Solution for Short-lived Dynamic Credentials

- Vault generates **temporary, short-lived AWS credentials** for CI/CD workflows.
- Workflow:
  1. Vault is preconfigured with AWS service account credentials and policies.
  2. GitHub Actions authenticates to Vault using **OIDC-based JWT tokens**.
  3. Vault dynamically generates credentials valid only for a short time (e.g., 10-15 minutes).
  4. GitHub Actions deploys infrastructure using these ephemeral credentials.
  5. Credentials expire immediately after CI completes, minimizing leak impact.

- Vault avoids storing static secrets in the pipeline.
- Provides fine-grained control and automatic credential rotation.
- Maximizes security and traceability in shared environments.

### 3.3 Vault Installation and Setup Demo

- Deploy Vault server on an Ubuntu EC2 instance (free tier recommended).
- Configure inbound security groups to allow Vault port (port 8200).
- Install Vault binary and run Vault in development mode on the instance.
- Authenticate to Vault UI or CLI using a root token.
- Enable AWS secret engine with AWS access keys.
- Generate AWS IAM user credentials with specific permission (e.g., S3 full access).
- Enable JWT authentication with OIDC discovery URL for GitHub Actions.
- Define Vault policies limiting access to specific secrets.
- Bind Vault policies to GitHub repository client identity.

### 3.4 GitHub Actions Integration

- Terraform configuration stored in Git repository.
- GitHub Actions workflow triggers on commit/push events.
- Workflow obtains temporary AWS credentials by querying Vault using the repository's OIDC JWT.
- Credentials are used to initialize Terraform, plan, and apply resources on AWS.
- Temporary AWS users are created dynamically by Vault with a fixed, short TTL.
- After workflow finishes, temporary credentials expire.
- The process enforces secure credential usage and disaster risk mitigation.

| Component              | Role/Function                                             | Notes                              |
|------------------------|-----------------------------------------------------------|-----------------------------------|
| Vault Server           | Hosts secrets backend and secret engine                   | Installed on EC2                   |
| AWS Secret Engine      | Generates AWS IAM credentials                              | Configured with service account   |
| JWT Auth Method        | Enables Vault authentication via GitHub OIDC tokens      | Facilitates secure CI authentication |
| GitHub Actions Runner  | Executes Terraform using temporary short-lived AWS keys   | Requests credentials from Vault   |
| Terraform scripts      | Infrastructure code creating resources on AWS             | Stored and versioned in Git       |

### 3.5 Key Interview Points

- Vault is more than just secrets storage; it **eliminates the risks of long-lived static credentials**.
- GitHub Actions uses OIDC authentication to securely request **ephemeral credentials** from Vault.
- Vault dynamically provisions short-lived IAM users/roles on AWS for Terraform execution.
- This setup enforces **least privilege, auditability, and secret lifecycle management** in infrastructure automation.

---

# Summary

The video systematically covers **three pillars to secure Terraform deployments in enterprise DevSecOps** environments:

1. **DevSecOps best practices:** Use `.gitignore`, enforce pre-commit scanning with GitLeaks, and run secrets scanning via CI/CD pipelines.
2. **Terraform misconfiguration detection with Checkov:** Automate static code security analysis to catch insecure Terraform settings uncatchable by native Terraform or git checks.
3. **Secrets management with HashiCorp Vault:** Dynamically generate short-lived AWS credentials accessed securely via GitHub Actions OIDC tokens, avoiding long-lived static secrets in CI/CD.

Together, these strategies enable **secure, auditable, and resilient management of Terraform infrastructure**, minimizing human errors, insider threats, and misconfiguration risks. The tutorial provides detailed, hands-on instructions and demo repository references for implementation in real-world scenarios.
