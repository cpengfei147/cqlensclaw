# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**CQLensClaw** - 基于 OpenClaw 构建的企业级 Agent Runtime 通用底座，面向多行业场景的可复制解决方案。

- **GitHub**: https://github.com/cpengfei147/cqlensclaw
- **Fork 来源**: openclaw/openclaw

## Core Principle: Fork + Deep Enhancement, Not Just a Wrapper

**重要**：我们不是简单地在 OpenClaw 外面"套一层"，而是：

1. **Fork 模式** - 维护 OpenClaw 的安全加固分支
2. **必须修复 OpenClaw 底层的安全漏洞和架构问题**
3. **不能交付给企业一个存在隐患和漏洞的版本**

### Security Issues to Fix (Before Production)

| Issue | Severity | Status |
|-------|----------|--------|
| ClawJacked 漏洞 (Gateway 认证) | Critical | Pending |
| 明文凭证存储 | Critical | Pending |
| Prompt 注入防护不足 | High | Pending |

### Layered Approach

| Problem Type | Where to Fix |
|--------------|--------------|
| 权限控制、审计日志、成本控制 | `packages/` 扩展层 |
| 底层安全漏洞、认证缺陷、凭证存储 | 修改 OpenClaw 源码 |

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Product Surface                              │
│        Web Console │ Chat/IM │ API/SDK │ Admin UI               │
└─────────────────────────────────────────────────────────────────┘
                                │
┌─────────────────────────────────────────────────────────────────┐
│                 Governance & Runtime Core                        │
│   Task Orchestrator │ Policy │ Approval │ Audit │ State Store   │
└─────────────────────────────────────────────────────────────────┘
                                │
┌─────────────────────────────────────────────────────────────────┐
│                    OpenClaw Adapter Layer                        │
│          + Security Hardening (凭证加密、输入审查)                │
└─────────────────────────────────────────────────────────────────┘
                                │
┌─────────────────────────────────────────────────────────────────┐
│                       OpenClaw Core                              │
│     Gateway │ Channels │ Sessions │ Tools │ Hooks │ Events      │
└─────────────────────────────────────────────────────────────────┘

+ Workspace Isolation（横切：多租户隔离）
+ Industry Packs（横切：可插拔行业包）
```

## Project Structure

### Our Modules (`packages/`)

| Module | Purpose | Priority |
|--------|---------|----------|
| `governance-gate/` | 治理核心：Policy、Risk、Approval、Audit、Cost | P0 |
| `runtime-core/` | 运行时核心：Task Engine、State Store、Event Bus | P0 |
| `enterprise-runtime/` | 整合入口 | P0 |

### OpenClaw Core (`src/`)

| Directory | Purpose |
|-----------|---------|
| `src/gateway/` | Gateway 服务核心（最核心） |
| `src/channels/` | 渠道系统 |
| `src/agents/` | Agent 执行引擎 |
| `src/hooks/` | Hook 系统（扩展点） |
| `src/plugins/` | 插件系统 |
| `extensions/` | 官方扩展插件（Telegram、Discord 等） |

## Development Commands

### OpenClaw Commands

```bash
# Install dependencies
pnpm install

# Build
pnpm build

# Type check
pnpm tsgo

# Lint/format
pnpm check
pnpm format:fix

# Tests
pnpm test
pnpm test:coverage

# Run CLI in dev
pnpm openclaw ...
```

### Runtime Requirements

- Node.js 22+
- pnpm (package manager)
- Bun supported for TypeScript execution

## Key Files to Understand

| File | Purpose |
|------|---------|
| `src/gateway/server.impl.ts` | Gateway 核心实现 |
| `src/plugins/types.ts` | Plugin 类型定义（开发插件必看） |
| `src/hooks/types.ts` | Hook 事件类型（监听事件必看） |
| `docs/plans/2026-03-20-technical-solution.md` | 完整技术方案 |
| `docs/plans/directory-structure-guide.md` | 目录结构说明 |

## Coding Conventions

- Language: TypeScript (ESM), strict typing, avoid `any`
- File naming: kebab-case (`policy-engine.ts`)
- Class naming: PascalCase (`PolicyEngine`)
- Tests: colocated `*.test.ts`
- Formatting: Oxlint + Oxfmt, run `pnpm check` before commits
- Keep files under ~700 LOC

## Communication

- 使用中文与用户交流
- 代码注释使用英文
