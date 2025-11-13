# DevSecOps and Related Practices

## What is DevSecOps, and how does it differ from traditional DevOps?

**DevSecOps** (Development, Security, and Operations) is a cultural and technical shift that integrates security practices directly into the DevOps lifecycle. It emphasizes the importance of security being a **shared responsibility** throughout development and deployment, rather than being an afterthought or handled solely by a separate security team.

---

## Static Application Security Testing (SAST) tools in CI/CD

SAST tools analyze **source code** or binaries to detect vulnerabilities **before runtime**.

**Common SAST tools:**

* **SonarQube**
* **Snyk**
* **CodeQL**

**Notes on CodeQL (used at Microsoft in ADO ecosystem):**

* Detects security vulnerabilities and coding flaws by analyzing code as a database and performing query-based analysis.
* Common issues detected:

  * **Security Vulnerabilities:** SQL Injection, XSS, hard-coded secrets, authentication & authorization flaws
  * **Code Quality Issues:** Memory leaks, unreachable code, use of deprecated functions
  * **Logical and Performance Bugs:** Incorrect error handling, inefficient loops & queries

> Many organizations use both: SonarQube for general quality and CodeQL for advanced security scanning, giving a complementary approach to SAST.
> SonarQube does pattern matching file by file, whereas CodeQL treat the code as db records, and fire query to detect issues. It makes CodeQL more powerful to detect issues spanning to multiple files.
> Example, you have QueryBuilder class which forumlates the query ["SELECT * FROM users WHERE username = '" + userInput + "'";]. There are multiple calling classes which is calling this builder class. SonarQube detects only the QueryBuilder class, CodeQL detects all calling classes so that you can change them to sanitize the SQL statement.
> SonarQube advantages over CodeQL: It has 40+ language support, IDE plugin, and rich historical data


---

## Dynamic Application Security Testing (DAST) tools in CI/CD

DAST tools test **running applications** for vulnerabilities such as SQL injection, XSS, and authentication flaws.

**Microsoft approach:**

* DAST tools are **not integrated into CI/CD pipelines**.
* Instead, a **centralized Web Scanning Tool / WCP Portal** is used for post-production security assessments:

  * Registers applications for scanning after deployment
  * Performs monthly scans for vulnerabilities (SQLi, XSS, authentication flaws)
  * Reports security issues to development teams
  * Conducts privacy reviews, including cookie scanning and data protection compliance

**Common DAST Tools (often missing in practice):**

* **OWASP ZAP (Zed Attack Proxy)** – Open-source tool for detecting common web application vulnerabilities; ideal for automation in pipelines.
* **Burp Suite** – Comprehensive DAST platform for web applications, offering both automated and manual security testing.
* **Arachni** – Framework-based DAST scanner for Ruby, JavaScript, and web technologies.
* **AppScan (HCL/AppScan Source)** – Enterprise-grade scanner that integrates into CI/CD for dynamic and interactive testing.
* **Netsparker / Invicti** – Automated DAST platform that identifies vulnerabilities and provides proof-based scanning to reduce false positives.

**Benefit:**
Integrating one or more of these tools provides early visibility into runtime vulnerabilities and improves the feedback loop for developers — preventing costly security issues from reaching production.

---

## Software Composition Analysis (SCA)

**SCA** is a security practice to identify and manage **vulnerabilities in open-source dependencies** within a project.

* Helps detect security flaws, license compliance issues, and outdated libraries.
* Popular tools: **Snyk**, **GitHub Dependabot**

---

## Infrastructure as Code (IaC) Security

As teams increasingly use **Infrastructure as Code (IaC)** tools such as **Terraform**, **Bicep**, and **ARM templates** to provision and configure cloud resources, securing infrastructure definitions becomes essential.

**IaC scanning** detects:

* Open network ports or overly permissive security groups
* Missing encryption on storage or databases
* Unrestricted access policies and privilege escalations

**Common IaC Security Tools:**

* **Checkov** – Detects misconfigurations in Terraform, CloudFormation, and Kubernetes manifests
* **Snyk IaC** – Identifies security issues early in pipelines
* **Terraform Sentinel** – Enforces compliance via policy-as-code

