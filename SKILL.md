---
name: token-saver
description: 项目记忆管理 / Project Memory Manager — 自动保存技术栈、架构决策、Bug、偏好、外部资源，支持搜索召回和对话联想。纯文件驱动，零外部依赖。Auto-save tech stack, decisions, bugs, preferences, references with search & auto-recall. File-driven, zero deps.
---

# 项目记忆管理 / Project Memory Manager

## 触发条件 / Triggers

### 触发关键词 / Trigger Keywords
**中文:** 项目记忆, 记住这个, 这个项目用了, 保存项目信息, 忘记这个记忆, 搜索记忆
**English:** project memory, remember this, save this, forget memory, search memory

### 命令 / Commands
- `/memory-search "<keyword>"` — 搜索项目记忆 / Search project memories
- `/memory-search "<keyword>" --global` — 搜索全局记忆 / Search global memories
- `/memory-search "<keyword>" --type <type>` — 按类型过滤 / Filter by type
- `/memory-list` — 列出项目记忆 / List project memories
- `/memory-list --global` — 列出全局记忆 / List global memories
- `/memory-list --all` — 包含 stale 条目 / Include stale entries
- `/memory-forget <id>` — 删除记忆 / Delete memory
- `/memory-forget <id> --keep` — 归档（保留文件）/ Archive (keep file)
- `/memory-open <id>` — 查看完整记忆 / View full memory

---

## 会话启动 / Session Startup

**每次对话开始时自动执行 / Auto-execute at start of every session:**

1. 检查项目记忆索引 / Check project memory index:
   - 读取 / Read `.claude/memory/MEMORY.md`
   - 如存在，静默加载索引（不逐条展开）/ If exists, silently load index
   - 如不存在，不提示 / If not, don't prompt

2. 检查全局记忆索引 / Check global memory index:
   - 读取 / Read `~/.claude-memory/MEMORY.md`
   - 如存在，静默加载 / If exists, silently load

---

## 自动初始化 / Auto-Initialization

**首次触发保存且目录不存在时 / When first save is triggered and directory doesn't exist:**

1. 创建目录结构 / Create directory structure:
```
.claude/memory/
  MEMORY.md
  memories/
```

2. 写入初始索引 / Write initial index:
```markdown
# Project Memory Index / 项目记忆索引

> Auto-managed, editable by hand. Grouped by type, stale entries excluded.
> 自动管理，可手工编辑。按类型分组，stale 条目不在此索引。

## tech-stack

## decision

## bug

## preference

## reference
```

3. 提示 / Notify: `📁 Initialized project memory directory / 已初始化项目记忆目录`

---

## 自动保存机制 / Auto-Save Mechanism

### 信号检测（全程监听） / Signal Detection (Continuous)

在对话中持续留意以下信号 / Watch for these signals during conversation:

| Signal / 信号 | Trigger pattern / 触发模式 | → Memory type / 类型 |
|---------------|--------------------------|---------------------|
| Tech stack declaration / 技术栈声明 | "This project uses / 这个项目用..." | `tech-stack` |
| Architecture decision / 架构决策 | "We chose X over Y because / 选了 X 而非 Y，因为..." | `decision` |
| Bug discovery / Bug 发现 | "There's a bug / known issue / 有个 Bug..." | `bug` |
| Project preference / 项目偏好 | "We use / standardize on / 这个项目统一用..." | `preference` |
| External resource / 外部资源 | "Docs at / API at / Figma / 文档在 / API 地址..." | `reference` |

**触发时机 / Trigger timing:** When a signal is detected, silently run the 3-gate filter before replying.
检测到信号时，回复用户之前先静默执行三道闸门过滤。

### 三道闸门 / Three-Gate Filter

**Gate 1 / 闸门 1 — Already known? / 是否已知？**
- Grep in `.claude/memory/memories/` for related keywords
- If identical memory exists (title + content highly similar) → skip / 跳过
- Not found / 未找到 → Gate 2 / 闸门 2

**Gate 2 / 闸门 2 — Conflicting update? / 是否有更新？**
- If same topic but different content (e.g. "MySQL 8.0" → "PostgreSQL 16")
- → Mark old memory `stale: true`, `updated: <today>`
- → Remove old entry from MEMORY.md index
- → Write new memory
- Completely new topic → Gate 3 / 闸门 3

