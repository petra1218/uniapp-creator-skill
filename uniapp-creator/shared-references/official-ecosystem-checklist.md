# 官方生态核对清单

## 使用方式

设计或执行时，按顺序过一遍。只要前一层能满足，就不要跳到后一层自定义实现。

如果当前项目存在「多个应用共用同一个 uniCloud 服务空间」，先不要直接假设官方库都适合共享。必须逐类核查「是否适合进入共享层」和「隔离边界是否成立」。

## 核对顺序总览

```
1. 项目内已有官方封装
2. 官方起点（uni-starter / uni-admin / uni-ui）
3. 官方模块与能力（uni-id / DB Schema / 云对象 / 云函数 / 云存储）
4. 最小改造
5. 自定义实现
```

---

## 一、项目内已有官方封装

先判断项目里是否已经有：

- 基于 `uni-ui`、`uni_modules`、`uniCloud` 的封装
- 官方模板遗留的业务骨架
- 统一登录、权限、云对象调用方式
- 现有 `pages.json` 和 `manifest.json` 配置

如果已有基线，沿用基线做增量，不要强行改模板。

---

## 二、官方起点选择

| 项目类型 | 推荐模板 | 关键组件 |
|---------|---------|---------|
| 客户端（需云能力 + 账号） | `uni-starter` | uni-id-pages + Pinia + 云对象封装 |
| 后台管理端 | `uni-admin` | RBAC 权限 + 动态菜单 + schema2code |
| 纯前端，无需云能力 | `uni-ui` | uni-ui 组件库 |
| 学习/原型验证 | `Hello uni-app` | 组件/API 演示 |

**客户端 + 后台管理端是多仓组合，不是单项目决策。**

---

## 三、uniCloud 能力细化核对

### 3.1 数据访问路径决策树

按顺序判断每个业务模块用哪条路径：

```
该模块是否需要登录用户读写自己的数据？
  → 是：优先 clientDB + DB Schema + uni-id
该模块是否需要业务编排（校验/扣减/状态推进）？
  → 是：云对象
该模块是定时任务/URL 化入口/第三方回调？
  → 是：云函数
以上都不满足，且第三方明确禁止？
  → 才考虑自定义接口
```

### 3.2 DB Schema 最佳实践（官方规范）

**每个业务集合必须配置 DB Schema**，至少包含：

```json
{
  "bsonType": "object",
  "required": ["field1", "field2"],
  "permission": {
    "read": "doc._id == auth.uid || 'admin' in auth.role",
    "create": "auth.uid != null",
    "update": "doc._id == auth.uid || 'admin' in auth.role",
    "delete": "'admin' in auth.role"
  },
  "properties": {
    "field1": { "bsonType": "string", "label": "字段1" },
    "field2": { "bsonType": "number" }
  },
  "indexes": [
    { "fields": ["user_id", "status"], "indexName": "user_status_idx" }
  ]
}
```

**必须明确：**
- `required` 字段（必填）
- `permission` 表达式（谁可以读/写/删）
- `properties` 类型与 label
- `indexes` 索引（高频查询字段必须建索引）

### 3.3 uni-id 多应用方案

当多个应用共用同一个 uniCloud 服务空间时，uni-id 通过 `dcloudAppid` 实现用户隔离：

**用户表隔离：**
```json
// 用户记录中的 dcloud_appid 数组
{
  "_id": "xxx",
  "username": "张三",
  "dcloud_appid": ["__UNI__A12345", "__UNI__B12345"]
}
```

**config.json 多应用配置：**
```json
[
  {
    "dcloudAppid": "__UNI__A12345",
    "isDefaultConfig": true,
    "passwordSecret": "same-secret-for-all-apps",
    "tokenSecret": "same-token-secret",
    "tokenExpiresIn": 7200
  },
  {
    "dcloudAppid": "__UNI__B12345",
    "passwordSecret": "same-secret-for-all-apps",
    "tokenSecret": "same-token-secret"
  }
]
```

**关键规则：**
- `passwordSecret` 必须相同才能跨端使用相同账号密码
- `dcloud_appid` 为空数组表示无法登录任何客户端
- 权限判断：`'admin' in auth.role` — admin 角色跨应用有效

**哪些可以共享：**
- 用户基础表（uni-id-users）
- 角色权限表（uni-id-roles, uni-id-permissions）
- 管理员账号（opendb-admin-users）

**哪些必须按应用隔离：**
- 业务数据表（通过 DB Schema 的 `dcloudAppid` 字段隔离）
- 业务云函数/云对象
- 云存储路径（按应用命名空间隔离）

### 3.4 uni-admin 后台管理最佳实践

**uni-admin 的两种菜单模式：**

| 类型 | 配置位置 | 适用场景 |
|------|---------|---------|
| 静态菜单 | `admin.config.js` → `staticMenu` | 所有管理员都能看到，固定不变的功能 |
| 动态菜单 | `opendb-admin-menus` 表 | 按角色动态生成，需可视化维护 |

