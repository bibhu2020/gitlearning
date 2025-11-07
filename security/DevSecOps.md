# DevSecOps and Related Practices

## What is DevSecOps, and how does it differ from traditional DevOps?

**DevSecOps** (Development, Security, and Operations) is a cultural and technical shift that integrates security practices directly into the DevOps lifecycle. It emphasizes the importance of security being a **shared responsibility** throughout development and deployment, rather than being an afterthought or handled solely by a separate security team.

---

## Static Application Security Testing (SAST) tools in CI/CD

SAST tools analyze **source code** or binaries to detect vulnerabilities **before runtime**.  

**Common SAST tools:**
- **SonarQube**
- **Snyk**
- **CodeQL**

**Notes on CodeQL (used at Microsoft in ADO ecosystem):**
- Detects security vulnerabilities and coding flaws by analyzing code as a database and performing query-based analysis.
- Common issues detected:
  - **Security Vulnerabilities:** SQL Injection, XSS, hard-coded secrets, authentication & authorization flaws
  - **Code Quality Issues:** Memory leaks, unreachable code, use of deprecated functions
  - **Logical and Performance Bugs:** Incorrect error handling, inefficient loops & queries

> SonarQube also performs coding quality and standards checks, which CodeQL does not.

---

## Dynamic Application Security Testing (DAST) tools in CI/CD

DAST tools test **running applications** for vulnerabilities such as SQL injection, XSS, and authentication flaws.

**Microsoft approach:**
- DAST tools are **not integrated into CI/CD pipelines**.
- Instead, a **centralized Web Scanning Tool / WCP Portal** is used for post-production security assessments:
  - Registers applications for scanning after deployment
  - Performs monthly scans for vulnerabilities (SQLi, XSS, authentication flaws)
  - Reports security issues to development teams
  - Conducts privacy reviews, including cookie scanning and data protection compliance

---

## Software Composition Analysis (SCA)

**SCA** is a security practice to identify and manage **vulnerabilities in open-source dependencies** within a project.  
- Helps detect security flaws, license compliance issues, and outdated libraries.
- Popular tools: **Snyk**, **GitHub Dependabot**

---

## What is OWASP?

**OWASP** (Open Web Application Security Project) is a nonprofit organization focused on **improving software and web application security**.  
- Provides free, open-source resources for application security.
- Raises awareness about security risks in software development.

**OWASP Top 10 (2021) includes:**
1. Injection (e.g., SQL Injection)
2. Broken Authentication
3. Sensitive Data Exposure
4. XML External Entities (XXE)
5. Broken Access Control
6. Security Misconfiguration
7. Cross-Site Scripting (XSS)
8. Insecure Deserialization
9. Using Components with Known Vulnerabilities
10. Insufficient Logging & Monitoring

---

## Other DevSecOps Practices

### Injecting tools into the pipeline
- Integrate **SAST, DAST, SCA** tools directly into CI/CD pipelines.
- Automated scans help detect vulnerabilities **before deployment**.

### Preventing credential check-in into Git
- Use **pre-commit hooks**, Git policies, and automated scanners to prevent secrets from being committed.
- Tools: **GitGuardian, TruffleHog, Azure DevOps branch policies**.

---

## Access Control in Azure DevOps (ADO)

### Organization Level
- **All Users:** Add AAD users, give project access, and define access levels
- **Group Rules:** Add AAD or DevOps groups to projects, define their access levels
- **Access Levels:** Predefined sets of permissions
- **Permissions:** Define group & user permissions for:
  - Repos
  - Pipelines
  - Test plans
  - Dashboards
  - Policies

### Project Level
- Access control at project scope allows finer control over repositories, pipelines, boards, test plans, and other resources.
- You can manage **users, groups, roles, and permissions** specifically for each project.

---

