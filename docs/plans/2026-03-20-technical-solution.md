# 企业级 Agent Runtime 技术方案

> 基于 OpenClaw 的行业通用 Agent Runtime 底座 - 技术方案

## 一、问题定义与目标

### 1.1 核心问题

企业想用 AI Agent 提升效率，但面临三个障碍：

| 障碍 | 企业担忧 | 我们的解法 |
|-----|---------|-----------|
| **不敢用** | Agent 会不会乱操作？出事谁负责？ | 治理层：审批、审计、风险分级 |
| **不好用** | 每个行业流程不同，通用 Agent 不够用 | 行业 Pack：可插拔的行业能力包 |
| **不能用** | 多部门怎么隔离？成本怎么控制？ | 多租户：Workspace 隔离、成本预算 |

### 1.2 行业背景（2026）

| 挑战 | 数据 |
|-----|------|
| 安全合规是首要障碍 | 50% 的 AI Agent 项目停留在试点阶段 |
| 治理模型不成熟 | 只有 1/5 的公司有成熟的 Agent 治理模型 |
| 集成复杂 | 46% 认为与现有系统集成是首要挑战 |
| 监管加强 | EU AI Act 2026年8月全面执行 |

### 1.3 成功标准

| 标准 | 可勘测指标 |
|-----|-----------|
| **可控** | 高风险操作 100% 经过审批 |
| **可追溯** | 每个 Action 都有审计日志 |
| **可隔离** | 不同 Workspace 数据完全隔离 |
| **可扩展** | 新行业 Pack 可在 1 周内接入 |
| **可落地** | 第一个闭环 2 周内跑通 |

---

## 二、技术决策与依据

### 2.1 决策 1：基于 OpenClaw + 安全加固层

| 方案 | 优点 | 缺点 |
|-----|------|------|
| A. 直接用 OpenClaw | 快速启动 | 继承其安全风险 |
| B. 完全自研 | 完全可控 | 耗时长，重复造轮子 |
| **C. OpenClaw + 安全加固层** | 借力 + 补短板 | 复杂度增加 |

**选择**：方案 C

**依据**：
- OpenClaw 的渠道能力成熟（20+ 渠道）
- 但其安全问题已被公开披露，企业场景必须加固
- 符合"先借力，再抽象"的策略

**安全加固措施**：

| OpenClaw 问题 | 加固措施 |
|--------------|---------|
| 凭证明文存储 | Credential Vault（加密存储） |
| 提示注入风险 | 输入过滤 + 输出审查 |
| 插件恶意风险 | 插件白名单 + 签名验证 |
| WebSocket 劫持 | 强制认证 + 来源验证 |

### 2.2 决策 2：Plugin 模式集成

| 方案 | 优点 | 缺点 |
|-----|------|------|
| **A. Plugin 模式** | 简单，紧密集成 | 受 OpenClaw 约束 |
| B. 独立服务 | 灵活，解耦 | 复杂，两个服务 |

**选择**：方案 A

**依据**：
- OpenClaw 的 Hook/Gateway Method 机制足够灵活
- 减少重复开发，快速验证
- 符合"先借力"的策略

### 2.3 决策 3：简化架构层数

| 方案 | 层数 | 复杂度 |
|-----|------|--------|
| 原方案 | 7 层 | 高 |
| **调整方案** | 5 层 + 2 横切 | 中 |

**选择**：5 层 + 2 横切

**依据**：
- Facade + Adapter 职责重叠，合并
- Industry Packs 应横切而非底层
- 简化后边界更清晰

---

## 三、系统架构

### 3.1 架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                     Product Surface                              │
│        Web Console │ Chat/IM │ API/SDK │ Admin UI               │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                 Governance & Runtime Core                        │
│   Task Orchestrator │ Policy │ Approval │ Audit │ State Store   │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    OpenClaw Adapter Layer                        │
│          + Security Hardening (凭证加密、输入审查)                │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                       OpenClaw Core                              │
│     Gateway │ Channels │ Sessions │ Tools │ Hooks │ Events      │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
│                    External Systems                              │
│         CRM │ ERP │ MCP │ Browser │ Email │ APIs                │
└ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘

