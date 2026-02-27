<div align="center">

# 🧱 KidBlocksOS

**a creative tablet OS for kids, powered by AI**

[![Status](https://img.shields.io/badge/status-proof%20of%20concept-orange?style=flat-square)](https://github.com/sleepycompile/kidblocksos)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)
[![Raspberry Pi](https://img.shields.io/badge/runs%20on-Raspberry%20Pi-c51a4a?style=flat-square&logo=raspberrypi&logoColor=white)](https://www.raspberrypi.com/)
[![OpenClaw](https://img.shields.io/badge/powered%20by-OpenClaw-8B5CF6?style=flat-square)](https://openclaw.ai)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat-square)](https://github.com/sleepycompile/kidblocksos/issues)

---

[**Skill**](skill/kidblocks-engine/SKILL.md) · [**Architecture**](docs/architecture.md) · [**Skill Guide**](docs/skill-guide.md) · [**Discussions**](https://github.com/sleepycompile/kidblocksos/discussions)

</div>

---

## the idea

kids are creative. they have ideas all day long. but most of the tools we hand them are just consumption devices. watch this, play that, scroll through whatever.

KidBlocksOS flips that. it's an OS where the only thing you can do is create.

a kid opens a studio, says what they want to build ("a penguin game where I jump on ice"), and the system builds it. either instantly from a template or through an AI agent that generates a full interactive app from scratch. the kid plays it, sees how it works through a visual programming layer, and saves it to their library.

no app store. no ads. no tracking. no internet required for the core experience.

a Raspberry Pi is the brain. [OpenClaw](https://openclaw.ai) is the agent framework that runs natively on the device, handling all AI generation through its local gateway. the OS ships with OpenClaw installed and a dedicated agent configured out of the box.

## what's in this repo

this is a proof of concept. we're releasing the brain (the OpenClaw skill that powers generation) and full documentation on how everything fits together.

| included | open source |
|----------|:-----------:|
| **Imagination Engine Skill** (OpenClaw) | ✅ |
| **Architecture docs** | ✅ |
| **Skill usage guide** | ✅ |
| OS shell and UI | no |
| pre-built image | no |
| template engine | no |

the skill is the interesting part. it's the full instruction set that teaches an OpenClaw agent to generate kid-safe interactive HTML5 apps from natural language. install it into any OpenClaw workspace and your agent knows how to build for kids.

we open sourced it because parents and educators should be able to read exactly what the AI is being told to do. and because the community can make it better.

---

## the imagination engine

the skill lives at [`skill/kidblocks-engine/SKILL.md`](skill/kidblocks-engine/SKILL.md). it's an OpenClaw agent skill that covers six creative studios with 40+ generation patterns.

<table>
<tr>
<td align="center" width="16%">

**🎮 Games**

platformer
catcher
maze
whack-a-mole
pong
runner
memory
target

</td>
<td align="center" width="16%">

**📖 Stories**

branching narrative
mad libs
comic strip
adventure

</td>
<td align="center" width="16%">

**🎵 Music**

piano
beat maker
sequencer
sound board
theremin
music box

</td>
<td align="center" width="16%">

**🎨 Art**

freehand draw
stamps
color mixer
pixel art
kaleidoscope
fireworks
patterns

</td>
<td align="center" width="16%">

**🔬 Science**

solar system
ecosystem
weather sim
body explorer
chemistry
dinosaurs
gravity

</td>
<td align="center" width="16%">

**🔧 Tinker**

calculator
clock/timer
flashcards
fortune teller
dice roller
palette gen
animations

</td>
</tr>
</table>

### content safety

three layers. not one.

| layer | where | what it does |
|-------|-------|-------------|
| **client filter** | before AI | regex catches violent, sexual, and inappropriate terms |
| **skill rules** | inside the OpenClaw agent prompt | explicit ban list plus safe reinterpretations |
| **sandbox** | after generation | iframe with `allow-scripts` only, no DOM escape possible |

the reinterpretation part matters. kids say stuff like "kill the enemies" because that's what they've seen in games. the AI doesn't refuse, it redirects:

| what the kid says | what gets built |
|-------------------|----------------|
| "kill the enemies" | "help the friends" |
| "gun game" | "water balloon launcher" |
| "scary monster" | "friendly monster who needs help" |
| "war" | "pillow fort battle" |

### age adaptive

the skill adjusts output based on the child's age.

| 5 to 6 | 7 to 8 | 9 to 10 |
|--------|--------|---------|
| no fail states | gentle progression | real challenge |
| one choice per page | two to three choices | complex branching |
| very short sentences | paragraph length | longer narratives |
| simple interactions | multi-step | strategy elements |

---

## using the skill

this is an OpenClaw skill. you need [OpenClaw](https://openclaw.ai) installed.

```bash
# install the skill into your OpenClaw workspace
mkdir -p ~/.openclaw/workspace/skills/kidblocks-engine
cp skill/kidblocks-engine/SKILL.md ~/.openclaw/workspace/skills/kidblocks-engine/

# then tell your agent:
# "read the kidblocks-engine skill and make a dinosaur platformer for age 7"
```

the agent reads the skill, follows the patterns and safety rules, and returns a JSON object with a complete HTML5 app and visual programming data.

on KidBlocksOS itself, the Electron shell talks to the local OpenClaw gateway through the Chat Completions API. the agent is preconfigured with the skill during first boot. kids never see any of this, they just describe what they want and it appears.

full details in the [skill guide](docs/skill-guide.md).

---

## how the system works

> full breakdown: [**docs/architecture.md**](docs/architecture.md)

```
kid says something
      |
      v
content safety filter (regex, blocks bad stuff)
      |
      v
template match? ----yes----> instant build (offline, no AI needed)
      |
      no
      |
      v
OpenClaw agent reads kidblocks-engine skill
      |
      v
agent generates complete HTML5 app
      |
      v
runs in sandboxed iframe (allow-scripts only)
      |
      v
kid plays their creation
```

a Raspberry Pi runs the whole thing. OpenClaw is installed natively and starts on boot as a systemd service. the gateway runs on localhost, never touches the open internet. 13 built-in templates cover the common cases without needing the agent at all.

### parental controls

- pin protected settings (hashed, not stored in plaintext)
- configurable screen time limits with break reminders
- bedtime enforcement that locks the device on schedule
- three safety levels: strict (no AI), standard (AI plus filtering), creative (9+)
- full activity log of every AI interaction, viewable behind the parent pin

### security

| what | how |
|------|-----|
| process isolation | dedicated user, minimal permissions |
| systemd hardening | ProtectSystem, NoNewPrivileges, ProtectKernelTunables |
| electron | context isolation, no node integration, CSP headers |
| network | OpenClaw gateway on localhost only, random auth token |
| storage | parent pin hashed with scrypt, filesystem permissions |
| generated content | sandboxed iframe, three layer content filter |

---

## status

proof of concept. actively being built.

- [x] six creative studios with 40+ patterns
- [x] 13 offline templates
- [x] OpenClaw agent with dedicated skill
- [x] first boot setup wizard (10 languages)
- [x] screen time, bedtime, break reminders
- [x] content safety (three layers)
- [x] parent pin (hashed)
- [x] activity logging and guardian report
- [x] voice input
- [x] TTS output
- [x] visual programming layer
- [ ] guardian channel via XMTP (parent notifications)
- [ ] OTA updates
- [ ] accessibility
- [ ] community skill marketplace
- [ ] multi-device sync

---

## contributing

we need help with:

- **new patterns** for the studios (game types, instruments, science sims)
- **safety rules** (better filtering, more reinterpretations)
- **age adaptation** (smarter difficulty scaling)
- **localization** (translating prompts and safety rules)
- **testing** across different OpenClaw configurations

see [CONTRIBUTING.md](CONTRIBUTING.md) for details.

---

## license

the imagination engine skill and all documentation are [MIT](LICENSE).

the OS shell, templates, and image are proprietary.

---

<div align="center">

built with 🧱 on Raspberry Pi · runs on [OpenClaw](https://openclaw.ai)

**[skill](skill/kidblocks-engine/SKILL.md)** · **[architecture](docs/architecture.md)** · **[guide](docs/skill-guide.md)** · **[discussions](https://github.com/sleepycompile/kidblocksos/discussions)**

</div>
