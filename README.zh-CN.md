<p align="center"><img src="assets/logo.svg" width="420" alt="agy-ox — ZCode × Antigravity"></p>

<p align="center">
  把本机 Google Antigravity CLI 雇成 ZCode 的 headless 工人。
</p>

<p align="center">
  <a href="https://zcode.z.ai/docs/plugin"><img src="assets/badges/zcode-plugin.svg" alt="ZCode plugin"></a>
  <a href="https://antigravity.google/docs/cli/headless/"><img src="assets/badges/agy-headless.svg" alt="agy official headless CLI"></a>
  <a href="LICENSE"><img src="assets/badges/license-mit.svg" alt="license MIT"></a>
</p>

<p align="center"><a href="README.md">English</a> · <a href="README.zh-CN.md">简体中文</a></p>

Marketplace、插件、Skill 都叫 **agy-ox**。斜杠命令挂在同一个 Skill 上。

ZCode 加载一个 Skill 和一组命令，Agent 直接跑官方 `agy -p`。没有 companion、没有后台任务队列、没有多角色模板。

![ZCode 对话调用 $agy-ox，再跑官方 agy -p，最后核对 JSON](assets/flow.svg)

**[安装](#在-zcode-里一键导入) · [行为](#行为约定) · [目录](#目录)**

## 在 ZCode 里一键导入

![四步：打开工作区、添加 Yuuqq/agy-ox、安装 agy-ox、输入 $agy-ox](assets/install.svg)

ZCode 没有独立的 Skill 商店。Skill 要打成插件，再通过 marketplace 分发。本仓库根目录的 `marketplace.json` 就是这份目录。

1. 先打开任意工作区。
2. **设置 → 插件 → Discover → 右上角 `+` → Add marketplace**。
3. 填入：

```text
Yuuqq/agy-ox
```

   完整 URL `https://github.com/Yuuqq/agy-ox` 也可以。
4. 在 **agy-ox** marketplace 里，对 **agy-ox** 插件点 **Get / Install**。新安装的插件默认启用。

然后在对话里：

```text
/agy-ox-plan 审查当前工作区的 diff
```

```text
/agy-ox-fix 按我们刚说的范围把这个重试 bug 修掉
```

```text
/agy-ox-ask 这个仓库的登录流程是怎么走的？
```

```text
/agy-ox-continue 按你刚才的方案改
```

```text
/agy-ox-doctor
```

`$agy-ox` 和 `/agy-ox` 仍是通用入口，会按任务推断 `plan` 还是 `accept-edits`。输入 `/` 也能在 Skills 分组里找到它。

### 本地目录（不经过 GitHub）

**设置 → 插件 → Discover → `+`**，选择本仓库的本地路径。ZCode 会校验根目录的 `marketplace.json`。

## 行为约定

![只读用 plan，改文件用 accept-edits](assets/modes.svg)

| 命令 | 官方参数 | 说明 |
| --- | --- | --- |
| `/agy-ox-plan` | `--mode plan` | 只读调研或审查 |
| `/agy-ox-fix` | `--mode accept-edits` | 自动批准工作区内写文件 |
| `/agy-ox-ask` | `--mode plan --effort low` | 短问答，超时 2 分钟 |
| `/agy-ox-continue` | `--conversation <id>` | 续上一次 JSON 里的会话 |
| `/agy-ox-doctor` | 不发 `-p` | 检查 `agy --version` 和 `agy models` |
| `/agy-ox` / `$agy-ox` | 按任务推断 | 没说要改文件时用 `plan` |
| 推理强度 | `--effort low\|medium\|high` | 只允许这三个值 |
| 回收结果 | `--output-format json` | 必须同时检查退出码、`status`、`error`、`response` |

默认**不会**加 `--dangerously-skip-permissions`。`accept-edits` 不会自动放行 `run_command`，shell 仍走 [Permissions](https://antigravity.google/docs/cli/permissions/)。

headless **soft-deny**（退出码 `0`，stderr 点名被拦的工具）时，加一条收窄的 `permissions.allow`，不要跳过权限。短参考：[`permissions-allow.md`](plugins/agy-ox/skills/agy-ox/permissions-allow.md)。

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

写入 `~/.gemini/antigravity-cli/settings.json`（Windows 是 `%USERPROFILE%\.gemini\antigravity-cli\settings.json`），再重跑。Windows 上如果 `command(git)` 盖不住子命令，改用 `command(regex:git .*)`。

在 Windows 上，长任务会先写入临时文件再整段交给 `-p`，避免 PowerShell 把任务正文拆成额外参数。Skill 里写了 bash / PowerShell / cmd 模板。

Skill 依据的官方文档：

- [Using AGY CLI](https://antigravity.google/docs/cli/using/)
- [Headless mode](https://antigravity.google/docs/cli/headless/)
- [Choose an execution mode](https://antigravity.google/docs/cli/modes/)
- [Permissions](https://antigravity.google/docs/cli/permissions/)

## 前置条件

ZCode 所用的那台机器上已经安装并登录 `agy`：

```bash
agy --version
```

没有命令时，按 [Installation & Auth](https://antigravity.google/docs/cli/install) 自行安装。本插件不会代装。

## 目录

```text
.
├── marketplace.json                 # ZCode marketplace 清单（仓库根目录）
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
        └── permissions-allow.md # soft-deny 之后
```

插件里的 Skill 必须是扁平的 `skills/<name>/SKILL.md`，不能再套一层分组目录。

## 和 agy-staff 的差别

[agy-staff](https://github.com/keli-wen/agy-staff) 是给 Claude Code / Codex / Pi 用的完整插件：多角色、Node companion、后台 job。**agy-ox** 是给 ZCode 的薄路径：直接调官方 `agy -p`，解析 JSON。

## 许可证

MIT
