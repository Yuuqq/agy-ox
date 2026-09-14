# After a soft-deny: add `permissions.allow`

Headless `agy` has no approval prompt. A tool that would have asked is **soft-denied**: the process may still exit `0` with `status=SUCCESS`, while stderr names the tool and how to allow it. The task is incomplete.

Do **not** add `--dangerously-skip-permissions` unless the user explicitly asked for that one run.

## 1. Quote stderr, then stop

Show the notice. Do not pretend the task finished. Do not silently redo the work in ZCode.

## 2. Add only the denied scope

File:

- macOS / Linux: `~/.gemini/antigravity-cli/settings.json`
- Windows: `%USERPROFILE%\.gemini\antigravity-cli\settings.json`

Merge into an existing `permissions.allow` array. Do not replace unrelated rules.

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

`command(prefix)` matches word-by-word from the left. `command(git)` covers `git status` and `git diff`. `command(npm test)` covers `npm test` and `npm test --coverage`.

| Need | Rule |
| --- | --- |
| git family | `command(git)` |
| only status / diff | `command(git status)`, `command(git diff)` |
| npm test | `command(npm test)` |
| named npm scripts | `command(regex:npm run (build\|lint\|test))` |
| Windows git + subcommands | `command(regex:git .*)` |

On Windows PowerShell / cmd, a command that cannot be split into words needs an exact match unless you use `regex:`. Prefer `command(regex:git .*)` when `command(git)` does not cover subcommands.

## 3. Precedence

Official order is **deny > ask > allow**. If `command(*)` is in `ask` or `deny`, a narrower `allow` rule will not win. Do not add `command(*)` to `allow`.

Workspace file read/write is already auto-allowed. `--mode accept-edits` still does not auto-approve `run_command`.

## 4. Rerun

Use the same `-p` text. If this chat has a `conversation_id`, pass `--conversation <id>`. Still omit `--dangerously-skip-permissions`.

## Out of scope

- Do not install or edit `agy` itself.
- Do not invent actions outside official `action(target)` form.
- Full syntax: https://antigravity.google/docs/cli/permissions/
- Headless soft-deny: https://antigravity.google/docs/cli/headless/
