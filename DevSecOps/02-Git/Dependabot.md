# 🚀 Dependabot Notes

## 🔹 What is Dependabot?

Dependabot is a tool by GitHub that automatically keeps your project dependencies up to date by scanning dependency files and creating pull requests for updates.

---

## 🔹 Why Use Dependabot?

* Improves security by fixing vulnerable dependencies
* Automates dependency updates
* Reduces manual effort
* Keeps builds stable and updated

---

## 🔹 Key Features

* Automatic dependency updates via Pull Requests
* Security alerts for vulnerable libraries
* Native integration with GitHub workflows
* Supports multiple ecosystems:

  * Maven (Java)
  * npm (Node.js)
  * pip (Python)
  * Docker
  * GitHub Actions

---

## 🔹 How Dependabot Works

1. Scans dependency files like:

   * pom.xml
   * package.json
   * requirements.txt
2. Checks latest versions from registries
3. Creates Pull Requests with updates
4. Developer reviews and merges

---

## 🔹 Configuration File

Location:
.github/dependabot.yml

### Example:

```yaml
version: 2
updates:
  - package-ecosystem: "maven"
    directory: "/"
    schedule:
      interval: "daily"
```

---

## 🔹 Supported Update Intervals

* daily
* weekly
* monthly

---

## 🔹 Example Use Case

A Maven-based Java project has an outdated dependency with a security vulnerability.

Dependabot:

* Detects the issue
* Creates a PR with updated version
* Developer reviews and merges
* CI/CD pipeline runs with secure dependencies

---

## 🔹 Dependabot vs Renovate

| Feature       | Dependabot      | Renovate          |
| ------------- | --------------- | ----------------- |
| Owner         | GitHub          | Open Source       |
| Setup         | Easy            | Complex           |
| Customization | Limited         | High              |
| Best For      | Simple projects | Enterprise setups |

---

## 🔹 Best Practices

* Enable security updates
* Always review PRs before merging
* Integrate with CI/CD pipelines
* Use appropriate update schedules
* Be cautious with auto-merge

---

## 🔹 Interview Questions

**1. What is Dependabot?**
Dependabot is a tool that automates dependency updates and security fixes in GitHub repositories.

**2. How does Dependabot work?**
It scans dependency files, checks for updates, and creates pull requests.

**3. Where is Dependabot configured?**
In `.github/dependabot.yml`.

**4. Difference between alerts and updates?**

* Alerts → Security vulnerability warnings
* Updates → Version upgrade pull requests

---

## 🔹 One-Line Definition

Dependabot is a GitHub tool that automatically detects, updates, and secures project dependencies.
