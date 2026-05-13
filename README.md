# uniapp-creator

一个正式发布的 `uni-app` / `uniCloud` 方向 skill。

`uniapp-creator` 面向的是 **uni-app 项目的设计与开发执行**，不是业务应用代码仓库。本仓库中的 `uniapp-creator/` 目录是正式 skill 本体，其余 `docs/`、`AGENTS.md` 等内容用于说明这个 skill 的设计思路、生成过程、整改过程与验证基线，供参考和复用。

## 这是什么

`uniapp-creator` 是一个面向 `uni-app` / `uniCloud` 的 skill 组合，目标是让相关项目具备一套更稳定的工作方式：

- 设计阶段先收敛项目边界、模板选择、官方生态复用和平台策略
- 执行阶段按模块和步骤落地，而不是直接大面积展开实现
- 前端功能、UI、交互统一考虑
- 数据结构变化、流程变化、任务推进都能持续回写文档

这套 skill 体系重点解决的是：

1. `uni-app` 项目从设计到执行的衔接问题
2. DCloud 官方生态优先复用的问题
3. Web / App / Harmony / 小程序的差异处理问题
4. 多仓协同、前后台协同、共享后台场景下的边界问题

## Skill 结构

主 skill 名称：`uniapp-creator`

下含两个子 skill：

- `uniapp-solution-architect`
  - 负责设计阶段
  - 明确 `HBuilderX` 建项方式、官方模板选择、官方生态复用、平台策略、模块编号与执行入口

- `uniapp-delivery-engineer`
  - 负责执行阶段
  - 按既有设计落地，优先复用官方能力，处理平台差异，并回写数据文档、流程文档、任务记录

此外还包含：

- `shared-references/`
  - 两个子 skill 共用的核对类 supporting files
- 各子 skill 自身 `references/`
  - 与角色直接相关的 supporting files

## 仓库结构

```text
.
├─ AGENTS.md
├─ docs/
│  ├─ agents/
│  ├─ design/
│  ├─ review/
│  ├─ tracking/
│  └─ *.md
└─ uniapp-creator/
   ├─ shared-references/
   ├─ uniapp-solution-architect/
   └─ uniapp-delivery-engineer/
```

### 目录说明

- `uniapp-creator/`
  - 正式 skill 本体
- `uniapp-creator/uniapp-solution-architect/`
  - 设计子 skill
- `uniapp-creator/uniapp-delivery-engineer/`
  - 执行子 skill
- `uniapp-creator/shared-references/`
  - 共享核对文件
- `docs/`
  - 这个 skill 的设计、整改、审查、验证与追踪资料

## 安装到 AI 开发工具

`uniapp-creator` 以目录形式发布，最适合安装到支持“本地 skills / agents / prompts 目录”的 AI 开发工具中。

核心原则只有一条：

- 把仓库中的 `uniapp-creator/` 目录放到目标工具可读取的 skills 目录中

安装后可用的名称是：

- 主 skill：`uniapp-creator`
- 子 skill：`uniapp-solution-architect`
- 子 skill：`uniapp-delivery-engineer`

### 通用安装方式

1. 下载或克隆本仓库
2. 找到工具本地的 skills 目录
3. 将 `uniapp-creator/` 复制或软链接到该目录
4. 重启工具，或执行工具自己的 skills 刷新动作

### 安装到 Codex / 本地 skills 系统

如果你的工具本身就是基于本地 skills 目录加载，直接把：

- `uniapp-creator/`

复制到该工具的 skills 根目录即可。

如果你的环境区分“系统 skills”和“用户 skills”，建议放到“用户 skills”目录，便于独立升级和维护。

### 安装到支持 AGENTS.md / repo instructions 的工具

有些工具并不直接扫描 skills 目录，而是更依赖仓库内的说明文件、agent 配置或 prompts 约定。

这类工具建议：

1. 保留本仓库中的 `uniapp-creator/`
2. 保留根目录 `AGENTS.md`
3. 在你的目标项目中，把 `uniapp-creator/` 作为本地 skill 资源引入
4. 在仓库说明或工具配置中，明确：
   - 设计阶段使用 `uniapp-solution-architect`
   - 执行阶段使用 `uniapp-delivery-engineer`

