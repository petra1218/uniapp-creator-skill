# 模板选择指引

## 目的

统一判断 uni-app 项目应以哪个官方起点进入，避免在设计或执行阶段各自得出不同模板结论。

**设计输出中必须包含 HBuilderX 建项方式和模板选择结论，不允许只写"用 uni-app"而不写具体起点。**

## 固定前提

- 新项目默认明确写出使用 `HBuilderX` 创建项目
- 只有在「确实是旧项目增量」时才不新建项目
- 先判断项目类型，再选模板，不反过来
- 客户端和后台管理端是两个独立入口，必须分开判断模板

## 选择顺序

1. 先判断是不是旧项目增量
2. 再判断是客户端、管理端，还是两者都要
3. 再判断是否需要 uniCloud、账号体系、业务骨架
4. 最后确定具体模板

---

## 客户端模板选择

| 场景 | 推荐模板 | 选择理由 | 不选它的情况 |
|------|---------|---------|------------|
| 通用前端 + uniCloud + 账号体系 + 业务骨架 | **`uni-starter`** | DCloud 官方业务起点，内置 uni-id-pages、登录页、用户中心、云对象调用封装，开箱即用 | 只需单页原型或纯展示页 |
| 通用前端，无需云能力和账号体系 | **`uni-ui`** 官方组件起点 | 以 uni-ui 组件库为主，适合快速搭建页面 | 需要账号、云端逻辑 |
| 仅验证 uni-app 基础能力 | **`Hello uni-app`** | 包含官方组件/API 演示，适合熟悉框架 | 需要承接真实业务 |
| 有特殊需求，无法用 uni-starter 覆盖 | **`空白项目`** + 手动组合 | 按需引入 uni-id、uni-pay、uni-ui 等模块 | 通用业务场景 |

### uni-starter 模板特点（重要）

`uni-starter` 是 DCloud 官方出品的企业级业务起点，**推荐作为含云能力和用户体系的客户端项目首选**：

```
uni-starter 包含：
├── 前端：uni-id-pages（登录/注册/重置密码页面）+ 用户中心 + Pinia 状态管理
├── 云端：uni-id-common（登录态校验）+ 基础云对象
├── 数据库：uni-id-users 等基础表 + DB Schema
└── 脚手架：自动关联云服务空间、一键发行
```

选 `uni-starter` 时，设计输出必须写明：
- 是否复用 `uni-id-pages` 的登录页，还是自定义
- 是否复用用户中心页面，还是重写
- 是否复用 Pinia 状态管理结构

---

## 后台管理端模板选择

| 场景 | 推荐模板 | 选择理由 | 不选它的情况 |
|------|---------|---------|------------|
| 需要管理后台 | **`uni-admin`** | DCloud 官方后台框架，内置菜单管理、角色权限、数据管理、换肤，开箱即用 | 只需简单数据管理 |
| 简单数据管理，无需复杂权限 | **`schema2code` 生成的页面** | 配置 DB Schema 后自动生成增删改查页面，适合轻量后台 | 需要多角色多权限 |
| 极简运营页面，直接复用客户端部分页面 | **客户端项目内嵌管理模块** | 仅在客户端 tabBar 外挂一个管理入口 | 正式运营后台 |

### uni-admin 模板特点（重要）

`uni-admin` 是 DCloud 官方的后台管理框架，**推荐作为含运营管理需求项目的后台首选**：

```
uni-admin 包含：
├── 前端：Vue3 + uni-ui，PC 宽屏优先，移动端自适应
├── 菜单系统：静态菜单（admin.config.js）+ 动态菜单（opendb-admin-menus）
├── 权限系统：基于 uni-id 的 RBAC（admin 角色 + 自定义角色）
├── 数据管理：schema2code 生成 + web 控制台
└── 样式：内置换肤机制（默认蓝色 + 绿柔皮肤）
```

选 `uni-admin` 时，设计输出必须写明：
- 是否复用 `uni-admin` 的默认登录页，还是自定义
- 是否使用动态菜单（推荐），还是静态菜单
- 管理员账号体系是否复用 `uni-admin` 自带的 `opendb-admin-users`
- 后台访问路径是否需要自定义（默认 `/admin/`）

---

## 多仓默认组合

### 客户端 + 后台管理端（常见组合）

这是最常见的组合，**两个必须作为独立项目设计**：

```
项目结构：
├── client/                    # 客户端项目（uni-starter）
│   ├── pages/                 # 用户端页面
│   ├── components/            # 用户端组件
│   ├── static/                # 用户端静态资源
│   ├── uniCloud/              # 客户端云函数/云对象
│   │   └── cloudfunctions/
│   │       └── client-co/     # 客户端业务云对象
│   └── manifest.json
│
└── admin/                     # 后台管理端项目（uni-admin）
    ├── pages/                 # 后台管理页面
    ├── components/            # 后台组件
    ├── static/                # 后台静态资源
    ├── uniCloud/              # 后台云函数/云对象（与客户端共享同一服务空间）
    │   └── cloudfunctions/
    │       └── admin-co/      # 后台管理云对象
    └── manifest.json
```

**关键约束：**
- 客户端和后台管理端共用**同一个 uniCloud 服务空间**
- 数据库集合按 `dcloudAppid` + DB Schema 权限表达式隔离
- 前端云对象和后台云对象分开命名（如 `transcription-co` vs `admin-transcription-co`）
- 后台管理端的基础路径默认 `/admin/`，通过 `manifest.json` 的 `h5.router.base` 修改
- 后台管理端的 `pages.json` 路由模式必须为 **hash**（uni-admin 强制要求）

