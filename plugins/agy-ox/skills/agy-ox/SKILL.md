---
name: agy-ox
description: Delegate a task to the local Google Antigravity CLI (agy) as a headless worker named agy-ox. Use when the user says $agy-ox, /agy-ox, /agy-ox-plan, /agy-ox-fix, /agy-ox-ask, /agy-ox-continue, /agy-ox-doctor, "have agy do X", "use agy-ox", "use agy", or wants Gemini/Antigravity to review, research, implement, continue, or check the local agy CLI. Do not use for work the current ZCode agent should do itself.
---

# agy-ox

Run the machine-local `agy` CLI in official headless print mode. This is a thin Skill, not a native ZCode subagent and not a Codex `agent_role` / `agent_path` worker.

Do not wrap `agy` in a companion, job daemon, or extra Node script. Do not invent Codex or ZCode native subagent fields.

## Command routing

If a slash command selected this skill, honor it. Do not infer a different mode.

| Command | `--mode` | `--effort` | `--print-timeout` | Notes |
| --- | --- | --- | --- | --- |
| `/agy-ox-plan` | `plan` | `medium` | `15m` | Read-only research, review, outline |
| `/agy-ox-fix` | `accept-edits` | `medium` | `15m` | Implement or fix files |
| `/agy-ox-ask` | `plan` | `low` | `2m` | Short question, no file edits |
| `/agy-ox-continue` | inherit, or pick from the follow-up | inherit unless the user set one | `15m` | Resume with `--conversation` |
| `/agy-ox-doctor` | — | — | — | Do not run `-p`. See Doctor |
| `/agy-ox` or `$agy-ox` | infer from the task | `medium` unless set | `15m` for real work, `2m` for a one-liner | `plan` unless the user asked to edit files |

User-specified `--effort` / `--model` / `--print-timeout` always win. `--effort` may only be `low`, `medium`, or `high`.

## Doctor

When the command is `/agy-ox-doctor`, or the user asks whether agy is installed:

1. Run `agy --version`.
2. If the binary is missing, stop. Point at https://antigravity.google/docs/cli/install and wait. Do not install it.
3. Run `agy models`. A model list means cached login works. An auth error means they must run interactive `agy` once.
4. Report a short table: binary path or command name, version, login (`ok` / `needs login`), and one next step.

Do not send a `-p` prompt during doctor.

## Prerequisites (all other commands)

1. Confirm `agy --version` works in the same environment that will run the task. If unsure, run Doctor first.
2. If `agy` is missing, stop. Point the user at the install page. Do not install it yourself.
3. Headless mode uses cached credentials. A first interactive `agy` login is required on this machine.

## Invoke

```text
agy -p "<task>" --output-format json --mode <plan|accept-edits> --effort <low|medium|high> --print-timeout <dur>
```

Required every run except doctor:

- `-p` / `--print` / `--prompt` — the user task, passed through verbatim
- `--output-format json` — one JSON envelope on stdout when the run ends

Never use `default` in headless mode: it wants interactive diff review, which print mode cannot show.

Allowed optional flags (official only):

- `--model <id>` — pin a slug from `agy models`. Do not guess unknown ids
- `--continue` / `-c` — most recent agy conversation on this machine
- `--conversation <id>` — resume a specific `conversation_id`
- `--sandbox` — only when the user asked for terminal sandboxing
- `--dangerously-skip-permissions` — **never default this on**

Do not pass `--permission-mode`. agy uses `--mode`.

## Windows and quoting

Short tasks can go in `-p` directly. Long tasks, or any task with quotes, newlines, or leading `--`, must go through a temp file first so the shell cannot split flags out of the task.

Bash / Git Bash:

```bash
task_file="$(mktemp)"
printf '%s' "$TASK" > "$task_file"
agy -p "$(cat "$task_file")" --output-format json --mode plan --effort medium --print-timeout 15m
```

PowerShell:

```powershell
$taskFile = Join-Path $env:TEMP ("agy-ox-" + [guid]::NewGuid().ToString() + ".txt")
Set-Content -Path $taskFile -Value $task -Encoding utf8
agy -p (Get-Content -Raw -Encoding utf8 $taskFile) --output-format json --mode plan --effort medium --print-timeout 15m
```

cmd.exe:

```bat
agy -p "%TASK%" --output-format json --mode plan --effort medium --print-timeout 15m
```

On cmd, prefer the PowerShell temp-file path whenever the task is longer than one line. Do not wrap the whole argv in one quoted string that also contains flags.

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

On success, give the user `response`. Keep `conversation_id` for `/agy-ox-continue`. Mention duration/tokens only if asked.

On soft-deny (exit `0`, `SUCCESS`, but the work is incomplete because tools were denied), quote the stderr notice and do not pretend the task finished.

On failure, quote `error` (or stderr if JSON is missing) and stop. Do not silently redo the work in ZCode.

## Continue

When the command is `/agy-ox-continue`, or the user says "continue the agy-ox task":

1. Take the follow-up text verbatim.
2. Prefer `--conversation <id>` from the last successful JSON in this chat.
3. If no id is known, use `--continue` for the most recent agy conversation on this machine. Say that you are using the machine-global last conversation.
4. Pick `--mode` from the follow-up: edits → `accept-edits`, otherwise `plan`. If the user named a mode, use that.

```text
agy -p "<follow-up>" --output-format json --conversation <id> --mode <plan|accept-edits> --effort medium --print-timeout 15m
```

## Prompt rules

- Pass the user's task through verbatim. Do not add persona templates unless the user asked.
- If the user authorized a costly or irreversible action (commit, push, paid API, deploy), keep that authorization in the prompt.
- After `accept-edits`, inspect `git status` / `git diff` before building on the result.

## Out of scope

- Do not start a background job system, companion CLI, or `stream-json` watcher.
- Do not fake ZCode plugin `agents/` as a native subagent (that field is not executed).
- Do not fake Codex `agent_role` / `agent_path`.
- Do not install or upgrade `agy` without the user doing it.
