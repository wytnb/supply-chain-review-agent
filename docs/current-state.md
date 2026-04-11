# 当前状态

## 已完成

- 项目已形成可导入 Dify 的 Workflow MVP DSL，当前工作流已包含 `Start`、`DocExtractor_Contract`、`DocExtractor_Quotation`、`DocExtractor_PO`、`LLM_Extract_Contract`、`LLM_Extract_Quotation`、`LLM_Extract_PO`、`Code_Normalize_Documents`、`Code_Match_Line_Items`、`Iteration_Line_Item_Check`、`Code_Header_And_Clause_Check`、`Code_Aggregate_Findings`、`Template_Render_Report`、`End`。
- 现有抽取节点采用 `Ollama / qwen3:8b` 作为 LLM 提供方，并使用结构化输出承载三单抽取结果。
- 抽取层已经显式包含 `*_raw / *_norm`、`low_confidence_fields`、关键条款布尔位等中间字段，后续规则层可以直接消费这些结果。
- 样本组 `group_01` 到 `group_20` 已经把 `PASS / REJECT / MANUAL_REVIEW` 的验收边界、正式风险与人工复核场景锁定。
- 已完成本地静态检查：YAML 可解析、文件无乱码占位符、新增节点和连线存在、Code 节点代码均带算法说明注释。
- 已完成 6 个代表场景的本地规则 smoke test：`G01`、`G03`、`G08` 输出 `REJECT`，`G18` 输出 `PASS`，`G19`、`G20` 输出 `MANUAL_REVIEW`。

## 进行中

- 当前状态应准确表述为“Dify Workflow DSL 已补全，待 Dify 导入与节点运行验证”，不再只是纯文档阶段。
- 当前实现事实已经锁定为 `Dify Workflow + Ollama/qwen3:8b` 的初审链路，但人工复核与留痕仍在 Dify 外部流程中完成。
- canonical 结论值、规则编码与中文业务名的映射已经定义清楚，但还需要通过最终文档和工作流节点串联到同一条执行链。

## 阻塞项

- 尚未完成 Dify 导入验证，因此还不能确认导入后的节点运行形状、Iteration 输出消费和 Template/End 最终表现是否与本地验证一致。
- 尚未做 20 组样本的全量跑批验证，因此当前只确认了代表场景，不代表全部样本都已验收通过。
- 人工复核界面、留痕存储和正式审批回写仍未纳入当前 Dify 工作流，相关闭环要由外部系统承担。

## 下一步

- 将当前 DSL 导入 Dify，验证节点连线、Iteration 输出、Template 渲染和 End 输出是否与设计一致。
- 用 20 组样本做全量跑批，确认 `PASS / REJECT / MANUAL_REVIEW`、规则编码和报告内容均符合预期。
- 在 Dify 导入后补充验证记录，把导入结果、节点输出形状、代表样本和全量样本结果写入 `docs/verification-log.md`。
- 继续把系统级事实沉淀到 `docs/system-overview.md`、`docs/decision-log.md`、`docs/business-knowledge.md`，避免实现口径和样本口径分裂。

## 给下一位接手者的提示

- 当前文档里的系统描述已经不是“仅有方案定义”，而是“工作流原型已存在，规则链路待闭环”。
- 如果继续推进实现，优先保持固定三单输入、8 类正式风险、2 类人工复核规则、三态主结论和外部人工复核闭环不变。
- 任何实现、验证或结论边界的变化，都要同步更新 `docs/current-state.md` 和 `docs/verification-log.md`。
