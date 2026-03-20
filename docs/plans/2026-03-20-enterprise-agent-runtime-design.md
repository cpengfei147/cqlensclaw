# 企业级 Agent Runtime 底座架构设计

> 基于 OpenClaw 的行业通用 Agent Runtime 底座

## 1. 战略定位

**核心目标**：在 OpenClaw 之上构建企业级治理能力，形成可复制的行业解决方案。

**独特价值组合**：
- 治理能力（审批、审计、策略）
- 多租户/工作区隔离
- 行业 Pack 可插拔
- 基于成熟的 OpenClaw 执行骨架

**与竞品差异**：

| 项目 | 核心能力 | 我们的差异 |
|-----|---------|-----------|
| Ark (麦肯锡) | K8s 原生 | 更轻量，不强制 K8s |
| Soul-Core | Rust 高性能 | 更易用，TypeScript 生态 |
| TamaleBot | 安全至上 | + 多租户 + 行业 Pack |
| OpenAI Runtime | 状态恢复 | + 审批流 + 企业治理 |

---

## 2. 整体架构

```
┌──────────────────────────────────────────────────────────────────┐
│                         Access Layer                              │
│   Web Console  │  Chat/IM Channels  │  API/SDK  │  Webhook/Cron  │
└──────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                         Runtime Core                              │
│        Task Engine │ Agent Loop │ State Store │ Event Bus        │
└──────────────────────────────────────────────────────────────────┘
                                │
                ┌───────────────┴───────────────┐
                ▼                               ▼
┌───────────────────────────┐   ┌───────────────────────────────────┐
│     Governance Gate       │   │       Execution Bridge            │
│ Policy│Risk│Approval│Audit│   │  OpenClaw Adapter│Connectors     │
│        Cost Budget        │   │                                   │
└───────────────────────────┘   └───────────────────────────────────┘
                │                               │
                └───────────────┬───────────────┘
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                          Data Layer                               │
│       Task Store │ Audit Store │ Memory Store │ Artifact Store   │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│              Workspace Isolation（横切所有层）                     │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│              Industry Packs（可插拔挂载）                          │
└──────────────────────────────────────────────────────────────────┘
```

**架构原则**：
- 5 层 + 2 横切，边界清晰
- Governance 作为"门"（拦截器模式），所有 Action 必须过门
- 通过 Adapter 隔离 OpenClaw，保持可替换性

---

## 3. 模块详细设计

### 3.1 Runtime Core（运行时核心）

系统心脏，负责任务编排和状态管理。

| 组件 | 职责 |
|-----|------|
| **Task Engine** | 任务创建、拆分、状态机管理 |
| **Agent Loop** | Think → Act → Observe 循环 |
| **State Store** | 检查点保存、中断恢复 |
| **Event Bus** | 模块间解耦通信、外部通知 |

**任务状态流转**：
```
pending → running → paused → running → completed
                 ↘        ↗
                   failed
```

**检查点机制**：
- 在关键节点自动保存
- 支持从任意检查点恢复
- 用于长任务、中断恢复

---

### 3.2 Governance Gate（治理门）

企业级核心能力，采用拦截器模式。

| 组件 | 职责 |
|-----|------|
| **Policy Engine** | 工具白名单/黑名单、数据访问边界 |
| **Risk Classifier** | 动作风险分级（低/中/高/严重） |
| **Approval Engine** | 高风险动作审批流 |
| **Audit Logger** | 全量操作日志（只追加） |
| **Cost Budget** | Token 预算控制、成本告警 |

**风险分级与执行路径**：
```
低风险   → 自动执行
中风险   → 建议后执行
高风险   → 进入审批
严重风险 → 默认拒绝
```

---

### 3.3 Execution Bridge（执行桥）

连接 Runtime 与实际执行能力。

**OpenClaw Adapter**：
- Gateway 通信
- Channel 消息转换
- Session 管理
- Tools 调用
- Browser 控制

**External Connectors**：
- CRM/ERP 连接器
- MCP Server
- HTTP APIs
- 邮件/IM 系统

**设计要点**：
- 对上层屏蔽 OpenClaw 内部细节
- 统一 Action 输入/输出格式
- 便于后续替换底层引擎

---

### 3.4 Data Layer（数据层）

| 存储 | 用途 | 第一版实现 |
|-----|------|-----------|
| Task Store | 任务状态、检查点 | SQLite |
| Audit Store | 操作日志 | SQLite + 文件 |
| Memory Store | 短期/长期记忆 | 内存 + SQLite |
| Artifact Store | 任务产出物 | 本地文件系统 |

**设计原则**：
- 第一版用轻量方案快速验证
- 接口抽象好，后续可升级

---

### 3.5 Workspace Isolation（工作区隔离）

