# 企业级 Agent Runtime 架构审视与落地路径

> 基于湘牛招投标项目的架构审视和 8 周落地计划

## 一、背景

### 1.1 项目定位

- **底座目标**：企业级 Agent Runtime 通用底座
- **验证项目**：湘牛招投标 AI 智能体
- **策略**：用湘牛项目验证底座，边交付边沉淀

### 1.2 湘牛需求概要

| 能力 | 说明 |
|------|------|
| 标讯监控 | 7x24 扫描招标网站，智能匹配推送 |
| 标书生成 | 基于历史标书生成初稿，格式检查 |
| 智能分析 | 竞争对手分析、报价建议 |
| 流程管家 | 节点跟踪、提醒、闭环 |
| 私有部署 | 数据不出域 |

---

## 二、架构审视发现的问题

### 2.1 原技术方案的盲区

| 盲区 | 影响 |
|------|------|
| **身份体系缺失** | 不知道"谁在用"，权限无从谈起 |
| **知识库权限缺失** | 员工能看到不该看的文档 |
| **可观测性缺失** | 出问题难定位 |

### 2.2 OpenClaw 现有能力局限

| OpenClaw 能做 | 企业需要但 OpenClaw 没有 |
|--------------|------------------------|
| `allowFrom` 白名单控制"谁能用" | 控制"谁能做什么"、"谁能看什么" |
| 按渠道独立配置 | 统一的 RBAC 权限模型 |
| 简单字符串匹配 | 角色、部门、项目多维度权限 |

### 2.3 关键认知

> **企业内部员工数据隔离** 和 **Tools/Skills/RAG 的分层权限** 是企业级底座的核心问题，不是某个客户的特殊需求。

---

## 三、修正后的架构

### 3.1 架构全景图

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Product Surface                                  │
│           Web Console │ Chat/IM │ API/SDK │ Admin UI                    │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      Identity & Access Layer                             │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                  │
│   │   Identity   │  │     RBAC     │  │   Session    │                  │
│   │   Provider   │  │    Engine    │  │  Enricher    │                  │
│   │ (企微/钉钉)   │  │  (角色权限)   │  │ (上下文注入) │                  │
│   └──────────────┘  └──────────────┘  └──────────────┘                  │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    Governance & Runtime Core                             │
│   ┌─────────────────────────────┐  ┌─────────────────────────────┐      │
│   │      Governance Gate        │  │       Runtime Core          │      │
│   │  Policy │ Risk │ Approval   │  │  Task │ State │ Event Bus   │      │
│   │  Audit  │ Cost Budget       │  │  Agent Orchestrator         │      │
│   └─────────────────────────────┘  └─────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                       Knowledge & RAG Layer                              │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                  │
│   │   Document   │  │    Access    │  │   Knowledge  │                  │
│   │    Tagger    │  │    Filter    │  │    Scope     │                  │
│   │  (文档打标)   │  │  (权限过滤)   │  │ (四级隔离)   │                  │
│   └──────────────┘  └──────────────┘  └──────────────┘                  │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      OpenClaw Adapter Layer                              │
│            + Security Hardening (凭证加密、输入审查、输出过滤)            │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          OpenClaw Core                                   │
│        Gateway │ Channels │ Sessions │ Tools │ Hooks │ Events           │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
│                         External Systems                                 │
│            CRM │ ERP │ MCP │ Browser │ Email │ APIs │ RAG               │
└ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘

═══════════════════════════════════════════════════════════════════════════
                              横切关注点
═══════════════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────────────┐
│                    Workspace Isolation（多租户隔离）                      │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                      Industry Packs（行业能力包）                         │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                     Observability（可观测性）                             │
└─────────────────────────────────────────────────────────────────────────┘
```

### 3.2 新增模块

| 模块 | 职责 | 优先级 |
|------|------|--------|
| `identity/` | 身份接入 + RBAC + Session 上下文 | P0 |
| `knowledge/` | 知识库分层 + 权限过滤 | P0 |
| `observability/` | 日志 + 追踪 + 指标 | P0 |

### 3.3 目录结构

```
packages/
├── identity/                 # 身份与权限
│   ├── src/
│   │   ├── provider/         # Identity Provider (企微/钉钉)
│   │   ├── rbac/             # RBAC Engine
│   │   ├── context/          # User Context & Session Enricher
│   │   └── index.ts
│   └── package.json
│
├── knowledge/                # 知识库权限
│   ├── src/
│   │   ├── scope/            # Knowledge Scope (四级隔离)
│   │   ├── filter/           # Access Filter
│   │   ├── tagger/           # Document Tagger
│   │   └── index.ts
│   └── package.json
│
├── observability/            # 可观测性
│   ├── src/
│   │   ├── logger/           # Structured Logger
│   │   ├── trace/            # Trace Context
│   │   ├── metrics/          # Metrics Collector
│   │   └── index.ts
│   └── package.json
│
├── governance-gate/          # 治理核心
├── runtime-core/             # 运行时核心
├── openclaw-adapter/         # OpenClaw 适配
├── workspace/                # 多租户隔离
├── industry-pack/            # 行业包框架
└── shared/                   # 共享模块
```

---

## 四、配置化 RBAC 设计

### 4.1 设计原则

- 不硬编码，配置即可
- 换客户只需换配置文件
- 后续可加管理界面生成配置

### 4.2 配置文件格式

```yaml
# config/permissions.yaml
roles:
  - name: admin
    permissions: ["*"]

  - name: manager
    permissions:
      - "project:*"
      - "document:read"
      - "bid:view"

  - name: staff
    permissions:
      - "project:own"
      - "document:read:public"
      - "bid:view:own"