**Benefit:** Prevents cloud misconfigurations and enforces compliance **before deployment**.

---

## Container and Image Security

Containerized environments introduce their own risks — vulnerabilities within images, misconfigured Dockerfiles, or embedded secrets can expose systems.

**Container scanning** helps detect:

* Vulnerable OS packages or dependencies
* Secrets or credentials baked into images
* Insecure configurations or permissions

**Popular Container Security Tools:**

* **Trivy**
* **Anchore**
* **Clair**
* **Snyk Container**

**Benefit:** Ensures only secure and compliant images are promoted through CI/CD pipelines and deployed to Kubernetes or cloud registries.

---

## Secrets Management in DevSecOps

Beyond preventing credential check-ins, DevSecOps emphasizes **secure storage, rotation, and access management of secrets**.

**Best Practices:**

* Store secrets in **Azure Key Vault**, **AWS Secrets Manager**, or **HashiCorp Vault**
* Use **Managed Identities** instead of static credentials
* Enforce automated secret rotation and access control policies

**Supporting Tools:**

* **GitGuardian** or **TruffleHog** for scanning repositories
* **Azure DevOps Branch Policies** and pre-commit hooks to block credential leaks

**Benefit:** Prevents sensitive data exposure and enforces consistent secret management practices.

---

## Continuous Monitoring and Security Posture Management

Security doesn’t end after deployment. Continuous monitoring ensures that applications, infrastructure, and configurations remain secure and compliant in real time.

**Key Tools and Practices:**

* **Microsoft Defender for Cloud** and **Azure Security Center** for cloud workload protection
* **AWS Security Hub** for centralized compliance reporting
* **Falco** for runtime security in Kubernetes
* **Prometheus + Grafana** for observability and anomaly detection

**Benefit:** Provides ongoing visibility into security posture and ensures adherence to frameworks such as **CIS Benchmarks**, **ISO 27001**, or **NIST**.

---

## Threat Modeling and Shift-Left Security

Security should be part of design discussions, not just testing. Threat modeling identifies potential risks before code is even written.

**Practices:**

* Conduct **threat modeling sessions** during design and architecture reviews
* Apply frameworks like **Microsoft STRIDE** or **OWASP Threat Dragon**
* Train teams to think about security implications early in SDLC

**Benefit:** Reduces costly late-stage fixes and promotes a **shift-left** security mindset across the organization.

---

## Access Control in Azure DevOps (ADO)

### Organization Level

* **All Users:** Add AAD users, give project access, and define access levels
* **Group Rules:** Add AAD or DevOps groups to projects, define their access levels
* **Access Levels:** Predefined sets of permissions
* **Permissions:** Define group & user permissions for:

  * Repos
  * Pipelines
  * Test plans
  * Dashboards
  * Policies

### Project Level

* Access control at project scope allows finer control over repositories, pipelines, boards, test plans, and other resources.
* You can manage **users, groups, roles, and permissions** specifically for each project.

---

## Summary

A mature DevSecOps implementation integrates multiple layers of security throughout the software delivery lifecycle:

| Security Layer            | Purpose                                     | Example Tools                           |
| ------------------------- | ------------------------------------------- | --------------------------------------- |
| **SAST**                  | Scan source code for vulnerabilities        | SonarQube, CodeQL                       |
| **DAST**                  | Test running applications dynamically       | OWASP ZAP, Burp Suite, AppScan, Arachni |
| **SCA**                   | Manage open-source dependencies             | Snyk, Dependabot                        |
| **IaC Security**          | Detect misconfigurations in cloud templates | Checkov, Snyk IaC                       |
| **Container Security**    | Scan container images for CVEs              | Trivy, Anchore                          |
| **Secrets Management**    | Secure storage and rotation of credentials  | Azure Key Vault, TruffleHog             |
| **Continuous Monitoring** | Runtime threat and compliance monitoring    | Defender for Cloud, Falco               |
| **Threat Modeling**       | Identify risks early in design              | STRIDE, Threat Dragon                   |

---

✅ **In summary:**
DevSecOps is not just about tools — it’s about embedding **security culture and automation** throughout the SDLC. By integrating these practices, organizations can deliver software that is **faster, safer, and more compliant** from code to cloud.
