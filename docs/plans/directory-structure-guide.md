# 工程目录结构说明

> OpenClaw 原有结构 + 企业级 Runtime 扩展结构对比

---

## 一、整体结构概览

```
cqlensclaw/
├── 📁 OpenClaw 原有目录
│   ├── src/              # 核心源码
│   ├── extensions/       # 官方扩展插件
│   ├── apps/             # 客户端应用
│   ├── docs/             # 文档
│   ├── ui/               # Web 控制台
│   ├── skills/           # 技能定义
│   ├── scripts/          # 构建脚本
│   └── ...
│
├── 📁 我们新增的目录
│   ├── packages/         # 企业级 Runtime 模块
│   └── docs/plans/       # 设计文档
│
└── 配置文件
```

---

## 二、OpenClaw 原有目录结构

### 2.1 根目录文件

| 文件/目录 | 含义 | 重要性 |
|----------|------|--------|
| `package.json` | 项目配置，依赖定义 | ⭐⭐⭐ |
| `tsconfig.json` | TypeScript 配置 | ⭐⭐⭐ |
| `pnpm-workspace.yaml` | pnpm 工作区配置 | ⭐⭐ |
| `openclaw.mjs` | CLI 入口文件 | ⭐⭐⭐ |
| `Dockerfile` | Docker 构建文件 | ⭐⭐ |
| `README.md` | 项目说明 | ⭐ |
| `CHANGELOG.md` | 变更日志 | ⭐ |
| `LICENSE` | MIT 许可证 | ⭐ |
| `SECURITY.md` | 安全策略 | ⭐⭐ |
| `AGENTS.md` / `CLAUDE.md` | AI 助手指令 | ⭐ |

### 2.2 src/ 目录（核心源码）

这是 OpenClaw 的核心代码目录。

```
src/
├── gateway/          # 🔥 Gateway 服务（核心）
├── channels/         # 🔥 渠道系统
├── agents/           # 🔥 Agent 执行引擎
├── config/           # 配置管理
├── hooks/            # Hook 系统
├── plugins/          # 插件系统
├── cli/              # CLI 命令
├── commands/         # 命令实现
├── infra/            # 基础设施
├── routing/          # 路由系统
├── browser/          # 浏览器自动化
├── cron/             # 定时任务
├── discord/          # Discord 渠道
├── telegram/         # Telegram 渠道
├── slack/            # Slack 渠道
├── signal/           # Signal 渠道
├── imessage/         # iMessage 渠道
├── web/              # WhatsApp Web 渠道
├── security/         # 安全相关
├── media/            # 媒体处理
├── process/          # 进程管理
├── terminal/         # 终端 UI
└── ...
```

#### src/gateway/ - Gateway 服务（最核心）

| 文件 | 含义 |
|-----|------|
| `server.impl.ts` | Gateway 服务器核心实现 |
| `server-methods.ts` | RPC 方法列表（40+ 个） |
| `server-chat.ts` | 消息处理和心跳管理 |
| `server-channels.ts` | 渠道管理 |
| `auth.ts` | 认证系统 |
| `control-plane-audit.ts` | 控制平面审计 |
| `exec-approval-manager.ts` | 执行审批管理 |

#### src/channels/ - 渠道系统

| 目录/文件 | 含义 |
|----------|------|
| `plugins/` | 渠道插件定义 |
| `plugins/types.ts` | 渠道插件类型定义 |
| `allowlists/` | 白名单管理 |
| `session.ts` | 会话管理 |

#### src/agents/ - Agent 执行引擎

| 文件 | 含义 |
|-----|------|
| `tool-catalog.ts` | 工具目录 |
| `provider-capabilities.ts` | Agent 能力配置 |
| `runner.ts` | Agent 运行器 |

#### src/hooks/ - Hook 系统

| 文件 | 含义 |
|-----|------|
| `types.ts` | Hook 事件类型定义 |
| `message-hook-mappers.ts` | 消息 Hook 映射 |

#### src/plugins/ - 插件系统

| 文件 | 含义 |
|-----|------|
| `types.ts` | 插件类型定义 |
| `registry.ts` | 插件注册中心 |
| `hooks.ts` | 插件 Hook 实现 |
| `runtime/` | 插件运行时 |

#### src/infra/ - 基础设施