┌─────────────────────────────────────────────────────────────────┐
│          Workspace Isolation（横切：多租户隔离）                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│          Industry Packs（横切：可插拔行业包）                     │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 层级说明

| 层 | 职责 | 自研/复用 |
|---|------|----------|
| Product Surface | 用户入口 | 部分自研 |
| Governance & Runtime | 治理 + 任务管理 | 自研 |
| OpenClaw Adapter | 适配 + 安全加固 | 自研 |
| OpenClaw Core | 执行引擎 | 复用 |
| External Systems | 外部系统 | 连接器 |
| Workspace Isolation | 多租户 | 自研 |
| Industry Packs | 行业能力 | 自研框架 |

---

## 四、模块划分

### 4.1 模块清单

| 模块 | 职责 | 优先级 |
|-----|------|--------|
| **governance-core** | 治理核心：Policy、Risk、Approval、Audit | P0 |
| **runtime-core** | 运行时核心：Task、State、Event | P0 |
| **openclaw-adapter** | OpenClaw 适配 + 安全加固 | P0 |
| **workspace** | 多租户隔离 | P1 |
| **industry-pack** | 行业包框架 | P1 |
| **shared** | 共享类型和工具 | P0 |

### 4.2 模块依赖关系

```
┌─────────────┐     ┌─────────────┐
│ console-api │────▶│runtime-core │
└─────────────┘     └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │ governance- │
                    │    core     │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │  openclaw-  │
                    │   adapter   │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │  OpenClaw   │
                    │    Core     │
                    └─────────────┘

横切：workspace（贯穿所有模块）
横切：industry-pack（贯穿所有模块）
```

### 4.3 各模块详细职责

#### governance-core（治理核心）

| 子模块 | 职责 | 达成效果 |
|-------|------|---------|
| Policy Engine | 定义规则：允许/禁止/需审批 | 管理员可配置哪些操作需要审批 |
| Risk Classifier | 自动判断操作风险等级 | 低风险自动执行，高风险触发审批 |
| Approval Engine | 审批流管理 | 审批人可通过 IM/Console 处理 |
| Audit Logger | 记录所有操作 | 满足 EU AI Act 审计要求 |
| Cost Budget | 预算控制 | 超预算自动告警或停止 |

#### runtime-core（运行时核心）

| 子模块 | 职责 | 达成效果 |
|-------|------|---------|
| Task Engine | 任务创建、拆分、状态管理 | 复杂任务可拆分为子任务 |
| State Store | 检查点保存/恢复 | 长任务可中断恢复 |
| Event Bus | 模块间事件通信 | 解耦，支持异步处理 |
| Agent Orchestrator | 多 Agent 协调 | 支持多 Agent 协作完成任务 |

#### openclaw-adapter（OpenClaw 适配 + 安全加固）

| 子模块 | 职责 | 达成效果 |
|-------|------|---------|
| Gateway Adapter | 封装 OpenClaw Gateway 通信 | 统一接口，隔离内部细节 |
| Channel Adapter | 统一多渠道消息格式 | 业务层不感知渠道差异 |
| Security Hardening | 凭证加密、输入过滤、输出审查 | 修复 OpenClaw 安全漏洞 |
| Session Bridge | 扩展 Session 能力 | 支持检查点、元数据 |

#### workspace（多租户隔离）

| 能力 | 职责 | 达成效果 |
|-----|------|---------|
| 数据隔离 | 不同 Workspace 数据完全分离 | A 租户看不到 B 租户数据 |
| 策略隔离 | 不同 Workspace 可有不同规则 | 每个租户可定制审批流程 |
| 资源隔离 | 预算、配额独立计算 | 一个租户超支不影响其他 |
| Agent 隔离 | 不同 Workspace 有独立 Agent | 避免数据泄露 |

