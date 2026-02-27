# Next.js 全栈开发者的 Claude Code 配置手册

> 基于 everything-claude-code 仓库，为 **Next.js 全栈开发者** 定制的取舍清单。
> 核心需求：持久化记忆、前端全栈工作流、不引入多余噪音。

---

## 目录

1. [配置原则](#1-配置原则)
2. [Rules — 要什么、不要什么](#2-rules)
3. [Agents — 要什么、不要什么](#3-agents)
4. [Commands — 要什么、不要什么](#4-commands)
5. [Skills — 要什么、不要什么](#5-skills)
6. [Hooks — 要什么、不要什么](#6-hooks)
7. [持久化记忆方案](#7-持久化记忆方案)
8. [MCP — 建议](#8-mcp)
9. [安装步骤（手动挑选法）](#9-安装步骤)
10. [你的 CLAUDE.md 模板](#10-claudemd-模板)

---

## 1. 配置原则

```
不装插件，只抄文件。
不全量安装，只挑需要的。
自己的配置体系优先，这个库只是参考。
```

你已经有自己的 Claude Code 配置（Stop Hook、Skills 等），所以：
- **不要** `claude plugin add` — 会注入 52 个 Skills + 33 个 Commands 污染你的环境
- **不要** `./install.sh` 全量安装 — 会覆盖你可能已有的 rules
- **要做的**：手动拷贝下面列出的文件，按需裁剪后放到你的 `~/.claude/` 下

---

## 2. Rules

Rules 是始终加载的规则文件，放在 `~/.claude/rules/` 下。

### 要 (10 个文件)

| 文件 | 来源 | 理由 |
|------|------|------|
| `common/coding-style.md` | `rules/common/` | 不可变性、文件大小控制（200-400 行）、错误处理 — 通用且有价值 |
| `common/security.md` | `rules/common/` | 安全检查清单（硬编码密钥/SQL注入/XSS/CSRF）— 必须有 |
| `common/testing.md` | `rules/common/` | TDD 流程、80% 覆盖率要求 |
| `common/git-workflow.md` | `rules/common/` | Conventional Commits 格式、PR 流程 |
| `common/development-workflow.md` | `rules/common/` | Plan → TDD → Review → Commit 完整流程 |
| `typescript/coding-style.md` | `rules/typescript/` | TS 不可变更新、Zod 验证、禁止 console.log（带 glob 条件加载） |
| `typescript/patterns.md` | `rules/typescript/` | ApiResponse 泛型、useDebounce Hook、Repository 模式 |
| `typescript/security.md` | `rules/typescript/` | TS 环境变量安全、密钥管理 |
| `typescript/testing.md` | `rules/typescript/` | Playwright E2E 指定 |
| `typescript/hooks.md` | `rules/typescript/` | PostToolUse 自动格式化、TypeScript 检查提示 |

### 不要 (19 个文件)

| 文件 | 理由 |
|------|------|
| `common/patterns.md` | 内容太泛（骨架项目、Repository 模式），TS 版本已覆盖 |
| `common/performance.md` | 模型选择策略你自己决定就行，不需要每次加载 |
| `common/hooks.md` | Hook 使用指南，看一遍就够了，不需要每次注入上下文 |
| `common/agents.md` | Agent 清单表，参考用，不需要每次加载浪费 token |
| `python/*` (5 个) | 你不写 Python |
| `golang/*` (5 个) | 你不写 Go |
| `swift/*` (5 个) | 你不写 Swift |

### 安装命令

```bash
# 手动拷贝（不用 install.sh）
mkdir -p ~/.claude/rules/common
mkdir -p ~/.claude/rules/typescript

# common（挑 5 个）
cp rules/common/coding-style.md ~/.claude/rules/common/
cp rules/common/security.md ~/.claude/rules/common/
cp rules/common/testing.md ~/.claude/rules/common/
cp rules/common/git-workflow.md ~/.claude/rules/common/
cp rules/common/development-workflow.md ~/.claude/rules/common/

# typescript（全部 5 个，都带 glob 条件加载）
cp rules/typescript/*.md ~/.claude/rules/typescript/
```

---

## 3. Agents

Agents 是可委派任务的子代理，放在 `~/.claude/agents/` 下。

### 要 (7 个)

| Agent | 模型 | 工具权限 | 理由 |
|-------|------|---------|------|
| `planner.md` | opus | Read/Grep/Glob（只读） | 复杂功能规划，独立上下文不污染主会话 |
| `code-reviewer.md` | sonnet | Read/Grep/Glob/Bash | 分级审查体系（CRITICAL→LOW），置信度过滤 |
| `tdd-guide.md` | sonnet | Read/Write/Edit/Bash/Grep/Glob | TDD 流程执行（RED→GREEN→REFACTOR） |
| `build-error-resolver.md` | sonnet | Read/Write/Edit/Bash/Grep/Glob | 构建错误快速修复，不做重构只修 bug |
| `security-reviewer.md` | opus | Read/Grep/Glob/Bash | 安全漏洞专项分析 |
| `e2e-runner.md` | sonnet | Read/Write/Edit/Bash/Grep/Glob | Playwright E2E 测试，Next.js 项目必备 |
| `refactor-cleaner.md` | sonnet | Read/Write/Edit/Bash/Grep/Glob | 死代码清理 |

### 建议裁剪

拷贝后建议做的修改：
- `code-reviewer.md` — 保留 Security、Code Quality、React/Next.js Patterns、Node.js/Backend Patterns 部分，可以删除你不需要的内容
- `e2e-runner.md` — 如果你不用 Agent Browser，可以删掉 Agent Browser 部分只留 Playwright

### 不要 (6 个)

| Agent | 理由 |
|-------|------|
| `architect.md` | planner 已覆盖架构规划，重复 |
| `go-build-resolver.md` | Go 专用 |
| `go-reviewer.md` | Go 专用 |
| `python-reviewer.md` | Python 专用 |
| `database-reviewer.md` | 内容好但太重（PostgreSQL 专项），用 postgres-patterns Skill 替代更轻量 |
| `doc-updater.md` | 文档更新用 Haiku 模型，ROI 不高，手动更新更可控 |

### 安装命令

```bash
mkdir -p ~/.claude/agents

cp agents/planner.md ~/.claude/agents/
cp agents/code-reviewer.md ~/.claude/agents/
cp agents/tdd-guide.md ~/.claude/agents/
cp agents/build-error-resolver.md ~/.claude/agents/
cp agents/security-reviewer.md ~/.claude/agents/
cp agents/e2e-runner.md ~/.claude/agents/
cp agents/refactor-cleaner.md ~/.claude/agents/
```

---

## 4. Commands

Commands 是 `/name` 触发的快捷 prompt，放在 `~/.claude/commands/` 下。

### 要 (8 个)

| Command | 做什么 | 理由 |
|---------|--------|------|
| `/plan` | 调用 planner agent 制定实现计划 | 复杂功能必备，确认后才动手 |
| `/tdd` | 调用 tdd-guide agent 执行 TDD | 先写测试再实现 |
| `/code-review` | 调用 code-reviewer agent 审查代码 | 每次改完代码后用 |
| `/build-fix` | 调用 build-error-resolver 修构建错误 | Next.js 构建报错时一键修复 |
| `/e2e` | 调用 e2e-runner agent 跑 E2E 测试 | Playwright 测试 |
| `/verify` | 运行完整验证循环（lint + type + test + build） | 提交前的最终检查 |
| `/test-coverage` | 检查测试覆盖率 | 确保 80%+ |
| `/refactor-clean` | 调用 refactor-cleaner 清理死代码 | 定期清理 |

### 可选 (4 个)

| Command | 理由 |
|---------|------|
| `/learn` | 手动从当前会话提取模式为 Skill — 如果你要用持续学习系统 |
| `/sessions` | 会话管理（加载/保存/列出）— 如果你要用会话持久化 |
| `/checkpoint` | 工作流检查点 — 长任务时有用 |
| `/skill-create` | 从 git 历史提取编码模式生成 Skill — 有趣但非必需 |

### 不要 (21 个)

| Command | 理由 |
|---------|------|
| `/go-build`, `/go-review`, `/go-test` | Go 专用 |
| `/python-review` | Python 专用 |
| `/update-docs`, `/update-codemaps` | 文档自动生成，过度工程化 |
| `/claw` | NanoClaw REPL，独立工具不是命令 |
| `/orchestrate` | 多 Agent 编排，高级用法暂不需要 |
| `/multi-*` (5 个) | 多 Agent 并行工作流，先精通单实例再说 |
| `/pm2` | PM2 进程管理，过度工程化 |
| `/setup-pm` | 包管理器设置，一次性操作 |
| `/eval`, `/learn-eval` | 评估系统，高级用法 |
| `/evolve` | Instinct 演化系统，高级用法 |
| `/instinct-*` (3 个) | Instinct v2 系统，高级用法 |

### 安装命令

```bash
mkdir -p ~/.claude/commands

cp commands/plan.md ~/.claude/commands/
cp commands/tdd.md ~/.claude/commands/
cp commands/code-review.md ~/.claude/commands/
cp commands/build-fix.md ~/.claude/commands/
cp commands/e2e.md ~/.claude/commands/
cp commands/verify.md ~/.claude/commands/
cp commands/test-coverage.md ~/.claude/commands/
cp commands/refactor-clean.md ~/.claude/commands/

# 可选
cp commands/sessions.md ~/.claude/commands/
```

---

## 5. Skills

Skills 是按需加载的知识包，放在 `~/.claude/skills/` 下。

### 要 (9 个)

| Skill | 大小 | 理由 |
|-------|------|------|
| `tdd-workflow/` | 410 行 | TDD 完整流程（RED→GREEN→REFACTOR）+ 测试模式模板 |
| `frontend-patterns/` | — | React/Next.js 组件模式、Hooks、状态管理 |
| `backend-patterns/` | — | API 设计、中间件、数据库操作模式 |
| `coding-standards/` | — | 通用编码标准（不可变性、命名、文件组织） |
| `e2e-testing/` | — | Playwright 详细指南（Page Object Model、选择器、CI 集成） |
| `security-review/` | — | 安全审查清单和流程 |
| `postgres-patterns/` | — | PostgreSQL 索引、RLS、查询优化速查表（你用 Supabase 就需要） |
| `strategic-compact/` | — | 战略性上下文压缩指南 + suggest-compact.js 脚本 |
| `verification-loop/` | — | lint → typecheck → test → build 完整验证循环 |

### 可选 (5 个)

| Skill | 理由 |
|-------|------|
| `continuous-learning/` | 持续学习 v1 — 如果你要自动提取会话模式 |
| `api-design/` | API 设计模式 — 如果你经常设计 RESTful API |
| `docker-patterns/` | Docker/Docker Compose — 如果你本地用 Docker 开发 |
| `deployment-patterns/` | CI/CD 和部署模式 — 如果你管部署流程 |
| `database-migrations/` | 数据库迁移模式 — 如果你经常写 migration |

### 不要 (38 个)

| 分类 | Skills | 理由 |
|------|--------|------|
| Python/Django | `python-*`, `django-*` (7 个) | 你不写 Python |
| Go | `golang-*` (2 个) | 你不写 Go |
| Java/Spring | `java-*`, `jpa-*`, `springboot-*` (6 个) | 你不写 Java |
| Swift | `swift-*`, `swiftui-*` (4 个) | 你不写 Swift |
| C++ | `cpp-*` (2 个) | 你不写 C++ |
| ClickHouse | `clickhouse-io/` | 专用数据库 |
| 高级 AI | `cost-aware-llm-pipeline/`, `foundation-models-on-device/`, `iterative-retrieval/`, `regex-vs-llm-structured-text/` | AI/ML 专项，非前端开发 |
| 特殊用途 | `liquid-glass-design/`, `nutrient-document-processing/`, `visa-doc-translate/` | 特定领域 |
| 系统管理 | `continuous-learning-v2/`, `skill-stocktake/`, `configure-ecc/`, `search-first/`, `content-hash-cache-pattern/`, `eval-harness/`, `project-guidelines-example/` | 元工具/高级用法/示例 |

### 安装命令

```bash
mkdir -p ~/.claude/skills

cp -r skills/tdd-workflow ~/.claude/skills/
cp -r skills/frontend-patterns ~/.claude/skills/
cp -r skills/backend-patterns ~/.claude/skills/
cp -r skills/coding-standards ~/.claude/skills/
cp -r skills/e2e-testing ~/.claude/skills/
cp -r skills/security-review ~/.claude/skills/
cp -r skills/postgres-patterns ~/.claude/skills/
cp -r skills/strategic-compact ~/.claude/skills/
cp -r skills/verification-loop ~/.claude/skills/
```

---

## 6. Hooks

Hooks 是自动执行的脚本，需要最谨慎。**合并到你已有的 settings.json 中，不要覆盖。**

### 要 (4 个)

| Hook | 事件 | 脚本 | 理由 |
|------|------|------|------|
| 拦截 dev server | PreToolUse (Bash) | 内联 node 脚本 | 防止 Claude 直接跑 `npm run dev` 卡住终端 |
| 自动格式化 | PostToolUse (Edit) | `post-edit-format.js` | 编辑后自动 Prettier/Biome，确定性保证 |
| TypeScript 检查 | PostToolUse (Edit) | `post-edit-typecheck.js` | 编辑 .ts/.tsx 后自动 tsc 检查 |
| 战略压缩提醒 | PreToolUse (Edit\|Write) | `suggest-compact.js` | 达到 50 次工具调用后提醒 /compact |

### 你已有的（保留）

| Hook | 事件 | 说明 |
|------|------|------|
| `stop-hook-git-check.sh` | Stop | 检查未提交/未推送代码 — 继续保留 |

### 不要

| Hook | 理由 |
|------|------|
| SessionStart (`session-start.js`) | 有一定价值但增加启动延迟，下面会给你更轻量的方案 |
| SessionEnd (`session-end.js`) | 会话结束自动保存摘要 — 见下方持久化记忆方案单独讨论 |
| `evaluate-session.js` | 持续学习系统 — 先不装，等你熟悉后再加 |
| `post-edit-console-warn.js` | console.log 警告 — 有用但和 typecheck 叠加会增加每次编辑的延迟 |
| `check-console-log.js` (Stop) | 和你已有的 git-check Stop Hook 叠加，增加延迟 |
| tmux 提醒 | 如果你不用 tmux 就没意义 |
| git push 提醒 | 你已有 git-check 更全面 |
| 文档文件警告 | 过于严格 |
| PR URL 日志 | 锦上添花，非必需 |
| 构建分析 | 示例性质 |

### 合并到 settings.json 的方式

你需要把新的 Hook **合并**到你已有的 `~/.claude/settings.json`，而不是覆盖：

```json
{
    "$schema": "https://json.schemastore.org/claude-code-settings.json",
    "hooks": {
        "PreToolUse": [
            {
                "matcher": "Bash",
                "hooks": [{
                    "type": "command",
                    "command": "node -e \"let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>{try{const i=JSON.parse(d);const cmd=i.tool_input?.command||'';if(/(npm run dev\\b|pnpm( run)? dev\\b|yarn dev\\b|bun run dev\\b)/.test(cmd)){console.error('[Hook] BLOCKED: Dev server must run in tmux');process.exit(2)}}catch{}console.log(d)})\""
                }],
                "description": "Block dev servers outside tmux"
            },
            {
                "matcher": "Edit|Write",
                "hooks": [{
                    "type": "command",
                    "command": "node /path/to/your/scripts/suggest-compact.js"
                }],
                "description": "Suggest compaction at logical intervals"
            }
        ],
        "PostToolUse": [
            {
                "matcher": "Edit",
                "hooks": [{
                    "type": "command",
                    "command": "node /path/to/your/scripts/post-edit-format.js"
                }],
                "description": "Auto-format JS/TS files after edits"
            },
            {
                "matcher": "Edit",
                "hooks": [{
                    "type": "command",
                    "command": "node /path/to/your/scripts/post-edit-typecheck.js"
                }],
                "description": "TypeScript check after editing .ts/.tsx"
            }
        ],
        "Stop": [
            {
                "matcher": "",
                "hooks": [{
                    "type": "command",
                    "command": "~/.claude/stop-hook-git-check.sh"
                }]
            }
        ]
    },
    "permissions": {
        "allow": ["Skill"]
    }
}
```

需要拷贝的脚本文件：

```bash
mkdir -p ~/.claude/scripts/hooks

cp scripts/hooks/post-edit-format.js ~/.claude/scripts/hooks/
cp scripts/hooks/post-edit-typecheck.js ~/.claude/scripts/hooks/
cp scripts/hooks/suggest-compact.js ~/.claude/scripts/hooks/

# 还需要拷贝依赖的工具库
mkdir -p ~/.claude/scripts/lib
cp scripts/lib/utils.js ~/.claude/scripts/lib/
cp scripts/lib/utils.d.ts ~/.claude/scripts/lib/
```

然后把 settings.json 中的路径改为你实际的路径。

---

## 7. 持久化记忆方案

这是你的核心需求。有三种方案，从简到繁：

### 方案 A：最简方案（推荐先用这个）

**不装任何 Hook，用 CLAUDE.md + /compact 手动管理。**

做法：
1. 在你的项目根目录维护一个好的 `CLAUDE.md`（见第 10 节模板）
2. 每次重要工作完成后，让 Claude 把关键决策和进度写到项目中的 `docs/SESSION_LOG.md`
3. 下次开新会话时，Claude 会自动读到 CLAUDE.md，你可以说"看一下 docs/SESSION_LOG.md 接着上次的进度"

优点：零配置，完全可控，信息在 Git 里有版本记录。

### 方案 B：Session 文件方案

装 SessionEnd Hook，自动保存会话摘要。

需要的文件：
```bash
cp scripts/hooks/session-end.js ~/.claude/scripts/hooks/
cp scripts/lib/utils.js ~/.claude/scripts/lib/  # 如果还没拷贝
```

添加到 settings.json 的 hooks 中：
```json
"SessionEnd": [
    {
        "matcher": "*",
        "hooks": [{
            "type": "command",
            "command": "node ~/.claude/scripts/hooks/session-end.js"
        }],
        "description": "Auto-save session summary"
    }
]
```

效果：每次会话结束自动在 `~/.claude/sessions/` 生成摘要文件（任务列表、修改的文件、使用的工具）。

加载方式：下次新会话时说 "加载上次的会话记录" 或添加 SessionStart Hook：

```json
"SessionStart": [
    {
        "matcher": "*",
        "hooks": [{
            "type": "command",
            "command": "node ~/.claude/scripts/hooks/session-start.js"
        }],
        "description": "Load previous session context"
    }
]
```

需要额外拷贝：
```bash
cp scripts/hooks/session-start.js ~/.claude/scripts/hooks/
cp scripts/lib/session-aliases.js ~/.claude/scripts/lib/
cp scripts/lib/session-aliases.d.ts ~/.claude/scripts/lib/
cp scripts/lib/session-manager.js ~/.claude/scripts/lib/
cp scripts/lib/session-manager.d.ts ~/.claude/scripts/lib/
cp scripts/lib/package-manager.js ~/.claude/scripts/lib/
cp scripts/lib/package-manager.d.ts ~/.claude/scripts/lib/
```

### 方案 C：完整持续学习方案（进阶）

在方案 B 的基础上，加上 `evaluate-session.js`，会话结束时不仅保存摘要，还会检测可提取的模式并保存为新 Skill。

**建议：先用方案 A 或 B 至少两周，熟悉后再升级到 C。**

---

## 8. MCP

### 不建议从这个库安装任何 MCP

`mcp-configs/mcp-servers.json` 是参考模板。你应该根据自己的实际需要单独安装：

```bash
# 如果你需要 GitHub MCP
claude mcp add github npx -y @modelcontextprotocol/server-github

# 如果你用 Supabase
claude mcp add supabase npx -y @supabase/mcp-server-supabase@latest --project-ref=YOUR_REF
```

### 经验法则

- 同时启用的 MCP < 10 个
- 不用的 MCP 在项目 `.claude/settings.json` 中禁用：
  ```json
  { "disabledMcpServers": ["supabase", "vercel"] }
  ```

---

## 9. 安装步骤（完整流程）

假设你在 everything-claude-code 仓库目录下执行：

```bash
# ============ Step 1: Rules ============
mkdir -p ~/.claude/rules/common
mkdir -p ~/.claude/rules/typescript

cp rules/common/coding-style.md ~/.claude/rules/common/
cp rules/common/security.md ~/.claude/rules/common/
cp rules/common/testing.md ~/.claude/rules/common/
cp rules/common/git-workflow.md ~/.claude/rules/common/
cp rules/common/development-workflow.md ~/.claude/rules/common/

cp rules/typescript/*.md ~/.claude/rules/typescript/

# ============ Step 2: Agents ============
mkdir -p ~/.claude/agents

cp agents/planner.md ~/.claude/agents/
cp agents/code-reviewer.md ~/.claude/agents/
cp agents/tdd-guide.md ~/.claude/agents/
cp agents/build-error-resolver.md ~/.claude/agents/
cp agents/security-reviewer.md ~/.claude/agents/
cp agents/e2e-runner.md ~/.claude/agents/
cp agents/refactor-cleaner.md ~/.claude/agents/

# ============ Step 3: Commands ============
mkdir -p ~/.claude/commands

cp commands/plan.md ~/.claude/commands/
cp commands/tdd.md ~/.claude/commands/
cp commands/code-review.md ~/.claude/commands/
cp commands/build-fix.md ~/.claude/commands/
cp commands/e2e.md ~/.claude/commands/
cp commands/verify.md ~/.claude/commands/
cp commands/test-coverage.md ~/.claude/commands/
cp commands/refactor-clean.md ~/.claude/commands/

# ============ Step 4: Skills ============
mkdir -p ~/.claude/skills

cp -r skills/tdd-workflow ~/.claude/skills/
cp -r skills/frontend-patterns ~/.claude/skills/
cp -r skills/backend-patterns ~/.claude/skills/
cp -r skills/coding-standards ~/.claude/skills/
cp -r skills/e2e-testing ~/.claude/skills/
cp -r skills/security-review ~/.claude/skills/
cp -r skills/postgres-patterns ~/.claude/skills/
cp -r skills/strategic-compact ~/.claude/skills/
cp -r skills/verification-loop ~/.claude/skills/

# ============ Step 5: Hook 脚本 ============
mkdir -p ~/.claude/scripts/hooks
mkdir -p ~/.claude/scripts/lib

cp scripts/hooks/post-edit-format.js ~/.claude/scripts/hooks/
cp scripts/hooks/post-edit-typecheck.js ~/.claude/scripts/hooks/
cp scripts/hooks/suggest-compact.js ~/.claude/scripts/hooks/
cp scripts/lib/utils.js ~/.claude/scripts/lib/
cp scripts/lib/utils.d.ts ~/.claude/scripts/lib/

# ============ Step 6: 手动合并 settings.json ============
# 不要直接覆盖！手动把 Hook 配置合并到你现有的 ~/.claude/settings.json
echo "请手动编辑 ~/.claude/settings.json 合并 Hook 配置（见手册第 6 节）"
```

---

## 10. CLAUDE.md 模板

以下是为 Next.js 全栈项目定制的 CLAUDE.md 模板，基于仓库中的 `examples/saas-nextjs-CLAUDE.md` 裁剪：

```markdown
# [项目名] — CLAUDE.md

## 技术栈
Next.js 15 (App Router) + TypeScript + Tailwind CSS + [数据库] + [认证方案]

## 关键规则
- Server Components 优先，Client Components 只用于交互
- 不可变更新（spread 代替 mutation）
- Zod 验证所有输入（API routes、forms、env vars）
- 禁止 console.log（有 Hook 自动检测）
- 文件控制在 200-400 行，最多 800 行

## 常用命令
- `pnpm dev` — 开发服务器
- `pnpm build` — 生产构建
- `pnpm test` — 运行测试
- `pnpm lint` — 代码检查
- `pnpm test:e2e` — E2E 测试

## 目录结构
src/
  app/           # App Router 页面
    api/         # API Routes
    (auth)/      # 认证页面
  components/
    ui/          # 基础 UI 组件
  hooks/         # 自定义 Hooks
  lib/           # 工具函数
  types/         # TypeScript 类型

## 工作流
1. `/plan` 制定计划
2. `/tdd` 编写测试 + 实现
3. `/code-review` 审查代码
4. `/verify` 最终验证（lint + type + test + build）

## 数据库
- [你的数据库相关规则]

## 环境变量
- 见 .env.example
```

**建议控制在 50-80 行。** 具体规则拆分到 `~/.claude/rules/` 中。

---

## 总结清单

| 组件 | 安装数 / 总数 | 上下文成本 |
|------|-------------|-----------|
| Rules | **10** / 29 | 始终加载（common）+ 按需（typescript） |
| Agents | **7** / 13 | 零（独立上下文窗口） |
| Commands | **8** / 33 | 仅触发时加载 |
| Skills | **9** / 52 | 元数据约 450 token，内容按需加载 |
| Hooks | **4** / 12 | 零（脚本级，不消耗 token） |
| MCP | **0** / 14 | 不安装，自行按需添加 |

**从 175+ 个文件精选到 38 个，覆盖你作为 Next.js 全栈开发者的全部核心需求。**
