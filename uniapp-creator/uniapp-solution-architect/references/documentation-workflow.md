# 文档工作流

## 读文档顺序

1. `docs/project-system.md`
2. 当前任务卡
3. 相关设计文档
4. 如涉及数据，再看数据文档
5. 如涉及流程或平台分支，再看流程文档

## 设计输出至少要落下什么

- `docs/AGENTS.md`
- `docs/design/best-practices.md`
- 总体设计
- 技术约定
- `docs/design/api-contract.md`
- `docs/design/ui-guidelines.md`
- `docs/design/admin.md`（如涉及后台）
- 开发清单
- 当前轮次下一步
- 数据文档要求（如有）
- Mermaid 要求（如有）

## 输出目录规则

- `docs/superpowers/specs/`、`docs/superpowers/plans/` 只放 superpowers 过程文档
- 目标项目最终文档一律分类放到 `docs/` 下
- 任务卡默认放在 `docs/tasks/`
- 如果目标项目最终需要仓库根目录的 `AGENTS.md`，以 `docs/AGENTS.md` 为单一事实来源
- 通用型产品先输出 `docs/design/best-practices.md`，再输出细节设计
- 关键设计决策不能只留在 `docs/superpowers/`；最终 `docs/` 必须自包含
- 只要涉及界面，`docs/design/ui-guidelines.md` 必须包含可直接给 AI 使用的页面提示词
- 最终 `docs/` 默认按固定章节骨架产出，不要自由发挥文档结构

## 执行完成后至少回写什么

1. 任务卡执行记录
2. `docs/project-system.md`
3. `docs/tracking/change-log.md`
4. 如有数据变化，更新数据文档
5. 如有流程变化，更新流程文档
6. 如有接口变化，更新 `docs/design/api-contract.md`
7. 如有后台结构变化，更新 `docs/design/admin.md`
8. 如有界面规则或 AI 美化提示词变化，更新 `docs/design/ui-guidelines.md`
9. 如有开发规则变化，更新 `docs/AGENTS.md`

## 如何和后续执行衔接

设计文档里必须直接写明：
- 下一步做哪个模块或步骤
- 适合先 `superpowers:writing-plans` 还是直接 `superpowers:executing-plans`
- 哪些步骤更适合 TDD
- 哪些问题已经按最佳实践直接落结论，哪些问题仍待用户确认
- `docs/AGENTS.md` 是否要同步到根目录 `AGENTS.md`，以及同步时机

## 设计结束前自检

- 不要只检查文件是否存在，要检查章节是否齐全
- 不要只看 `docs/superpowers/` 过程稿，要检查最终 `docs/`
- 如果还是旧骨架，先升级文档，再结束设计
- `overview.md` 中只允许一套目录结构
- `data-model.md` 必须使用 uniCloud 官方 DB Schema 格式（`properties: {}`，不是 `columns: []`）
- 每张任务卡的目标目录名必须与 `overview.md` 建项结论中的项目名一致
- 跨文档一致性：api-contract 中的云对象 ↔ data-model 中的集合、overview 项目名 ↔ 任务卡目标目录、flows 中的调用 ↔ api-contract 中的方法

## 跨文档一致性校验步骤

设计完成前，按以下步骤执行跨文档一致性校验：

1. 从 `api-contract.md` 提取所有云对象/云函数方法及其操作的集合名
2. 逐一在 `data-model.md` 中确认对应集合定义存在
3. 从 `overview.md` 提取项目名（如 `rc-transcribe-client`、`rc-transcribe-admin`）
4. 逐一在所有任务卡中确认 `## 目标仓库或目录` 引用的项目名一致
5. 从 `flows.md` 的 Mermaid 图中提取所有云对象/云函数调用
6. 逐一在 `api-contract.md` 中确认对应方法存在
7. 如果发现不一致，以 data-model.md 和 overview.md 为基准，补全或修正缺失方
