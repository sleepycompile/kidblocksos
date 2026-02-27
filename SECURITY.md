# Security Policy

KidBlocksOS is designed for children. Security is not optional — it's foundational.

## Reporting Vulnerabilities

If you discover a security vulnerability, **please do not open a public issue.**

Email: **w2927864@gmail.com**

Include:
- Description of the vulnerability
- Steps to reproduce
- Potential impact (especially regarding child safety)
- Suggested fix (if you have one)

We will respond within 48 hours and work with you on a fix before any public disclosure.

## Scope

### In Scope

- **Content safety bypass** — any way to make the skill generate inappropriate content for children
- **Sandbox escape** — any way generated HTML could access the parent frame or OS
- **Credential exposure** — API keys, PINs, or tokens accessible where they shouldn't be
- **Parental control bypass** — circumventing PIN, screen time, bedtime, or safety levels

### Out of Scope

- Vulnerabilities in upstream dependencies (Electron, Node.js, Raspberry Pi OS) — report to those projects
- Physical access attacks (if someone has root on the Pi, all bets are off)
- Social engineering the parent into giving the child the PIN

## Security Model

See [docs/architecture.md — Security Architecture](docs/architecture.md#security-architecture) for the full threat model and mitigations.

### Key Principles

1. **Defense in depth** — three layers of content safety (client filter → skill rules → iframe sandbox)
2. **Least privilege** — dedicated user, systemd hardening, minimal permissions
3. **No trust in generated content** — iframe sandbox with `allow-scripts` only, no `allow-same-origin`
4. **Secrets are hashed** — parent PIN uses scrypt, gateway tokens are random
5. **Local-first** — gateway bound to localhost, no external network exposure

## Supported Versions

| Version | Supported |
|---------|-----------|
| 0.1.x (beta) | ✅ |

## Acknowledgments

We appreciate responsible disclosure and will credit security researchers (with permission) in our release notes.