---

## HBuilderX 建项详细操作步骤

### 步骤 1：创建客户端项目（以 uni-starter 为例）

1. 打开 HBuilderX → 菜单 → `文件` → `新建` → `项目`
2. 选择分类：`uni-app`
3. 选择模板：`uni-starter`（或根据上表选择其他模板）
4. 填写项目名称（如 `rc-transcription-client`）
5. 选择存放路径（路径中不要包含中文或特殊字符）
6. 选择 Vue 版本：**推荐 Vue 3**（编译速度更快，不支持低版本 Android）
7. 点击 `创建`

### 步骤 2：关联 uniCloud 服务空间

**方式 A：通过初始化向导（推荐）**

项目创建后，HBuilderX 会自动弹出 `uniCloud 初始化向导`：

1. 实名认证（根据法律要求，开通云服务器需完成实名认证）
2. 创建服务空间 → 选择云厂商（**支付宝云/阿里云/腾讯云**，三家均提供免费额度）
3. 选择刚才创建的服务空间
4. 上传部署资源：
   - 右键 `uniCloud/cloudfunctions` → `上传部署`（上传所有云函数）
   - 右键 `uniCloud/database` → `初始化数据库`（HBuilderX ≥ 3.9）

**方式 B：手动关联已有服务空间**

如果已有服务空间，或向导中途关闭：

1. 在项目根目录右键 → `创建uniCloud云服务空间` → 选择云厂商
2. 或在 `uniCloud` 目录右键 → `关联云服务空间或项目...`
3. 关联后在 `uniCloud/cloudfunctions` 右键 → `上传部署`
4. 在 `uniCloud/database` 右键 → `初始化数据库`

### 步骤 3：创建后台管理端项目（uni-admin）

1. HBuilderX → `文件` → `新建` → `项目`
2. 选择分类：`uni-app`
3. 选择模板：`uni-admin`
4. 填写项目名称（如 `rc-transcription-admin`）
5. 重复步骤 2，**关联同一个服务空间**（客户端和后台共用同一个空间）

**重要：uni-admin 和客户端项目必须部署到同一个 uniCloud 服务空间**，但各自有独立的前端项目目录。

### 步骤 4：运行与调试

| 操作 | 客户端 | 后台管理端 |
|------|-------|----------|
| 运行到浏览器 | `运行` → `运行到浏览器` → `Chrome` | `运行` → `运行到浏览器` → `Chrome` |
| 访问地址 | `http://localhost:5173/` | `http://localhost:5173/admin/` |
| 切换云函数连接 | 底部工具栏：`连接本地云函数` / `连接云端云函数` | 同左 |

### 步骤 5：发行前检查

| 检查项 | 说明 |
|--------|------|
| `manifest.json` 基础配置 | AppID、版本号、版本名称已填写 |
| `pages.json` 路由 | 所有路由路径有对应 .vue 文件，tabBar 路径正确 |
| 条件编译标识 | `#ifdef`、`#ifndef` 覆盖所有平台差异代码段 |
| 云函数 `package.json` | 依赖已锁定版本（无 `^`/`~`），所有依赖已上传 |
| DB Schema | schema 文件已在目标服务空间导入并校验通过 |
| 隐私协议 | uni-app 强制要求已配置（`manifest.json` → `app-plus` → `privacy`） |

---

## 模板对比速查

| 维度 | uni-starter | uni-admin | uni-ui | Hello uni-app |
|------|------------|----------|--------|--------------|
| **含 uniCloud** | ✅ 含 | ✅ 含 | ❌ 不含 | ❌ 不含 |
| **含账号体系** | ✅ uni-id-pages | ✅ uni-id | ❌ 不含 | ❌ 不含 |
| **含后台管理** | ❌ 不含 | ✅ 含 | ❌ 不含 | ❌ 不含 |
| **含 Pinia** | ✅ 含 | ❌ 不含 | ❌ 不含 | ❌ 不含 |
| **含 uni-ui** | ✅ 含 | ✅ 含 | ✅ 含 | ✅ 含 |
| **适用场景** | 客户端业务起点 | 后台管理起点 | 纯页面组件 | 框架学习 |

---

## 输出要求

设计输出 `docs/design/overview.md` 的 `## HBuilderX 建项方式` 章节必须包含：

```markdown
## HBuilderX 建项方式

### 客户端项目
- **建项工具**：HBuilderX
- **模板**：`uni-starter`（或具体模板名）
- **Vue 版本**：Vue 3（推荐）
- **uniCloud 服务空间**：{空间名}（{云厂商}，{免费额度说明}）
- **选择理由**：
  1. {理由1}
  2. {理由2}
- **不选其他模板的原因**：{与当前项目直接相关的差异}

### 后台管理端项目
- **建项工具**：HBuilderX
- **模板**：`uni-admin`
- **uniCloud 服务空间**：与客户端共用同一空间
- **选择理由**：{理由}
- **管理后台访问路径**：`/admin/`（默认）

### 项目目录结构
{文字描述或 ASCII 图示}
```

---

## 相关官方资源

- uni-starter 模板：https://ext.dcloud.net.cn/plugin?id=5747
- uni-admin 模板：https://ext.dcloud.net.cn/plugin?id=3268
- HBuilderX 下载：https://hx.dcloud.net.cn/
- uniCloud 快速入门：https://doc.dcloud.net.cn/uniCloud/quickstart.html
