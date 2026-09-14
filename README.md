# agy-ox

Hire the local [Google Antigravity CLI](https://antigravity.google/docs/cli/using/) (`agy`) as a **ZCode** headless worker. The installed plugin and chat skill stay `$agy-worker`.

This is the thin path: ZCode loads one Skill and one slash command, then the Agent runs official `agy -p` print mode. There is no companion process, job queue, or persona stack.

[简体中文](README.zh-CN.md)

## One-click import in ZCode

ZCode has no standalone skill marketplace. Skills ship as a plugin catalog. This repo **is** that catalog.

1. Open a workspace in ZCode.
2. **Settings → Plugins → Discover → `+` → Add marketplace**.
3. Paste:

```text
Yuuqq/agy-ox
```

   GitHub URL `https://github.com/Yuuqq/agy-ox` also works.
4. Under the new **agy-ox** marketplace, click **Get / Install** on the `agy-worker` plugin. Newly installed plugins are enabled by default.

Then in chat:

```text
$agy-worker review the current working-tree diff
```

```text
/agy-worker implement the retry fix we just discussed
```

Typing `/` also lists the Skill under the Skills group.

### Local folder (no GitHub)

**Settings → Plugins → Discover → `+`**, choose this cloned directory. ZCode validates `marketplace.json` at the repo root.

## What it does

| User intent | Official flag | Notes |
| --- | --- | --- |
| Research, review, explain, plan | `--mode plan` | Read-only investigation |
| Implement / fix / edit files | `--mode accept-edits` | Auto-approves workspace file writes |
| Reasoning depth | `--effort low\|medium\|high` | Only these three values |
| Result | `--output-format json` | Agent must check exit code, `status`, `error`, `response` |

`--dangerously-skip-permissions` is **not** on by default. `accept-edits` does not auto-approve `run_command`; shell tools still follow [Permissions](https://antigravity.google/docs/cli/permissions/).

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
├── marketplace.json              # ZCode marketplace catalog (repo root)
└── plugins/agy-worker/
    ├── .zcode-plugin/plugin.json
    ├── commands/agy-worker.md    # /agy-worker
    └── skills/agy-worker/SKILL.md
```

Skills inside a plugin must stay in this flat `skills/<name>/SKILL.md` layout.

## Not agy-staff

[agy-staff](https://github.com/keli-wen/agy-staff) is a full Claude Code / Codex / Pi plugin with personas, a Node companion, and background jobs. This repo is the ZCode equivalent of a single Codex `agy-worker` Skill: call official `agy -p` and parse JSON.

## License

MIT
