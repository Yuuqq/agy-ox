<p align="center"><img src="assets/logo.svg" width="420" alt="agy-ox — ZCode × Antigravity"></p>

<p align="center">
  Hire the local Google Antigravity CLI as a headless worker for ZCode.
</p>

<p align="center">
  <a href="https://zcode.z.ai/en/docs/plugin"><img src="assets/badges/zcode-plugin.svg" alt="ZCode plugin"></a>
  <a href="https://antigravity.google/docs/cli/headless/"><img src="assets/badges/agy-headless.svg" alt="agy official headless CLI"></a>
  <a href="LICENSE"><img src="assets/badges/license-mit.svg" alt="license MIT"></a>
</p>

<p align="center"><a href="README.md">English</a> · <a href="README.zh-CN.md">简体中文</a></p>

Marketplace, plugin, and Skill are named **agy-ox**. Slash commands hang off that same Skill.

ZCode loads one Skill plus several commands. The Agent then runs official `agy -p` print mode. There is no companion process, job queue, or persona stack.

![ZCode chat calls $agy-ox, which runs official agy -p, then checks JSON](assets/flow.svg)

**[Install](#one-click-import-in-zcode) · [Modes](#what-it-does) · [Layout](#layout)**

## One-click import in ZCode

![Four steps: open a workspace, add Yuuqq/agy-ox, install agy-ox, type $agy-ox](assets/install.svg)

ZCode has no standalone skill marketplace. Skills ship as a plugin catalog. This repo **is** that catalog.

1. Open a workspace in ZCode.
2. **Settings → Plugins → Discover → `+` → Add marketplace**.
3. Paste:

```text
Yuuqq/agy-ox
```

   GitHub URL `https://github.com/Yuuqq/agy-ox` also works.
4. Under the **agy-ox** marketplace, click **Get / Install** on the **agy-ox** plugin. Newly installed plugins are enabled by default.

Then in chat:

```text
/agy-ox-plan review the current working-tree diff
```

```text
/agy-ox-fix implement the retry fix we just discussed
```

```text
/agy-ox-ask what does the auth flow do?
```

```text
/agy-ox-continue apply that plan
```

```text
/agy-ox-doctor
```

`$agy-ox` and `/agy-ox` still work as a general entry that infers `plan` vs `accept-edits`. Typing `/` also lists the Skill under the Skills group.

### Local folder (no GitHub)

**Settings → Plugins → Discover → `+`**, choose this cloned directory. ZCode validates `marketplace.json` at the repo root.

## What it does

![plan for read-only work, accept-edits for file writes](assets/modes.svg)

| Command | Official flags | Notes |
| --- | --- | --- |
| `/agy-ox-plan` | `--mode plan` | Read-only research or review |
| `/agy-ox-fix` | `--mode accept-edits` | Auto-approves workspace file writes |
| `/agy-ox-ask` | `--mode plan --effort low` | Short question, 2m timeout |
| `/agy-ox-continue` | `--conversation <id>` | Resume the last JSON conversation |
| `/agy-ox-doctor` | no `-p` | Check `agy --version` and `agy models` |
| `/agy-ox` / `$agy-ox` | inferred | `plan` unless the user asked to edit files |
| Reasoning depth | `--effort low\|medium\|high` | Only these three values |
| Result | `--output-format json` | Agent must check exit code, `status`, `error`, `response` |

`--dangerously-skip-permissions` is **not** on by default. `accept-edits` does not auto-approve `run_command`; shell tools still follow [Permissions](https://antigravity.google/docs/cli/permissions/).

On a headless **soft-deny** (exit `0`, stderr names the tool), add a scoped `permissions.allow` rule. Do not skip permissions. Short reference: [`permissions-allow.md`](plugins/agy-ox/skills/agy-ox/permissions-allow.md).

```json
{
  "permissions": {
    "allow": [
      "command(git)",
      "command(npm test)"
    ]
  }
}
```

Write that into `~/.gemini/antigravity-cli/settings.json` (Windows: `%USERPROFILE%\.gemini\antigravity-cli\settings.json`), then rerun. On Windows, use `command(regex:git .*)` when `command(git)` does not cover subcommands.

On Windows, long prompts go through a temp file before `-p` so PowerShell quoting cannot split flags out of the task. The Skill has bash, PowerShell, and cmd templates.

Docs used by the Skill:

- [Using AGY CLI](https://antigravity.google/docs/cli/using/)
- [Headless mode](https://antigravity.google/docs/cli/headless/)
- [Choose an execution mode](https://antigravity.google/docs/cli/modes/)
- [Permissions](https://antigravity.google/docs/cli/permissions/)

## Prerequisite

Install and sign in `agy` on the same machine ZCode uses:

```bash
agy --version
```

If the binary is missing, follow [Installation & Auth](https://antigravity.google/docs/cli/install). The plugin will not install `agy` for you.

## Layout

```text
.
├── marketplace.json                 # ZCode marketplace catalog (repo root)
└── plugins/agy-ox/
    ├── .zcode-plugin/plugin.json
    ├── commands/
    │   ├── agy-ox.md            # /agy-ox
    │   ├── agy-ox-plan.md       # /agy-ox-plan
    │   ├── agy-ox-fix.md        # /agy-ox-fix
    │   ├── agy-ox-ask.md        # /agy-ox-ask
    │   ├── agy-ox-continue.md   # /agy-ox-continue
    │   └── agy-ox-doctor.md     # /agy-ox-doctor
    └── skills/agy-ox/
        ├── SKILL.md             # $agy-ox
        └── permissions-allow.md # after a soft-deny
```

Skills inside a plugin must stay in this flat `skills/<name>/SKILL.md` layout.

## Not agy-staff

[agy-staff](https://github.com/keli-wen/agy-staff) is a full Claude Code / Codex / Pi plugin with personas, a Node companion, and background jobs. **agy-ox** is the thin ZCode path: call official `agy -p` and parse JSON.

## License

MIT
