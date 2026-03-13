# Security Engineering Meta-Prompt

A structured protocol for embedding security into AI-assisted software development — covering threat modeling, secure coding patterns, input validation, cryptography, dependency security, secrets management, and security testing.

## What Is a Meta-Prompt?

A meta-prompt is a structured protocol document that initializes an AI-assisted development environment before task execution begins. Instead of starting with an ad-hoc prompt, the meta-prompt establishes workflow structure, behavioral constraints, and quality standards so that subsequent prompts operate inside a disciplined, well-defined system.

## This Meta-Prompt

Security Engineering ensures that security is a structural property of the system from its first line of code, not a checklist applied before launch. It covers:

- **Threat Modeling** — STRIDE analysis, attack surface mapping, trust boundary identification
- **Authentication and Authorization** — OAuth 2.0/OIDC, password hashing (bcrypt/Argon2id), RBAC/ABAC, session management
- **Input Validation and Output Encoding** — injection prevention, XSS prevention, Content Security Policy, file upload validation
- **Cryptography** — TLS 1.2+, AES-256 encryption at rest, key management, HMAC verification
- **Dependency and Supply Chain Security** — lock files, vulnerability scanning, dependency evaluation
- **Secrets Management** — pre-commit scanning, secrets manager integration, rotation schedules, leak response
- **Security Testing** — SAST, DAST, SCA, penetration testing, vulnerability tracking
- **Security Logging** — authentication events, authorization failures, administrative actions, tamper-resistant audit trails

This meta-prompt is invoked when threat modeling should inform design, when authentication or authorization logic is being generated, when cryptographic decisions are being made, or when security audit findings need remediation.

## Part of the Meta-Prompt Ecosystem

This meta-prompt works alongside companion protocols:

| Meta-Prompt | Repository | Purpose |
|---|---|---|
| LLM-Native Software Engineering | https://github.com/pragnakar/LLM_NATIVE_SOFTWARE_ENGINEERING | Architecture and application development |
| Deployment Engineering | https://github.com/pragnakar/Deployment_Engineering | Infrastructure, containerization, and deployment |
| DevOps | https://github.com/pragnakar/DevOps | Runtime operations, observability, and reliability |
| Database | https://github.com/pragnakar/Database | Data persistence, schema design, and management |
| UI/UX | https://github.com/pragnakar/UI-UX | Interface and experience design |
| Security Engineering | https://github.com/pragnakar/Security_Engineering | Threat modeling, secure coding, and vulnerability management |
| MLOps | https://github.com/pragnakar/MLOps | Model lifecycle, training pipelines, and production ML |
| API Design | https://github.com/pragnakar/API_Design | Interface contracts, versioning, and developer experience |
| Testing Strategy | https://github.com/pragnakar/Testing-Strategy | Test architecture, coverage strategy, and quality verification |
| Documentation | https://github.com/pragnakar/Documentation | Technical writing, knowledge management, and developer docs |

## Usage

1. Load `Security_Engineering.md` into your AI coding agent when threat modeling, authentication, authorization, or security architecture work begins
2. The agent follows the Principles, Practices, and Agent Instructions to generate secure code: parameterized queries, proper password hashing, RBAC checks, input validation, and structured security logging
3. Use the Quick-Start Checklist to verify all security baselines before production deployment
4. Reference Integration Points to coordinate with Deployment Engineering (infrastructure security), DevOps (security monitoring), and Database (data access controls)

## License

MIT

## Author

Pragnakar Pedapenki
