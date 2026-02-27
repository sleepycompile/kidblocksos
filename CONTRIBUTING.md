# contributing

this project is for children. everything here should reflect that.

## what you can work on

### the imagination engine skill

`skill/kidblocks-engine/SKILL.md` is the instruction set that teaches AI to generate kid-safe interactive apps. we need:

- new game patterns (puzzle types, educational games, more instruments)
- new science simulations (circuits, optics, biology)
- better safety reinterpretation rules
- age adaptation improvements
- localization (translate safety rules and prompts to other languages)
- performance hints for constrained hardware

### documentation

- fix errors, clarify confusing parts
- add examples
- improve the architecture docs

## how to test

before submitting anything, test your changes:

1. install the modified SKILL.md into your OpenClaw workspace
2. ask your agent to generate across all six studios
3. make sure output is valid JSON with html and logic fields
4. open the HTML in a browser at 1024x600
5. try touch interactions (Chrome DevTools device mode)
6. check that there are no external network requests
7. try edge cases that should be caught by safety rules

## submitting

1. fork the repo
2. make a branch
3. make your changes
4. test (see above)
5. open a PR with a clear explanation of what you changed and why

one concern per PR. don't mix pattern additions with safety rule changes. if your change touches content safety, explain the implications.

## questions

open a [discussion](https://github.com/sleepycompile/kidblocksos/discussions).