users:
  - id: "wecom:zhangsan"
    roles: ["staff"]
    projects: ["proj-001", "proj-002"]

  - id: "wecom:lisi"
    roles: ["manager"]
    department: "招投标部"
```

### 4.3 核心接口

```typescript
interface UserContext {
  userId: string;           // "wecom:zhangsan"
  name: string;
  roles: string[];          // ["staff"]
  permissions: string[];    // ["project:own", "document:read:public"]
  projects?: string[];      // ["proj-001", "proj-002"]
  department?: string;
}

class RBACEngine {
  constructor(configPath: string);
  getUserContext(userId: string): UserContext;
  hasPermission(user: UserContext, permission: string): boolean;
  filterByScope(user: UserContext, resources: Resource[]): Resource[];
}
```

---

## 五、知识库权限设计

### 5.1 四级隔离

| 级别 | 说明 | 示例 |
|------|------|------|
| 企业级 | 全员可见 | 公司制度、产品手册 |
| 部门级 | 本部门可见 | 部门流程、培训资料 |
| 项目级 | 项目成员可见 | 投标资料、报价方案 |
| 个人级 | 仅自己可见 | 个人笔记、草稿 |

### 5.2 数据模型

```typescript
interface Document {
  id: string;
  content: string;
  scope: "enterprise" | "department" | "project" | "personal";
  scopeId?: string;
  metadata: {
    source: string;
    uploadedBy: string;
    uploadedAt: Date;
  };
}
```

### 5.3 检索逻辑

```typescript
function queryWithPermission(query: string, user: UserContext): Document[] {
  const allowedScopes = [
    { scope: "enterprise" },
    { scope: "department", scopeId: user.department },
    { scope: "project", scopeId: user.projects },
    { scope: "personal", scopeId: user.userId },
  ];

  return ragSearch(query, { filter: allowedScopes });
}
```

---

## 六、湘牛交付线 — 8 周路线图

### 6.1 总览

```
Week 2        Week 4        Week 6        Week 8
  │             │             │             │
  ▼             ▼             ▼             ▼
┌─────┐      ┌─────┐      ┌─────┐      ┌─────┐
│身份 │ ───▶ │知识库│ ───▶ │招投标│ ───▶ │治理 │
│权限 │      │RAG  │      │功能 │      │流程 │
└─────┘      └─────┘      └─────┘      └─────┘
```

### 6.2 Week 1-2：环境 + 身份 + 配置化权限

| 任务 | 交付物 |
|------|--------|
| 部署 OpenClaw | 能收发消息 |
| 对接企业微信 | 获取用户ID |
| 配置化 RBAC | `permissions.yaml` + 规则引擎 |
| Session 注入 | 每个请求携带 UserContext |

**验收标准**：
- 员工发消息 → 系统识别身份 → 返回正确的 UserContext
- 配置文件修改后，权限立即生效

### 6.3 Week 3-4：知识库 + 权限过滤的 RAG

| 任务 | 交付物 |
|------|--------|
| 文档导入 + 打标 | 每个文档有 scope 标签 |
| 权限过滤检索 | RAG 查询按 UserContext 过滤 |
| 基础问答 | 能回答企业知识问题 |

**验收标准**：
- 普通员工查询 → 只返回授权文档
- 管理员查询 → 返回所有文档

### 6.4 Week 5-6：招投标核心功能

| 任务 | 交付物 |
|------|--------|
| 标讯监控 | 定时抓取，智能匹配推送 |
| 标书辅助 | 基于历史标书生成初稿 |
| 项目绑定 | 员工只能操作自己负责的项目 |

### 6.5 Week 7-8：治理 + 可观测

| 任务 | 交付物 |
|------|--------|
| 审计日志 | 记录谁在什么时候做了什么 |
| 任务跟踪 | 投标流程节点提醒 |
| 基础日志 | 系统运行日志 |

---

## 七、技术选型

| 组件 | 选型 | 理由 |
|------|------|------|
| 权限配置 | YAML 文件 | 简单、可读、易修改 |
| 规则引擎 | 简单 JS 实现 | 第一版不需要复杂引擎 |
| RAG | LanceDB / Chroma | 轻量、支持过滤 |
| 文档解析 | LangChain loaders | 支持多格式 |
| 日志 | Winston / Pino | 成熟、结构化 |

---

## 八、风险与应对

| 风险 | 应对 |
|------|------|
| 企业微信对接复杂 | 先用 OpenClaw 现有渠道能力，不重新开发 |
| 权限规则复杂 | 第一版简化为两级（企业/项目），后续再扩展 |
| RAG 效果不好 | 先用成熟方案（LanceDB），效果不行再调优 |
| 招标网站反爬 | 先做 1-2 个网站，控制频率 |

---

*文档生成日期：2026-03-20*
*版本：1.0*