#### industry-pack（行业包框架）

| 能力 | 职责 | 达成效果 |
|-----|------|---------|
| Pack 定义规范 | 定义行业包结构 | 新行业可快速接入 |
| Pack 加载器 | 运行时加载行业包 | 无需重启即可切换 |
| Pack 验证器 | 校验行业包合法性 | 防止恶意包 |

---

## 五、数据模型

### 5.1 基础实体

| 实体 | 说明 | 关键字段 |
|-----|------|---------|
| **Workspace** | 工作区/租户 | id, name, quota, policy_id |
| **Agent** | AI 代理 | id, workspace_id, role, capabilities |
| **Task** | 任务 | id, workspace_id, agent_id, status, parent_task_id |
| **Action** | 具体操作 | id, task_id, tool_name, risk_level, result |
| **Checkpoint** | 检查点 | id, task_id, state_snapshot, created_at |

### 5.2 治理实体

| 实体 | 说明 | 关键字段 |
|-----|------|---------|
| **Policy** | 策略规则 | id, workspace_id, rules[] |
| **Approval** | 审批记录 | id, action_id, approver, decision, reason |
| **AuditLog** | 审计日志 | id, workspace_id, action, actor, timestamp |
| **CostRecord** | 成本记录 | id, workspace_id, tokens, cost_usd, date |

### 5.3 行业包实体

| 实体 | 说明 | 关键字段 |
|-----|------|---------|
| **IndustryPack** | 行业包 | id, name, version, components[] |
| **PackComponent** | 包组件 | type (prompt/sop/tool/template), content |

### 5.4 实体关系

```
Workspace
    │
    ├──▶ Agent (1:N)
    │       │
    │       └──▶ Task (1:N)
    │               │
    │               ├──▶ Action (1:N)
    │               │       │
    │               │       └──▶ Approval (1:1)
    │               │
    │               └──▶ Checkpoint (1:N)
    │
    ├──▶ Policy (1:1)
    │
    ├──▶ AuditLog (1:N)
    │
    ├──▶ CostRecord (1:N)
    │
    └──▶ IndustryPack (N:M)
```

### 5.5 状态流转

**Task 状态**：
```
pending → running → paused → running → completed
              │                           │
              └──────▶ failed ◀───────────┘
```

**Action 状态**：
```
pending → risk_assessed → approved/auto_approved → executing → completed
              │                    │                              │
              └─▶ rejected         └─▶ approval_pending           └─▶ failed
```

---

## 六、实现路径

### 6.1 分阶段交付

| 阶段 | 目标 | 周期 | 交付物 |
|-----|------|------|--------|
| **Phase 0** | 环境就绪 | 1 周 | OpenClaw 跑通、项目骨架 |
| **Phase 1** | 最小闭环 | 2 周 | Action → 审计日志 |
| **Phase 2** | 治理能力 | 3 周 | 风险分级 + 审批流 |
| **Phase 3** | 任务管理 | 2 周 | Task + 检查点恢复 |
| **Phase 4** | 多租户 | 2 周 | Workspace 隔离 |
| **Phase 5** | 行业包 | 2 周 | Pack 框架 + 示例包 |

**总计：12 周（3 个月）**

### 6.2 Phase 0：环境就绪（1 周）

| 任务 | 验收标准 |
|-----|---------|
| 部署 OpenClaw Gateway | 能通过一个渠道收发消息 |
| 创建项目骨架 | packages/ 目录结构就位 |
| 配置开发环境 | TypeScript + SQLite 可运行 |
| 跑通 Plugin 机制 | 自定义 Hook 能被触发 |

### 6.3 Phase 1：最小闭环（2 周）

**目标**：每个 Action 都有审计日志

| 任务 | 验收标准 |
|-----|---------|
| 实现 openclaw-adapter | 能监听 tool:execute 事件 |
| 实现 AuditLogger | 每个 Action 写入 SQLite |
| 实现查询接口 | 能按 workspace/时间 查询日志 |

