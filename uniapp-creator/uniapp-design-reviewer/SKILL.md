---
name: uniapp-design-reviewer
description: 用于 uniapp-creator 的设计与执行阶段之间。在 solution-architect 完成设计输出后、delivery-engineer 开始执行前，对 docs/ 下所有设计文档进行跨文档一致性审查，自动修复检测到的问题，输出审查修复日志。触发词：审查设计、检查设计文档、设计审查、design review、检查docs、校验设计、审查文档一致性。
trigger_phrases:
  - 审查设计
  - 检查设计文档
  - 设计审查
  - design review
  - 检查docs
  - 校验设计
  - 审查文档一致性
  - 设计中间审查
  - review design
version: "1.1"
agent_created: true
author: qiang
---

# uniapp-design-reviewer

## 文档信息

- 当前版本：`v1.1`
- 更新时间：`2026-05-27`
- 当前定位：设计→执行的中间审查与修复层

## 概览

在 `uniapp-solution-architect` 完成设计输出后、`uniapp-delivery-engineer` 开始执行前，对 `docs/` 下所有设计文档进行系统性审查。

核心职责：
1. **检查** — 跨文档一致性校验（6组关系 + 数据单位 + 状态枚举）
2. **修复** — P0 和 P1 问题直接修复（修改文档文件）
3. **报告** — 输出简洁的审查修复日志
4. **放行** — 确认文档可以交付给 delivery-engineer

不做的事：
- 不做需求的合规性判断（那是 solution-architect 的职责）
- 不做架构方案评估（那是 solution-architect 的职责）
- 不重新生成大段文档
- 不修改 skill 文件本身

## 使用时机

满足以下条件时使用：
- `docs/` 目录下已有完整的设计文档（overview, api-contract, data-model, flows, ui-guidelines, admin, best-practices, AGENTS, project-system, change-log）
- `docs/tasks/` 目录下已有全部任务卡
- 准备将设计交付给 `uniapp-delivery-engineer` 开始执行

如果设计文档还不完整，先回退到 `uniapp-solution-architect` 补全。

## 执行流程

### Step 1: 扫描文档

读取 `docs/` 下所有最终文档，建立文档清单：

```
必读文件：
- docs/design/overview.md
- docs/design/best-practices.md
- docs/design/api-contract.md
- docs/design/data-model.md
- docs/design/flows.md
- docs/design/ui-guidelines.md
- docs/design/admin.md
- docs/AGENTS.md
- docs/project-system.md
- docs/tracking/change-log.md
- docs/tasks/*.md（所有任务卡）
```

### Step 2: 5组跨文档一致性校验

逐组执行以下校验，发现问题立即记录：

#### 组 1：api-contract.md ↔ data-model.md
- api-contract 中每个云对象/云函数方法涉及的集合是否在 data-model 中存在
- data-model 中定义的每个业务集合，是否在 api-contract 中有对应的操作方法
- **修复原则**：以 data-model 为基准，api-contract 缺失则补充方法桩

#### 组 2：overview.md 项目名 ↔ 任务卡目标目录
- 所有任务卡 `## 目标仓库或目录` 中引用的项目名，是否与 overview `## HBuilderX 建项方式` 完全一致
- **修复原则**：以 overview 为基准，直接修正任务卡中的项目名

#### 组 3：flows.md ↔ api-contract.md
- flows.md 中 Mermaid 图里出现的每个云对象调用，是否在 api-contract 中有对应方法
- api-contract 中的 URL 化入口，是否在 flows 中有对应的异步流程图
- **修复原则**：以 api-contract 为基准，修正 flows 中的方法名引用

#### 组 4：overview.md 页面清单 ↔ ui-guidelines.md
- overview 中列出的每个页面，是否在 ui-guidelines 中有对应的页面级提示词
- **修复原则**：以 overview 为基准，缺失的提示词标注为"P2 待补充"（不阻塞执行）

#### 组 5：overview.md 页面清单 ↔ 任务卡输出文件路径
- 所有任务卡的 `## 输出文件` 和 `## 范围` 中的页面路径，是否与 overview `## 页面结构与关键入口` 逐字符一致
- **修复原则**：以 overview 为基准，直接修正任务卡中不一致的路径名

#### 组 6：overview.md 服务商类型 ↔ 全文档 uniCloud 目录名
- overview `## HBuilderX 建项方式` 中声明的服务商类型（阿里云/支付宝云/腾讯云），是否与**所有文档**中引用的 `uniCloud` 目录名一致（阿里云→`uniCloud-aliyun/`、支付宝云→`uniCloud-alipay/`、腾讯云→`uniCloud-tcb/`）
- 检查范围覆盖：
  - `overview.md` 目录结构中的 `uniCloud` 目录名
  - `api-contract.md` 中所有 `uniCloud/cloudfunctions/` 等部署路径
  - `data-model.md` 中 DB Schema 部署路径（如 `uniCloud/database/`）
  - 所有任务卡的云函数目标路径
- **修复原则**：以 overview.md 建项结论中的服务商类型为基准，全局替换所有文档中不一致的目录名

### Step 3: 专项检查

#### 3.1 数据单位一致性
- 全文搜索 data-model.md 中所有时长字段（label 含"时长""额度""分钟""秒"）
- 全文搜索所有金额字段（label 含"金额""价格""元""分"）
- 逐字段核对存储单位是否统一：时长统一用秒、金额统一用分
- **修复原则**：如发现混用，统一为秒/分并修正字段名、label、description、初始数据

