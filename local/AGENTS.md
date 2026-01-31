注意如果你读到这个AGENTS.md说明你是本地起的，配置跟项目根目录的AGENTS.md有所不同，所以一切以实际为准

# 本地调研速记（local 专用）

## 目录与 Git 根
- 当前目录：`/home/snw/Codex/Router/Router_new/CodexDev/local`
- 本目录所在的 Git 根：`/home/snw/Codex/Router/Router_new/CodexDev`
- **Router 项目根目录**：`/home/snw/Codex/Router/Router_new`
- 进入任何子目录先读 `notes.md`（若存在），并主动去读项目根的 `AGENTS.md`

## 本地启动 Router（开发）
在 Router 根目录执行：
```bash
cp .env.template .env   # 可选，按需配置
go mod download
go run ./cmd/router --log-dir ./logs
```
前端热更新（可选）：
```bash
npm install --prefix web
npm start --prefix web   # 自动代理到 http://localhost:3011
```

## 线上 vs 本地（关键差异）
- 线上通常由 systemd 启动（WorkingDirectory 必须是仓库根，否则 .env 不加载）
- 线上通过 Nginx 反代到 `127.0.0.1:3011` + HTTPS 终止
- 线上需显式配置 `UCAN_AUD` / `AUTO_REGISTER_ENABLED`（systemd 不自动加载 .env）
- DB 由 `.env` 的 `SQL_DSN` 决定：`postgres://...` 才是 PG，否则可能落到 SQLite

## 必读文件
- 项目根说明：`/home/snw/Codex/Router/Router_new/AGENTS.md`
- 快速开始：`/home/snw/Codex/Router/Router_new/README.md`

## 本地启动记录
- 详细步骤与排错：`/home/snw/Codex/Router/Router_new/CodexDev/local/localUp.md`
