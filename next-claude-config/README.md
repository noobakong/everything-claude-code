# Next.js Fullstack Developer — Claude Code Config

A curated, reviewed, and fixed subset of [everything-claude-code](https://github.com/anthropics/everything-claude-code), optimized for **JS/TS frontend fullstack engineers** working with Next.js, React, Node.js, and Vercel.

## Design Philosophy

This config is designed to **complement the superpowers plugin**, not replace it.

```
superpowers (workflow engine)     +  next-claude-config (domain knowledge)
─────────────────────────────        ─────────────────────────────────────
HOW to work:                         WHAT standards to follow:
  brainstorming → planning             project rules (coding style, git, security)
  → TDD → code review                 domain knowledge (API, backend, Docker)
  → verification → completion          security audit (OWASP, secrets, checklist)
```

**Superpowers** handles planning, TDD, code review, verification, and debugging workflows.
**This config** provides project-level rules, domain-specific knowledge, and security capabilities that superpowers does not cover.

### What was intentionally excluded to avoid conflicts

| Excluded | Reason | Covered by |
|----------|--------|------------|
| planner / code-reviewer / tdd-guide agents | superpowers has superior workflow integration | `superpowers:writing-plans`, `superpowers:code-reviewer`, `superpowers:test-driven-development` |
| refactor-cleaner agent | Functionality overlap | `code-simplifier` plugin |
| /plan, /tdd, /code-review, /verify, /refactor-clean commands | superpowers auto-triggers these | superpowers skills |
| tdd-workflow skill | superpowers TDD has stricter enforcement (Iron Laws) | `superpowers:test-driven-development` |
| frontend-patterns skill | Overlap with plugin | `frontend-design` plugin |
| post-edit-typecheck.js hook | Duplicate type checking | `typescript-lsp` plugin |

## What's Included

| Category | Count | Purpose |
|----------|-------|---------|
| **rules/** | 6 files | Always-on guidelines: security, coding style, git workflow, TypeScript conventions |
| **agents/** | 2 agents | Security auditing (read-only) and build error resolution |
| **skills/** | 6 skills | Domain knowledge: API design, coding standards, Docker, security review, backend patterns, context compaction |
| **commands/** | 2 commands | `/build-fix` and `/test-coverage` |
| **scripts/** | 2 scripts | Post-edit auto-formatting hook + shared utilities |

## Directory Structure

```
next-claude-config/
├── README.md
├── agents/
│   ├── build-error-resolver.md       # Build error fixing agent
│   └── security-reviewer.md          # Security audit agent (read-only)
├── rules/
│   ├── common/
│   │   ├── security.md               # Security rules (OWASP, secrets, etc.)
│   │   ├── coding-style.md           # General coding conventions
│   │   └── git-workflow.md           # Commit message & PR workflow
│   └── typescript/
│       ├── coding-style.md           # TS-specific style rules
│       ├── patterns.md               # TS design patterns
│       └── hooks.md                  # React hooks rules
├── skills/
│   ├── api-design/SKILL.md           # RESTful API design patterns
│   ├── coding-standards/SKILL.md     # Code quality standards
│   ├── docker-patterns/SKILL.md      # Docker/containerization patterns
│   ├── security-review/SKILL.md      # Security review checklist (OWASP Top 10)
│   ├── strategic-compact/SKILL.md    # Context window compaction strategy
│   └── backend-patterns/SKILL.md     # Node.js/API backend patterns
├── commands/
│   ├── build-fix.md                  # /build-fix — Fix build errors
│   └── test-coverage.md              # /test-coverage — Coverage report
└── scripts/
    ├── hooks/
    │   └── post-edit-format.js        # Auto-format after editing JS/TS
    └── lib/
        └── utils.js                   # Shared utilities for hooks
```

## Installation

### 1. Copy files to your project

```bash
# From the everything-claude-code repo root:

# Rules (highest value — always-on project conventions)
cp -r next-claude-config/rules/ YOUR_PROJECT/.claude/rules/

# Agents
cp -r next-claude-config/agents/ YOUR_PROJECT/.claude/agents/

# Skills
cp -r next-claude-config/skills/ YOUR_PROJECT/.claude/skills/

# Commands
cp -r next-claude-config/commands/ YOUR_PROJECT/.claude/commands/

# Scripts (for hooks)
cp -r next-claude-config/scripts/ YOUR_PROJECT/.claude/scripts/
```

### 2. Configure hooks in settings.json

Add to your project's `.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "node .claude/scripts/hooks/post-edit-format.js"
          }
        ]
      }
    ]
  }
}
```

### 3. Priority guide

| Priority | What to Install | Why |
|----------|----------------|-----|
| **High** | `rules/` (all 6 files) | Always-on quality guardrails — this is the highest-value addition |
| **High** | `agents/security-reviewer.md` | Security audit capability that superpowers lacks |
| **Medium** | `skills/backend-patterns/`, `skills/api-design/` | Domain knowledge for Node.js/API development |
| **Medium** | `scripts/hooks/post-edit-format.js` | Automatic formatting after edits |
| **Low** | `skills/docker-patterns/` | Only if your project uses Docker |
| **Low** | `skills/strategic-compact/` | Context management for long sessions |

## Prerequisites (Global Plugins)

This config assumes you have these plugins enabled globally:

| Plugin | Role |
|--------|------|
| **superpowers** | Workflow engine: planning, TDD, code review, verification, debugging |
| **frontend-design** | Frontend UI generation and React patterns |
| **typescript-lsp** | TypeScript type checking after edits |
| **code-simplifier** | Code cleanup and refactoring |
| **context7** | Up-to-date library documentation |
| **playwright** | Browser automation for E2E |

## Context Cost

| Category | ~Tokens | Loading |
|----------|---------|---------|
| Rules (all 6) | ~2,500 | Always loaded |
| Agents (2) | ~500 each | On invocation only |
| Skills (6) | ~1,500 each | On trigger only |
| Commands (2) | ~300 each | On invocation only |

**Worst case if everything loads**: ~13K tokens. In practice, only rules are always loaded (~2.5K).

## Fixes Applied

This config includes 4 fixes compared to the upstream `everything-claude-code`:

| # | File | Fix | Reason |
|---|------|-----|--------|
| 1 | `agents/security-reviewer.md` | Removed `Write` and `Edit` from tools | Security auditor should be read-only |
| 2 | `rules/common/git-workflow.md` | Removed `Attribution disabled via settings.json` line | Claude tool config leaked into git rules |
| 3 | `skills/security-review/SKILL.md` | Removed Section 9 "Blockchain Security (Solana)" and wallet checklist item | Irrelevant to frontend fullstack; `verify` import was incorrect |
| 4 | `skills/backend-patterns/SKILL.md` | Added serverless warning to RateLimiter and JobQueue | In-memory state is lost between invocations in Vercel/Lambda |

## License

Same as the upstream repository.
