---
name: agy-ox
description: Delegate a task to the local Google Antigravity CLI (agy) as a headless subagent named agy-ox. Use when the user says $agy-ox, /agy-ox, "have agy do X", "use agy-ox", "use agy", "delegate to Antigravity/Gemini", wants a second-model review/research, or wants agy to implement or fix code. Do not use for work the current ZCode agent should do itself.
---

# agy-ox

Run the machine-local `agy` CLI in official headless print mode. This is a thin Skill, not a native ZCode subagent and not a Codex `agent_role` / `agent_path` worker.

Do not wrap `agy` in a companion, job daemon, or extra Node script. Do not invent Codex or ZCode native subagent fields.

## Prerequisites

1. Confirm `agy --version` works in the same environment that will run the task.
2. If `agy` is missing, stop. Point the user at https://antigravity.google/docs/cli/install and wait for them to install and sign in. Do not install it yourself.
3. Headless mode uses cached credentials. A first interactive `agy` login is required on this machine.

## Invoke

```text
agy -p "<task>" --output-format json --mode <plan|accept-edits> --effort <low|medium|high>
```

Required every run:

- `-p` / `--print` / `--prompt` — the user task, passed through verbatim
- `--output-format json` — one JSON envelope on stdout when the run ends

Choose `--mode` from official execution modes:

| Task | `--mode` | Why |
| --- | --- | --- |
| Read-only: research, review, explain, outline, Q&A | `plan` | Read-only investigation; presents a plan instead of writing code |
| Implement, fix, edit, generate files | `accept-edits` | Auto-approves workspace file writes/creates |

Never use `default` in headless mode: it wants interactive diff review, which print mode cannot show.

Allowed optional flags (official only):

- `--effort low|medium|high` — reasoning effort. Use only these three values. Default `medium` if the user did not specify.
- `--model <id>` — pin a slug from `agy models`. Do not guess unknown ids.
- `--print-timeout <dur>` — default is `5m`. Use `15m` for real coding/review work unless the user set another limit.
- `--continue` / `-c` — continue the most recent agy conversation
- `--conversation <id>` — resume a specific `conversation_id` from a previous JSON result
- `--sandbox` — only when the user asked for terminal sandboxing
- `--dangerously-skip-permissions` — **never default this on**. Add it only when the user explicitly authorizes auto-approving every tool call

Do not pass `--permission-mode`. agy uses `--mode`.

## Permissions

Official split:

- `--mode` controls file-edit confirmation (`plan` / `accept-edits` / `default`).
- Tool permissions (`run_command`, URL, MCP, non-workspace files) are a separate engine.

In headless mode there is no approval prompt. A tool that would have asked is **soft-denied**: the process may still exit `0`, with a notice on stderr naming the tool and how to allow it.

Workspace file read/write is auto-allowed. Shell commands default to Ask, so they are soft-denied unless already allowed in `~/.gemini/antigravity-cli/settings.json`.

`--mode accept-edits` does **not** auto-approve `run_command`. If an implement/fix run is blocked on shell tools, report the stderr notice and ask whether to:

1. add a scoped `permissions.allow` rule, or
2. rerun with `--dangerously-skip-permissions` after explicit user approval

Prefer scoped allow rules. See https://antigravity.google/docs/cli/headless/ and https://antigravity.google/docs/cli/permissions/.

## Collect the result

`stdout` is the JSON envelope; diagnostics go to `stderr`. Parse stdout as JSON. Then check **all** of:

1. Process exit code
2. `status`
3. `error` (present on failure)
4. `response` (the deliverable text)

Treat the run as failed unless exit code is `0` **and** `status` is `SUCCESS`. Do not trust a nonempty `response` when `status` is `ERROR`, `CANCELED`, `INTERRUPTED`, `INVALID`, `WAITING`, or `RUNNING`.

On success, give the user `response`. Keep `conversation_id` for follow-ups. Mention duration/tokens only if asked.

On soft-deny (exit `0`, `SUCCESS`, but the work is incomplete because tools were denied), quote the stderr notice and do not pretend the task finished.

On failure, quote `error` (or stderr if JSON is missing) and stop. Do not silently redo the work in ZCode.

## Follow-up

Reuse the previous `conversation_id`:

```text
agy -p "<follow-up>" --output-format json --conversation <id> --mode <plan|accept-edits> --effort <low|medium|high>
```

`--continue` is only for the most recent agy conversation on this machine.

## Prompt rules

- Pass the user's task verbatim. Do not add persona templates unless the user asked.
- If the user authorized a costly or irreversible action (commit, push, paid API, deploy), keep that authorization in the prompt.
- For a long task, write it to a temp file and pass that file's contents as a single `-p` argument so shell quoting cannot split flags out of the task.
- After `accept-edits`, inspect `git status` / `git diff` before building on the result.

## Out of scope

- Do not start a background job system, companion CLI, or `stream-json` watcher.
- Do not fake ZCode plugin `agents/` as a native subagent (that field is not executed).
- Do not fake Codex `agent_role` / `agent_path`.
- Do not install or upgrade `agy` without the user doing it.