| 文件 | 含义 |
|-----|------|
| `exec-approvals.ts` | 执行审批 |
| `session-cost-usage.ts` | 成本追踪 |
| `device-auth-store.ts` | 设备认证存储 |
| `net/` | 网络相关（代理、SSRF 防护） |

#### src/config/ - 配置管理

| 目录/文件 | 含义 |
|----------|------|
| `sessions/` | 会话配置和存储 |
| `config.ts` | 配置管理器 |
| `types.*.ts` | 各类型配置定义 |

### 2.3 extensions/ 目录（扩展插件）

OpenClaw 官方维护的扩展插件。

```
extensions/
├── telegram/         # Telegram 扩展
├── discord/          # Discord 扩展
├── slack/            # Slack 扩展
├── feishu/           # 飞书扩展
├── msteams/          # Microsoft Teams 扩展
├── matrix/           # Matrix 扩展
├── googlechat/       # Google Chat 扩展
├── line/             # LINE 扩展
├── irc/              # IRC 扩展
├── nostr/            # Nostr 扩展
├── bluebubbles/      # BlueBubbles (iMessage) 扩展
├── memory-lancedb/   # LanceDB 记忆存储
├── memory-core/      # 记忆核心
├── voice-call/       # 语音通话
├── diffs/            # 差异对比
└── ...
```

每个扩展的结构：
```
extensions/{name}/
├── src/
│   ├── index.ts      # 入口文件
│   └── channel.ts    # 渠道实现
├── package.json
└── README.md
```

### 2.4 apps/ 目录（客户端应用）

```
apps/
├── macos/            # macOS 原生应用
├── ios/              # iOS 应用
├── android/          # Android 应用
└── shared/           # 共享代码
```

### 2.5 ui/ 目录（Web 控制台）

```
ui/
├── src/
│   └── ui/
│       ├── app.ts              # 应用入口
│       ├── chat/               # 聊天 UI
│       ├── controllers/        # 控制器
│       ├── components/         # 组件
│       └── ...
├── index.html
├── vite.config.ts
└── package.json
```

### 2.6 其他目录

| 目录 | 含义 |
|-----|------|
| `docs/` | 文档（Mintlify 格式） |
| `skills/` | 技能定义文件 |
| `scripts/` | 构建和工具脚本 |
| `test/` | 测试配置 |
| `test-fixtures/` | 测试数据 |
| `patches/` | 依赖补丁 |
| `assets/` | 静态资源 |
| `git-hooks/` | Git 钩子 |
| `Swabble/` | Swabble 相关（内部工具） |

---

## 三、我们新增的目录结构

### 3.1 packages/ 目录（企业级 Runtime）

```
packages/
├── governance-core/          # 🔥 治理核心
│   ├── src/
│   │   ├── policy/           # Policy Engine - 策略引擎
│   │   ├── risk/             # Risk Classifier - 风险分级
│   │   ├── approval/         # Approval Engine - 审批引擎
│   │   ├── audit/            # Audit Logger - 审计日志
│   │   └── cost/             # Cost Budget - 成本预算
│   ├── package.json
│   └── README.md
│
├── runtime-core/             # 🔥 运行时核心
│   ├── src/
│   │   ├── task/             # Task Engine - 任务引擎
│   │   ├── state/            # State Store - 状态存储/检查点
│   │   ├── event/            # Event Bus - 事件总线
│   │   └── orchestrator/     # Agent Orchestrator - Agent 编排
│   ├── package.json
│   └── README.md
│
├── openclaw-adapter/         # 🔥 OpenClaw 适配层
│   ├── src/
│   │   ├── gateway/          # Gateway Adapter - 网关适配
│   │   ├── channel/          # Channel Adapter - 渠道适配
│   │   ├── session/          # Session Bridge - 会话桥接
│   │   └── security/         # Security Hardening - 安全加固
│   ├── package.json
│   └── README.md
│
├── workspace/                # 多租户隔离
│   ├── src/
│   │   ├── isolation/        # 数据/策略/资源隔离
│   │   └── management/       # Workspace 管理
│   ├── package.json
│   └── README.md
│
├── industry-pack/            # 行业包框架
│   ├── src/
│   │   ├── loader/           # Pack 加载器
│   │   ├── validator/        # Pack 验证器
│   │   └── schema/           # Pack 规范定义
│   ├── packs/                # 示例行业包
│   │   └── demo-pack/
│   ├── package.json
│   └── README.md
│
├── shared/                   # 共享模块
│   ├── src/
│   │   ├── types/            # 共享类型定义
│   │   ├── utils/            # 工具函数
│   │   └── db/               # 数据库抽象
│   ├── package.json
│   └── README.md
│
└── enterprise-runtime/       # 整合入口
    ├── src/
    │   └── index.ts          # 统一导出
    ├── package.json
    └── README.md
```

