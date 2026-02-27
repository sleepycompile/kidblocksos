# architecture

how KidBlocksOS works from power on to creation.

## the stack

```
Raspberry Pi
├── Raspberry Pi OS Lite (Bookworm, arm64)
│   └── kidblocks.target (custom systemd boot target)
│
├── Cage (Wayland compositor)
│   └── Electron (kiosk mode)
│       ├── main process (node.js)
│       │   ├── IPC handlers (projects, config, TTS, wifi)
│       │   └── OpenClaw gateway client
│       └── renderer (chromium)
│           ├── OS UI (lock screen, home, studios, wizard, settings)
│           ├── template engine (13 patterns)
│           ├── content safety filter
│           └── sandboxed iframe (where generated apps run)
│
└── OpenClaw (installed natively, starts on boot)
    └── gateway (localhost:8089)
        └── imagination engine agent
            └── kidblocks-engine skill
```

there's no desktop environment. the Pi boots directly into the kiosk. the only thing on screen is KidBlocksOS.

OpenClaw runs as a native systemd service. it's not bolted on as an afterthought. the OS is built around it.

## boot sequence

### first boot (fresh install)

```
power on
  -> Pi OS boots to multi-user.target
  -> kidblocks-firstboot.service kicks in
    -> installs system packages (cage, fonts, audio libs)
    -> creates a dedicated kidblocks user
    -> installs Node.js 22
    -> runs npm install for Electron
    -> installs OpenClaw globally (npm install -g openclaw)
    -> configures OpenClaw workspace with kidblocks-engine skill
    -> drops in systemd service files
    -> sets kidblocks.target as default boot
    -> deletes itself
    -> reboots
```

### normal boot (after setup)

```
power on
  -> Pi OS boots to kidblocks.target
  -> kidblocks-gateway.service starts
    -> OpenClaw gateway comes online at localhost:8089
    -> agent loads with kidblocks-engine skill in workspace
  -> kidblocks-kiosk.service starts
    -> cage compositor grabs tty7
    -> electron launches in kiosk mode
    -> lock screen shows up
```

### first user session (setup wizard)

when the kid taps the lock screen and no profile exists yet, the wizard runs. two phases:

**parent zone (8 steps):**
1. "hand this to a grown-up"
2. language (10 options)
3. timezone
4. wifi setup
5. screen time limits and bedtime
6. content safety level
7. parent pin (4 digit, confirmed, hashed)
8. AI provider API key (configures the OpenClaw agent's provider)

**kid zone (4 steps):**
9. "now hand this to your kid!"
10. what's your name?
11. how old are you? (5 through 10)
12. pick your buddy (emoji avatar)

the API key step writes to the OpenClaw config and starts the gateway. the parent never has to know what OpenClaw is. they just paste a key and it works.

## the creation pipeline

what happens when a kid says "make a penguin game where I slide on ice":

```
"make a penguin game where I slide on ice"
  |
  +-- content safety filter (client-side regex)
  |   blocked? -> gentle redirect ("how about a superhero who saves everyone?")
  |   passed? -> continue
  |
  +-- template matching (engine.js matchIntent function)
  |   matched? -> generate HTML from template (instant, offline)
  |   no match + OpenClaw healthy? -> continue
  |   no match + OpenClaw down? -> "that one needs the AI brain! try connecting to wifi"
  |
  +-- OpenClaw generation
  |   electron main process POSTs to local OpenClaw gateway:
  |
  |   POST http://127.0.0.1:8089/v1/chat/completions
  |   {
  |     model: "openclaw:main",
  |     messages: [
  |       { role: "system", content: "read kidblocks-engine skill..." },
  |       { role: "user", content: '{"description":"penguin sliding...","studio":"games","age":7}' }
  |     ]
  |   }
  |
  |   agent reads SKILL.md from workspace, generates JSON with html + logic
  |
  +-- launch
      html goes into sandboxed iframe (allow-scripts only)
      logic goes into the KidBlocks visual programming drawer
      activity gets logged
      creation can be saved, modified, or rebuilt
```

## data layout

### projects (saved creations)

```
/opt/kidblocks/data/projects/{id}/
├── manifest.json    # name, studio, template, icon, timestamps
├── app.html         # the generated HTML
└── logic.json       # visual programming data (things + rules)
```

### configuration

```
/opt/kidblocks/data/config/
├── profile.json     # child's name, age, avatar
├── parent.json      # pin hash and salt (scrypt)
├── device.json      # language, timezone
├── timekeeper.json  # daily limit, break interval, bedtime, wake
├── safety.json      # level, blocked topics
├── engine.json      # gateway token, configured flag
├── tts.json         # ElevenLabs key, voice choice
├── wifi.json        # ssid, password
└── usage-YYYY-MM-DD.json  # daily screen time tracking
```

### OpenClaw config

```
/opt/kidblocks/openclaw-config/
├── openclaw.json           # gateway config, provider settings
├── workspace/
│   ├── AGENTS.md           # agent identity and behavior
│   └── skills/
│       └── kidblocks-engine/
│           └── SKILL.md    # the imagination engine skill
```

### logs

```
/opt/kidblocks/data/logs/
├── activity-YYYY-MM-DD.jsonl   # everything (creations, voice, sessions)
├── first-boot.log              # first boot setup output
└── electron.log                # electron stderr
```

## the OpenClaw agent

KidBlocksOS runs its own OpenClaw instance. this is not the host system's agent. it's a separate installation with its own config, workspace, and purpose.

| property | value |
|----------|-------|
| config home | /opt/kidblocks/openclaw-config |
| gateway port | 8089 |
| binding | 127.0.0.1 only |
| auth | random token generated per device |
| agent | single agent ("main") |
| skill | kidblocks-engine |
| model | whatever provider the parent configured |
| workspace | /opt/kidblocks/openclaw-config/workspace |

the Electron app talks to it through the Chat Completions API endpoint that OpenClaw exposes. the agent reads the skill from its workspace, follows the patterns, and returns structured JSON. it has no extra tools enabled, no file access beyond the workspace, no internet browsing. just the skill and the model.

this is what makes the whole thing work. OpenClaw handles the agent lifecycle, the gateway, the auth, the model routing. the OS just sends prompts and gets back apps.

## security model

### who we're protecting against

this is a children's device. the threats that matter:

1. **bad content** - kid types something the AI shouldn't generate
2. **sandbox escape** - generated HTML tries to reach the OS
3. **credential theft** - API keys or pins get exposed
4. **physical access** - someone plugs in a keyboard or SSHes in
5. **network attacks** - malicious wifi, MITM

### how each is handled

| threat | mitigation |
|--------|-----------|
| bad content | three layer filter: regex -> skill rules -> sandbox |
| sandbox escape | iframe sandbox="allow-scripts" (no allow-same-origin) |
| credential theft | hashed pin (scrypt), random gateway token, file permissions |
| physical access | kiosk blocks shortcuts, pin gates settings |
| network attacks | OpenClaw gateway on localhost only, electron blocks navigation |

### electron hardening

```javascript
contextIsolation: true
nodeIntegration: false

mainWindow.webContents.on('will-navigate', (e) => e.preventDefault());
mainWindow.webContents.setWindowOpenHandler(() => ({ action: 'deny' }));

// content security policy
"default-src 'self' 'unsafe-inline' 'unsafe-eval' data: blob: file:;
 connect-src 'self' http://127.0.0.1:* https://api.elevenlabs.io"
```

### systemd hardening

```ini
ProtectSystem=strict
ReadWritePaths=/opt/kidblocks/data
ProtectHome=yes
NoNewPrivileges=yes
ProtectKernelTunables=yes
ProtectKernelModules=yes
ProtectControlGroups=yes
```

## template engine

13 built-in templates handle common requests without touching OpenClaw.

the matching works through regex on the input text. "penguin sliding on ice" matches the platformer template because it detects a character (penguin emoji) plus a game-like context. the template function takes the extracted parameters and returns a complete HTML page.

```
input: "catch falling pizza"
  -> regex: /catch|fall|basket|collect/ -> catcher template
  -> regex: /pizza/ -> items = pizza emoji set
  -> TEMPLATES.catcher({ catcherEmoji: '🧺', items: '🍕,🍕,🧀,🍅,🫓', speed: 3 })
  -> complete HTML5 game
```

each template also has a logic generator that produces the visual programming view:

```javascript
LOGIC_GEN.catcher = (params) => ({
  things: [
    { name: 'Basket', icon: '🧺', props: [{ label: 'Speed', type: 'slider' }] }
  ],
  rules: [
    { when: 'item falls into basket', then: 'score goes up' },
    { when: 'item hits the ground', then: 'missed +1' }
  ]
});
```

## KidBlocks visual programming

the "Inside" button on any creation opens a drawer that shows kids how their app works. it uses two concepts:

**things** - characters, objects, and their adjustable properties. sliders change values in real time by sending postMessage to the iframe.

**rules** - when/then pairs written in plain language. "when the cat catches a fish, score goes up by 1."

this is not drag-and-drop coding. it's a transparency layer. kids see the logic behind what they built and can tweak properties without understanding code.

## voice

input: Web Speech API (SpeechRecognition). respects the language setting from the wizard.

output: ElevenLabs API if configured (cached to disk by content hash). falls back to Web Speech API SpeechSynthesis.

## screen time (TimeKeeper)

ticks every 30 seconds once a session starts:

- checks daily limit (warns at 5 min and 2 min remaining, locks at zero)
- checks break interval (nudges the kid to stretch)
- checks bedtime (locks the entire device with a dark sleep screen)
- saves usage to disk per day

## guardian channel (planned)

not implemented yet. the plan is XMTP-based parental notifications:

- what the child built
- content flags (blocked inputs)
- screen time milestones
- bedtime enforcement triggers
- daily summary

parent commands coming later: extend time, lock now, adjust safety level remotely.

currently everything logs locally and is viewable in the settings guardian report.
