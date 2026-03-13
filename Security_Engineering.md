# Security Engineering

## A Meta-Prompt for Threat Modeling, Secure Development, and Vulnerability Management

**Version:** 1.0
**Repository:** https://github.com/pragnakar/Security_Engineering
**Parent Meta-Prompt:** LLM-Native Software Engineering — https://github.com/pragnakar/LLM_NATIVE_SOFTWARE_ENGINEERING
**Companion Meta-Prompts:** Deployment Engineering, DevOps, Database, UI-UX, MLOps, API Design, Testing Strategy, Documentation, Scrum

---

## Thesis

Security is not a feature added before launch — it is a structural property of the system from its first line of code. Every layer of a software system — application code, APIs, data stores, infrastructure, and deployment pipelines — introduces attack surface. Security engineering ensures that attack surface is identified, minimized, and defended systematically, not reactively after a breach. This meta-prompt embeds security thinking into AI-assisted development so that LLM-generated code meets the same security standards as human-authored code — or higher, since AI agents can apply security rules consistently across every file they touch.

## When This Meta-Prompt Is Invoked

Invoke this meta-prompt when:
- A new system or service is being designed and threat modeling should inform architectural decisions
- An LLM coding agent is generating authentication, authorization, or access control logic
- APIs are being designed or reviewed for security posture
- Input validation, output encoding, or data sanitization is being implemented
- Cryptographic decisions are being made (hashing, encryption, signing, key management)
- A security audit or penetration test has surfaced findings that need remediation
- Dependencies are being added and supply chain risk needs assessment
- Compliance requirements (SOC 2, GDPR, HIPAA, PCI-DSS) need to be mapped to engineering controls

## Scope

**This meta-prompt covers:**
- Threat modeling: STRIDE analysis, attack surface mapping, trust boundaries
- Authentication and authorization: identity verification, role-based access control, session management
- Input validation and output encoding: injection prevention, XSS prevention, content security
- Cryptography: hashing (passwords, data integrity), encryption (at rest, in transit), key management
- Dependency and supply chain security: vulnerability scanning, lock files, update policies
- Secrets management: detection, storage, rotation, and leak prevention
- Secure coding patterns: defense in depth, fail-secure defaults, least privilege
- Security testing: static analysis (SAST), dynamic analysis (DAST), dependency scanning (SCA)
- Incident preparedness: security logging, audit trails, breach response readiness
- Compliance mapping: translating regulatory requirements into engineering controls

**This meta-prompt does NOT cover:**
- Infrastructure provisioning and network security configuration — handled by **Deployment Engineering**
- Runtime security monitoring, alerting, and incident response operations — handled by **DevOps**
- Database access controls and encryption configuration — handled by the **Database** meta-prompt
- API design patterns and versioning — handled by the **API Design** meta-prompt
- Application architecture and build workflow — handled by **LLM-Native Software Engineering**

## Principles

1. **Security is a design constraint, not a testing phase.** Security requirements are defined alongside functional requirements in SPEC.md. A system whose security properties are only evaluated after implementation will always have structural vulnerabilities that are expensive to fix.

2. **Assume all input is hostile.** Every input from every source — user forms, API requests, file uploads, environment variables, database results, third-party API responses — is validated and sanitized before use. Trust boundaries are explicit and enforced in code.

3. **Fail secure, not fail open.** When a system encounters an unexpected state — an unrecognized permission, a missing authentication token, a malformed request — the default behavior is denial, not access. Errors that silently grant access are the most dangerous class of security bug.

4. **Defense in depth — no single layer is sufficient.** Input validation at the API gateway does not eliminate the need for parameterized queries at the database layer. TLS in transit does not eliminate the need for encryption at rest. Every security control assumes the layer above it has already failed.

5. **Least privilege everywhere.** Every service account, database user, API key, and human operator receives the minimum permissions required for their function. Broad permissions are never granted for convenience and narrowed later.

6. **Secrets have a lifecycle: creation, storage, rotation, and revocation.** A secret that is created and never rotated is a time bomb. A secret that is revoked but not replaced is a denial-of-service. All four phases are planned and automated.

7. **Dependencies are attack surface.** Every third-party package adds code that the team did not write, review, or control. Dependencies are pinned, scanned for known vulnerabilities, and evaluated for maintenance health before adoption.

