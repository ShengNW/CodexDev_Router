# API 对接速览（只读）

本文件供“对接负责人/秘书”快速了解 Router 的 API 现状与信息入口。  
仅用于调研与沟通，不涉及代码修改。

## 1) 必读规范与现有文档
- **公司 Interface 规范与分层**：`/root/code/router/router_new/API_reference.md`
- **对外 v1 文档（JWT + public/admin/internal）**：`/root/code/router/router_new/docs/API.v1.md`
- **旧接口 → 新接口映射表**：`/root/code/router/router_new/CodexDev/API/api-v1-mapping.md`
- **架构梳理与调用链路摘要**：`/root/code/router/router_new/CodexDev/API/notes.md`

> 提醒：旧 `/api/*` 与 `/v1/*` 属兼容通道，不作为对外首选说明。对外统一推荐 `/api/v1/*`。

## 2) 入口代码（查接口/逻辑最快路径）
路由入口与注册：
- `internal/transport/http/router/api.go`（/api/v1/public + /api/v1/admin + /api/v1/internal）
- `internal/transport/http/router/relay.go`（OpenAI 兼容 /v1）
- `internal/transport/http/router/dashboard.go`（/dashboard 与 /v1/dashboard）

鉴权与限流：
- `internal/transport/http/middleware/auth.go`（JWT/TokenAuth）
- `internal/transport/http/middleware/rate-limit.go`
- `internal/transport/http/middleware/cors.go`
- `internal/transport/http/middleware/distributor.go`（渠道分发）

模型调用与计费链路：
- `internal/admin/controller/relay.go`（入口/重试）
- `internal/relay/controller/*`（文本/图片/音频）
- `internal/relay/adaptor/*`（各供应商适配器）
- `internal/relay/billing/*`（计费与扣费）
- `internal/relay/meta/*`（元信息/上下文）

## 3) 当前对外 API 结构（简要）
**规范路径（推荐）：**
- `/api/v1/public/*`：用户侧 + OpenAI 兼容调用
- `/api/v1/admin/*`：运营/管理
- `/api/v1/internal/*`：预留（当前无接口）

**兼容路径（历史保留）：**
- `/api/*`：旧管理/用户接口
- `/v1/*` 与 `/dashboard/*`：OpenAI 兼容

**兼容开关：**
- `DISABLE_OPENAI_COMPAT=true` 可禁用 `/v1/*` 与 `/dashboard/*`

## 4) JWT 与权限说明（对外沟通口径）
统一用 JWT：
```
Authorization: Bearer <JWT>
```
JWT 来源：
- 钱包登录（/api/v1/public/common/auth 或 /api/v1/public/auth）
- `/api/v1/public/profile` 支持钱包 JWT 或 UCAN

权限分层：
- public：普通用户 JWT
- admin：管理员 JWT
- root：Root JWT（系统配置）

## 5) 测试与排障提示（只读）
最小可用性验证：
- `GET /api/v1/public/models`
- `POST /api/v1/public/chat/completions`

排障日志与环境：
- 运行方式、PG 连接等信息在根目录 `AGENTS.md`

---

如需更完整接口清单、请求/响应结构，请优先查 `docs/API.v1.md` 与 `api-v1-mapping.md`，  
若需要确认实现细节，再定位到 `internal/transport/http/router/api.go` 等代码入口。
