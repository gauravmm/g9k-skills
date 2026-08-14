# g9k-skills

Small, focused agent skills published in the standard `skills/<slug>/SKILL.md` format for use with the Skills ecosystem.

This repository currently includes:

- `codebase-simplification-review`, a skill for doing whole-codebase simplification reviews that prioritize deleting complexity, collapsing duplicate paths, and identifying unnecessary abstractions.
- `understand-unfamiliar-json`, a skill for inspecting unknown JSON with `jqi` before writing `jq` filters, parsers, or transformations.
- `ste-writing`, a skill for rewriting prose into ASD-STE100 Simplified Technical English, with a linter that scores the mechanical subset. By Ege Çelebi, from [`woosal1337/blog`](https://github.com/woosal1337/blog/tree/main/videos/ep01-the-cure-for-ai-slop) ("The Cure for AI Slop"), included here under its MIT license.
- `terse-comments`, a skill for writing and tightening comments and docstrings so they state the decision and only what the code cannot say. Written by me, and it layers on top of `ste-writing`.
- `python-project-scaffold`, a skill for imposing one shape on a Python project: uv, ruff, pyright in strict mode, pre-commit, GitHub CI, one self-check, and a CLAUDE.md that governs docs and comments.
- `typography`, a skill for setting type: scales, measure, leading, pairing, variable fonts, and font loading, with references for patterns, failure modes, and validations. From [`omer-metin/skills-for-antigravity`](https://github.com/omer-metin/skills-for-antigravity/tree/main/skills/typography), included here under its Apache 2.0 license, unchanged apart from an attribution line.
- `md2paper`, a skill for sending markdown to a Viwoods AiPaper e-ink tablet as a PDF set for its screen, bundling the `md2paper` script it documents.

## Install

Install the whole repository:

```bash
npx skills add gauravmm/g9k-skills
```

Install a specific skill in this repo:

```bash
npx skills add gauravmm/g9k-skills --skill codebase-simplification-review
npx skills add gauravmm/g9k-skills --skill understand-unfamiliar-json
npx skills add gauravmm/g9k-skills --skill ste-writing
npx skills add gauravmm/g9k-skills --skill terse-comments
npx skills add gauravmm/g9k-skills --skill python-project-scaffold
npx skills add gauravmm/g9k-skills --skill typography
npx skills add gauravmm/g9k-skills --skill md2paper
```

## License

MIT, Copyright (c) 2026 Gaurav Manek. The `ste-writing` skill is the work of Ege Çelebi and keeps its own MIT license and copyright, in `skills/ste-writing/LICENSE`. The `typography` skill comes from `omer-metin/skills-for-antigravity` and keeps its own Apache 2.0 license, in `skills/typography/LICENSE`.

## More specific skills

Some more involved and domain-specific skills I've developed live here:

- [`gauravmm/mcp_gateway_maker/`](https://github.com/gauravmm/mcp_gateway_maker/blob/master/README.md)
  An MCP security proxy and skill workflow for analyzing upstream MCP servers, logging their behavior, and generating filters or rewrite plugins to reduce risk. It is built for probing a server's security surface and then using targeted skills to propose mitigations, from simple YAML policies to content-aware plugins.
- [`gauravmm/HomeAssistant-pyscript-Conversion-Skill`](https://github.com/gauravmm/HomeAssistant-pyscript-Conversion-Skill)
  A Home Assistant conversion skill that helps turn YAML automations into `pyscript`, using clustering and AI-assisted grouping so related automations can be merged into cleaner Python files. It is aimed at larger or messier HA setups where manual one-by-one conversion would be slow and error-prone.
