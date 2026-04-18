# 项目 README

## 项目概览

- 项目名称：`供应链采购合同与 PO 审查 Agent`
- 一句话说明：面向企业内部采购审查流程的业务型 AI Agent 样板项目，用于对合同、报价单、PO 三单做字段抽取、一致性校验、风险识别、人工复核和留痕。
- 主要负责人或团队：当前未指定，按样板项目方式维护。
- 当前阶段：`文档对齐 / 方案定义`

## 项目要做什么

本项目聚焦制造业与贸易型企业的采购审查环节。系统接收采购合同、供应商报价单和采购订单三类文档，抽取关键信息，按统一规则完成跨文档比对，并输出结构化审查报告。

V1 的重点不是做聊天机器人，也不是做自动审批，而是展示一个可解释、可复核、可留痕的业务型 Agent 闭环：`AI 初审 + 人工复核 + 审查留痕`。项目同时服务于作品集展示、样板项目沉淀和后续行业方案复用。

## 如何开始
### 人类
下载`tasks/供应链采购文件审查agent.yml`并导入Dify，在Dify中正常使用
### AI
1. 阅读 `AGENTS.md`。
2. 阅读 `docs/project-brief.md` 了解业务背景、目标和范围。
3. 阅读 `docs/current-state.md` 了解当前进展、缺口和下一步。
4. 如果任务涉及架构、边界或对象模型，阅读 `docs/system-overview.md`。
5. 如果任务涉及业务规则、字段语义或风险口径，阅读 `docs/business-knowledge.md`。
6. 在默认假设“已经验证过”之前，先查看 `docs/verification-log.md`。

## 文档地图

- `docs/project-brief.md`：PRD 级业务背景、V1 目标、范围、约束和当前里程碑。
- `docs/business-knowledge.md`：三类文档语义、风险口径、人工复核边界、领域术语和默认业务假设。
- `docs/system-overview.md`：V1 目标架构、核心对象、Agent 分工、数据流、边界与集成约束。
- `docs/current-state.md`：当前已完成的文档化工作、未实现项、阻塞项与下一步。
- `docs/decision-log.md`：已锁定的 V1 产品与方案决策。
- `docs/verification-log.md`：文档复核、测试、跳过项与已知未验证范围。
- `memory/OBSERVATIONS.md`：项目短期记忆与后续会话提示。
- `tasks/合同_报价单_PO 样本组/`：20 组标准样本与边界样本。

## 工作约定

- 以上文档默认视为当前项目的事实源。
- 当需求、范围、架构边界或验证状态发生变化时，需要在同一任务里同步更新文档。
- 与实现状态相关的事实优先写入 `docs/current-state.md` 和 `docs/verification-log.md`，不要把目标设计误写成已实现能力。
- 与业务长期稳定相关的事实优先写入 `docs/business-knowledge.md`。