**可勘测指标**：
- 100% 的 Action 有审计记录
- 查询延迟 < 100ms

### 6.4 Phase 2：治理能力（3 周）

**目标**：高风险操作需要审批

| 任务 | 验收标准 |
|-----|---------|
| 实现 Risk Classifier | 能根据规则判断风险等级 |
| 实现 Policy Engine | 能配置白名单/黑名单规则 |
| 实现 Approval Engine | 审批请求能通过 IM 通知 |
| 实现 Security Hardening | 凭证加密存储 |

**可勘测指标**：
- 高风险操作 100% 进入审批
- 凭证无明文存储

### 6.5 Phase 3：任务管理（2 周）

**目标**：长任务可中断恢复

| 任务 | 验收标准 |
|-----|---------|
| 实现 Task Engine | 能创建/拆分/跟踪任务 |
| 实现 State Store | 检查点能保存/恢复 |
| 实现 Event Bus | 任务状态变更能触发事件 |

**可勘测指标**：
- 任务中断后能从检查点恢复
- 恢复后状态与中断前一致

### 6.6 Phase 4：多租户（2 周）

**目标**：不同 Workspace 完全隔离

| 任务 | 验收标准 |
|-----|---------|
| 实现 Workspace 模型 | 能创建/管理多个 Workspace |
| 实现数据隔离 | 查询自动带 workspace_id |
| 实现策略隔离 | 不同 Workspace 可配置不同规则 |
| 实现资源隔离 | 成本按 Workspace 统计 |

**可勘测指标**：
- Workspace A 无法访问 Workspace B 数据
- 成本统计按 Workspace 独立

### 6.7 Phase 5：行业包（2 周）

**目标**：新行业可快速接入

| 任务 | 验收标准 |
|-----|---------|
| 定义 Pack 规范 | 文档 + JSON Schema |
| 实现 Pack 加载器 | 配置即可加载新 Pack |
| 创建示例 Pack | 一个完整的行业包示例 |

**可勘测指标**：
- 新行业包可在 1 周内接入
- 无需修改底座代码

### 6.8 里程碑总览

```
Week 1     Week 3      Week 6       Week 8       Week 10      Week 12
  │          │           │            │            │            │
  ▼          ▼           ▼            ▼            ▼            ▼
┌────┐    ┌─────┐    ┌──────┐    ┌───────┐    ┌───────┐    ┌──────┐
│ P0 │───▶│ P1  │───▶│  P2  │───▶│  P3   │───▶│  P4   │───▶│  P5  │
│环境│    │审计 │    │治理  │    │任务   │    │多租户 │    │行业包│
└────┘    └─────┘    └──────┘    └───────┘    └───────┘    └──────┘
  │          │           │            │            │            │
  ▼          ▼           ▼            ▼            ▼            ▼
OpenClaw   最小闭环    企业可用    生产可用    复制就绪    商业就绪
可运行     可验证      可演示      可上线      可扩展      可销售
```

---

## 七、风险与应对

| 风险 | 概率 | 影响 | 应对措施 |
|-----|------|------|---------|
| OpenClaw 安全漏洞被利用 | 中 | 高 | Phase 1 即实现 Security Hardening |
| OpenClaw 大版本不兼容 | 低 | 高 | Adapter 层隔离，锁定版本 |
| 审批流阻塞业务 | 中 | 中 | 支持超时自动升级/自动通过配置 |
| 性能瓶颈 | 低 | 中 | 审计日志异步写入，批量处理 |
| 行业 Pack 无人买单 | 中 | 高 | 先做通用底座，找到客户再定制 |

---

## 八、技术选型

