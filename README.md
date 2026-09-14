<p align="center"><img src="assets/logo.svg" width="196" height="56" alt="agy-ox"></p>

<p align="center">
  <a href="README.md">English</a> ·
  <a href="README.zh-CN.md">简体中文</a>
</p>

<p align="center">
  <a href="https://zcode.z.ai/en/docs/plugin"><img src="assets/badges/zcode-plugin.svg" alt="ZCode plugin"></a>
  <a href="https://antigravity.google/docs/cli/headless/"><img src="assets/badges/agy-headless.svg" alt="agy official headless CLI"></a>
  <a href="LICENSE"><img src="assets/badges/license-mit.svg" alt="license MIT"></a>
</p>

# agy-ox

Hire the local [Google Antigravity CLI](https://antigravity.google/docs/cli/using/) (`agy`) as a **ZCode** headless worker.

Marketplace, plugin, skill, and slash command are all named **agy-ox**.

ZCode loads one Skill and one command. The Agent then runs official `agy -p` print mode. There is no companion process, job queue, or persona stack.

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
$agy-ox review the current working-tree diff
```

```text
/agy-ox implement the retry fix we just discussed
```

Typing `/` also lists the Skill under the Skills group.

### Local folder (no GitHub)

**Settings → Plugins → Discover → `+`**, choose this cloned directory. ZCode validates `marketplace.json` at the repo root.

## What it does

![plan for read-only work, accept-edits for file writes](assets/modes.svg)

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
├── marketplace.json           # ZCode marketplace catalog (repo root)
└── plugins/agy-ox/
    ├── .zcode-plugin/plugin.json
    ├── commands/agy-ox.md     # /agy-ox
    └── skills/agy-ox/SKILL.md # $agy-ox
```

Skills inside a plugin must stay in this flat `skills/<name>/SKILL.md` layout.

## Not agy-staff

[agy-staff](https://github.com/keli-wen/agy-staff) is a full Claude Code / Codex / Pi plugin with personas, a Node companion, and background jobs. **agy-ox** is the thin ZCode path: call official `agy -p` and parse JSON.

## License

MIT
