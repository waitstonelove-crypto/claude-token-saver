# Claude Code 项目记忆插件

让 Claude Code 自动记住当前项目的技术栈、架构决策、已知 Bug、偏好和外部资源。纯文件驱动，零外部依赖。

## 安装

### 方式 1: Claude Code 插件市场（推荐）

```
/claude plugins install project-memory
```

### 方式 2: 手动安装

```bash
git clone https://github.com/<your-repo>/claude-project-memory.git
cp -r claude-project-memory ~/.claude/skills/project-memory/
```

### 方式 3: 项目内安装（团队共享）

将 `SKILL.md` 复制到项目 `.claude/skills/` 目录即可。

## 快速开始

### 初始化

在项目根目录启动 Claude Code 后，使用 `/project-memory` 命令激活插件。

Claude 会自动检查 `.claude/memory/MEMORY.md` 是否存在。首次使用时，当你说出第一条值得记住的信息时会自动初始化。

### 自动保存

不用刻意操作。当你在对话中说出以下内容时，Claude 自动识别并保存：

- "这个项目用 Go 1.22 + Gin 框架" → `[tech-stack]`
- "选了 JWT 认证而非 session，因为多服务器部署" → `[decision]`
- "Safari 下登录页布局有问题，还没修" → `[bug]`
- "测试统一用 vitest" → `[preference]`

保存后会有一行简短提示：`📝 已保存: [decision] JWT 认证方案`

### 搜索记忆

```
/memory-search "数据库"              → 搜索项目记忆
/memory-search "命名规范" --global   → 搜索全局记忆
/memory-search "API" --type decision → 按类型过滤
```

### 查看和删除

```
/memory-list              → 列出所有记忆
/memory-open 1            → 查看第 1 条完整内容
/memory-forget 1          → 删除第 1 条
/memory-forget 1 --keep   → 归档（保留文件但不显示）
```

## 记忆类型

| 类型 | 说明 | 示例 |
|------|------|------|
| `tech-stack` | 技术栈、依赖、运行环境 | Go 1.22 + Gin + PostgreSQL 16 |
| `decision` | 架构决策及原因 | 选 JWT 而非 session，因为多服务器 |
| `bug` | 已知 Bug 及上下文 | Safari 下登录页布局崩溃 |
| `preference` | 项目级偏好和规范 | 测试统一用 vitest |
| `reference` | 外部资源链接 | API 文档在 https://... |

## 存储位置

```
项目目录/.claude/memory/    ← 项目特定记忆（随 git 共享）
  MEMORY.md                 ← 索引文件
  memories/                 ← 独立记忆条目

~/.claude-memory/           ← 跨项目全局记忆
  MEMORY.md
  memories/
```

## 团队协作

`.claude/memory/` 目录是普通 markdown 文件，可直接提交到 git。团队成员 `git pull` 后即共享项目记忆。

## 自动过期

超过 90 天未更新的记忆，在搜索时会标注并提示。当新信息与旧信息冲突时，自动归档旧记忆。

## 不保存的内容

为保持记忆库整洁，以下内容不会自动保存：
- 代码片段（代码在 git 里，记忆只存 WHY）
- 临时调试日志和报错堆栈
- 从 package.json / go.mod 能直接读出的依赖列表
- 会话进度和待办事项

## 许可

MIT