**Gate 3 / 闸门 3 — Worth saving? / 是否值得存？**
- DO NOT save / 不存:
  - Temp debug logs, stack traces / 临时调试日志、报错堆栈
  - Dependencies readable from package.json/go.mod
  - Code snippets (code in git, memory stores WHY)
  - Session progress / to-do items (not project memory scope)
  - Trivial one-liners you won't need again
- Worth saving → execute write

### 写入操作 / Write Operation

1. 生成文件名 / Generate filename: `<type>-<slug>.md` (e.g. `decision-api-auth.md`)
2. 确保目录存在 / Ensure `.claude/memory/memories/` exists
3. 写入记忆文件 / Write memory file:

```markdown
---
name: <type>-<slug>
type: <type>
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
stale: false
tags: [<tag1>, <tag2>]
---

<Content: 2-5 bullet points, include rationale and constraints>
<正文：2-5 个要点，包含原因和约束>
```

4. 更新索引 / Update `.claude/memory/MEMORY.md`:
   - Add line under `## <type>` section
   - Format: `- [Title](memories/<filename>.md) — one-line description / 一句话描述`

5. 轻提示 / Brief notification: `📝 Saved / 已保存: [<type>] <title>`

### 保存后撤回 / Undo Save

如果用户说 "forget that / 忘记刚才那条" / "undo / 撤回":
- Delete the memory file just written
- Remove from MEMORY.md index
- Reply: `🗑️ Undone / 已撤回: [<type>] <title>`

---

## 自动召回（对话联想） / Auto-Recall (Conversational Association)

**核心原则 / Core principle:** When user asks about something covered by project memory, proactively reference it. No need for `/memory-search` first.
当用户提问涉及项目已有记忆时，主动关联，不需要用户先输入 `/memory-search`。

### 触发场景 / Trigger Scenarios

| Scenario / 场景 | Example question / 示例问题 | Check memory type / 检查类型 |
|-----------------|---------------------------|---------------------------|
| Tech/dependency / 技术选型 | "What database/ORM/framework? / 用什么数据库/框架？" | `tech-stack` |
| Architecture / 架构方案 | "Why did we choose X? / 为什么选 X？" | `decision` |
| Bug/troubleshooting / 问题排查 | "There's a bug / XX is broken / 有个 Bug" | `bug` |
| Convention / 规范风格 | "How do we write tests? / 测试怎么写？" | `preference` |
| External link / 外部资源 | "Where are the docs/designs? / 文档在哪？" | `reference` |

### 执行流程 / Execution Flow

1. **Detect / 检测** — Does the user's question relate to known project info?
2. **Match index / 匹配索引** — Check entries in pre-loaded MEMORY.md
3. **Read detail / 读详情** — If index summary is insufficient, read the full memory file
4. **Answer + cite / 联想回答** — Proactively reference memory with source attribution

### 回答格式 / Response Format

```
<Normal answer / 正常回答>

📎 Project memory / 参考项目记忆:
- [<type>] <title> (<date>)
```

### 示例 / Example

```
User:    What database are we using? / 我们后端数据库用的什么？

Claude:  Based on project memory, we use PostgreSQL 16 with Redis 7
         for caching, on Go 1.22 + Gin. / 根据项目记忆，后端 PostgreSQL
         16 + Redis 7 缓存，Go 1.22 + Gin。

         📎 Project memory:
         - [tech-stack] Go + SvelteKit Stack (2026-05-24)

---

User:    How should I build the login page? Any gotchas?
         登录页怎么做？有什么要注意的？

Claude:  Key points for login:
         1. Auth: JWT + refresh token (not session-based)
         2. Watch Safari < 17.4 flexbox compatibility
         3. ...

         📎 Project memory:
         - [decision] JWT Auth (2026-05-24)
         - [bug] Safari Login Flexbox (2026-05-24)
```

### 静默规则 / Silence Rules

- Memory answers the question → cite and answer / 引用并回答
- Memory irrelevant → don't force / 不强行引用
- Memory > 90 days old → flag `⚠️ may be outdated / 可能过时`
- New info conflicts with memory → trigger Gate 2 (update & archive)

---

## 搜索与召回 / Search & Recall

### /memory-search

When user types `/memory-search "<keyword>"`:

1. **Determine scope / 确定范围:**
   - Default: `.claude/memory/` (project memories)
   - `--global` → `~/.claude-memory/` (global memories)
   - `--type <type>` → filter by type only

