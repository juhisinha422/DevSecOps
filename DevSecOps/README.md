### Summary of DevSecOps Zero to Hero Series by Abishek

This comprehensive video series by Abishek covers **DevSecOps fundamentals and practical implementation** across various key areas of the DevOps lifecycle, focusing heavily on security integration. The series is aimed at DevOps engineers transitioning into DevSecOps roles or stepping up in their organizations by embedding security into every phase of development, deployment, and operations.

---

### Overview & Structure of the Series

The series is divided into **seven detailed parts**:

| Part | Topic                                      | Key Focus                                                                                           |
|-------|--------------------------------------------|---------------------------------------------------------------------------------------------------|
| 1     | Introduction to DevSecOps                   | What DevSecOps is, scope beyond CI/CD, threat modeling using OAS Thread Dragon, security reports   |
| 2     | Securing Git and GitHub                     | .gitignore, pre-commit hooks, Git secrets scanning (GitLeaks), branch protection, RBAC, Dependabot|
| 3     | Infrastructure as Code (Terraform) Security| Best practices, Checkov for Terraform misconfigurations, HashiCorp Vault integration for secrets   |
| 4     | Container Security                          | Running containers as non-root, multi-stage Docker builds, distroless images, Docker ignore, runtime hardening |
| 5     | Kubernetes Security                         | Namespaces, RBAC, Network policies, Kyverno for policy enforcement, Kubernetes secrets, External Secrets Operator |
| 6     | Application Security (SAST, SCA, DAST)     | Static Application Security Testing (SonarQube), Software Composition Analysis (pip audit), Dynamic Application Security Testing (OWASP ZAP) |
| 7     | End-to-End DevSecOps Implementation        | Three-tier app (frontend, backend, DB), infrastructure setup (Terraform + AWS EKS), containerization, Kubernetes manifests, CI/CD pipeline with GitHub Actions and Argo CD |

---

### Key Insights and Concepts

#### 1. **DevSecOps Fundamentals**

- **DevSecOps = DevOps + Security Mindset**  
  It is not a separate role but an integrated approach where security is embedded from the start ("shift-left").  
- Early DevOps lacked security focus, leading to vulnerabilities; hence, DevSecOps emerged.  
- DevSecOps covers security in **all DevOps activities**, including Git, Terraform, Containers, Kubernetes, CI/CD, not just security stages in pipelines.  
- DevSecOps helps address **AI-generated code vulnerabilities** by enforcing strong pipeline checks (e.g., GitLeaks, Snyk).  

#### 2. **Threat Modeling with OAS Thread Dragon**

- Visual modeling of app/infrastructure architecture with components, actors, data flows, and trust boundaries.  
- Identifies vulnerabilities like **information disclosure, privilege escalation, DOS attacks** via automated threat detection.  
- Generates **detailed PDF security reports** for management and development teams to advocate DevSecOps adoption.  
- Uses the **STRIDE framework** for threat classification.

#### 3. **Git Security Best Practices**

- Use **`.gitignore`** to prevent committing sensitive files (e.g., `.env`, terraform state).  
- Implement **pre-commit hooks** (custom scripts or pre-commit framework with GitLeaks) to detect secrets/tokens in any committed files.  
- Scan entire repo history for leaked secrets using GitLeaks CLI regularly.  
- Integrate GitLeaks in GitHub Actions to block secrets in PRs.  
- Enforce **branch protection rules** and **RBAC** in repositories.  
- Use **mandatory reviews** and **CODEOWNERS** to control PR approvals.  
- Use **Dependabot** to automatically update vulnerable package versions.

#### 4. **Terraform Security**

- Avoid hardcoding credentials; use `.gitignore` for `.env` and state files.  
- Use pre-commit hooks to block secrets in Terraform files.  
- Use **Checkov** to detect insecure Terraform configurations (e.g., public S3 buckets).  
- Use **HashiCorp Vault** for **short-lived credentials** to avoid long-lived secrets in GitHub Actions; Vault issues ephemeral AWS IAM credentials via OIDC integration.  
- Terraform code should be versioned and deployed via CI/CD pipelines for auditability and rollback.

#### 5. **Container Security**

- Avoid running containers as **root user**; create and switch to non-root user in Dockerfile.  
- Use **multi-stage builds** to separate build-time dependencies from production runtime artifacts, drastically reducing image size and attack surface.  
- Use **distroless or slim base images** to minimize unnecessary packages/binaries.  
- Use **`.dockerignore`** to exclude unwanted files from the image build context.  
- Harden container runtime with Docker run parameters such as `--read-only`, `--tmpfs`, `--cap-drop=all`, `--security-opt=no-new-privileges`, resource limits (`--memory`, `--cpus`).

#### 6. **Kubernetes Security**

- Use **Namespaces** to logically isolate teams/applications in a single cluster; control resource quotas per namespace.  
- Use **RBAC (Role-Based Access Control)** to assign fine-grained permissions to service accounts running pods.  
- Use **Network Policies** to restrict pod-to-pod communication, allowing only trusted pods (e.g., frontend can access backend, attacker pod cannot).  
- Use **Kyverno** for policy enforcement via admission controllers, e.g., block pods using `latest` tag, enforce resource limits, require probes.  
- Use **Kubernetes Secrets** for storing sensitive data; secrets are base64 encoded but not encrypted by default.  
- Use **External Secrets Operator (ESO)** integrated with Vault to securely fetch secrets from external secret stores and avoid storing secrets directly in Git.

#### 7. **Application Security (AppSec)**

