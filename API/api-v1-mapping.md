# /api → /api/v1 分层映射

## 分层原则
- public：面向终端用户/前端/集成方的自助接口
- admin：运营/管理接口（Admin/Root 权限）
- internal：内部系统/任务接口（当前预留）

## 说明
- 旧 `/api/*` 路径全部保留（兼容期），新增 `/api/v1/{public|admin|internal}/*` 镜像路由。
- 已有 `/api/v1/public/*` 钱包/UCAN 相关接口保持不变。
- **冲突处理**：`/api/models` 与 OpenAI 兼容的 `/api/v1/public/models` 冲突，因此镜像路径使用 `/api/v1/public/models-all`（功能等同于旧 `/api/models`）。

## 已符合规范（保持不变）
- `/api/v1/public/common/auth/*`（proto 钱包认证）
- `/api/v1/public/auth/*`（web3 钱包认证）
- `/api/v1/public/profile`（UCAN 或钱包 JWT）

## Public（用户/自助）

### 公共信息 / 找回密码
| 方法 | 旧路径 | 新路径 | 权限/中间件 | Handler |
| --- | --- | --- | --- | --- |
| GET | `/api/status` | `/api/v1/public/status` | - | `admin.GetStatus` |
| GET | `/api/notice` | `/api/v1/public/notice` | - | `admin.GetNotice` |
| GET | `/api/about` | `/api/v1/public/about` | - | `admin.GetAbout` |
| GET | `/api/home_page_content` | `/api/v1/public/home_page_content` | - | `admin.GetHomePageContent` |
| GET | `/api/reset_password` | `/api/v1/public/reset_password` | `CriticalRateLimit` | `admin.SendPasswordResetEmail` |
| POST | `/api/user/reset` | `/api/v1/public/user/reset` | `CriticalRateLimit` | `admin.ResetPassword` |

### 钱包 OAuth
| 方法 | 旧路径 | 新路径 | 权限/中间件 | Handler |
| --- | --- | --- | --- | --- |
| GET | `/api/oauth/wallet/nonce` | `/api/v1/public/oauth/wallet/nonce` | `CriticalRateLimit` | `auth.WalletNonce` |
| POST | `/api/oauth/wallet/login` | `/api/v1/public/oauth/wallet/login` | `CriticalRateLimit` | `auth.WalletLogin` |
| POST | `/api/oauth/wallet/bind` | `/api/v1/public/oauth/wallet/bind` | `CriticalRateLimit`, `UserAuth` | `auth.WalletBind` |

### 用户自助
| 方法 | 旧路径 | 新路径 | 权限/中间件 | Handler |
| --- | --- | --- | --- | --- |
| POST | `/api/user/register` | `/api/v1/public/user/register` | `CriticalRateLimit` | `user.Register` |
| POST | `/api/user/login` | `/api/v1/public/user/login` | `CriticalRateLimit` | `user.Login` |
| GET | `/api/user/logout` | `/api/v1/public/user/logout` | - | `user.Logout` |
| GET | `/api/user/self` | `/api/v1/public/user/self` | `UserAuth` | `user.GetSelf` |
| PUT | `/api/user/self` | `/api/v1/public/user/self` | `UserAuth` | `user.UpdateSelf` |
| DELETE | `/api/user/self` | `/api/v1/public/user/self` | `UserAuth` | `user.DeleteSelf` |
| GET | `/api/user/dashboard` | `/api/v1/public/user/dashboard` | `UserAuth` | `user.GetUserDashboard` |
| GET | 无旧路径 | `/api/v1/public/user/spend/overview` | `UserAuth` | `user.GetUserSpendOverview` |
| GET | `/api/user/available_models` | `/api/v1/public/user/available_models` | `UserAuth` | `admin.GetUserAvailableModels` |
| GET | `/api/user/token` | `/api/v1/public/user/token` | `UserAuth` | `user.GenerateAccessToken` |
| GET | `/api/user/aff` | `/api/v1/public/user/aff` | `UserAuth` | `user.GetAffCode` |
| POST | `/api/user/topup` | `/api/v1/public/user/topup` | `UserAuth` | `user.TopUp` |

补充说明：
- `/api/user/dashboard` 与 `/api/v1/public/user/dashboard` 支持新增查询参数：
  `start_timestamp` / `end_timestamp` / `granularity=hour|day|week|month|year` / `models` / `include_meta=1`（详见 `docs/API.v1.md`）。

### 个人 Token 管理
| 方法 | 旧路径 | 新路径 | 权限/中间件 | Handler |
| --- | --- | --- | --- | --- |
| GET | `/api/token` | `/api/v1/public/token` | `UserAuth` | `token.GetAllTokens` |
| GET | `/api/token/search` | `/api/v1/public/token/search` | `UserAuth` | `token.SearchTokens` |
| GET | `/api/token/:id` | `/api/v1/public/token/:id` | `UserAuth` | `token.GetToken` |
| POST | `/api/token` | `/api/v1/public/token` | `UserAuth` | `token.AddToken` |
| PUT | `/api/token` | `/api/v1/public/token` | `UserAuth` | `token.UpdateToken` |
| DELETE | `/api/token/:id` | `/api/v1/public/token/:id` | `UserAuth` | `token.DeleteToken` |

### 个人日志
| 方法 | 旧路径 | 新路径 | 权限/中间件 | Handler |
| --- | --- | --- | --- | --- |
| GET | `/api/log/self` | `/api/v1/public/log/self` | `UserAuth` | `log.GetUserLogs` |
| GET | `/api/log/self/stat` | `/api/v1/public/log/self/stat` | `UserAuth` | `log.GetLogsSelfStat` |
| GET | `/api/log/self/search` | `/api/v1/public/log/self/search` | `UserAuth` | `log.SearchUserLogs` |

