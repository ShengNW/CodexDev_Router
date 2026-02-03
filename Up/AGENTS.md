# 启动总览（通用）

本目录用于汇总“通用启动逻辑”，分为：
- 公网（服务器）启动
- 本地（开发机）启动

若在服务器环境执行上线/重启，请优先遵循项目根目录 `AGENTS.md` 的具体链路要求。

---

## 公网（服务器）启动逻辑

目标：确保公网可访问且正确连接 PG。

核心链路：
- systemd 管理服务
- Router 监听 3011
- Nginx 反代到 127.0.0.1:3011
- SQL_DSN 指向 PG

推荐流程（高层）：
1) 更新代码并构建
2) 重启服务
3) 验证 PG
4) 验证公网

关键校验：
- 日志需出现 `openPostgreSQL`
- `curl http://127.0.0.1:3011/api/status` 返回 success:true
- `curl -I https://router.yeying.pub` 返回 200

常见风险：
- 忘记构建前端（web/build）
- SQL_DSN 丢失导致回落 SQLite
- Nginx 反代端口不一致

---

## 本地（开发机）启动逻辑

目标：快速起服务进行功能验证。

推荐流程（高层）：
1) 准备环境：Go 1.22+，Node 18+，npm
2) npm 安装与构建前端
3) go run 或 build 启动后端
4) 本地健康检查

关键校验：
- 日志出现 `openSQLite`（默认）或 `openPostgreSQL`（如配置了 SQL_DSN）
- `curl http://localhost:3011/api/status` 返回 success:true

常见风险：
- tiktoken 下载失败（代理指向 127.0.0.1:7890）
  解决：移除 .env 里的 HTTPS_PROXY 或显式 `env -u ...` 启动
- Node 版本过旧导致 Vite 构建失败

---

## 使用提示（对 Codex 的指令示例）
- 服务器："你现在在服务器环境，帮我公网启动" → 遵循项目根目录 `AGENTS.md`
- 本地："你现在在本地环境，帮我本地启动" → 按本文件流程执行
