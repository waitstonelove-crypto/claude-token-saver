#TOKEN SAVER/ Claude Code Project Memory Plugin / Claude Code 项目记忆插件

Let Claude Code automatically remember the current project's tech stack, architectural decisions, known bugs, preferences, and external resources. Pure file-driven, zero external dependencies.

让 Claude Code 自动记住当前项目的技术栈、架构决策、已知 Bug、偏好和外部资源。纯文件驱动，零外部依赖。

## Installation / 安装

### Method 1: Claude Code Plugin Marketplace (Recommended) / 方式 1: Claude Code 插件市场（推荐）

```
/claude plugins install token-saver
```

### Method 2: Manual Install / 方式 2: 手动安装

```bash
git clone https://github.com/<your-repo>/claude-token-saver.git
cp -r claude-token-saver ~/.claude/skills/token-saver/
```

### Method 3: In-Project Install (Team Sharing) / 方式 3: 项目内安装（团队共享）

Copy `SKILL.md` to the project's `.claude/skills/` directory.

将 `SKILL.md` 复制到项目 `.claude/skills/` 目录即可。

## Quick Start / 快速开始

### Initialize / 初始化

After starting Claude Code in the project root, activate the plugin with `/token-saver`.

在项目根目录启动 Claude Code 后，使用 `/token-saver` 命令激活插件。

Claude will automatically check if `.claude/memory/MEMORY.md` exists. On first use, it auto-initializes the first time you say something worth remembering.

Claude 会自动检查 `.claude/memory/MEMORY.md` 是否存在。首次使用时，当你说出第一条值得记住的信息时会自动初始化。

### Auto-Save / 自动保存

No deliberate action needed. Claude automatically recognizes and saves when you mention things like:

不用刻意操作。当你在对话中说出以下内容时，Claude 自动识别并保存：

- "This project uses Go 1.22 + Gin framework" → `[tech-stack]`
- "Chose JWT auth over sessions because of multi-server deployment" → `[decision]`
- "Login page layout is broken on Safari, haven't fixed it yet" → `[bug]`
- "All tests use vitest" → `[preference]`

A brief notification will appear after saving: `📝 Saved: [decision] JWT auth solution`

保存后会有一行简短提示：`📝 已保存: [decision] JWT 认证方案`

### Search Memory / 搜索记忆

```
/memory-search "database"              → Search project memory
/memory-search "convention" --global   → Search global memory
/memory-search "API" --type decision   → Filter by type
```

```
/memory-search "数据库"              → 搜索项目记忆
/memory-search "命名规范" --global   → 搜索全局记忆
/memory-search "API" --type decision → 按类型过滤
```

### View & Delete / 查看和删除

```
/memory-list              → List all memories
/memory-open 1            → View full content of entry 1
/memory-forget 1          → Delete entry 1
/memory-forget 1 --keep   → Archive (keep file, hide from display)
```

```
/memory-list              → 列出所有记忆
/memory-open 1            → 查看第 1 条完整内容
/memory-forget 1          → 删除第 1 条
/memory-forget 1 --keep   → 归档（保留文件但不显示）
```

## Memory Types / 记忆类型

| Type / 类型 | Description / 说明 | Example / 示例 |
|-------------|-------------------|----------------|
| `tech-stack` | Tech stack, dependencies, runtime / 技术栈、依赖、运行环境 | Go 1.22 + Gin + PostgreSQL 16 |
| `decision` | Architectural decisions & rationale / 架构决策及原因 | JWT over sessions for multi-server / 选 JWT 而非 session，因为多服务器 |
| `bug` | Known bugs & context / 已知 Bug 及上下文 | Login layout broken on Safari / Safari 下登录页布局崩溃 |
| `preference` | Project-level conventions / 项目级偏好和规范 | Tests unified with vitest / 测试统一用 vitest |
| `reference` | External resource links / 外部资源链接 | API docs at https://... / API 文档在 https://... |

## Storage Location / 存储位置

```
<project>/.claude/memory/    ← Project-specific memory (shared via git)
  MEMORY.md                  ← Index file / 索引文件
  memories/                  ← Individual memory entries / 独立记忆条目

~/.claude-memory/            ← Cross-project global memory / 跨项目全局记忆
  MEMORY.md
  memories/
```

## Team Collaboration / 团队协作

The `.claude/memory/` directory contains plain markdown files, ready for git. Team members get shared project memory after `git pull`.

`.claude/memory/` 目录是普通 markdown 文件，可直接提交到 git。团队成员 `git pull` 后即共享项目记忆。

## Auto-Expiry / 自动过期

Memories not updated for over 90 days are flagged and prompted during search. When new information conflicts with old, the old memory is automatically archived.

超过 90 天未更新的记忆，在搜索时会标注并提示。当新信息与旧信息冲突时，自动归档旧记忆。

## What's NOT Saved / 不保存的内容

To keep the memory library clean, the following are not auto-saved:
- Code snippets (code lives in git; memory stores only the WHY)
- Temporary debug logs and error stack traces
- Dependency lists directly readable from package.json / go.mod
- Session progress and to-do items

为保持记忆库整洁，以下内容不会自动保存：
- 代码片段（代码在 git 里，记忆只存 WHY）
- 临时调试日志和报错堆栈
- 从 package.json / go.mod 能直接读出的依赖列表
- 会话进度和待办事项

## License / 许可

MIT