2. **Search logic / 搜索逻辑 (plain text matching / 纯文本匹配):**
   - Read MEMORY.md index, match by title keywords
   - For matched candidates, read full memory files
   - Further match in body content
   - Also match `tags` in frontmatter
   - Exclude stale entries by default; `--all` includes them

3. **Output format / 展示格式:**
```
🔍 "<keyword>" — Found N memories / 找到 N 条记忆

1. [<type>] <title> (<created>)
   <first 1-2 lines / 正文前 1-2 行>

2. [<type>] <title> (<created>)
   <first 1-2 lines>

> /memory-open <id>  view full / 展开
> /memory-forget <id>  delete / 删除
```
No results: `🔍 "<keyword>" — No memories found / 未找到相关记忆`

### /memory-list

When user types `/memory-list`:

1. Read MEMORY.md index
2. Display grouped by type:
```
📋 Project Memory / 项目记忆 — N entries

## tech-stack
1. Title (date) — one-line description / 一句话描述

## decision
2. Title (date) — one-line description
```
3. Default: exclude stale; `--all` includes stale (marked ⚠️)
4. `--global` → list global memories

### /memory-open

When user types `/memory-open <id>`:

1. Look up memory by ID from last `/memory-list` or `/memory-search` result
2. Read full memory file, display frontmatter + body

### /memory-forget

When user types `/memory-forget <id>`:

1. Look up memory by ID
2. Show title for confirmation
3. `--keep` → set `stale: true`, remove from index, keep file
4. No `--keep` → delete file + remove from index
5. Reply: `🗑️ Deleted / 已删除: [<type>] <title>` or `📝 Archived / 已归档: [<type>] <title>`

### ID 映射 / ID Mapping

Temporary sequential IDs (1, 2, 3...) assigned in each `/memory-list` / `/memory-search` result. Map ID → filename, valid for current session. `/memory-open` and `/memory-forget` resolve via this mapping.

---

## 生命周期管理 / Lifecycle Management

### 过期检测 / Expiry Detection

1. **Check on read / 读取时检查:**
   - Check `created` / `updated` field
   - Over **90 days** → label `⚠️ 90+ days, may be outdated / 90+ 天前可能过时`
   - Can ask user if update is needed

2. **Conflict detection (auto-archive) / 冲突检测（自动归档）:**
   - New memory with same topic but contradictory content
   - Auto-mark old memory `stale: true`, remove from index
   - Notify: `📝 Saved (old memory auto-archived) / 已保存（旧记忆已自动归档）`

3. **Explicit staleness / 显式过时:**
   - User says "no longer using / 已经换了", "bug fixed / 修好了", "outdated / 过时了"
   - Immediately mark as stale, reply: `📝 Archived / 已归档: [<type>] <title>`

### stale 规则 / Stale Rules

- Keep original file, modify frontmatter: `stale: true`, `updated: <today>`
- Remove from MEMORY.md index
- Default: hidden from `/memory-list` and `/memory-search`
- `--all` flag includes stale entries (marked ⚠️)

---

## 输出规范 / Output Conventions

### 记忆类型标签 / Type Labels

| Type / 类型 | Display / 显示 |
|-------------|---------------|
| tech-stack | `[tech-stack]` |
| decision | `[decision]` |
| bug | `[bug]` |
| preference | `[preference]` |
| reference | `[reference]` |

### 静默操作 / Silent Operations (no user interruption / 不打断用户)

- Loading MEMORY.md index at session start
- Gate 1: skip duplicate → silent
- Gate 3: skip noise → silent

### 提示操作 / Notified Operations (one-liner / 一行提示)

- Save new: `📝 Saved / 已保存: [<type>] <title>`
- Save update: `📝 Saved / 已保存: [<type>] <title> (old archived / 旧记忆已归档)`
- Delete: `🗑️ Deleted / 已删除: [<type>] <title>`
- Init directory: `📁 Initialized / 已初始化 project memory directory`

---

## 文件路径参考 / File Path Reference

| Purpose / 用途 | Path / 路径 |
|---------------|------------|
| Project memory index / 项目记忆索引 | `<project_root>/.claude/memory/MEMORY.md` |
| Project memory entry / 项目记忆条目 | `<project_root>/.claude/memory/memories/<type>-<slug>.md` |
| Global memory index / 全局记忆索引 | `~/.claude-memory/MEMORY.md` |
| Global memory entry / 全局记忆条目 | `~/.claude-memory/memories/<type>-<slug>.md` |
