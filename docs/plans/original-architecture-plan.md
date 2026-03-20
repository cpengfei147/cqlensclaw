# 基于 OpenClaw 的 Agent Runtime 技术架构图 + 模块拆分清单 + 3 个月研发计划

## 1. 文档目标

本文档用于指导一个“基于 OpenClaw 的行业通用 Agent Runtime 底座”建设方案，适合作为 Claude Code 的输入材料，用于进一步生成：

- 技术方案细化
- 目录结构设计
- 模块代码骨架
- 任务拆分与研发计划
- API / SDK / 数据模型定义

本文档的核心思想不是“直接把 OpenClaw 当成企业平台”，而是：

> 用 OpenClaw 作为接入与执行骨架，用自研 Runtime Facade + Governance Layer 构建真正可复用的行业通用 Agent Runtime。

---

## 2. 总体建设目标

我们要建设的不是一个单一行业 Agent，而是一套 **行业通用的 Agent Runtime 底座**，用于承载不同垂直行业的 Agent 应用。

### 目标能力

- 支持多通道接入（Chat / IM / API / Web）
- 支持任务生命周期管理
- 支持工具调用与动作执行
- 支持审批、审计、策略治理
- 支持记忆与知识分层管理
- 支持多租户 / 多工作区
- 支持行业 Pack 机制
- 支持后续替换底层执行引擎，不与 OpenClaw 强耦合

### 定位

- **OpenClaw**：执行与接入骨架
- **自研 Runtime**：状态、治理、审计、审批、租户、行业抽象
- **Industry Pack**：行业流程、提示词、工具、SOP、审批模板

---

## 3. 技术架构图