### 安装到不支持原生 skills 的 AI 工具

如果某个工具不支持直接安装 skills，也可以把本仓库当作“结构化提示词资源”使用：

1. 将 `uniapp-creator/uniapp-solution-architect/SKILL.md` 作为设计阶段系统提示词或项目提示词
2. 将 `uniapp-creator/uniapp-delivery-engineer/SKILL.md` 作为执行阶段系统提示词或项目提示词
3. 将 `shared-references/` 和各自 `references/` 作为补充参考资料

这种方式不能完全等价于原生 skills 调度，但依然能复用本仓库的主体规则。

## 使用方法

### 方法一：按子 skill 分阶段调用

如果你希望明确分阶段工作，建议按下面方式使用。

#### 1. 设计阶段

在以下场景优先使用 `uniapp-solution-architect`：

- 还没确定 `HBuilderX` 建项方式
- 还没确定官方模板
- 还没确定项目边界、平台范围、多仓关系
- 还没确定官方模块复用策略

典型请求：

- “请先为这个 uni-app 项目做方案设计，明确 HBuilderX 模板、模块编号和平台策略”
- “这个项目有用户端、共享后台和管理后台，先帮我拆设计和执行边界”

#### 2. 执行阶段

在以下场景优先使用 `uniapp-delivery-engineer`：

- 已经有明确设计边界
- 已经有模块编号或任务卡
- 要继续补数据结构、云端逻辑、前端页面或管理后台
- 要按官方优先原则继续开发并回写文档

典型请求：

- “按现有设计继续做订单模块，优先复用官方能力”
- “用 schema2code 思路补 schema、云对象和前端列表页，并同步回写文档”

### 方法二：直接作为本地 skill 使用

如果你的工具已经成功加载本地 skill，可直接按下面名称使用：

- `uniapp-creator`
- `uniapp-solution-architect`
- `uniapp-delivery-engineer`

推荐做法：

- 用 `uniapp-solution-architect` 处理前期方案
- 用 `uniapp-delivery-engineer` 处理后续落地
- 让工具在同一个项目上下文中持续引用这套 skill

### 方法三：作为创建同类 skill 的参考仓库

如果你不是直接使用这个 skill，而是想创建自己的领域 skill，这个仓库也可以作为参考。

建议重点查看：

- `uniapp-creator/`
  - 看正式 skill 如何组织
- `docs/project-system.md`
  - 看总控入口如何维护
- `docs/design/system-design.md`
  - 看稳定设计基线如何收敛
- `docs/tracking/`
  - 看任务卡、变更记录和验证闭环如何建立

## 设计原则

`uniapp-creator` 当前坚持以下原则：

- 官方优先：优先复用 `uni-app` / `uniCloud` 官方模板、组件、插件、模块、设计资源
- 设计与执行拆分：设计定义边界，执行按边界落地
- 前端一体化：功能、UI、交互一起考虑
- 平台差异先分类：编译期、运行期、能力差异与降级
- 文档持续回写：数据、流程、任务状态都留痕
- 避免过度设计：不为完整性额外堆低价值文档和方法论

## 当前状态

当前仓库已经完成：

- `uniapp-creator` 主 skill 与两个子 skill 的拆分
- supporting files 的共享与角色化收敛
- 最小静态验证基线
- 仓库级 `AGENTS.md` 与 `docs/agents/` 配置

当前仍待继续：

- 真实前向测试
- 基于前向测试结果的最后一轮修订

这些“待继续”项属于 **skill 的迭代完善**，不影响本仓库作为正式 skill 发布和参考使用。

## 文档说明

本仓库中除了正式 skill 目录外，还保留了完整文档主线。它们不是 skill 本体的一部分，而是这个 skill 的参考资料。

主要包括：

- 设计基线
- 整改计划
- 审查记录
- 任务卡
- 变更记录
- 验证基线

如果你只是想使用 skill，重点看 `uniapp-creator/` 即可。  
如果你想理解这个 skill 是如何一步步收敛出来的，再阅读 `docs/`。

## License

MIT License
