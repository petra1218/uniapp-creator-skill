# 文档工作流

## 读文档顺序

1. 如当前轮次还没有实现计划，先生成或读取 `docs/superpowers/plans/YYYY-MM-DD-<topic>.md`
2. `docs/project-system.md`
3. 当前任务卡
4. 相关设计文档
5. 如存在 `docs/design/best-practices.md`，先读它
6. 如有数据变化风险，读数据文档
7. 如有流程变化风险，读流程文档

## 输出目录规则

- `docs/superpowers/specs/`、`docs/superpowers/plans/` 只放 superpowers 过程文档
- 目标项目最终文档一律分类放到 `docs/` 下
- 任务卡默认放在 `docs/tasks/`
- `docs/AGENTS.md` 是开发 AI 规则文档的单一事实来源
- `docs/design/best-practices.md` 是通用型产品默认模式与主动取舍的单一事实来源
- 不要把只存在于 `docs/superpowers/` 的内容当成最终事实来源
- 涉及界面实现时，`docs/design/ui-guidelines.md` 中的页面级提示词是事实来源之一
- 如果最终 `docs/` 还是旧骨架，先补骨架，再实施

## 每轮执行后至少回写

1. 任务卡执行记录
2. `docs/project-system.md`
3. `docs/tracking/change-log.md`
4. 数据文档（如有）
5. 流程文档（如有）
6. 接口文档（如有）
7. 后台文档（如有）
8. 界面文档（如有）
9. `docs/AGENTS.md`（如规则有变化）

## 回写时必须写清楚

- 修改成果
- 影响文件
- 当前版本号
- 下一步建议
- `docs/AGENTS.md` 的根目录同步状态

## 执行计划来源

- 当前需求没有实施计划：先使用 `superpowers:writing-plans`
- 已有实施计划并进入落地：显式使用 `superpowers:executing-plans` 或 `superpowers:subagent-driven-development`
- 不要跳过计划链路直接实现
- 如果实现需要偏离最佳实践摘要，先回退补设计，不要边做边改口径
- 如果任务卡还不足以直接实施，先展开成可执行计划
- 如果页面提示词缺失，先补 `docs/design/ui-guidelines.md`
- 如果 URL 化入口合同、异常流程分支、任务卡执行方式等缺失，先补最终文档