```text
┌──────────────────────────────────────────────────────────────────────┐
│                         Product Surface / Access                     │
│  Web Console  |  Chat/IM Channels  |  API/SDK  |  Admin/Ops UI      │
└──────────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────────┐
│                    Runtime Control & Governance Layer                │
│  Task Orchestrator | Policy Engine | Approval Engine | Audit/Trace  │
│  Tenant/Workspace  | Artifact Svc  | Memory Svc      | Job Scheduler│
└──────────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────────┐
│                        Runtime Facade / Core API                     │
│  Agent API | Task API | Action API | Event API | Tool API | State API│
│  Standard internal contracts shielding business from OpenClaw internals│
└──────────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────────┐
│                        OpenClaw Adapter Layer                        │
│  Gateway Adapter | Channel Adapter | Skill Adapter | Browser Adapter │
│  Webhook Adapter | Cron Adapter    | Control UI Bridge | Routing Adp │
└──────────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────────┐
│                            OpenClaw Core                             │
│  Gateway (always-on) | sessions | channels | tools | events         │
│  WebSocket control plane | HTTP APIs | Control UI | hooks            │
│  multi-channel inbox | multi-agent routing | local-first execution   │
└──────────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────────┐
│                     External Systems / Execution Targets             │
│ CRM | ERP | Ticketing | Email | Browser | Files | MCP | Internal APIs│
└──────────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────────┐
│                       Industry Packs / Vertical Kits                 │
│ Roofing Pack | Energy Pack | Manufacturing Pack | Service Pack       │
│ SOPs | prompts | tools | approval templates | KPI models             │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 4. 架构设计原则

### 4.1 通用的是 Runtime，不是行业流程

底座应该通用化：

- 任务
- 状态
- 动作
- 工具调用
- 策略
- 审批
- 审计
- 记忆
- 工件
- 多租户

行业差异不应写入底层内核，而应沉淀在 Industry Pack 层。

### 4.2 业务层不能直接依赖 OpenClaw 内部实现

必须通过 Adapter 层隔离 OpenClaw。

原因：

- OpenClaw 是一个快速演进的项目
- 直接依赖其内部协议和结构，未来升级成本高
- 底层引擎未来可能替换，不应绑死

### 4.3 默认半自动，不默认全自动

企业级 Agent Runtime 应优先采用：

- 低风险动作自动执行
- 中风险动作建议后执行
- 高风险动作进入审批
- 异常动作升级人工接管

### 4.4 先借力，再抽象，再替换

建议分三阶段演进：

1. 借 OpenClaw 快速搭底座
2. 抽象出自研 Runtime API
3. 逐步替换部分底层能力

---

## 5. 模块拆分清单

建议拆成 8 个一级模块。

---

## 5.1 Runtime Core

负责统一内部语义和核心运行模型。

### 5.1.1 Agent Registry

职责：

- 注册 Agent
- 管理角色、能力标签、模型配置
- 管理 Agent 与 Workspace 的绑定关系

建议接口：

- `registerAgent()`
- `updateAgentProfile()`
- `getAgentById()`
- `listAgentsByWorkspace()`

### 5.1.2 Task Engine

职责：

- 创建任务
- 拆分任务
- 跟踪任务状态
- 支持长任务、恢复任务、多步骤任务

建议接口：

- `createTask()`
- `updateTaskStatus()`
- `resumeTask()`
- `cancelTask()`
- `listTaskActions()`

### 5.1.3 Action Executor

职责：

- 标准化工具调用
- 管理执行超时、重试、失败处理
- 记录 action 结果

建议接口：

- `executeAction()`
- `retryAction()`
- `markActionFailed()`

### 5.1.4 State Store

职责：

- 保存任务状态
- 保存会话状态
- 保存运行快照
- 用于恢复执行

---

## 5.2 OpenClaw Adapter Layer

负责把 OpenClaw 现有能力封装成内部稳定接口。

### 5.2.1 Gateway Adapter

职责：

- 封装 OpenClaw Gateway 通信
- 统一鉴权与连接管理
- 屏蔽原始协议细节

### 5.2.2 Channel Adapter

职责：

- 映射多消息渠道
- 把外部输入统一为标准消息事件

输入来源示例：

- Slack
- Teams
- Telegram
- Discord
- API Trigger
- Web Console

### 5.2.3 Skill Adapter

职责：

- 封装 OpenClaw skills
- 使其成为可审计、可治理的标准 Action

### 5.2.4 Browser Adapter

职责：

- 复用 OpenClaw 的 browser control 能力
- 封装网页自动化任务

### 5.2.5 Cron / Hook Adapter

职责：

- 支持定时触发
- 支持事件触发任务
- 把外部 webhook 纳入任务编排

### 5.2.6 Control UI Bridge

职责：

- 复用 OpenClaw Control UI 能力
- 同时接入自研 Ops Console

---

## 5.3 Governance Layer

这是企业运行时最关键的一层。

### 5.3.1 Policy Engine

职责：

- 工具白名单 / 黑名单
- 数据访问边界
- 模型使用限制
- 动作风险控制

规则示例：

- 某工作区禁止外部浏览器操作
- 某 Agent 不允许写 CRM，只允许读
- 某租户高风险动作必须二次审批

### 5.3.2 Approval Engine

职责：

- 高风险动作审批
- 人工驳回 / 批准 / 转交
- 保留审批记录

### 5.3.3 Audit & Trace

职责：

- 记录任务和动作执行轨迹
- 提供 trace、回放、问题排查能力
- 记录输入、输出、工具调用、耗时

### 5.3.4 Credential Vault

职责：

- 统一管理 API Key / Token / 系统凭证
- 避免凭证出现在日志和明文配置中

### 5.3.5 Risk Classifier

职责：

- 自动判断动作风险等级
- 输出建议执行路径：自动执行 / 审批 / 拒绝

---

## 5.4 Data & Memory Layer

### 5.4.1 Short-term Memory

职责：

- 当前任务上下文
- 最近会话和动作轨迹

### 5.4.2 Long-term Memory

职责：

- 用户长期偏好
- 组织习惯
- 历史任务摘要

### 5.4.3 Knowledge Connector

职责：

- 企业知识库接入
- SOP / FAQ / 文档索引
- 与记忆层分离

### 5.4.4 Artifact Store

职责：

- 存储任务输出产物
- 支持 Markdown / JSON / PDF / 截图 / 表单 / 工单

---

## 5.5 Multi-tenant & Workspace Layer

### 5.5.1 Tenant Manager

职责：

- 管理企业租户
- 资源隔离
- 配额和后续计费准备

### 5.5.2 Workspace Manager

职责：

- 组织部门 / 项目 / 业务线隔离
- 绑定不同 Agent 与策略

### 5.5.3 RBAC / Permission

职责：

- 管理员
- 操作员
- 审批人
- 观察者

---

## 5.6 External Integration Hub

### 5.6.1 System Connectors

目标系统示例：

- CRM
- ERP
- 工单系统
- 邮件系统
- IM
- 文件系统
- MCP Server
- 内部 API

### 5.6.2 Connector SDK

职责：

- 定义统一接入标准
- 标准化认证、错误处理、日志、限流

---

## 5.7 Industry Pack Layer

这是业务复用和商业复制的关键层。

每个 Industry Pack 包含：

- Prompt Pack
- SOP Pack
- Tool Pack
- Approval Template
- KPI Schema
- Output Template

行业示例：

- Roofing Pack
- Manufacturing Pack
- Energy Pack
- After-sales Service Pack

---

## 5.8 Product Surface

### 5.8.1 Ops Console

职责：

- 查看任务
- 查看审批
- 查看 trace
- 手动接管

### 5.8.2 Biz Console

职责：

- 业务触发任务
- 查看执行结果
- 处理异常

### 5.8.3 API / SDK Gateway

职责：

- 提供外部系统触发入口
- 支持系统对系统集成

### 5.8.4 Chat Surface

职责：

- 利用 OpenClaw 多通道入口承接人机交互

---

## 6. 推荐数据对象标准

建议第一版先定义 7 个核心对象。

### 6.1 Agent

```json
{
  "agent_id": "string",
  "tenant_id": "string",
  "workspace_id": "string",
  "role": "string",
  "capabilities": ["string"],
  "model_profile": "string",
  "policy_profile": "string"
}
```

### 6.2 Task

```json
{
  "task_id": "string",
  "type": "string",
  "input": {},
  "priority": "normal",
  "status": "pending",
  "deadline": null,
  "owner": "string"
}
```

### 6.3 Action

```json
{
  "action_id": "string",
  "task_id": "string",
  "tool_name": "string",
  "params": {},
  "result": {},
  "risk_level": "low",
  "duration_ms": 0
}
```

### 6.4 Artifact

```json
{
  "artifact_id": "string",
  "task_id": "string",
  "type": "markdown",
  "schema": "string",
  "uri": "string",
  "version": 1
}
```

### 6.5 Event

```json
{
  "event_id": "string",
  "type": "string",
  "source": "string",
  "correlation_id": "string",
  "timestamp": "string"
}
```

### 6.6 Policy

```json
{
  "policy_id": "string",
  "allowed_tools": ["string"],
  "restricted_scopes": ["string"],
  "approval_rules": ["string"],
  "model_constraints": ["string"]
}
```

### 6.7 Approval

```json
{
  "approval_id": "string",
  "task_id": "string",
  "approver": "string",
  "decision": "approved",
  "reason": "string",
  "timestamp": "string"
}
```

---

## 7. 建议技术选型

### 第一版建议

- 控制与接入底层：OpenClaw Gateway
- 服务端语言：TypeScript 或 Python
- Runtime Facade / Orchestration：自研轻量服务
- 数据库：PostgreSQL
- 缓存 / 队列：Redis
- 对象存储：MinIO / S3 Compatible
- 前端：React / Next.js
- 鉴权：Keycloak / Auth0 / 自有 IAM

### 建议原则

- 第一版优先复用 OpenClaw 的接入与执行能力
- 第一版不重写 Control Plane
- 第一版重点补齐治理、状态、审批、审计
- 第二阶段再逐步降低对 OpenClaw 的强依赖

---

## 8. 3 个月研发计划

---

## 第 1 个月：把底座跑起来

### 第 1-2 周

目标：建立基础骨架

交付内容：

- OpenClaw Gateway 基础部署
- Runtime Facade 项目骨架
- Gateway Adapter / Channel Adapter 原型
- 统一对象模型：Agent / Task / Action / Event / Artifact
- PostgreSQL + Redis 初始化

验收标准：

- 能从一个消息渠道或 API 入口触发任务
- 能通过 Facade 调用 OpenClaw 并记录 Action
- 数据库中可看到任务状态流转

### 第 3-4 周

目标：完成最小可用 Runtime

交付内容：

- Task Engine v1
- Action Executor v1
- Audit & Trace v1
- Ops Console 最小版
- 一个 Demo Connector（例如 CRM 或内部 API）

验收标准：

- 可查看任务列表、动作列表、失败原因
- 工具调用结果有日志
- 失败任务可人工重试

---

## 第 2 个月：把治理补齐

### 第 5-6 周

目标：引入企业必需的可控能力

交付内容：

- Policy Engine v1
- Approval Engine v1
- Risk Classifier v1
- Credential Vault v1

验收标准：

- 高风险动作不能直接执行
- 审批流可工作
- 凭证不出现在业务日志中

### 第 7-8 周

目标：做可持续运行能力

交付内容：

- Memory Service v1
- Artifact Store v1
- Cron / Hook Adapter
- 恢复与重试机制
- 任务超时与补偿逻辑

验收标准：

- 长任务可跨会话保留状态
- 定时任务可运行
- 每个任务都能落地为 Artifact 或结构化输出

---

## 第 3 个月：行业化与多租户化

### 第 9-10 周

目标：让底座具备复制能力

交付内容：

- Tenant Manager v1
- Workspace Manager v1
- RBAC v1
- Connector SDK v1
- Industry Pack Framework v1

验收标准：

- 至少支持 2 个 Workspace
- 不同 Workspace 有不同工具权限
- 行业 Pack 可通过配置挂载

### 第 11-12 周

目标：交付第一个行业包

交付内容：

- 一个完整行业包，例如：
  - Roofing Pack
  - Manufacturing Inspection Pack
  - Energy Forecast Pack
- Biz Console v1
- API / SDK v1
- 试点客户演示环境

验收标准：

- 从业务触发到产出结果形成闭环
- 有审批、有日志、有工件
- 第二个行业包可复用 70% 以上底座代码

---

## 9. 3 个月后应达到的结果

3 个月后，应拿到的不是“一个更大的 OpenClaw”，而是：

### 平台资产

- Runtime Facade
- Governance Layer
- Tenant + Workspace 模型
- Connector SDK
- Industry Pack 框架
- 可演示的试点环境

### OpenClaw 的角色

- Gateway / Control Plane Provider
- 多通道接入 Provider
- Skills / Browser / Hooks Provider
- 第一阶段执行骨架

---

## 10. 最容易踩的坑

### 10.1 直接改 OpenClaw 源码做业务

问题：

- 升级困难
- 维护成本高
- 容易与上游版本冲突

建议：

- 始终通过 Adapter 封装

### 10.2 一开始就做全行业万能平台

问题：

- 范围失控
- 需求不断膨胀
- 很难形成闭环

建议：

- 先做底座共性 + 一个行业 Pack

### 10.3 没有审批和审计就上企业场景

问题：

- 容易出错
- 无法追责
- 客户不放心

建议：

- 默认高风险动作必须审批

### 10.4 把记忆、知识库、工件混在一起

问题：

- 无法治理
- 无法优化
- 无法分层存储

建议：

- 分离为 Memory / Knowledge / Artifact 三层

### 10.5 只做聊天入口，不做 API 入口

问题：

- 无法融入企业系统流程
- 难以系统对系统协作

建议：

- 第一版就保留 API Trigger 能力

---

## 11. 推荐给 Claude Code 的下一步任务

可以基于本文件继续生成以下内容：

### 11.1 代码仓结构

建议生成：

- monorepo 目录结构
- runtime-core 模块
- openclaw-adapter 模块
- governance 模块
- memory 模块
- connector-sdk 模块
- industry-pack 示例模块
- ops-console / biz-console 前端结构

### 11.2 接口定义

建议生成：

- Task API
- Action API
- Policy API
- Approval API
- Artifact API
- Connector SDK 接口规范

### 11.3 数据库设计

建议生成：

- PostgreSQL 表结构
- 索引建议
- 审计日志表
- 任务状态表
- 工件表
- 审批表

### 11.4 首个行业 Pack 示例

建议优先生成：

- Roofing Pack
或
- Manufacturing Inspection Pack

### 11.5 研发任务拆解

建议生成：

- 12 周 Sprint Backlog
- 每周交付项
- 风险项
- 验收标准

---

## 12. 一句话总结

> 基于 OpenClaw 做行业通用 Agent Runtime 的正确方式，不是把 OpenClaw 直接包装成企业平台，而是把它作为“接入与执行骨架”，在其上构建你们自己的 Runtime Facade、Governance Layer、Tenant Layer 与 Industry Pack 体系。

