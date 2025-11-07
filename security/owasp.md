# OWASP (Open Web Application Security Project)

When you build or secure applications on Azure, **OWASP** gives you a universal reference framework to evaluate application-level security — regardless of platform or language.

**For example:**
- When you deploy an app on Azure App Service, you still need to ensure it’s protected from SQL injection or cross-site scripting (XSS).
- Azure provides infrastructure-level protection, but application-layer vulnerabilities remain your responsibility.
- That’s where OWASP helps — it defines what threats to look for and how to mitigate them.

## OWASP Top 10
These are the ten most common and impactful web application security risks identified by OWASP.

| #       | Category                                       | Description                                                          | Example / Mitigation                                                              |
| ------- | ---------------------------------------------- | -------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| **A01** | **Broken Access Control**                      | Users can access data or functions they shouldn’t.                   | Enforce least privilege; validate roles in code; use Azure AD RBAC.               |
| **A02** | **Cryptographic Failures**                     | Weak or missing encryption allows data leaks.                        | Use HTTPS/TLS; encrypt with Key Vault-managed keys.                               |
| **A03** | **Injection**                                  | Malicious input (e.g., SQL, NoSQL, OS commands) manipulates queries. | Use parameterized queries and ORM frameworks.                                     |
| **A04** | **Insecure Design**                            | Poor architectural security decisions.                               | Apply secure design reviews and threat modeling early.                            |
| **A05** | **Security Misconfiguration**                  | Default settings, verbose errors, or open ports.                     | Harden app services, limit public exposure, automate compliance via Azure Policy. |
| **A06** | **Vulnerable and Outdated Components**         | Using old or unpatched libraries.                                    | Automate dependency scanning in CI/CD.                                            |
| **A07** | **Identification and Authentication Failures** | Weak authentication, missing MFA, insecure tokens.                   | Use Azure AD, enforce MFA, secure tokens in Key Vault.                            |
| **A08** | **Software and Data Integrity Failures**       | Unverified updates or CI/CD pipeline weaknesses.                     | Sign images/artifacts; enable Defender for DevOps.                                |
| **A09** | **Security Logging and Monitoring Failures**   | Inadequate logging prevents attack detection.                        | Enable Azure Monitor, Sentinel, Defender for Cloud.                               |
| **A10** | **Server-Side Request Forgery (SSRF)**         | Server makes untrusted requests on behalf of attacker.               | Validate URLs, use managed identity, restrict outbound network access.  |

## A01 — Broken Access Control

What it is
Users can access or act on resources they shouldn’t (view other users’ data, call admin APIs, etc.).

Vulnerable example (BOLA — Broken Object Level Authorization)

Node.js/Express endpoint:

```js
// VULNERABLE: no ownership check
app.get('/invoices/:id', async (req, res) => {
  const invoice = await db.query('SELECT * FROM invoices WHERE id = $1', [req.params.id]);
  res.json(invoice.rows[0]);
});
```

If Alice is logged in she can call `/invoices/42` — but so could she call `/invoices/43` and see Bob’s invoice.

Fixed example

```js
// FIX: verify ownership server-side
app.get('/invoices/:id', async (req, res) => {
  const invoice = await db.query('SELECT * FROM invoices WHERE id = $1', [req.params.id]);
  if (!invoice.rows[0]) return res.status(404).send('Not found');
  if (invoice.rows[0].owner_user_id !== req.user.sub) return res.status(403).send('Forbidden');
  res.json(invoice.rows[0]);
});
```

Here `req.user.sub` is the authenticated user's ID (from JWT/Azure AD).

Azure config example (protecting an API with roles)

Define App Roles in Azure AD App Registration (e.g., `InvoiceReader`, `InvoiceAdmin`).

In your API, validate the user’s role claim and enforce authorization:

```js
if (!req.user.roles.includes('InvoiceAdmin')) return res.status(403).send('Admin only');
```