#### 3.2 状态枚举一致性
- 提取 data-model.md 中 `rc_tasks.status` 的 enum 数组
- 提取 flows.md 中状态映射表的服务端状态列
- 对比两边状态枚举是否完全一致
- **修复原则**：以 flows.md 为基准（flows 通常从需求推导更准确），修正 data-model.md 的 enum

#### 3.3 定时任务 Schema 完整性
- 检查 api-contract.md 中每个 `cron-xxx` 云函数是否包含：触发条件、入参说明、出参 JSON Schema
- **修复原则**：如只有文字描述无 Schema，补充标准出参 Schema

#### 3.4 人工处理实现方式一致性
- 检查 admin.md、overview.md 后台页面清单、TASK-009 输出文件三者对人工处理的描述是否一致
- 确认没有创建独立的人工处理页面（应为详情页内嵌面板）
- **修复原则**：统一为"详情页内嵌面板"方案

#### 3.5 需求禁止项验证
- 对照需求文件中的"明确禁止 skill 擅自补充的内容"
- 全文搜索设计文档，确认没有出现禁止内容
- **修复原则**：如发现违规，直接删除相关描述

### Step 4: 自动修复

对检测到的 P0 和 P1 问题，直接修改文件：

**P0（自动修复，不询问）：**
- 数据单位混用 → 统一修正
- 状态枚举不一致 → 以 flows 为基准修正
- 页面路径不一致 → 以 overview 为基准修正
- 任务卡项目名不一致 → 直接修正
- api-contract 方法缺失 → 补充方法桩

**P1（自动修复，记录日志）：**
- ui-guidelines 页面提示词缺失 → 标注为 P2 待补充
- best-practices 待确认问题未落地 → 补充说明
- 跨云对象调用链不完整 → 补充调用关系

**P2（不自动修复，记录到报告）：**
- 辅助页面缺少提示词
- 配置项描述不够详细
- change-log 内容过于简略

### Step 5: 输出审查修复日志

生成 `docs/design/cross-review-fix-log.md`，格式：

```markdown
# 设计审查修复日志

> 审查日期：YYYY-MM-DD
> 审查人：uniapp-design-reviewer v1.0
> 设计 Skill 版本：uniapp-solution-architect vX.X

## 审查结果摘要

| 检查项 | 结果 |
|--------|------|
| api-contract ↔ data-model | ✅/⚠️/❌ |
| overview 项目名 ↔ 任务卡 | ✅/⚠️/❌ |
| flows ↔ api-contract | ✅/⚠️/❌ |
| overview 页面 ↔ ui-guidelines | ✅/⚠️/❌ |
| overview 页面 ↔ 任务卡路径 | ✅/⚠️/❌ |
| overview 服务商类型 ↔ 目录名 | ✅/⚠️/❌ |
| 数据单位一致性 | ✅/⚠️/❌ |
| 状态枚举一致性 | ✅/⚠️/❌ |
| 定时任务 Schema | ✅/⚠️/❌ |
| 人工处理一致性 | ✅/⚠️/❌ |
| 需求禁止项 | ✅/⚠️/❌ |

## 已修复的问题

| # | 级别 | 问题 | 涉及文件 | 修复操作 |
|---|------|------|---------|---------|
| 1 | P0 | ... | ... | ... |

## 仍存在的问题（不阻塞执行）

| # | 级别 | 问题 | 建议 |
|---|------|------|------|

## 执行放行结论

✅ 可以交付给 delivery-engineer / ⚠️ 建议修复 P0 问题后再交付
```

### Step 6: 更新 project-system.md

在 `docs/project-system.md` 的 `## 当前阶段` 中更新为"设计审查完成"，在 `## 交付同步状态` 中增加 `cross-review-fix-log.md` 的状态。

## 修复原则总表

| 冲突场景 | 基准方 | 修正方 |
|---------|--------|--------|
| 项目名不一致 | overview.md 建项结论 | 任务卡 |
| 页面路径不一致 | overview.md 页面清单 | 任务卡 + ui-guidelines |
| 方法名/调用链不一致 | api-contract.md | flows.md |
| 状态枚举不一致 | flows.md 状态映射表 | data-model.md |
| 数据单位不一致 | best-practices.md 规则 | data-model.md + api-contract.md |
| 集合缺失 | data-model.md 是基准 | api-contract.md 补充引用 |
| uniCloud 目录名不一致 | overview.md 服务商类型 | 所有文档中的目录名引用 |

## 不要这样做

不要：
- 修改需求文件
- 添加需求中没有的功能
- 删除需求中明确要求的功能
- 做架构方案的二次决策（那是 solution-architect 的职责）
- 在报告中写冗长的分析过程——报告必须简洁
- 对 P2 问题做自动修复（只记录，不修改）
- 跳过 Step 6 的 project-system.md 更新

## 与其他 skill 的衔接

```
uniapp-solution-architect (v1.19+)
    │  产出 docs/
    ▼
uniapp-design-reviewer (本 skill)
    │  审查 → 修复 → 输出审查日志 → 放行
    ▼
uniapp-delivery-engineer (v1.19+)
    │  执行前检查 cross-review-fix-log.md
    │  确认 P0 已修复
    ▼
    开始执行
```
