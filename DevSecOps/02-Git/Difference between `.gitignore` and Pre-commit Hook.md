## 🔹 Difference between `.gitignore` and Pre-commit Hook

### ✅ `.gitignore`

* A configuration file used by Git
* Specifies files and directories that Git should ignore
* Prevents unnecessary files from being tracked or committed

**Examples:**

* `node_modules/`
* `.env`
* `*.log`

**Key Points:**

* Works before staging (`git add`)
* Applies only to untracked files
* Does not enforce any rules on commits
* Already tracked files are not affected

---

### ✅ Pre-commit Hook

* A script that runs automatically before a commit is created
* Used to validate, check, or block commits

**Common Uses:**

* Code linting (ESLint, Checkstyle)
* Running unit tests
* Secret scanning
* Code formatting

**Example:**

```bash
#!/bin/sh
npm run lint
if [ $? -ne 0 ]; then
  echo "Lint failed. Commit blocked."
  exit 1
fi
```

**Key Points:**

* Runs after staging but before commit
* Can block commits if checks fail
* Helps enforce coding standards and security policies

---

## 🔥 Key Differences

| Feature     | `.gitignore`            | Pre-commit Hook                |
| ----------- | ----------------------- | ------------------------------ |
| Purpose     | Ignore files            | Validate commits               |
| Stage       | Before staging          | Before commit                  |
| Type        | Configuration file      | Script                         |
| Enforcement | No                      | Yes (can block commit)         |
| Usage       | Avoid unnecessary files | Ensure code quality & security |

---

## ✅ Best Practice (4+ Years Experience)

* Use `.gitignore` to exclude unwanted files from the repository
* Use pre-commit hooks to enforce:

  * Code quality checks
  * Security validations
  * Standardized commits

👉 Both are used together in real-world DevOps workflows.

**.gitignore works on the files that will avoid committing of secure files to the git repositories.
Files that are sensitive like .env, terraform files.**

**Pre-commit hook is a script that will see for a particular pattern like secret pattern. Like developer encoded secrets in the code file .py file or .java file itself.**

## Official Website: - https://pre-commit.com/
