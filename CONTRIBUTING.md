# Contributing to KidBlocksOS

Thank you for your interest in making KidBlocksOS better for kids everywhere.

## What You Can Contribute To

This repo contains the **Imagination Engine skill** and **documentation**. Contributions to both are welcome.

### Imagination Engine Skill

The skill at `skill/kidblocks-engine/SKILL.md` is the instruction set that teaches AI models to generate kid-safe interactive apps.

**We especially need:**
- New game patterns (puzzle types, educational games)
- New instrument patterns (guitar, drums, synthesizer)
- New science simulations (circuits, optics, biology)
- Better safety reinterpretation rules
- Age-adaptive difficulty improvements
- Localization (translate safety rules and prompts to other languages)
- Performance optimization hints for constrained hardware

### Documentation

- Fix errors or clarify confusing sections
- Add examples
- Improve the architecture documentation
- Add FAQ content

## How to Test Changes

Before submitting a PR, test your skill changes:

1. Copy the modified `SKILL.md`
2. Use it as a system prompt with any LLM (Claude, GPT-4, etc.)
3. Test with diverse prompts across all 6 studios
4. Verify the output is valid JSON with `html` and `logic` fields
5. Open the generated HTML in a browser at **1024×600** resolution
6. Test touch interactions (Chrome DevTools → device mode)
7. Verify **no external network requests** (check Network tab)
8. Verify content safety (try edge cases that should be reinterpreted)

## Submitting Changes

1. Fork the repository
2. Create a branch (`git checkout -b improve-game-patterns`)
3. Make your changes
4. Test thoroughly (see above)
5. Commit with a clear message
6. Open a Pull Request

### PR Guidelines

- **One concern per PR** — don't mix pattern additions with safety rule changes
- **Explain the "why"** — not just what you changed, but why it's better
- **Include test prompts** — show us what you tested and the results
- **Safety first** — if your change affects content safety, explain the implications

## Code of Conduct

This project is for children. All contributions and interactions must reflect that:

- Be kind and constructive
- Prioritize child safety above all else
- No content that would be inappropriate for the target audience (ages 5-10)
- Respect the work of others

## Questions?

Open a [Discussion](https://github.com/sleepycompile/kidblocksos/discussions) — we're happy to help.