**推荐：默认使用动态菜单**，优点是可在管理后台直接增删菜单，无需改代码。

**权限判断（组件内）：**
```vue
<template>
  <view>
    <!-- 有 USER_ADD 权限的用户可见 -->
    <button v-if="$hasPermission('USER_ADD')">新增</button>
    <!-- 有 admin 角色的用户可见 -->
    <button v-if="$hasRole('admin')">删除</button>
  </view>
</template>
```

**数据权限（Schema 层）：**
```json
// 在业务表的 schema 文件中配置
{
  "permission": {
    "read": "'admin' in auth.role || 'READ_DATA' in auth.permission"
  }
}
```

**uni-admin 与客户端共用服务的关键约束：**
- 后台基础路径默认 `/admin/`（通过 `manifest.json` → `h5.router.base` 修改）
- 路由模式必须为 **hash**（uni-admin 强制要求）
- 客户端和后台的云对象/云函数在同一个服务空间，但命名分开（`xxx-co` vs `admin-xxx-co`）

### 3.5 uni-pay 支付聚合最佳实践

`uni-pay` 是 DCloud 官方的支付聚合，支持微信/支付宝，覆盖 App/小程序/H5 多端。

**接入模式：**
```
客户端 → uni-pay 云对象 → uni-pay 统一下单
              ↓
         第三方支付渠道
              ↓
         支付回调云函数（webhook-pay-notify）
              ↓
         订单状态更新 + 权益发放
```

**回调设计必须包含：**
- 幂等键（用商户订单号 `out_trade_no` 作为幂等键）
- 签名校验（`uniPayCommon.verifyPayNotify`）
- 状态映射：`pending → paying → paid / failed`
- 重复回调处理（先查订单状态，已 paid 则直接返回成功）

### 3.6 云存储最佳实践

| 场景 | 推荐方案 |
|------|---------|
| 客户端上传文件 | `uni-file-picker` → 客户端直传云存储 |
| 获取文件访问 | 云函数调用 `getTempFileURL` 获取临时链接 |
| 删除文件 | 云函数执行 `deleteFile`，不把删除权交给前端 |
| 大文件/私密文件 | 云函数中转（但优先考虑临时链接方案） |

**文件路径命名规范：**
```
cloudstorage://{app_id}/{user_id}/{task_id}/{filename}
// 例如：cloudstorage://__UNI__A12345/uid001/task001/recording.m4a
```

---

## 四、第三方能力接入合同

出现支付、语音转写、OCR、AI、内容安全、地图等第三方能力时，必须输出最小合同：

| 字段 | 必须明确 |
|------|---------|
| provider | 第三方名称和能力边界 |
| provider_task_id | 第三方任务 ID |
| 本地任务映射 | 本地 task_id 与 provider_id 的映射 |
| 提交参数 | 关键入参摘要 |
| 回调/轮询 | 回调 URL 或轮询方式 |
| 状态映射 | 成功/失败/处理中的状态映射 |
| 幂等设计 | 重复回调/请求的处理方式 |
| 成本风险 | 配额、并发、限流风险 |

---

## 五、设计或执行时必须回答

- 当前方案复用了哪些官方能力？
- 哪些地方只是做最小改造？
- 哪些地方需要业务包装？
- 哪些地方必须自定义，以及为什么？

**多应用共用时，还必须回答：**
- 哪些官方库进入共享层，理由是什么？
- 哪些官方库不进入共享层，理由是什么？
- 哪些能力可以用 `dcloudAppid` 做应用级隔离？
- 哪些能力不能只靠 `dcloudAppid`？
- 文件上传、访问链接、删除权限分别落在哪层？

---

## 六、官方插件推荐清单

| 类型 | 推荐插件 | 地址 |
|------|---------|------|
| 客户端业务起点 | uni-starter | ext.dcloud.net.cn/plugin?id=5747 |
| 后台管理框架 | uni-admin | ext.dcloud.net.cn/plugin?id=3268 |
| 用户体系页面 | uni-id-pages | ext.dcloud.net.cn/plugin?id=8592 |
| 支付聚合 | uni-pay | ext.dcloud.net.cn/plugin?id=5406 |
| 短信服务 | uni-sms | ext.dcloud.net.cn/plugin?id=... |
| 内容安全 | uni-sec-check | ext.dcloud.net.cn/plugin?id=... |
| 地图能力 | uni-map | ext.dcloud.net.cn/plugin?id=... |
| App 更新 | uni-upgrade-center | ext.dcloud.net.cn/plugin?id=... |
| 即时通讯 | uni-im | ext.dcloud.net.cn/plugin?id=... |
| CMS | uni-cms | ext.dcloud.net.cn/plugin?id=... |
