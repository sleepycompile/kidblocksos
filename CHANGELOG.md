# Changelog

All notable changes to the KidBlocksOS Imagination Engine skill and documentation.

## [0.1.0-beta.1] — 2026-02-27

### Added
- **Imagination Engine skill** (`kidblocks-engine/SKILL.md`)
  - 6 creative studios: Games, Stories, Music, Art, Science, Tinker
  - 40+ generation patterns across all studios
  - Content safety rules with safe reinterpretation
  - Age-adaptive difficulty (5-6, 7-8, 9-10)
  - Touch-first design requirements for 7" touchscreen
  - Web Audio API sound generation patterns
  - Performance constraints for Raspberry Pi 5
  - Structured JSON output format (HTML + visual logic)
- **Architecture documentation** (`docs/architecture.md`)
  - Full system stack breakdown
  - Boot sequence (first boot + normal boot)
  - Creation pipeline (input → safety → template/AI → sandbox)
  - Data model (projects, config, logs)
  - OpenClaw integration details
  - Security architecture and threat model
- **Skill usage guide** (`docs/skill-guide.md`)
  - OpenClaw integration
  - Raw LLM API usage (Python example)
  - Paste-into-chat usage
  - Input/output format reference
  - Testing guide

### Security
- Content safety: 3-layer protection (client filter → skill rules → iframe sandbox)
- Iframe sandbox: `allow-scripts` only (no `allow-same-origin`)
- PIN hashing with scrypt
- Random gateway token per device
- SystemD hardening (ProtectSystem, NoNewPrivileges)
