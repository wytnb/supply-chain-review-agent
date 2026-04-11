# 系统概览

## 摘要

- 系统用途：对采购合同、供应商报价单和 PO 执行字段抽取、标准化、跨文档比对、规则审查和审查报告生成。
- 当前架构成熟度：`Dify Workflow DSL 已补全，待 Dify 导入与运行验证`
- 当前实现事实：抽取层已接入 `Dify Workflow + Ollama/qwen3:8b`，后续判断层由 `Code` 和 `Iteration` 节点承接。

## 节点级数据流

1. `Start`
   - 输入：`contract_file`、`quotation_file`、`po_file`、`review_task_id`、`reviewer_role`
   - 输出：审查上下文、文件句柄和任务标识
   - 作用：接收三单输入，校验任务级上下文，准备进入文档提取链路
2. `DocExtractor_Contract` / `DocExtractor_Quotation` / `DocExtractor_PO`
   - 输入：单个原始文件
   - 输出：对应文档的可读文本
   - 作用：把 PDF、DOCX、XLSX、CSV 等文本型文档转成下游可消费的文本内容
3. `LLM_Extract_Contract` / `LLM_Extract_Quotation` / `LLM_Extract_PO`
   - 输入：上一步文本内容
   - 输出：结构化抽取对象，包含文档头部、行项目、关键条款、`low_confidence_fields` 和 `*_raw / *_norm` 字段
   - 作用：只负责抽取和低置信度标注，不直接下正式风险结论
4. `Code_Normalize_Documents`
   - 输入：3 份结构化抽取结果，以及 `review_task_id`、`reviewer_role`
   - 输出：`normalized_documents`、`normalization_warnings`、`document_overview`
   - 作用：统一空白、全半角、币种、税率、数量、单价、日期、付款条件、交付字段，并生成行项目 `match_key`
5. `Code_Match_Line_Items`
   - 输入：`normalized_documents`
   - 输出：`matched_items`
   - 作用：按 `item_code` 优先、否则 `item_name + spec_model` 的规则对齐三单行项目；无法唯一匹配时只保留人工复核候选
6. `Iteration_Line_Item_Check`
   - 输入：`matched_items`
   - 输出：逐行检查结果数组
   - 作用：对每个匹配行项执行同一套确定性规则，产出 `item_findings`、`item_manual_flags`、`item_evidence`
7. `Code_Header_And_Clause_Check`
   - 输入：`normalized_documents`、`normalization_warnings`
   - 输出：`header_findings`
   - 作用：检查供应商、付款条件、币种、税率、交期和关键条款等头部字段差异
8. `Code_Aggregate_Findings`
   - 输入：逐行检查结果、头部检查结果、`document_overview`
   - 输出：`review_report_json`
   - 作用：扁平化结果、去重、排序、生成 `final_decision`、`findings`、`summary` 和 `audit_stub`
9. `Template_Render_Report`
   - 输入：`review_report_json`
   - 输出：`review_report_markdown`
   - 作用：把已确定的 JSON 结果渲染为固定结构的审查报告
10. `End`
    - 输入：`review_report_markdown`、`review_report_json`
    - 输出：同名最终结果
    - 作用：作为 Workflow 的最终输出节点，提供给前端或后端 API 调用

## 核心对象

- `DocumentPackage`
  - 固定包含 `contract`、`quotation`、`po` 三类文档
  - 缺任一文档时不进入正式审查
- `ExtractedDocument`
  - 统一抽取对象，覆盖文档编号、主体、商务条款、收货地点、交期、行项目和关键条款
- `LineItem`
  - 统一行项目对象，用于跨文档匹配与差异判断
- `ReviewFinding`
  - 单条发现对象，区分 `risk`、`manual_review_flag`、`info`
- `ReviewReport`
  - 最终审查输出对象，承载主结论、比对结果、风险、人工复核事项和审查留痕
- `FinalDecision`
  - 固定枚举：`PASS`、`REJECT`、`MANUAL_REVIEW`

## 决策边界

- LLM 只负责抽取、结构化和低置信度标注，不负责正式风险裁决。
- 正式风险与主结论由 `Code` / `Iteration` 节点按确定性规则输出。
- `Template` 只负责把确定后的 JSON 渲染成 Markdown 报告，不参与判定。
- 人工复核、责任人记录和正式写回流程在 Dify 外部完成，Dify 只输出审查结果和 `audit_stub`。
- 行项目匹配无法唯一确定时，不允许自动下正式风险结论，只能进入人工确认。
- 近似名、自然语言条款等边界情况不做激进自动归一，V1 以保守审查为主。

## 边界与集成

- 外部系统：
  - V1 不接入 ERP、OA、SRM 的正式写回流程。
  - V1 不扩展到发票、收货单、入库单等第四类及以上单据。
- 接口或 API：
  - 当前只锁定对象模型和流程边界，尚未定义外部 API 协议。
  - 后续实现应围绕 `DocumentPackage` 输入和 `ReviewReport` 输出展开。
- 涉及的文件、数据集或仓库：
  - PRD：`tasks/PRD_V1_供应链文件审查Agent.md`
  - 样本：`tasks/合同_报价单_PO 样本组/group_01` 到 `group_20`

## 运行说明

- 部署或运行时假设：
  - 项目定位为私有化部署样板，默认运行在企业内网或本地演示环境。
  - 当前 workflow DSL 已完成静态校验与代表样本 smoke test，但尚未完成 Dify 导入与全链路验证。
- 失败或降级策略：
  - 文档缺失、字段抽取不完整或行项目无法唯一匹配时，应停止自动放行并转入人工确认或重新发起审查。
  - 对自然语言条款的语义不确定场景，应优先降级到 `MANUAL_REVIEW`。
- 可观测性或留痕说明：
  - 每次审查需要记录审查任务 ID、文档版本或哈希、模型版本、规则版本、AI 初审结论、人工复核人、人工复核结论、最终结论和时间戳。