Checklist

- Ownership checks exist for every object read/write.
- API enforces role/claim checks regardless of front-end UI.
- Sequential numeric IDs replaced by GUIDs or opaque identifiers when possible.

---

## A02 — Cryptographic Failures

What it is
Failure to encrypt, weak algorithms, or poor key management.

Vulnerable example (hardcoded secret + plaintext transport)

`appsettings.json` contains:

```json
{
  "ConnectionString": "Server=...;User Id=app;Password=SuperSecret123;"
}
```

and the web app allows HTTP (not HTTPS). Secrets are in source control and traffic is unencrypted.

Fixed example

1. Move secret to Key Vault (example CLI)

```bash
az keyvault secret set --vault-name MyVault --name "DbPassword" --value "SuperSecret123"
# In app, use Managed Identity to fetch secret from Key Vault, NOT from config
```

2. Enforce HTTPS in App Service

In Azure Portal → App Service → TLS/SSL settings → "HTTPS Only" = On.

3. Use modern password hashing

```js
const hash = await bcrypt.hash(password, 12); // bcrypt (or Argon2)
```

Checklist

- No secrets in code/config in repo.
- Key Vault + Managed Identity used for secrets.
- HTTPS enforced (TLS 1.2+).
- Sensitive DB columns protected with TDE / Always Encrypted if needed.

---

## A03 — Injection

What it is
Attackers send input that the app treats as code (SQL, shell, LDAP, etc.).

Vulnerable SQLi example (string concatenation)

```js
// BAD: vulnerable to SQL injection
const query = `SELECT * FROM users WHERE username = '${req.query.username}'`;
db.query(query, (err, res) => { ... });
```

If attacker sends `username=admin'--`, the SQL logic is altered.

Fixed example (parameterized)

```js
// GOOD: parameterized query
const text = 'SELECT * FROM users WHERE username = $1';
const values = [req.query.username];
const result = await db.query(text, values);
```

NoSQL example (unsafe query building)

Vulnerable MongoDB usage:

```js
// BAD: unsanitized JSON string parsed into a query object
const filter = JSON.parse(req.body.filter);
const docs = await db.collection('items').find(filter).toArray();
```

An attacker can pass:

```json
{"$where": "function() { return this.price < 0; }"}
```

to execute arbitrary JS when server allows it.

Fix (allowlist / sanitize)

Only accept whitelisted fields and types:

```js
const allowedFields = ['category','priceMin','priceMax'];
const filter = {};
if (allowedFields.includes(req.body.field)) filter[req.body.field] = req.body.value;
```

Azure-specific mitigation

Use Web Application Firewall (WAF) rules at Application Gateway or Front Door as an additional layer (WAF helps block common SQLi patterns but is not a replacement for parameterization).

Checklist

- All DB access uses parameterized queries or prepared statements.
- No `eval()` or unsafe deserialization.
- ORM raw queries use parameter binding.
- DAST and SAST included in CI.

---

## A04 — Insecure Design

What it is
Flaws in architecture / design — not just code — that create security gaps (e.g., poor multi-tenancy isolation, missing threat modeling).

Concrete example (multi-tenancy design flaw)

Design: single shared database table `users` with `tenant_id`, and UI assumes tenant isolation.
Vulnerability: a background job forgets to filter by `tenant_id` and copies all users to an export.

Remediation (threat modeling + separation)

Run a threat model: draw data flows and list trust boundaries (example: user → web app → API → storage).

Architect strong tenant isolation:

- Option A: Separate DB per tenant (strong isolation).
- Option B: Shared DB + strict `tenant_id` enforcement at every layer, with DB row-level security (RLS) enforced in DB engine.

Azure example (isolation)

- Use Azure SQL Row-Level Security (RLS) or separate resource groups/subscriptions per tenant for strict isolation.
- Use Private Endpoints to keep data plane traffic inside VNets.

Checklist

