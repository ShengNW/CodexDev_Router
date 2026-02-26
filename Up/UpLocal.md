# Router 部署实战手册（解压后起步）

适用环境：`Ubuntu 24.04.2 LTS` + `x86_64` + `systemd` + `Nginx`  
文档起点：你已经把安装包解压到服务器指定目录。  
本次实测包：`router_new-20260226-081156-9574086.tar.gz`

---

## 0. 开局先定盘：你现在要达成的目标

你要完成的是一条完整链路，而不是只“跑起来一个进程”：

1. 用 `scripts/starter.sh` 手工验证程序可起停。
2. 把手工命令收敛到 `systemd`，让服务可开机自启、可托管重启。
3. 用 `Nginx` 把公网请求转发到 `127.0.0.1:3011`。
4. 验证数据库确实连的是 PostgreSQL（日志看到 `openPostgreSQL`）。

如果这四步都成立，才算部署闭环完成。

---

## 1. 一图看懂整体架构

```mermaid
flowchart LR
    A[Browser / API Client] -->|HTTPS 443| B[Nginx]
    B -->|Proxy to 127.0.0.1:3011| C[Router Process]
    D[systemd] -->|start/stop/restart| E[scripts/starter.sh]
    E -->|manage pid + nohup| C
    C -->|SQL_DSN| F[(PostgreSQL)]
    C -->|serve static| G[web/dist]
```

关键理解：

- `starter.sh` 是“进程控制器”，负责起停和 PID 管理。
- `systemd` 是“总调度器”，调用 `starter.sh`，负责托管与自启动。
- `Nginx` 只负责公网入口与反向代理，不直接管理进程。

---

## 2. 先站稳脚：确认解压目录与内容

假设你解压到了：

- `/opt/router/releases/router_new-20260226-081156-9574086`

进入目录并检查核心文件：

```bash
cd /opt/router/releases/router_new-20260226-081156-9574086
ls -la
```

你应看到至少这几项：

- `build/router`（后端二进制）
- `.env.template`（配置模板）
- `web/dist/`（前端静态资源）
- `scripts/starter.sh`（启动脚本）

---

## 3. 配置是生命线：先准备 `.env`

从模板生成配置（首次部署）：

```bash
cd /opt/router/releases/router_new-20260226-081156-9574086
cp -n .env.template .env
```

编辑 `.env`，最关键是 `SQL_DSN`。示例：

```env
SQL_DSN=postgres://router_user:router_pass@10.0.0.10:5432/router_db?sslmode=disable
```

原理说明（非常重要）：

- 程序会在工作目录读取 `.env`。
- `SQL_DSN` 是 `postgres://...` 才会连 PG。
- 如果 `SQL_DSN` 丢失或为空，可能回落到 SQLite。

---

## 4. 手工验收 `starter.sh`：先把单机起停跑通

先加执行权限（保险起见）：

```bash
cd /opt/router/releases/router_new-20260226-081156-9574086
chmod +x scripts/starter.sh build/router
```

### 4.1 启动

```bash
cd /opt/router/releases/router_new-20260226-081156-9574086
./scripts/starter.sh start
```

### 4.2 健康检查

```bash
curl -sS http://127.0.0.1:3011/api/status
```

看到 `success:true` 即服务活着。

### 4.3 检查数据库类型

```bash
cd /opt/router/releases/router_new-20260226-081156-9574086
rg -n "openPostgreSQL|openSQLite|openMySQL" logs/starter.log logs/oneapi-*.log -S
```

你要看到 `openPostgreSQL`。

### 4.4 停止与重启

```bash
cd /opt/router/releases/router_new-20260226-081156-9574086
./scripts/starter.sh stop
./scripts/starter.sh restart
./scripts/starter.sh stop
```

`starter.sh` 行为速记：

- `start`：若已运行则不重复启动（幂等）
- `stop`：先优雅停止，超时后强制结束
- `restart`：先停后起

---

## 5. 交给 systemd 托管（生产标准做法）

建议用固定软链接 `current` 指向当前版本，便于升级切换：

```bash
sudo mkdir -p /opt/router/releases
sudo ln -sfn /opt/router/releases/router_new-20260226-081156-9574086 /opt/router/current
```

### 5.1 systemd 服务文件

创建 `/etc/systemd/system/router.service`：

```bash
sudo tee /etc/systemd/system/router.service >/dev/null <<'EOF'
[Unit]
Description=Router Service (managed by starter.sh)
After=network-online.target
Wants=network-online.target

[Service]
Type=forking
WorkingDirectory=/opt/router/current
ExecStart=/opt/router/current/scripts/starter.sh start
ExecStop=/opt/router/current/scripts/starter.sh stop
ExecReload=/opt/router/current/scripts/starter.sh restart
PIDFile=/opt/router/current/run/router.pid
Restart=always
RestartSec=3
Environment=ROUTER_PORT=3011
Environment=ROUTER_LOG_DIR=/opt/router/current/logs

[Install]
WantedBy=multi-user.target
EOF
```

