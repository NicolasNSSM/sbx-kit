# sbx-kit

Kits for [Docker Sandboxes](https://docs.docker.com/ai/sandboxes/) (`sbx`): one directory per coding agent, each preconfigured with plugins, skills, tools and guardrails.

## Kits

| Kit | Agent | What it adds |
| --- | --- | --- |
| [`Claude/`](Claude/AGENTS.md) | Claude Code | caveman plugin, plan mode at startup, `grill-me` skill, rtk, no `git commit`/`git push`, auto-updates on start |

Each kit's `AGENTS.md` documents its layout, internals and gotchas.

## Requirements

- `sbx` >= 0.36 (schema v2 kits)

## Usage

Clone or download this repo, then from its root:

```bash
sbx run ./<kit> <workspace>    # e.g. sbx run ./Claude ~/Projects/my-app
```

The kit path must start with `./` or `../`.

`setup.install` and `files/` apply only when the sandbox is created. After changing them, recreate the sandbox with `sbx rm <sandbox>`.