```
┌────────────────────────────────────────────────────────┐
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │ Workspace A  │  │ Workspace B  │  │ Workspace C  │ │
│  │ • 独立 Policy│  │ • 独立 Policy│  │ • 独立 Policy│ │
│  │ • 独立 Agents│  │ • 独立 Agents│  │ • 独立 Agents│ │
│  │ • 独立数据   │  │ • 独立数据   │  │ • 独立数据   │ │
│  │ • 独立预算   │  │ • 独立预算   │  │ • 独立预算   │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ │
└────────────────────────────────────────────────────────┘
```

**隔离维度**：数据、策略、资源（预算/配额）

---

### 3.6 Industry Pack（行业包框架）

```
一个 Industry Pack 包含：
├── prompts/        # 行业提示词
├── sops/           # 标准操作流程
├── tools/          # 行业专用工具
├── approvals/      # 审批模板
├── schemas/        # 输出格式定义
└── kpis/           # 指标模型
```

**加载方式**：配置化挂载，不改底座代码

---

## 4. 执行流示意

```
用户消息
    │
    ▼
[Access Layer] 接收请求
    │
    ▼
[Runtime Core] 创建 Task，启动 Agent Loop
    │
    ▼
Agent 决定执行 Action
    │
    ├──────────────────────────────┐
    ▼                              ▼
[Governance Gate]            [Execution Bridge]
    │                              │
    ├─ Policy 检查 ────────────────┤
    ├─ Risk 分级                   │
    │   ├─ 低风险 → 直接执行 ──────→│
    │   ├─ 中风险 → 建议后执行 ────→│
    │   └─ 高风险 → 进入审批        │
    │                              │
    ├─ Cost 检查                   │
    ├─ Audit 记录                  │
    └─ Credential 注入 ───────────→│
                                   │
                                   ▼
                          [OpenClaw / External]
                                   │
                                   ▼
                            执行结果返回
                                   │
                                   ▼
                      [State Store] 保存检查点
                                   │
                                   ▼
                         继续下一轮 Agent Loop
```

---

## 5. 原子闭环定义

按优先级排序的可验证闭环：

| # | 原子闭环 | 验证点 | 依赖 |
|---|---------|--------|------|
| 1 | 消息 → Agent → 响应 | OpenClaw 基础能力跑通 | 无 |
| 2 | 任务 → Action → 结果记录 | 每次执行都有记录 | #1 |
| 3 | Action → 审计日志 | 可查询、可追溯 | #2 |
| 4 | 高风险 Action → 审批 → 执行 | 人在回路 | #2, #3 |
| 5 | 任务中断 → 状态保存 → 恢复 | 长任务可靠性 | #2 |
| 6 | Workspace A ≠ Workspace B | 隔离验证 | #2 |

---

## 6. 技术选型（第一版）

| 类别 | 选择 | 原因 |
|-----|------|------|
| 语言 | TypeScript | 与 OpenClaw 一致 |
| 数据库 | SQLite | 轻量，快速验证 |
| 缓存 | 内存 | 简化，后续可换 Redis |
| 执行底座 | OpenClaw | 成熟的多渠道能力 |

---

## 7. 与原方案对照

| 原方案 | 本设计 | 变化原因 |
|--------|--------|---------|
| 7 层架构 | 5 层 + 2 横切 | 简化，边界更清晰 |
| Facade + Adapter 分离 | 合并为 Execution Bridge | 职责重叠 |
| Governance 在顶层 | Governance 作为"门" | 借鉴 TamaleBot 拦截器模式 |
| 无成本控制 | 新增 Cost Budget | 借鉴 Soul-Core |
| Industry Pack 在底层 | Industry Pack 可插拔 | 更符合插件化逻辑 |

---

## 8. 风险与待定项

| 风险 | 缓解措施 |
|-----|---------|
| 行业 Pack 无明确方向 | 先定义 Pack 框架，具体行业后补 |
| OpenClaw 升级风险 | Adapter 层做实，验证可替换性 |
| 无具体客户场景 | 找内部场景先跑通闭环 |

---

## 9. 下一步

1. **选择第一个原子闭环开始实现**
2. **确定集成方式**（Plugin 模式 vs 独立服务）
3. **搭建项目骨架**

---

## 参考项目

- [Ark (麦肯锡)](https://github.com/mckinsey/agents-at-scale-ark) - K8s 原生声明式
- [Soul-Core](https://github.com/AMAI-LABS/soul-core) - Rust 高性能运行时
- [TamaleBot-OSS](https://github.com/SudoDog-official/tamalebot-oss) - 安全至上
- [Yugent](https://github.com/team-virgo/yugent) - 简单模块化
- [OpenAI Agent Runtime](https://github.com/study8677/openai-agent-runtime) - 生命周期管理

---

*文档生成日期：2026-03-20*
*基于头脑风暴会话整理*