- **SAST (Static Application Security Testing):** Tools like SonarQube analyze source code for vulnerabilities such as SQL injection, insecure SSL, hardcoded secrets, CSRF etc.  
- **SCA (Software Composition Analysis):** Tools like `pip-audit` identify vulnerable dependencies (direct and transitive) based on CVEs.  
- **DAST (Dynamic Application Security Testing):** Tools like OWASP ZAP scan running applications by sending thousands of API calls to detect runtime vulnerabilities.  
- Demo uses **OWASP Juice Shop**, an intentionally vulnerable e-commerce app, to demonstrate DAST.

#### 8. **End-to-End DevSecOps Implementation**

- Example project: **Journey** - a lightweight blogging app with React frontend, NodeJS backend, and Postgres DB.  
- Steps covered:  
  - Run app locally on EC2 instance.  
  - Infrastructure provisioning with Terraform (VPC, EKS cluster in private subnet).  
  - Containerize frontend/backend with Docker multi-stage builds, non-root users.  
  - Create Docker Compose for local multi-container orchestration.  
  - Write Kubernetes manifests including deployments, services, PVC, storage class, secrets, network policies.  
  - Push Docker images to GitHub Container Registry.  
  - Deploy app on EKS cluster with autoscaling nodes (EKS Autopilot mode).  
  - Implement full CI/CD pipeline using GitHub Actions: linting, SCA, Docker build, image scanning (Trivy), manifest scanning (Checkov), Dockerfile linting, Kubernetes manifest update.  
  - Use Argo CD for GitOps-style continuous delivery syncing manifests to the cluster automatically.

---

### Quantitative Comparison: Docker Image Sizes (Example)

| Image Type          | Image Size (MB) | Description                                           |
|---------------------|-----------------|-------------------------------------------------------|
| Default Docker Build | ~401            | Single-stage build, root user, many packages          |
| Multi-stage Build    | ~79.5           | Separates build/runtime, smaller runtime image        |
| Distroless Image     | ~52             | Minimal base image, further reduces attack surface     |

---

### Core DevSecOps Practices Emphasized

- **Shift Left Security:** Embed security from the earliest phases (code commit, infra as code, container build).  
- **Automate Security Checks:** Pre-commit hooks, CI/CD pipeline scans (linting, SCA, image scanning, manifest scanning).  
- **Use Short-lived Secrets:** Vault for ephemeral credentials to reduce risk of leaks.  
- **Least Privilege:** Non-root containers, RBAC in Kubernetes.  
- **Network Segmentation:** Kubernetes Namespaces and Network Policies.  
- **Continuous Monitoring and Enforcement:** Kyverno for policy enforcement, Dependabot for dependency updates, Argo CD for automated deployments.  
- **Comprehensive Application Security:** Combine SAST, SCA, and DAST tools.  

---

### Summary Table: Security Tools and Their Purpose

| Tool/Concept          | Purpose                                                   | Usage Context                     |
|-----------------------|-----------------------------------------------------------|----------------------------------|
| **OAS Thread Dragon** | Threat modeling & architecture vulnerability detection    | Infrastructure & app architecture |
| **GitLeaks**          | Detect secrets in Git commits and history                 | Git repos & CI/CD                |
| **Pre-commit Framework** | Pattern-based secret detection before commit            | Git pre-commit                  |
| **Checkov**            | Detect insecure Terraform/Kubernetes manifest configs    | IaC scanning                   |
| **HashiCorp Vault**    | Manage short-lived secrets, dynamic credentials           | Secret management                |
| **Trivy**              | Container image vulnerability scanning                    | Container security              |
| **Kyverno**            | Kubernetes policy enforcement via admission controllers   | K8s cluster governance          |
| **SonarQube (SAST)**   | Static code vulnerability detection                       | Application source code         |
| **pip-audit (SCA)**    | Detect vulnerable dependencies                            | Application dependency check    |
| **OWASP ZAP (DAST)**   | Runtime security scanning via HTTP APIs                   | Application penetration testing |
| **Argo CD**            | GitOps continuous delivery sync & deployment              | Kubernetes CD                   |

---

### Key Takeaways

- **DevSecOps is an integrated mindset, not just a tool or role.** Every DevOps activity must incorporate security.  
- Security needs to be **shifted left**, starting from code commit and infrastructure planning stages.  
- Automated tools and pipelines are essential to **detect, prevent, and mitigate security risks** in code, infrastructure, containers, and orchestration platforms.  
- **Secrets management and least privilege** are critical pillars to minimize risk exposure.  
- Kubernetes security is multi-faceted: use namespaces, RBAC, network policies, admission control policies, and secure secrets management.  
- Application security requires a combination of **code scanning, dependency analysis, and runtime testing** to cover all attack surfaces.  
- End-to-end DevSecOps pipelines combine all these practices to ensure continuous, automated security across the software delivery lifecycle.

---

### Recommended Actions for Learners

- Follow the **GitHub repository** provided with detailed manifests, scripts, and demo applications.  
- Practice **drawing threat models** using OAS Thread Dragon for your projects.  
- Implement **pre-commit hooks and Git secret scanning** before pushing code.  
- Use **Checkov and Trivy** in your CI pipeline to scan IaC and container images.  
- Explore **HashiCorp Vault** for dynamic secrets management integration.  
- Learn Kubernetes security essentials: namespaces, RBAC, network policies, and Kyverno.  
- Experiment with **SonarQube, pip-audit, and OWASP ZAP** for application security.  
- Build and run **multi-stage Docker builds** and optimize image sizes using distroless images.  
- Develop a **GitHub Actions pipeline** replicating the CI/CD example for your projects.  
- Integrate **Argo CD** for GitOps-style continuous deployment.

---

This series offers a **well-rounded, practical, and security-focused roadmap for mastering DevSecOps**, with hands-on demos, real-world tools, and best practices spanning code to cloud.
