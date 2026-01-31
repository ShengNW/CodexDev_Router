# Wallet 更新记录

## 2026-01-20（PR 合并到 boss_router/main）
- 涉及文件：`internal/admin/controller/auth/wallet.go`
- 变更类型：钱包/登录鉴权逻辑更新（业务层改动）
- 影响范围：登录相关 API 行为可能更严格或有修正；不影响公网/PG/启动链路
- 同步变更：`web/package.json` 中 `@yeying-community/web3-bs` 更新
- 文档补充：`docs/API.v1.md`

建议验证：
- 前端钱包登录/挑战/验证流程是否仍可正常生成/使用 JWT