| 类别 | 选型 | 依据 |
|-----|------|------|
| 语言 | TypeScript | 与 OpenClaw 一致，复用类型 |
| 运行时 | Node.js 22+ | OpenClaw 要求 |
| 数据库 | SQLite（第一版） | 轻量，快速验证 |
| 缓存 | 内存（第一版） | 简化，后续可换 Redis |
| 消息队列 | Event Bus（内存） | 第一版简化，后续可换 |
| 加密 | Node.js crypto | 凭证加密存储 |

---

## 九、目录结构

```
openclaw-main/
├── packages/
│   ├── governance-core/          # 治理核心
│   │   ├── src/
│   │   │   ├── policy/           # Policy Engine
│   │   │   ├── risk/             # Risk Classifier
│   │   │   ├── approval/         # Approval Engine
│   │   │   ├── audit/            # Audit Logger
│   │   │   └── cost/             # Cost Budget
│   │   ├── package.json
│   │   └── README.md
│   │
│   ├── runtime-core/             # 运行时核心
│   │   ├── src/
│   │   │   ├── task/             # Task Engine
│   │   │   ├── state/            # State Store
│   │   │   ├── event/            # Event Bus
│   │   │   └── orchestrator/     # Agent Orchestrator
│   │   ├── package.json
│   │   └── README.md
│   │
│   ├── openclaw-adapter/         # OpenClaw 适配 + 安全加固
│   │   ├── src/
│   │   │   ├── gateway/          # Gateway Adapter
│   │   │   ├── channel/          # Channel Adapter
│   │   │   ├── session/          # Session Bridge
│   │   │   └── security/         # Security Hardening
│   │   ├── package.json
│   │   └── README.md
│   │
│   ├── workspace/                # 多租户隔离
│   │   ├── src/
│   │   │   ├── isolation/        # 数据/策略/资源隔离
│   │   │   └── management/       # Workspace 管理
│   │   ├── package.json
│   │   └── README.md
│   │
│   ├── industry-pack/            # 行业包框架
│   │   ├── src/
│   │   │   ├── loader/           # Pack 加载器
│   │   │   ├── validator/        # Pack 验证器
│   │   │   └── schema/           # Pack 规范定义
│   │   ├── packs/                # 示例行业包
│   │   │   └── demo-pack/
│   │   ├── package.json
│   │   └── README.md
│   │
│   └── shared/                   # 共享类型和工具
│       ├── src/
│       │   ├── types/            # 共享类型定义
│       │   ├── utils/            # 工具函数
│       │   └── db/               # 数据库抽象
│       ├── package.json
│       └── README.md
│
├── docs/
│   └── plans/
│       ├── original-architecture-plan.md
│       ├── 2026-03-20-enterprise-agent-runtime-design.md
│       └── 2026-03-20-technical-solution.md
│
└── ...（OpenClaw 原有文件）
```

---

## 十、参考资料

### 行业报告
- [State of AI Agents 2026 - Arcade.dev](https://www.arcade.dev/blog/5-takeaways-2026-state-of-ai-agents-claude/)
- [State of AI in Enterprise - Deloitte](https://www.deloitte.com/us/en/what-we-do/capabilities/applied-artificial-intelligence/content/state-of-ai-in-the-enterprise.html)
- [AI Governance Auditing Best Practices - Sweep](https://www.sweep.io/blog/ai-governance-auditing-best-practices-every-enterprise-needs-in-2026)

### OpenClaw 相关
- [OpenClaw Security Issues 2026](https://www.bitrue.com/blog/openclaw-ai-agent-security-issues-2026)
- [ClawJacked Vulnerability - The Hacker News](https://thehackernews.com/2026/02/clawjacked-flaw-lets-malicious-sites.html)
- [OpenClaw Review - Cybernews](https://cybernews.com/ai-tools/openclaw-review/)

### 监管要求
- [EU AI Act 2026 Guide - Sombra](https://sombrainc.com/blog/ai-regulations-2026-eu-ai-act)
- [AI Governance Framework - Ethyca](https://www.ethyca.com/news/ai-governance)

---

*文档生成日期：2026-03-20*
*版本：1.0*
