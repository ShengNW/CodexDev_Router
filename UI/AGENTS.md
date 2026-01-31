# Router 前端交互速查（给 Codex/同事）

目的：让你不必全仓库搜索，就能理解前端交互的入口、调用链路、页面分布和后端接口对接位置。

---

## 1) 前端总体架构（最短理解路径）
- **前端目录**：`web/`
- **技术栈**：React 18 + Vite + react-router-dom v7 + semantic-ui + i18next + axios
- **核心调用链**：`src/index.jsx` → `src/App.jsx`(路由) → `pages/*`(页面) → `components/*`(表格/表单) → `helpers/api.jsx`(axios) → 后端 `/api/*`
- **状态来源**：`/api/status` → `localStorage` → `Header/Footer/各种页面`

---

## 2) 启动与构建（开发 & 线上）
- **开发**：在 `web/` 下执行 `npm run dev`
  - Vite 端口：`5181`（见 `web/vite.config.mjs`）
  - 默认代理：`/api` → `http://localhost:3011`
  - 若需要跨域指向别的后端：设置 `VITE_SERVER=http://host:port`
- **构建**：`npm run build --prefix web`
  - 输出目录：`web/build`
  - 后端通过 `embed.go` + `internal/transport/http/router/web.go` 直接提供静态资源
  - 若设置 `FRONTEND_BASE_URL`，后端会重定向到外部前端

---

## 3) 入口与路由（页面总览）
- **入口**：`web/src/index.jsx`
  - 注入 `StatusProvider` / `UserProvider`
  - 包裹 `BrowserRouter` + `Header` + `Footer`
  - 全局提示：`react-toastify`
- **路由**：`web/src/App.jsx`
  - 首页：`/` → `pages/Home`
  - 登录/注册：`/login` / `/register`
  - 权限页（需登录）：`/channel`、`/token`、`/redemption`、`/user`、`/log`、`/setting`、`/topup`、`/dashboard`
  - 其它：`/about`、`/chat`、`/reset`、`/user/reset`

---

## 4) 状态与本地缓存（非常关键）
- **状态加载**：`App.jsx` 启动时请求 `/api/status`
  - 写入 `localStorage`：`system_name`、`logo`、`footer_html`、`quota_per_unit`、`display_in_currency`、`chat_link`
  - 这些值影响：`Header`、`Footer`、配额显示、聊天入口等
- **用户状态**：`UserContext` + `localStorage.user`
  - `PrivateRoute` 仅检查 `localStorage.user`
  - 登出时会清理 `user` + `wallet_token`

---

## 5) API 统一入口与鉴权逻辑
- **API 实例**：`web/src/helpers/api.jsx`
  - `baseURL` 来自 `VITE_SERVER` 或当前域
  - 请求拦截：从 `localStorage.user` 或 `wallet_token` 填充 `Authorization`
  - 响应拦截：`401` 时尝试 `web3` 刷新 token
- **错误/提示**：`web/src/helpers/utils.jsx` → `showError/showSuccess/showNotice`

---

## 6) 认证/登录链路
- **账号密码登录**：`LoginForm.jsx`
  - `POST /api/user/login` → 写入 `localStorage.user` → 跳转 `/token`
- **钱包登录**：`services/web3Auth.jsx` + `helpers/web3.jsx`
  - 调用 `@yeying-community/web3-bs` 的 `loginWithChallenge`
  - baseUrl：`/api/v1/public/auth`
  - 登录成功后调用 `/api/user/self` 获取用户信息
  - token 存在 `localStorage.wallet_token`
- **钱包绑定**：`PersonalSetting.jsx`
  - `POST /api/v1/public/common/auth/challenge`
  - `POST /api/oauth/wallet/bind`

---

## 7) 主要页面与对应组件（去哪里改）

### 首页/关于/聊天
- 首页：`pages/Home/index.jsx`
  - `/api/notice`（弹公告）、`/api/home_page_content`（首页内容，支持 URL/Markdown）
