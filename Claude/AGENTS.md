# Claude sbx kit

Personal Docker Sandboxes (`sbx`) kit that starts Claude Code with:
- caveman plugin active by default, with its statusline badge
- plan mode at startup, auto as the default permission mode (bypass still reachable with Shift+Tab)
- `grill-me` + `grilling` skills (mattpocock/skills; `grill-me` only delegates to `grilling`)
- rtk installed and hooked into Claude Code
- caveman, skills, rtk and Claude itself updated on every start, before the agent launches

## Layout
```
spec.yaml                              # v2 kit, kind: sandbox, extends: claude
files/home/.claude/kit-settings.json   # our Claude settings, loaded via --settings (incl. git commit/push deny rules)
files/home/.claude/CLAUDE.md           # global agent rules: no commit/push, hand the commands to the user
files/home/.local/bin/claude-launch    # entrypoint: updates, then exec claude "$@"
```

## Workflow
- The kit is this directory (the one holding `spec.yaml`), wherever the repo was cloned or downloaded. Paths below are relative to it.
- Run from the kit's parent directory: `sbx run ./<kit-dir-name> <workspace>` (e.g. `sbx run ./Claude <workspace>`).
- `setup.install` and `files/` apply only at creation: recreate with `sbx rm <sandbox>` after changing them. Launcher changes only need an agent restart.

## Gotchas
- Schema v2 kits need sbx >= 0.36. sbx comes from the AUR package `docker-sbx` (update with `yay`).
- Local kit paths must start with `./` or `../`; an absolute path fails with "is not a sandbox or known agent".
- `sandbox.command` only replaces the arguments after the entrypoint binary, so the launcher must be `sandbox.entrypoint`; flags go in `command`.
- The default claude kit runs `claude --dangerously-skip-permissions`, which overrides `defaultMode`; plan mode comes from `--permission-mode plan`.
- `files/` copies drop the exec bit: `setup.install` runs `chmod 0755` on the launcher.
- `~/.claude.json`, `~/.claude/settings.json` and `~/.claude/.config.json` are managed by the claude kit and get overwritten: never ship them in `files/`. Our settings live in `kit-settings.json`, passed with `--settings`.
- `rtk init -g --auto-patch` writes its hook into the managed `settings.json`; it has survived so far. Check with `rtk init -g --show` and move the hook into `kit-settings.json` if it disappears.
- The launcher re-applies, before each start, what must live in the managed files:
  - `settings.json`: `statusLine` (caveman's SessionStart hook only looks there, otherwise it offers to add the badge every session) and `permissions.defaultMode: "auto"` (nested under `permissions`, not top-level; the kit sets `bypassPermissions` there, which is what triggers the "make auto mode your default?" offer).
  - `~/.claude.json`: `hasResetAutoModeOptInForDefaultOffer` and `hasSeenAutoModeEntryWarning`, the flags Claude writes when the offer is accepted (found by diffing after answering Yes).
- caveman's cache path includes its version (e.g. `.../caveman/3.1.0/src/hooks/`), so the statusline command locates `caveman-statusline.sh` with `find`.
- `~/.claude/CLAUDE.md` is not managed by the claude kit; `rtk init -g` (install step) appends its `@RTK.md` reference to our file since `files/` lands first.
- No commit/push is enforced twice: rules in `CLAUDE.md` (agent gives the commands instead) and `permissions.deny` in `kit-settings.json`, which blocks even in auto/bypass mode. Deny rules match command prefixes, so `git -C dir commit` slips past them; the instructions cover that.
- `setup.startup` does not gate the entrypoint: anything that must finish before the agent starts goes in the launcher.
- Network is default-deny: new download domains go in `permissions.network.allow`; find blocked ones with `sbx policy log` on the host.
- Never put secrets in the kit; use `sbx secret set` or the `credentials` block.

## Verify after a change
- `[CAVEMAN]` badge at startup, plan mode shown, no "add statusline" offer
- `/grill-me` available
- `rtk init -g --show` all `[ok]`, and `~/.claude/CLAUDE.md` holds both our rules and `@RTK.md`
- asking the agent to commit makes it print the commands instead
- `claude --version` is the latest