### 5.2 载入并启动

```bash
sudo systemctl daemon-reload
sudo systemctl enable router
sudo systemctl restart router
sudo systemctl status router --no-pager -l
```

### 5.3 systemd 下的关键验证

```bash
curl -sS http://127.0.0.1:3011/api/status
journalctl -u router --since "10 min ago" --no-pager | rg "openPostgreSQL|openSQLite|openMySQL" -S
```

必须命中 `openPostgreSQL`。

---

## 6. Nginx 反代到 3011（公网入口）

编辑 `/etc/nginx/conf.d/router.conf`：

```nginx
server {
    listen 80;
    server_name router.your-domain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name router.your-domain.com;

    ssl_certificate     /etc/letsencrypt/live/router.your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/router.your-domain.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:3011;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

检查并重载：

```bash
sudo nginx -t
sudo systemctl reload nginx
```

公网验证：

```bash
curl -I https://router.your-domain.com
```

---

## 7. 升级时如何“稳准狠”切换版本

升级的关键不是“快”，而是“可回滚”。

```mermaid
sequenceDiagram
    participant O as Old Version(current)
    participant S as systemd(router)
    participant N as New Version(release)
    S->>O: stop
    Note over O,S: 释放3011端口，清理旧pid
    O-->>N: 复制 .env
    S->>N: 切换 current 软链接
    S->>N: start
    N-->>S: /api/status = success:true
    Note over S,N: 若失败，软链接切回旧版本并 restart
```

标准切换命令（新版本已解压）：

```bash
export NEW=/opt/router/releases/<new_package_dir>
sudo cp -a /opt/router/current/.env "$NEW/.env"
sudo systemctl stop router
sudo ln -sfn "$NEW" /opt/router/current
sudo systemctl start router
sudo systemctl status router --no-pager -l
```

回滚命令（秒级恢复）：

```bash
sudo systemctl stop router
sudo ln -sfn /opt/router/releases/<old_package_dir> /opt/router/current
sudo systemctl start router
```

---

## 8. `starter.sh` 起停命令卡片（可直接抄）

```bash
cd /opt/router/current
./scripts/starter.sh start
./scripts/starter.sh stop
./scripts/starter.sh restart
```

可选环境变量：

- `ROUTER_PORT`：默认 `3011`
- `ROUTER_LOG_DIR`：默认 `./logs`

示例（临时改端口）：

```bash
cd /opt/router/current
ROUTER_PORT=3012 ./scripts/starter.sh restart
```

---

## 9. 一次性验收清单（上线前逐条打勾）

```bash
# 1) 进程由 systemd 托管
systemctl is-enabled router
systemctl is-active router

# 2) 本机健康检查
curl -sS http://127.0.0.1:3011/api/status

# 3) 数据库类型确认
journalctl -u router --since "today" --no-pager | rg "openPostgreSQL|openSQLite|openMySQL" -S

# 4) Nginx 正常
nginx -t
curl -I https://router.your-domain.com
```

验收标准：

- `is-enabled=enabled`
- `is-active=active`
- `/api/status` 返回 `success:true`
- 日志出现 `openPostgreSQL`
- 公网 `HTTP 200`（或 301 到 HTTPS 后 200）

---

## 10. 故障排查：三大高频问题

### 问题 A：服务起来了，但数据库不对（变成 SQLite）

排查：

```bash
cd /opt/router/current
ls -la .env
rg -n "^SQL_DSN=" .env
journalctl -u router --since "20 min ago" --no-pager | rg "openPostgreSQL|openSQLite|openMySQL" -S
```

原因：`.env` 缺失、`SQL_DSN` 空值、工作目录不对。

---

### 问题 B：`502 Bad Gateway`

排查：

```bash
systemctl status router --no-pager -l
curl -sS http://127.0.0.1:3011/api/status
nginx -t
systemctl status nginx --no-pager -l
```

原因：Router 没监听成功，或 Nginx 反代端口写错。

---

### 问题 C：`starter.sh stop` 说停了但端口还占用

排查：

```bash
ss -lntp | rg ":3011"
ps -ef | rg "build/router|cmd/router" -S
cat /opt/router/current/run/router.pid 2>/dev/null || true
```

原因：历史残留进程或 PID 文件不一致。先人工清理残留，再 `systemctl restart router`。

---

## 11. 给新手的一句总纲

把它记成三层：

1. `starter.sh` 负责“进程起停”。
2. `systemd` 负责“长期托管与开机自启”。
3. `Nginx` 负责“公网入口与 HTTPS”。

每次上线都按这个次序验收：  
`Router(本机)` -> `PG(日志)` -> `Nginx(公网)`。  
顺序不乱，系统就稳。
