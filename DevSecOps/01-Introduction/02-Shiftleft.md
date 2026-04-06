# Shift Left Security Notes

## What is Shift Left?
Shift Left means moving security practices **earlier in the SDLC**.

Instead of testing security at the end, we test during:
- Coding
- Build
- Testing phase

---

## Why Shift Left?
- Fixing bugs early is cheaper
- Reduces production risks
- Improves code quality

---

## How to Implement Shift Left
- Secure coding practices
- Code reviews
- Static code analysis (SAST)
- Dependency scanning (SCA)
- Automated security tests in CI/CD

---

## Example
Before:
Code → Build → Deploy → Security Testing ❌

After:
Code → Security Scan → Build → Test → Deploy ✅

---

## Tools
- SonarQube (SAST)
- Snyk (Dependency scanning)
- GitHub Advanced Security
- OWASP ZAP (DAST)

---

## Benefits
- Early detection of vulnerabilities
- Faster development cycle
- Reduced cost
