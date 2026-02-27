# using the imagination engine skill

this is an [OpenClaw](https://openclaw.ai) agent skill. it teaches an OpenClaw agent to generate complete, self-contained HTML5 apps from natural language descriptions. built for kids ages 5 to 10 on a touchscreen, but the output works in any browser.

## setup

you need OpenClaw installed. if you don't have it, see [openclaw.ai](https://openclaw.ai).

```bash
# install the skill into your workspace
mkdir -p ~/.openclaw/workspace/skills/kidblocks-engine
cp skill/kidblocks-engine/SKILL.md ~/.openclaw/workspace/skills/kidblocks-engine/
```

once the skill is in your workspace, your agent can read it on demand.

## usage

tell your OpenClaw agent to read the skill and generate something:

> "read the kidblocks-engine skill and make a cat maze game for a 7 year old"

the agent reads the skill, follows the patterns and safety rules, and returns JSON with the generated app.

### how KidBlocksOS uses it

on the actual device, the Electron shell sends structured requests to the local OpenClaw gateway through the Chat Completions API:

```
POST http://127.0.0.1:8089/v1/chat/completions

{
  "model": "openclaw:main",
  "messages": [
    {
      "role": "system",
      "content": "you are the KidBlocksOS Imagination Engine. read skills/kidblocks-engine/SKILL.md..."
    },
    {
      "role": "user",
      "content": "{\"description\": \"cat maze game\", \"studio\": \"games\", \"age\": 7}"
    }
  ]
}
```

the agent picks up the skill from its workspace, generates the app, and returns the result. kids never see any of this.

## input format

when sending structured requests (like the OS does), the user message is a JSON object:

```json
{
  "description": "what the child asked for",
  "studio": "games",
  "age": 7,
  "safety": "standard"
}
```

| field | required | notes |
|-------|----------|-------|
| description | yes | natural language, what to build |
| studio | no | games, stories, music, art, science, or tinker. defaults to games |
| age | no | 5 through 10. defaults to 7 |
| safety | no | strict, standard, or relaxed. defaults to standard |

you can also just talk to your agent naturally ("make me a piano for a 6 year old") and it will figure it out from the skill.

## output format

the skill tells the agent to return JSON only. no markdown, no explanation.

```json
{
  "html": "<!DOCTYPE html><html>...full app...</html>",
  "logic": {
    "things": [
      {
        "name": "Player",
        "icon": "🐧",
        "props": [
          { "label": "Speed", "type": "slider", "min": 1, "max": 10, "value": 5 }
        ]
      }
    ],
    "rules": [
      { "when": "Player touches a star", "then": "Score goes up by 1" },
      { "when": "All stars collected", "then": "You win!" }
    ]
  }
}
```

**html**: complete self-contained page. all CSS and JS inline. no external dependencies. no images. visuals use emoji, Canvas, or CSS. sound uses Web Audio API.

**logic**: visual representation of app behavior for the KidBlocks programming layer. if you don't need it, ignore it.

## studios

### games (Game Forge)
platformer, catcher, maze, whack-a-mole, pong, runner, memory, target practice

### stories (Story Weaver)
branching narrative, mad libs, comic strip, adventure

### music (Beat Lab)
piano, beat maker, sequencer, sound board, theremin, music box

### art (Canvas Magic)
freehand drawing, stamps, color mixer, pixel art, kaleidoscope, fireworks, pattern maker

### science (Lab Explorer)
solar system, ecosystem, weather sim, body explorer, chemistry mixer, dinosaur timeline, gravity sim

### tinker (Gadget Shop)
calculator, clock/timer, flashcards, fortune teller, dice roller, color palette, animations

## content safety

the skill has explicit rules about what never gets generated:

- violence, weapons, death
- sexual content
- drugs, alcohol
- bullying, hate speech
- horror, jump scares
- gambling mechanics

instead of refusing, the agent reinterprets. "gun game" becomes "water balloon launcher". "kill enemies" becomes "help friends". keeps the creative flow going while keeping the output safe.

## performance notes

the skill targets constrained hardware:

- keep DOM elements under 200 for games
- use requestAnimationFrame, not setInterval
- canvas games should clear and redraw each frame
- throttle touch events to 16ms minimum
- use CSS transform and opacity for animations (GPU accelerated)
- target 30fps

these constraints mean the generated apps are lightweight and run well on the Pi.

## testing changes

if you modify the skill:

1. install it in your OpenClaw workspace
2. ask your agent to generate across all six studios:
   - "make a penguin platformer" (games)
   - "tell me a story about a brave fox" (stories)
   - "I want a drum machine" (music)
   - "drawing app with rainbow brush" (art)
   - "show me the solar system" (science)
   - "build a calculator" (tinker)
3. check that output is valid JSON
4. open the HTML at 1024x600 in a browser
5. test touch (Chrome DevTools device mode)
6. check Network tab for external requests (there should be none)
7. try edge cases that should be caught by safety rules