## Practices

### Threat Modeling

- Conduct a STRIDE analysis (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege) for every new service or significant feature before implementation begins.
- Identify trust boundaries: where does data cross from trusted to untrusted zones? Every trust boundary crossing requires explicit validation.
- Map the attack surface: list all entry points (APIs, webhooks, file upload endpoints, message queue consumers, CLI interfaces) and the data each accepts.
- Document threats, their severity (Critical, High, Medium, Low), and the specific mitigation for each in the project specification. Unmitigated threats above Low severity require documented acceptance by the project owner.
- Revisit the threat model when new entry points, data flows, or third-party integrations are added.

### Authentication and Authorization

- Use established authentication frameworks and protocols (OAuth 2.0, OpenID Connect, SAML). Do not implement custom authentication schemes unless the specific requirement cannot be met by established protocols.
- Hash passwords with bcrypt (cost factor 12+), scrypt, or Argon2id. Never use MD5, SHA-1, or unsalted SHA-256 for password storage.
- Enforce multi-factor authentication (MFA) for all administrative and privileged accounts.
- Implement role-based access control (RBAC) or attribute-based access control (ABAC) with permissions checked at every API endpoint, not only at the UI layer.
- Session tokens are generated with cryptographically secure random number generators, transmitted only over HTTPS, marked `HttpOnly`, `Secure`, and `SameSite=Strict` (or `Lax` where cross-site navigation is required).
- Token expiration is enforced server-side. Expired tokens return 401, not stale data.

### Input Validation and Output Encoding

- Validate all input against an allowlist schema: expected type, length, format, and range. Reject anything that does not match.
- Use parameterized queries for all database access. No exceptions, no escape-hatch string interpolation.
- Apply context-appropriate output encoding: HTML encoding for web content, URL encoding for URL parameters, JSON encoding for API responses, shell escaping for command execution.
- Implement Content Security Policy (CSP) headers on all web responses. A strict CSP prevents the execution of injected scripts even when XSS defenses at the application layer fail.
- File uploads are validated by content type (magic bytes, not just extension), size-limited, stored outside the web root, and served with `Content-Disposition: attachment` unless explicitly required for inline display.
- Rate-limit all authentication endpoints and public-facing APIs. Rate limits are enforced at the infrastructure layer, not only in application code.

### Cryptography

- Use TLS 1.2 or higher for all data in transit. TLS 1.0 and 1.1 are not acceptable.
- Encrypt all sensitive data at rest using AES-256 or equivalent. Encryption keys are stored in a key management service (AWS KMS, GCP KMS, HashiCorp Vault), never alongside the encrypted data.
- Use HMAC-SHA256 or higher for data integrity verification.
- Never implement custom cryptographic algorithms. Use well-established libraries (libsodium, OpenSSL, Go crypto, Python cryptography).
- Cryptographic key rotation is automated and tested. Old keys remain available for decrypting existing data during the rotation window.

### Dependency and Supply Chain Security

- Pin all dependencies to exact versions in lock files (package-lock.json, poetry.lock, go.sum, Cargo.lock). Floating version ranges in production are not permitted.
- Run automated dependency vulnerability scanning (Dependabot, Snyk, Trivy, npm audit) on every pull request and on a daily schedule against the main branch.
- Critical and high-severity dependency vulnerabilities are triaged within 48 hours. A vulnerability in a dependency that is reachable from application code is treated as an application vulnerability.
- Evaluate new dependencies before adoption: maintenance activity (last commit, release cadence), known vulnerability history, license compatibility, and transitive dependency count.
- Minimize the dependency tree. A dependency that provides one function used in one file is a candidate for replacement with application code.

### Secrets Management

- Scan all commits for secrets using pre-commit hooks (gitleaks, detect-secrets, truffleHog). Block commits that contain secrets.
- Store all secrets in a secrets manager (AWS Secrets Manager, HashiCorp Vault, GCP Secret Manager). Applications reference secrets by name, never by value in code.
- Rotate all secrets on a defined schedule: API keys (90 days), database credentials (90 days), signing keys (annually). Automated rotation is preferred; manual rotation is documented as a runbook.
- If a secret is leaked (committed to a repository, logged, or exposed in an error response), it is revoked and rotated immediately — not just removed from the visible location.
- Audit secret access patterns. A service that suddenly accesses secrets it has never accessed before is an anomaly that warrants investigation.