- 关于：`pages/About/index.jsx`
  - `/api/about`
- 聊天：`pages/Chat/index.jsx`
  - 纯 iframe，`chat_link` 来自 `/api/status`

### Channel 管理（管理员）
- 页面：`pages/Channel/index.jsx`
  - 列表逻辑：`components/ChannelsTable.jsx`
  - 详情编辑：`pages/Channel/EditChannel.jsx`（非常大）
  - 相关接口：`/api/channel/*`、`/api/channel/models`、`/api/group`
  - 类型定义：`constants/channel.constants.jsx`

### Token 管理
- 页面：`pages/Token/index.jsx`
  - 列表逻辑：`components/TokensTable.jsx`
  - 编辑/新增：`pages/Token/EditToken.jsx`
  - 相关接口：`/api/token/*`、`/api/user/available_models`

### Redemption 兑换码
- 页面：`pages/Redemption/index.jsx`
  - 列表逻辑：`components/RedemptionsTable.jsx`
  - 编辑/新增：`pages/Redemption/EditRedemption.jsx`
  - 相关接口：`/api/redemption/*`

### 用户管理（管理员）
- 页面：`pages/User/index.jsx`
  - 列表逻辑：`components/UsersTable.jsx`
  - 编辑/新增：`pages/User/EditUser.jsx`、`pages/User/AddUser.jsx`
  - 相关接口：`/api/user/*`、`/api/user/manage`

### 日志
- 页面：`pages/Log/index.jsx`
  - 列表/统计：`components/LogsTable.jsx`
  - 相关接口：`/api/log/*`
  - 管理员能筛选用户/渠道，普通用户只看 `/api/log/self`

### 设置
- 页面：`pages/Setting/index.jsx`
  - 个人：`components/PersonalSetting.jsx`
  - 运营：`components/OperationSetting.jsx`
  - 系统：`components/SystemSetting.jsx`
  - 其他：`components/OtherSetting.jsx`
  - 统一接口：`/api/option`

### 仪表盘 & 充值
- 仪表盘：`pages/Dashboard/index.jsx`
  - `/api/user/dashboard`
- 充值：`pages/TopUp/index.jsx`
  - `/api/user/topup`，`/api/user/self`

---

## 8) 国际化 & 文案
- i18n 入口：`web/src/i18n.jsx`
- 语言包：`web/src/locales/zh/translation.json`、`web/src/locales/en/translation.json`
- Header 语言切换：`components/Header.jsx`

---

## 9) 和后端交互的关键点（排错必看）
- `ITEMS_PER_PAGE` 必须和后端一致：`constants/common.constant.jsx`
- 所有列表类接口都使用分页参数 `p`
- `localStorage.status` 影响页面显示（logo/系统名/页脚/单位换算/聊天链接）
- `FRONTEND_BASE_URL` 会让后端不再直接提供 `web/build`，而是重定向

---

## 10) 可能“看似存在但未接入”的前端
- `components/GitHubOAuth.jsx` / `components/LarkOAuth.jsx` 有逻辑但路由未接入
- `pages/Wallet/index.jsx` 未被 `App.jsx` 路由使用（测试工具页）

---

## 11) 快速定位一类“交互问题”的方法
- **页面不显示/内容异常** → 先看 `App.jsx` 是否正常拉到 `/api/status`
- **接口没走** → `helpers/api.jsx` 是否被正确使用？是否设置 `VITE_SERVER`？
- **权限问题** → `localStorage.user` 是否存在？`PrivateRoute` 只看本地
- **语言问题** → 检查 `locales` 和 `useTranslation` 的 key

---

如需深入修改某个页面：优先从 `pages/*` 入口找，再追到 `components/*Table` 或具体表单组件，最后才去 `helpers/api.jsx`/`helpers/utils.jsx` 看通用逻辑。
