# API Notes
这里记录 API 相关的调研与实现历史，便于后续快速理解 Router 的 API 架构与调用逻辑。

## 总览
Router 对外接口主要分两大类：
1) **服务端管理/用户 API（/api 旧路由 + /api/v1/public + /api/v1/admin）**
2) **模型供应商聚合的 OpenAI 兼容 API（/v1 + /dashboard，迁移到 /api/v1/public/*）**

整体入口链路：
`cmd/router/main.go` → `internal/app/app.go` → `internal/transport/http/router/main.go`  
其中 `SetApiRouter` / `SetDashboardRouter` / `SetRelayRouter` 分别挂载不同 API 类别。

## A. 旧 Router 自带 API（legacy）
路由前缀：`/api`  
路由定义：`internal/transport/http/router/api.go`

主要接口类型：
- **公共信息（无需登录）**
  `/api/status`, `/api/notice`, `/api/about`, `/api/home_page_content`  
  找回密码：`/api/reset_password`, `/api/user/reset`
- **用户体系（注册/登录/自助）**
  `/api/user/register`, `/api/user/login`, `/api/user/logout`  
  `/api/user/self`, `/api/user/dashboard`, `/api/user/available_models`, `/api/user/token`, `/api/user/topup`
- **管理员与运营（Admin 权限）**
  `/api/user`（用户管理）  
  `/api/channel`（渠道管理）  
  `/api/redemption`（兑换码）  
  `/api/log`（日志）  
  `/api/group`（分组）
- **Root 权限**
  `/api/option`（系统配置）

鉴权机制（旧 API）：
- 支持 **Cookie Session** + **Authorization header**
- middleware 在 `internal/transport/http/middleware/auth.go`
- 优先钱包 JWT，其次 access token
- 角色等级：`RoleCommonUser=1`, `RoleAdminUser=10`, `RoleRootUser=100`  
  定义于 `internal/admin/model/user.go`

## B. 新增 Interface 规范 API（/api/v1/public）
路由前缀：`/api/v1/public`  
路由定义：`internal/transport/http/router/api.go`

当前已补齐分层：
- `/api/v1/public`：镜像 `/api` 公共/自助接口（另含钱包/UCAN）
- `/api/v1/admin`：镜像 `/api` 管理接口
- `/api/v1/internal`：预留空组（当前无接口）
映射清单见 `CodexDev/API/api-v1-mapping.md`（注意 `/api/models` 镜像为 `/api/v1/public/models-all`，以避开 OpenAI 兼容 `/api/v1/public/models`）。

接口清单：
- **proto 风格钱包认证**
  `/api/v1/public/common/auth/challenge`  
  `/api/v1/public/common/auth/verify`  
  `/api/v1/public/common/auth/refreshToken`  
  处理逻辑：`internal/admin/controller/auth/wallet.go`
- **web3 风格钱包认证**
  `/api/v1/public/auth/challenge`  
  `/api/v1/public/auth/verify`  
  `/api/v1/public/auth/refresh`  
  `/api/v1/public/auth/logout`  
  处理逻辑：`internal/admin/controller/auth/wallet.go`
- **公开 profile（UCAN 或钱包 JWT）**
  `/api/v1/public/profile`  
  处理逻辑：`internal/admin/controller/auth/ucan.go`

UCAN 相关：
- 配置项：`UCAN_AUD / UCAN_RESOURCE / UCAN_ACTION`
- 初始化：`common/init.go`
- 校验逻辑：`common/ucan.go`

## C. 模型供应商聚合 API（OpenAI 兼容）
路由前缀：`/v1` 与 `/dashboard`（兼容期），以及新增 `/api/v1/public/*`（推荐）  
路由定义：`internal/transport/http/router/relay.go` + `internal/transport/http/router/dashboard.go` + `internal/transport/http/router/api.go`

OpenAI 兼容路由（TokenAuth + Distribute）：
- `/v1/models` 与 `/api/v1/public/models`（列模型）
- `/v1/completions`, `/v1/chat/completions`, `/v1/embeddings`, `/v1/audio/*`, `/v1/moderations`, `/v1/images/generations`  
  对应新增 `/api/v1/public/*` 同名路径
- 未实现的接口走 `RelayNotImplemented`

Dashboard 兼容路由（TokenAuth）：
- `/dashboard/billing/subscription`
- `/dashboard/billing/usage`
- `/v1/dashboard/billing/subscription`
- `/v1/dashboard/billing/usage`

## D. /v1 直连禁用开关
- 环境变量 `DISABLE_OPENAI_COMPAT=true` 时，不注册 `/v1/*` 与 `/dashboard/*`
- `/api/v1/public/*` 仍可用，用于替代 `/v1/*`

## E. Relay 核心调用链路（模型调用流程）
1) **TokenAuth 鉴权 + 模型校验**  
   `internal/transport/http/middleware/auth.go`  
   - 优先钱包 JWT，失败回退到 `sk-` 令牌  
   - 校验模型/网段/可用模型列表  
   - 解析请求中的 model（`internal/transport/http/middleware/utils.go`）
2) **Distribute 选渠道**  
   `internal/transport/http/middleware/distributor.go`  
   - 根据用户组 + 模型选择可用 channel  
   - 写入 ctx：channel / key / baseURL / model mapping
3) **Relay 入口与重试**  
   `internal/admin/controller/relay.go`  
   - 根据路径决定 relayMode  
   - 失败可按规则重试其他 channel
4) **请求转换/计费/供应商适配**  
   `internal/relay/controller/*`  
   - 预扣费、转换请求、调用适配器、解析响应、结算扣费
5) **适配器层（供应商集成）**  
   `internal/relay/adaptor/*`  
   - 通过 `internal/relay/adaptor.go` 按 APIType 选实现  
   - 供应商包括 OpenAI/Gemini/Anthropic/阿里/腾讯/智谱/Ollama 等

## F. 前端与文档提示
- 旧 API 文档：`docs/API.md`（说明 Cookie/Token 两种鉴权，但 API 列表不全）
- 前端会调用 `api/v1/public/*` 与 `/api/*`（见 `web/src/helpers/web3.jsx`、`web/src/components/PersonalSetting.jsx`）

## G. 迁移期策略
- **短期**：`/v1/*` 继续保留，但新接入与前端改用 `/api/v1/public/*`
- **管理端**：运营/管理接口迁移到 `/api/v1/admin/*`（旧 `/api/*` 继续镜像）
- **关闭直连**：待外部调用迁移完毕后，可设 `DISABLE_OPENAI_COMPAT=true` 禁用 `/v1/*` 与 `/dashboard/*`
- **风险提示**：部分第三方 SDK 默认调用 `/v1/*`，需要提前替换基地址

## H. 常见定位入口（快速查找）
- 路由总表：`internal/transport/http/router/api.go`, `internal/transport/http/router/relay.go`
- 鉴权：`internal/transport/http/middleware/auth.go`
- 渠道选择：`internal/transport/http/middleware/distributor.go`
- Relay 入口：`internal/admin/controller/relay.go`
- 适配器注册：`internal/relay/adaptor.go`

## I. 近期新增
- 新增 admin 接口：`POST /api/v1/admin/channel/preview/models`  
  用途：创建/编辑渠道时预览 OpenAI 兼容渠道的模型列表；鉴权 `AdminAuth`；不提供旧 `/api/*` 映射。

## J. 接口定义同步提示
- 当前仓库无 `interface/` 目录；如需更新接口 proto，请在 `yeying/api/<app>` 补充对应接口定义。

## K. 用户 Dashboard 统计扩展
- `/api/v1/public/user/dashboard`（及旧 `/api/user/dashboard`）新增参数：
  `start_timestamp` / `end_timestamp` / `granularity=hour|day|week|month|year` / `models` / `include_meta=1`。
- `include_meta=1` 返回 `meta.providers`（模型供应商树）以及 `meta.granularity/start/end`。
- **模型供应商解析规则（后端统一）**：
  - 若模型名包含 `/`，取 `/` 前缀作为 provider（小写）。
  - 否则按前缀匹配：  
    `gpt-*`, `o1*`, `o3*`, `chatgpt-*` → openai  
    `claude-*` → anthropic  
    `gemini-*` → google  
    `deepseek-*` → deepseek  
    `qwen-*`, `qwq-*`, `qvq-*` → qwen  
    `glm-*`, `cogview-*` → zhipu  
    `ERNIE-*` → baidu  
    其他 → unknown

## L. 用户花费总览
- `GET /api/v1/public/user/spend/overview`（JWT / UserAuth）
  - period：`last_week | last_month | this_year | last_year | last_12_months | all_time`
  - 返回昨日/周期的消费与充值汇总（quota），以及对应的时间范围（Unix 秒）。
