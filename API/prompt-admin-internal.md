你是本仓库的执行型 Codex，负责“/api/v1/admin / internal 分层迁移”。你要大胆推进并落地代码，不要只提建议。先阅读：
- `/root/code/router/router_new/AGENTS.md`
- `/root/code/router/router_new/CodexDev/API/notes.md`

你必须严格遵循以下原则：
- **只基于现有接口**做分层改版映射，不新增业务功能。
- **每一个现有 /api 接口都必须有明确的新路径映射**（/api/v1/public 或 /api/v1/admin 或 /api/v1/internal）。
- 已经符合公司规范的接口可保留不变（例如现有 `/api/v1/public/*` 钱包/UCAN相关）。
- 旧 `/api/*` 路径全部保留（兼容期），新路径是“镜像路由”。

分层依据（来自业务角色/权限关系）：
1) **public（自助/用户侧）**
   - 面向终端用户/前端/集成方的自助功能
   - 包含：登录注册、找回、钱包登录、用户自助信息、个人令牌、个人日志、兑换码自用、个人设置、用户侧 models list
2) **admin（后台管理/运营）**
   - Admin/Root 执行的管理操作（用户、渠道、日志、兑换码、手工充值、全局 token 管理、分组）
   - Root 专属设置也归入 admin 类别（运营/系统设置）
3) **internal（内部系统/任务）**
   - 若当前没有明确 internal 接口，可先留空或只在文档中保留占位；不要凭空发明内部接口。

实现任务（必须完成）：

A. **枚举并映射所有现有 /api 路由**
- 主要文件：`internal/transport/http/router/api.go`
- 你要列出每一个 /api 路径，给出对应的新路径映射
- 新路径格式：`/api/v1/{public|admin|internal}/...`
  （最小改动原则：保留原路径后半部分，仅替换前缀）
- 每条映射需标注：旧路径、HTTP 方法、权限、中间件、Handler

B. **实际代码实现镜像路由**
- 在 `internal/transport/http/router/api.go` 中新增 `/api/v1/public` 与 `/api/v1/admin` 路由组
- 对每条旧接口新增对应新接口，绑定同一 Handler 与中间件
- `/api/v1/internal` 如果暂无接口，也请预留空组（便于未来扩展）

C. **输出“映射清单文档”**
- 如果映射较多，请新建：`/root/code/router/router_new/CodexDev/API/api-v1-mapping.md`
- 文档需包含：
  - 分层原则（简述）
  - public/admin/internal 三类清单
  - 每条接口的**旧路径 → 新路径**映射表（含权限、handler）
  - 已经是 /api/v1/public 的接口注明“已符合规范，保持不变”
- 同时更新 `CodexDev/API/notes.md`，添加指向该映射文档的链接和迁移说明

分层映射参考（你必须严格覆盖全部旧接口）：

【public】（用户/集成方/前端自助）
- 登录注册/找回：`/api/user/register`, `/api/user/login`, `/api/user/logout`, `/api/reset_password`, `/api/user/reset`
- 钱包登录/绑定：`/api/oauth/wallet/nonce`, `/api/oauth/wallet/login`, `/api/oauth/wallet/bind`
- 用户自助信息：`/api/user/self`, `/api/user/dashboard`, `/api/user/available_models`
- 个人设置：`/api/user/self`(PUT/DELETE), `/api/user/aff`, `/api/user/token`, `/api/user/topup`
- 个人 token：`/api/token/*`（UserAuth）
- 个人日志：`/api/log/self`, `/api/log/self/stat`, `/api/log/self/search`
- 用户侧 models list：`/api/models`, `/api/channel/models`
- 公共信息：`/api/status`, `/api/notice`, `/api/about`, `/api/home_page_content`

【admin】（后台管理/运营）
- 用户管理：`/api/user`(GET/POST/PUT/DELETE), `/api/user/search`, `/api/user/:id`, `/api/user/manage`
- 渠道管理：`/api/channel/*`（除 `/models` 外）
- 兑换码管理：`/api/redemption/*`
- 日志管理：`/api/log`, `/api/log/stat`, `/api/log/search`, `/api/log`(DELETE)
- 分组管理：`/api/group`
- Root 设置：`/api/option`（RootAuth）

【internal】
- 若当前无明确 internal 接口，先在文档留空并说明 “待内部门户/Job 接入后补齐”。

交付要求：
- 必须有代码改动（镜像路由）
- 必须有映射文档
- 每个旧接口都要有新路径映射（逐条核对，不可遗漏）
- 保持旧接口不动（兼容期）
- 不修改 Nginx/部署，仅在文档中可提出建议

汇报格式：
- 变更文件列表
- 新增的 /api/v1/public & /api/v1/admin 路由清单
- 映射文档路径
- 是否存在潜在破坏性变动
