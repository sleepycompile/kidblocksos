<div align="center">

# 🧱 KidBlocksOS

**AI-powered creative tablet OS for kids ages 5-10**

*Children describe ideas. The system builds them. In seconds.*

[![Beta](https://img.shields.io/badge/status-beta-orange?style=flat-square)](https://github.com/sleepycompile/kidblocksos)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)
[![Raspberry Pi](https://img.shields.io/badge/platform-Raspberry%20Pi%205-c51a4a?style=flat-square&logo=raspberrypi&logoColor=white)](https://www.raspberrypi.com/)
[![OpenClaw](https://img.shields.io/badge/powered%20by-OpenClaw-8B5CF6?style=flat-square)](https://openclaw.ai)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat-square)](https://github.com/sleepycompile/kidblocksos/issues)

---

[**Skill**](skill/kidblocks-engine/SKILL.md) · [**Architecture**](docs/architecture.md) · [**Skill Guide**](docs/skill-guide.md) · [**Discussions**](https://github.com/sleepycompile/kidblocksos/discussions)

</div>

---

## What Is KidBlocksOS?

KidBlocksOS turns a **Raspberry Pi 5** with a 7" touchscreen into an AI-powered creative tablet designed exclusively for children.

A child opens a studio — Games, Stories, Music, Art, Science, or Tinker — and describes what they want to make. The system either matches it to one of **13 built-in templates** (instant, offline) or sends it to the **Imagination Engine** (an OpenClaw AI agent) which generates a complete interactive app from scratch.

The child plays their creation, sees how it works through a visual programming layer, and saves it to their library.

No app store. No ads. No tracking. No internet required for core features.

### The Pipeline

```
 "make a penguin game          Content Safety         Template Match?
  where I slide on ice"   ──►  Filter (3 layers)  ──►  Yes ──► Instant build
         │                                              No  ──► AI Generation
         │                                                        │
         ▼                                                        ▼
┌─────────────────┐                                  ┌──────────────────────┐
│ 🎤 Voice or ⌨️   │                                  │  Imagination Engine   │
│ Text Input      │                                  │  (OpenClaw Agent)     │
└─────────────────┘                                  │  + kidblocks-engine   │
                                                     │    skill              │
         ┌───────────────────────────────────────────┘
         ▼
┌─────────────────────────────────────────┐
│  Sandboxed iframe (allow-scripts only)  │
│  Complete HTML5 app — games, music,     │
│  art tools, science sims, stories       │
└─────────────────────────────────────────┘
```

## What's In This Repo

This is a **semi-open-source proof of concept**. We're open-sourcing the brain — the AI skill that powers generation — along with complete documentation of how the system works.

| What | Open Source? | Location |
|------|:-----------:|----------|
| **Imagination Engine Skill** | ✅ | [`skill/kidblocks-engine/SKILL.md`](skill/kidblocks-engine/SKILL.md) |
| **Architecture Documentation** | ✅ | [`docs/architecture.md`](docs/architecture.md) |
| **Skill Usage Guide** | ✅ | [`docs/skill-guide.md`](docs/skill-guide.md) |
| KidBlocksOS Shell (Electron) | ❌ | Proprietary |
| Pre-built OS Image | ❌ | Proprietary |
| Template Engine (13 patterns) | ❌ | Proprietary |

### Why This Model?

The skill is the creative intelligence. By open-sourcing it:

- **Educators and parents** can inspect exactly what the AI is instructed to do
- **Developers** can use the skill with OpenClaw or any LLM to generate kid-safe apps
- **The community** can improve patterns, safety rules, and age adaptation
- **Transparency** — the content safety rules are readable, auditable, and improvable

The OS shell is a hardware-specific integration. The skill works anywhere.

---

## The Imagination Engine Skill

The skill at [`skill/kidblocks-engine/SKILL.md`](skill/kidblocks-engine/SKILL.md) teaches an AI agent to generate complete, self-contained HTML5 apps. It covers:

### 6 Creative Studios — 40+ Patterns

<table>
<tr>
<td align="center" width="16%">

**🎮 Games**

Platformer
Catcher
Maze
Whack-a-mole
Pong
Runner
Memory
Target

</td>
<td align="center" width="16%">

**📖 Stories**

Branching narrative
Mad libs
Comic strip
Adventure

</td>
<td align="center" width="16%">

**🎵 Music**

Piano
Beat maker
Sequencer
Sound board
Theremin
Music box

</td>
<td align="center" width="16%">

**🎨 Art**

Freehand draw
Stamps
Color mixer
Pixel art
Kaleidoscope
Fireworks
Patterns

</td>
<td align="center" width="16%">

**🔬 Science**

Solar system
Ecosystem
Weather
Body explorer
Chemistry
Dinosaurs
Gravity

</td>
<td align="center" width="16%">

**🔧 Tinker**

Calculator
Clock/Timer
Flashcards
Fortune teller
Dice roller
Palette gen
Animations

</td>
</tr>
</table>

### Content Safety

Three-layer protection system:

| Layer | Where | What |
|-------|-------|------|
| **Client filter** | Before AI | Regex blocks violent/sexual/inappropriate terms |
| **Skill rules** | In the AI prompt | Explicit ban list + safe reinterpretations |
| **Sandbox** | After generation | `iframe sandbox="allow-scripts"` — no DOM escape |

Safe reinterpretation examples:

| Child says | AI generates |
|-----------|-------------|
| "kill the enemies" | "help the friends" |
| "gun game" | "water balloon launcher" |
| "scary monster" | "friendly monster who needs help" |
| "war" | "pillow fort battle" |

### Age-Adaptive

| Ages 5-6 | Ages 7-8 | Ages 9-10 |
|----------|----------|-----------|
| No fail states | Gentle progression | Real challenge |
| 1 choice per page | 2-3 choices | Complex branching |
| Very short sentences | Paragraphs | Longer narratives |
| Simple interactions | Multi-step | Strategy elements |

---

## Quick Start

### Use the Skill with OpenClaw

```bash
# Install into any OpenClaw agent
mkdir -p ~/.openclaw/workspace/skills/kidblocks-engine
cp skill/kidblocks-engine/SKILL.md ~/.openclaw/workspace/skills/kidblocks-engine/

# Tell your agent:
# "Read the kidblocks-engine skill. Make a dinosaur platformer for age 7."
```

### Use with Any LLM

```python
import anthropic, json

with open("skill/kidblocks-engine/SKILL.md") as f:
    skill = f.read()

client = anthropic.Anthropic()
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    system=skill,
    messages=[{
        "role": "user",
        "content": json.dumps({
            "description": "dinosaur jumping over volcanoes",
            "studio": "games",
            "age": 7,
            "safety": "standard"
        })
    }],
    max_tokens=8000
)

result = json.loads(response.content[0].text)

# result["html"]  → complete playable HTML5 app
# result["logic"] → visual programming data (things + rules)

with open("my-game.html", "w") as f:
    f.write(result["html"])
```

### Paste Into Any Chat UI

1. Copy the contents of [`SKILL.md`](skill/kidblocks-engine/SKILL.md)
2. Paste as the system prompt (or first message)
3. Ask: *"Make a piano app for a 6 year old"*
4. Save the `html` from the JSON response as a `.html` file
5. Open in any browser

See the full [**Skill Guide**](docs/skill-guide.md) for detailed usage.

---

## Architecture

> Full deep dive: [**docs/architecture.md**](docs/architecture.md)

### System Stack

```
┌─────────────────────────────────────────────────┐
│                 Raspberry Pi 5                   │
├─────────────────────────────────────────────────┤
│  Raspberry Pi OS Lite (Bookworm, arm64)          │
│  └── kidblocks.target (custom systemd target)    │
├─────────────────────────────────────────────────┤
│  Cage Compositor (Wayland)                       │
│  └── Electron (kiosk, context-isolated)          │
│      ├── Main: IPC, config, TTS, WiFi, gateway   │
│      └── Renderer: Studios, wizard, templates    │
│          └── Sandboxed iframe (generated apps)   │
├─────────────────────────────────────────────────┤
│  OpenClaw Gateway (localhost:8089)               │
│  └── Imagination Engine agent                    │
│      └── kidblocks-engine skill                  │
└─────────────────────────────────────────────────┘
```

### Security Hardening

| Layer | Protection |
|-------|-----------|
| **Process** | Dedicated `kidblocks` user, minimal group permissions |
| **SystemD** | `ProtectSystem=strict`, `NoNewPrivileges`, `ProtectKernelTunables` |
| **Electron** | `contextIsolation: true`, `nodeIntegration: false`, CSP headers |
| **Network** | Gateway on `127.0.0.1` only, random auth token, navigation blocked |
| **Storage** | Parent PIN hashed (scrypt), filesystem permissions |
| **Content** | 3-layer safety (regex → skill rules → iframe sandbox) |

### Parental Controls

- **PIN-gated settings** — hashed, not plaintext
- **Screen time limits** with configurable daily cap and break reminders
- **Bedtime enforcement** — device locks at scheduled time
- **Safety levels** — Strict (no AI), Standard (AI + filtering), Creative (9+)
- **Activity logging** — every AI interaction viewable in Guardian Report

---

## Hardware

### Minimum

- Raspberry Pi 5 (4GB RAM)
- 7" official touchscreen (1024×600)
- 32GB SD card
- Power supply (5V 5A USB-C)

### Recommended

- Raspberry Pi 5 (8GB RAM)
- 7" official touchscreen
- 64GB SD card
- 3D-printed case/stand
- WiFi connection (for AI features)
- Anthropic API key

Templates work completely offline. AI generation requires WiFi + API key.

---

## Roadmap

- [x] 6 creative studios with 40+ patterns
- [x] 13 offline templates
- [x] First-boot setup wizard (10 languages)
- [x] Screen time, bedtime, break reminders
- [x] Content safety (3 layers)
- [x] Parent PIN (hashed)
- [x] Activity logging + Guardian Report
- [x] Voice input (Web Speech API)
- [x] TTS output (ElevenLabs + fallback)
- [x] KidBlocks visual programming layer
- [ ] Guardian Channel via XMTP (parent notifications)
- [ ] OTA updates
- [ ] Accessibility (screen reader, high contrast, switch access)
- [ ] Community skill marketplace
- [ ] Multi-device sync
- [ ] Coding studio (visual → real code transition)

---

## Contributing

We welcome contributions to the Imagination Engine skill and documentation.

**Areas we need help:**

| Area | What |
|------|------|
| **New patterns** | Add game types, instruments, science sims |
| **Safety rules** | Better content filtering and reinterpretation |
| **Age adaptation** | Smarter difficulty scaling |
| **Localization** | Translate skill prompts and safety rules |
| **Performance** | Optimize generated HTML for constrained hardware |
| **Testing** | Test prompts across different LLMs |

### How to Contribute

1. Fork this repo
2. Edit the skill or docs
3. Test your changes (see [Skill Guide — Testing](docs/skill-guide.md#testing))
4. Open a PR with a clear description of what changed and why

### Discussions

Have ideas, questions, or want to show what you've built? Head to [**Discussions**](https://github.com/sleepycompile/kidblocksos/discussions).

---

## License

The Imagination Engine skill and documentation are released under the **[MIT License](LICENSE)**.

KidBlocksOS (the Electron shell, templates, and OS image) is proprietary.

---

<div align="center">

Built with 🧱 on Raspberry Pi · Powered by [OpenClaw](https://openclaw.ai)

**[Skill](skill/kidblocks-engine/SKILL.md)** · **[Architecture](docs/architecture.md)** · **[Guide](docs/skill-guide.md)** · **[Discussions](https://github.com/sleepycompile/kidblocksos/discussions)**

</div>