### Security Testing

- Integrate Static Application Security Testing (SAST) into the CI pipeline. SAST runs on every pull request and blocks merge on critical findings.
- Run Software Composition Analysis (SCA) alongside SAST to detect vulnerable dependencies.
- Conduct Dynamic Application Security Testing (DAST) against staging environments before promotion to production.
- Perform manual penetration testing annually or after major architectural changes.
- Maintain a vulnerability tracking system. Every finding has an owner, a severity, and a remediation deadline.

### Security Logging and Audit Trails

- Log all authentication events: successful logins, failed logins, password changes, MFA enrollment, session creation and destruction.
- Log all authorization failures: denied API requests, insufficient permission errors, access attempts to resources outside the user's scope.
- Log all administrative actions: configuration changes, user management, permission grants and revocations.
- Security logs are stored in a tamper-resistant system with minimum 1-year retention for compliance.
- Never log secrets, passwords, full credit card numbers, or other sensitive data. Log the event, not the value.

## Agent Instructions

When an LLM coding agent is operating under this meta-prompt:

**Do:**
- Use parameterized queries for all database access — never string interpolation
- Hash passwords with bcrypt (cost 12+), scrypt, or Argon2id
- Generate RBAC/ABAC permission checks at every API endpoint
- Set `HttpOnly`, `Secure`, and `SameSite` attributes on all session cookies
- Generate Content Security Policy headers on all web responses
- Validate all input against allowlist schemas (type, length, format, range)
- Apply context-appropriate output encoding (HTML, URL, JSON, shell)
- Reference secrets by environment variable name, never by value
- Include rate limiting on authentication endpoints and public APIs
- Generate security-relevant log entries for authentication and authorization events

**Do NOT:**
- Use MD5, SHA-1, or unsalted SHA-256 for password hashing
- Generate custom authentication protocols when OAuth 2.0/OIDC suffice
- Use string concatenation to build SQL queries, shell commands, or HTML output
- Log passwords, tokens, API keys, or other secret values
- Generate code that fails open — missing authentication defaults to denial
- Hardcode cryptographic keys, API keys, or credentials in source code
- Use `eval()`, `exec()`, or equivalent dynamic code execution with user-controlled input
- Trust client-side validation as a security control — always validate server-side
- Generate wildcard CORS policies (`Access-Control-Allow-Origin: *`) for authenticated endpoints

**Verify before proceeding:**
- Are all database queries parameterized?
- Are all passwords hashed with bcrypt/scrypt/Argon2id (not MD5/SHA)?
- Does every API endpoint check authorization before processing the request?
- Are all secrets referenced by name, not hardcoded in source?
- Is input validation applied at every trust boundary?
- Are security events (auth success/failure, access denied) logged?
- Does the system fail secure (deny by default) on unexpected states?

## Integration Points

- **LLM-Native Software Engineering:** Security requirements are defined in SPEC.md before build phases begin. Verification prompts for each phase include security review items from this meta-prompt's checklist. The AGENT.md directive includes security constraints as non-negotiable rules.

- **Deployment Engineering:** Deployment Engineering provisions the infrastructure security controls: network segmentation, firewall rules, non-root containers, image scanning. This meta-prompt defines the application-layer security requirements that operate within that infrastructure. Secrets management configuration is jointly owned: Deployment Engineering provisions the vault; this meta-prompt defines what goes in it and how it is rotated.

- **DevOps:** DevOps monitors security-relevant operational signals: anomalous access patterns, failed authentication spikes, unusual API call volumes. Security incident response follows DevOps incident management processes with security-specific runbooks. Security logging feeds into DevOps observability pipelines.

- **Database:** The Database meta-prompt defines per-application database users with minimum permissions, TLS for connections, and encryption at rest. This meta-prompt provides the threat model context for those decisions: which data is sensitive, what access patterns are expected, and what constitutes anomalous database access.

- **API Design:** The API Design meta-prompt defines endpoint structure and versioning. This meta-prompt defines the security controls applied to those endpoints: authentication requirements, authorization checks, input validation schemas, rate limits, and CORS policies. Every API endpoint inherits security requirements from this meta-prompt.