### 3.2 docs/plans/ 目录（设计文档）

```
docs/plans/
├── original-architecture-plan.md      # 原始架构规划（你的输入）
├── 2026-03-20-enterprise-agent-runtime-design.md  # 架构设计
├── 2026-03-20-technical-solution.md   # 技术方案
└── directory-structure-guide.md       # 本文档
```

---

## 四、目录对比总结

### 4.1 OpenClaw vs 我们的扩展

| 类别 | OpenClaw 原有 | 我们新增 |
|-----|-------------|---------|
| **核心代码** | `src/` | `packages/` |
| **扩展插件** | `extensions/` | `packages/industry-pack/packs/` |
| **设计文档** | `docs/` (用户文档) | `docs/plans/` (设计文档) |
| **配置** | 根目录 | 复用原有 |
| **构建** | `scripts/` | 复用原有 |

### 4.2 职责划分

| 目录 | 职责 | 所有者 |
|-----|------|--------|
| `src/gateway/` | Gateway 核心 | OpenClaw |
| `src/channels/` | 渠道系统 | OpenClaw |
| `src/agents/` | Agent 执行 | OpenClaw |
| `src/hooks/` | Hook 系统 | OpenClaw |
| `src/plugins/` | 插件框架 | OpenClaw |
| `packages/governance-core/` | 治理能力 | 我们 |
| `packages/runtime-core/` | 任务管理 | 我们 |
| `packages/openclaw-adapter/` | 适配 + 安全 | 我们 |
| `packages/workspace/` | 多租户 | 我们 |
| `packages/industry-pack/` | 行业包 | 我们 |

### 4.3 我们如何与 OpenClaw 集成

```
┌─────────────────────────────────────────────────────────────┐
│                     我们的 packages/                         │
│  governance-core │ runtime-core │ workspace │ industry-pack │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ 通过 Plugin SDK 集成
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                  packages/openclaw-adapter/                  │
│     封装 OpenClaw 内部接口，提供安全加固                      │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ 调用
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                     OpenClaw 原有代码                        │
│  src/gateway │ src/channels │ src/agents │ src/hooks        │
└─────────────────────────────────────────────────────────────┘
```

---

## 五、关键文件速查

### 5.1 开发时最常用的文件

| 文件 | 用途 |
|-----|------|
| `package.json` | 依赖管理、脚本命令 |
| `src/gateway/server.impl.ts` | Gateway 核心（了解入口） |
| `src/plugins/types.ts` | Plugin 类型（开发插件必看） |
| `src/hooks/types.ts` | Hook 事件类型（监听事件必看） |
| `packages/*/src/index.ts` | 我们各模块入口 |

### 5.2 调试时需要关注的目录

| 目录 | 调试场景 |
|-----|---------|
| `~/.openclaw/` | 运行时数据（会话、日志、配置） |
| `~/.openclaw/sessions/` | 会话数据 |
| `~/.openclaw/logs/` | 日志文件 |
| `~/.openclaw/config.json` | 配置文件 |

---

## 六、命名约定

### 6.1 OpenClaw 命名约定

| 类型 | 约定 | 示例 |
|-----|------|------|
| 文件名 | kebab-case | `server-methods.ts` |
| 类名 | PascalCase | `GatewayServer` |
| 函数名 | camelCase | `handleMessage` |
| 常量 | UPPER_SNAKE | `DEFAULT_PORT` |

### 6.2 我们的命名约定（保持一致）

| 类型 | 约定 | 示例 |
|-----|------|------|
| 包名 | kebab-case | `governance-core` |
| 文件名 | kebab-case | `policy-engine.ts` |
| 类名 | PascalCase | `PolicyEngine` |
| 接口名 | PascalCase + 前缀 I 可选 | `Policy` 或 `IPolicy` |

---

*文档生成日期：2026-03-20*