- Threat modeling done for major components.
- Multi-tenant boundaries explicitly enforced and tested.
- Sensitive operations require multi-step approvals where applicable.

---

## A05 — Security Misconfiguration

What it is
Unintentionally exposing services, using default credentials, verbose errors, or permissive cloud settings.

Vulnerable example (public storage)

An Azure Blob container was created with public access:

`https://mystorage.blob.core.windows.net/publiccontainer/photo.jpg`

Fixed example (private container + SAS)

Set container access level to private in Portal or CLI:

```bash
az storage container set-permission --account-name mystorage --name publiccontainer --public-access off
```

If apps need temporary access, use SAS tokens with short expiry or use Managed Identity to access Key Vault and then storage.

Other misconfig example (debug mode enabled)

Django app running in production with `DEBUG=True` — error pages leak stack traces, secrets, file paths.

Fix

Set `DEBUG=False` for production and sanitize error messages. Centralize error details to Application Insights or logs (not public pages).

Checklist

- No public storage unless explicitly required.
- Debug endpoints disabled in prod.
- NSGs and firewalls restrict inbound/external access.
- Azure Policy blocks risky configurations (e.g., public blobs).

---

## A06 — Vulnerable and Outdated Components

What it is
Using libraries, packages, containers or images with known vulnerabilities.

Vulnerable example

`package.json` includes `"lodash": "4.17.0"` (old) — contains known RCE vulnerabilities.

Fixed workflow

Enable Dependabot or Renovate to create PRs for version bumps.

Add SCA scanning in CI (GitHub Dependabot alerts or Snyk).

Example GitHub Actions step:

```yaml
- name: Run npm audit
  run: npm audit --production
```

For containers: use a curated base image and scan images in Azure Container Registry (ACR) with Defender for Containers.

Checklist

- Automated dependency updates enabled (Dependabot).
- SCA in CI and policy to block deploys with critical CVEs.
- Container images scanned and signed before deployment.

---

## A07 — Identification & Authentication Failures

What it is
Broken authentication flows, weak token handling, lack of MFA, or failing to validate tokens.

Vulnerable example (invalid token validation)

API accepts JWT without checking signature or issuer:

```js
const token = req.headers.authorization.split(' ')[1];
const payload = JSON.parse(Buffer.from(token.split('.')[1], 'base64').toString());
if (!payload) { /* accept anyway */ }
```

An attacker can forge payload and gain access.

Fixed example (validate JWT properly)

Using `passport-azure-ad` / `jsonwebtoken` libraries:

```js
const jwks = jwksClient({...});
const getKey = (header, cb) => jwks.getSigningKey(header.kid, (err, key) => cb(null, key.getPublicKey()));
jwt.verify(token, getKey, { audience: AZURE_AD_APP_ID, issuer: `https://sts.windows.net/${TENANT_ID}/` }, (err, decoded) => {
  if (err) return res.status(401).send('Invalid token');
  req.user = decoded;
});
```

Azure features to use

Use Azure AD for authentication (offload password complexity, token issuance).

Enforce Conditional Access and MFA for admins and risky sign-ins.

Checklist

- Tokens validated (signature, issuer, audience, expiry).
- MFA enforced for privileged roles.
- Refresh tokens rotated and revocable.

---

## A08 — Software & Data Integrity Failures

What it is
Artifacts (images, binaries, IaC templates) or data that can be tampered with before or during deployment.

Vulnerable example

CI/CD pipeline pulls a container image from Docker Hub `mycompany/app:latest` without verifying the source. A malicious image with the same tag is published and deployed.

Fixed example (image signing + ACR)

Push and sign images to Azure Container Registry (ACR) and enable content trust:

- Use `docker trust` / Notary or Sigstore (increasingly preferred).
- In CI, require image signature verification before deploy:

```bash
# pseudo-steps
az acr repository show --name myacr --image myapp:sha256:abcd1234
# verify signature with Sigstore/cosign
```

Lock down the production registry to only allow signed images from approved build pipeline service identity.

Checklist

- Artifacts signed and verified prior to deployment.
- CI credentials scoped and rotated.
- IaC templates validated and stored in source control with branch protections.

---

## A09 — Security Logging & Monitoring Failures

What it is
Missing or insufficient logs and alerts — attackers operate undetected.

Vulnerable example

App writes logs locally but never forwards them; logs are ephemeral and lost on pod restart. No alerts for repeated login failures or privilege changes.

Fixed example (centralized logging + alerting)

Instrument app with structured logs (JSON) and send to Application Insights:

```js
appInsights.trackEvent({ name: "login_failed", properties: { user: username, ip: req.ip }});
```

Configure Log Analytics and create Sentinel alerts:

Example alert: trigger when `SigninLogs` contains `RiskLevel` = `high` OR when there are >50 failed logins in 10 minutes.

Checklist

- Auth failures, privilege changes, and unusual queries are logged with user and IP.
- Logs centralized, immutable, and retained per policy.
- SIEM (Sentinel) rules and runbooks exist for incident response.

---

## A10 — Server-Side Request Forgery (SSRF)

What it is
App fetches attacker-supplied URLs, causing the server to contact internal services (metadata, internal APIs).

Vulnerable example (image proxy)

A service fetches images by URL for preview:

```js
// VULNERABLE: simply fetches whatever URL user gives
const resp = await fetch(req.query.url);
const body = await resp.buffer();
res.type(resp.headers.get('content-type')).send(body);
```

Attacker supplies `http://169.254.169.254/metadata/instance/compute` and reads instance metadata.

Fixed example (proxy with allowlist and scanning)

```js
// SAFE: allowlist + parsed host check
const allowedHosts = ['images.example-cdn.com','trusted.img.host'];
const parsed = new URL(req.query.url);
if (!allowedHosts.includes(parsed.hostname)) return res.status(400).send('Host not allowed');
const resp = await fetch(req.query.url, { timeout: 5000 });
// optionally scan/validate content-type and size
```

Azure networking defense

Block outbound access to `169.254.169.254` (IMDS) where possible or protect IMDS with endpoint protection for VMs/PaaS.

Use Azure Firewall and NSGs to block server egress to internal ranges or use forced-proxy so the proxy enforces allowlist.

Checklist

- Never allow arbitrary server-side fetches without validation.
- Use allowlist for domains and reject private IP ranges.
- Monitor for unusual outbound connections to internal endpoints.

---

## Final recommendations — how to use this document as a consultant

- Map OWASP to the application: for each item, list where in the app the risk exists and paste the concrete vulnerable code or infra config.
- Fix small issues first: parameterize queries, remove secrets from repos, enforce HTTPS.
- Automate enforcement: Azure Policy, CI gates (SAST/SCA/DAST), and PR checks.
- Instrument & test: add logging, run DAST and pen tests focused on the explicit examples above.
- Train teams: use the vulnerable / fixed examples in developer training sessions — hands-on fixes stick.

## Appendix — Quick mapping (OWASP → Azure features, with short examples)

- Broken Access Control → Azure AD App Roles (validate roles claim)
- Cryptographic Failures → Key Vault + Managed Identity (example CLI shown above)
- Injection → Parameterized queries + WAF at App Gateway/Front Door
- Insecure Design → Threat modeling + Azure Landing Zones for isolation
- Security Misconfiguration → Azure Policy blocking public blobs / enforcing HTTPS
- Vulnerable Components → Dependabot + Defender for Containers scanning ACR images
- Auth Failures → Conditional Access + MFA in Azure AD
- Integrity Failures → Signed artifacts in ACR + pipeline verification
- Logging Failures → App Insights + Sentinel alerts (example event/alert idea above)
- SSRF → NSG/Firewall + proxy allowlist for external fetches