- **Testing Strategy:** Security testing (SAST, DAST, SCA, penetration testing) is coordinated with the Testing Strategy meta-prompt's overall test plan. Security test results feed into the same verification pipeline defined by Testing Strategy.

- **Documentation:** Security architecture decisions, threat models, and compliance mappings are documented following the Documentation meta-prompt's standards. Security runbooks follow the operational documentation patterns defined there.

- **UI-UX:** UI components handling authentication, authorization, and sensitive data require security review. This meta-prompt defines security requirements for forms (CSRF tokens, input validation), session management (cookie attributes), and output encoding; UI-UX implements those requirements in component design. Content Security Policy headers restrict what frontend code can load and execute.

- **MLOps:** ML systems introduce unique security concerns: model artifact integrity, training data access controls, adversarial input robustness, and PII handling in prediction logs. This meta-prompt defines the threat model for ML pipelines; MLOps implements the operational controls.

- **Scrum:** Security requirements are non-negotiable backlog items, not optional enhancements. Threat modeling sessions occur during sprint planning for features that introduce new attack surface. Security findings from SAST/DAST generate backlog items with priority determined by severity. Definition of Done for any feature touching authentication, data storage, or external APIs includes a security checklist review.

## Anti-Patterns

| Anti-Pattern | Why It's Harmful |
|---|---|
| Security as a pre-launch checklist | Architectural security flaws found at the end of development are exponentially more expensive to fix than flaws caught during design |
| Rolling custom authentication | Custom auth schemes lack the cryptographic review and battle-testing of established protocols; they introduce novel vulnerabilities that standard libraries have already solved |
| String concatenation for SQL/HTML/shell commands | The #1 cause of injection vulnerabilities; parameterized queries and context-aware encoding eliminate entire vulnerability classes |
| Logging sensitive data for debugging | Passwords and tokens in logs are accessible to anyone with log access; they persist in log aggregation systems indefinitely |
| Fail-open error handling | A try/catch that returns 200 OK when authorization fails grants access to every attacker who triggers the exception path |
| Trusting client-side validation as a security boundary | Client-side validation is a UX convenience; any attacker can bypass it by sending requests directly to the API |
| Shared database credentials across services | A compromise of one service grants the attacker access to all other services' data; blast radius is maximized |
| Pinning to version ranges instead of exact versions | A compromised patch release is automatically pulled into the next build; exact pinning forces explicit review of every version change |
| Secrets in environment files committed to git | Git history is permanent; even deleted files are recoverable; a secret committed once is compromised forever |
| Wildcard CORS on authenticated endpoints | Any website can make authenticated requests to the API, enabling CSRF-like attacks and data exfiltration |

## Quick-Start Checklist

```
[ ] Conduct STRIDE threat model for the system
[ ] Map all trust boundaries and entry points
[ ] Define security requirements in SPEC.md
[ ] Implement authentication using OAuth 2.0 / OIDC (not custom)
[ ] Hash all passwords with bcrypt (cost 12+) / scrypt / Argon2id
[ ] Implement RBAC/ABAC permission checks at every API endpoint
[ ] Validate all input at every trust boundary (allowlist schema)
[ ] Use parameterized queries for all database access
[ ] Apply context-appropriate output encoding (HTML, URL, JSON)
[ ] Set Content Security Policy headers on all web responses
[ ] Configure session cookies: HttpOnly, Secure, SameSite
[ ] Enable TLS 1.2+ for all data in transit
[ ] Encrypt all sensitive data at rest (AES-256)
[ ] Store all secrets in a secrets manager (not in code or env files)
[ ] Configure pre-commit secret scanning (gitleaks / detect-secrets)
[ ] Pin all dependencies to exact versions with lock files
[ ] Integrate SAST and SCA into CI pipeline
[ ] Rate-limit authentication endpoints and public APIs
[ ] Log all authentication and authorization events
[ ] Configure security log retention (minimum 1 year)
[ ] Schedule dependency vulnerability triage (within 48 hours for critical)
[ ] Document secret rotation schedule and automate where possible
[ ] Plan annual penetration test
```

## Version History

- v1.0 — 2026-03-13 — Initial publication: full meta-prompt with Principles, Practices, Agent Instructions, Integration Points, Anti-Patterns, and Quick-Start Checklist

---

*Part of the Meta-Prompt Ecosystem: https://github.com/pragnakar*