### 模型/渠道
| 方法 | 旧路径 | 新路径 | 权限/中间件 | Handler |
| --- | --- | --- | --- | --- |
| GET | `/api/models` | `/api/v1/public/models-all` | `UserAuth` | `admin.ListAllModels` |
| GET | `/api/channel/models` | `/api/v1/public/channel/models` | `UserAuth` | `admin.DashboardListModels` |

## Admin（后台管理）

### 用户管理
| 方法 | 旧路径 | 新路径 | 权限/中间件 | Handler |
| --- | --- | --- | --- | --- |
| GET | `/api/user` | `/api/v1/admin/user` | `AdminAuth` | `user.GetAllUsers` |
| GET | `/api/user/search` | `/api/v1/admin/user/search` | `AdminAuth` | `user.SearchUsers` |
| GET | `/api/user/:id` | `/api/v1/admin/user/:id` | `AdminAuth` | `user.GetUser` |
| POST | `/api/user` | `/api/v1/admin/user` | `AdminAuth` | `user.CreateUser` |
| POST | `/api/user/manage` | `/api/v1/admin/user/manage` | `AdminAuth` | `user.ManageUser` |
| PUT | `/api/user` | `/api/v1/admin/user` | `AdminAuth` | `user.UpdateUser` |
| DELETE | `/api/user/:id` | `/api/v1/admin/user/:id` | `AdminAuth` | `user.DeleteUser` |

### 系统配置（Root）
| 方法 | 旧路径 | 新路径 | 权限/中间件 | Handler |
| --- | --- | --- | --- | --- |
| GET | `/api/option` | `/api/v1/admin/option` | `RootAuth` | `option.GetOptions` |
| PUT | `/api/option` | `/api/v1/admin/option` | `RootAuth` | `option.UpdateOption` |

### 渠道管理
| 方法 | 旧路径 | 新路径 | 权限/中间件 | Handler |
| --- | --- | --- | --- | --- |
| GET | `/api/channel` | `/api/v1/admin/channel` | `AdminAuth` | `channel.GetAllChannels` |
| GET | `/api/channel/search` | `/api/v1/admin/channel/search` | `AdminAuth` | `channel.SearchChannels` |
| GET | `/api/channel/:id` | `/api/v1/admin/channel/:id` | `AdminAuth` | `channel.GetChannel` |
| GET | `/api/channel/test` | `/api/v1/admin/channel/test` | `AdminAuth` | `channel.TestChannels` |
| GET | `/api/channel/test/:id` | `/api/v1/admin/channel/test/:id` | `AdminAuth` | `channel.TestChannel` |
| GET | `/api/channel/update_balance` | `/api/v1/admin/channel/update_balance` | `AdminAuth` | `channel.UpdateAllChannelsBalance` |
| GET | `/api/channel/update_balance/:id` | `/api/v1/admin/channel/update_balance/:id` | `AdminAuth` | `channel.UpdateChannelBalance` |
| POST | 无旧路径 | `/api/v1/admin/channel/preview/models` | `AdminAuth` | `channel.PreviewChannelModels` |
| POST | `/api/channel` | `/api/v1/admin/channel` | `AdminAuth` | `channel.AddChannel` |
| PUT | `/api/channel` | `/api/v1/admin/channel` | `AdminAuth` | `channel.UpdateChannel` |
| DELETE | `/api/channel/disabled` | `/api/v1/admin/channel/disabled` | `AdminAuth` | `channel.DeleteDisabledChannel` |
| DELETE | `/api/channel/:id` | `/api/v1/admin/channel/:id` | `AdminAuth` | `channel.DeleteChannel` |

### 兑换码管理
| 方法 | 旧路径 | 新路径 | 权限/中间件 | Handler |
| --- | --- | --- | --- | --- |
| GET | `/api/redemption` | `/api/v1/admin/redemption` | `AdminAuth` | `admin.GetAllRedemptions` |
| GET | `/api/redemption/search` | `/api/v1/admin/redemption/search` | `AdminAuth` | `admin.SearchRedemptions` |
| GET | `/api/redemption/:id` | `/api/v1/admin/redemption/:id` | `AdminAuth` | `admin.GetRedemption` |
| POST | `/api/redemption` | `/api/v1/admin/redemption` | `AdminAuth` | `admin.AddRedemption` |
| PUT | `/api/redemption` | `/api/v1/admin/redemption` | `AdminAuth` | `admin.UpdateRedemption` |
| DELETE | `/api/redemption/:id` | `/api/v1/admin/redemption/:id` | `AdminAuth` | `admin.DeleteRedemption` |

### 日志管理
| 方法 | 旧路径 | 新路径 | 权限/中间件 | Handler |
| --- | --- | --- | --- | --- |
| GET | `/api/log` | `/api/v1/admin/log` | `AdminAuth` | `log.GetAllLogs` |
| DELETE | `/api/log` | `/api/v1/admin/log` | `AdminAuth` | `log.DeleteHistoryLogs` |
| GET | `/api/log/stat` | `/api/v1/admin/log/stat` | `AdminAuth` | `log.GetLogsStat` |
| GET | `/api/log/search` | `/api/v1/admin/log/search` | `AdminAuth` | `log.SearchAllLogs` |

### 分组管理
| 方法 | 旧路径 | 新路径 | 权限/中间件 | Handler |
| --- | --- | --- | --- | --- |
| GET | `/api/group` | `/api/v1/admin/group` | `AdminAuth` | `group.GetGroups` |

## Internal
- 暂无接口，预留 `/api/v1/internal/*` 供内部任务/系统级接口接入。
