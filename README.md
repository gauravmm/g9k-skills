# g9k-skills

Small, focused agent skills published in the standard `skills/<slug>/SKILL.md` format for use with the Skills ecosystem.

This repository currently includes:

- `codebase-simplification-review`, a skill for doing whole-codebase simplification reviews that prioritize deleting complexity, collapsing duplicate paths, and identifying unnecessary abstractions.
- `understand-unfamiliar-json`, a skill for inspecting unknown JSON with `jqi` before writing `jq` filters, parsers, or transformations.

## Install

Install the whole repository:

```bash
npx skills add gauravmm/g9k-skills
```

Install a specific skill in this repo:

```bash
npx skills add gauravmm/g9k-skills --skill codebase-simplification-review
npx skills add gauravmm/g9k-skills --skill understand-unfamiliar-json
```

## More specific skills

Some more involved and domain-specific skills I've developed live here:

- [`gauravmm/mcp_gateway_maker/`](https://github.com/gauravmm/mcp_gateway_maker/blob/master/README.md)
  An MCP security proxy and skill workflow for analyzing upstream MCP servers, logging their behavior, and generating filters or rewrite plugins to reduce risk. It is built for probing a server's security surface and then using targeted skills to propose mitigations, from simple YAML policies to content-aware plugins.
- [`gauravmm/HomeAssistant-pyscript-Conversion-Skill`](https://github.com/gauravmm/HomeAssistant-pyscript-Conversion-Skill)
  A Home Assistant conversion skill that helps turn YAML automations into `pyscript`, using clustering and AI-assisted grouping so related automations can be merged into cleaner Python files. It is aimed at larger or messier HA setups where manual one-by-one conversion would be slow and error-prone.
