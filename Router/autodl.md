# Autodl 上 Codex CLI 走 Router 配置记录

目的：在全新的 Autodl 机器上，直接运行 `codex`（读取配置）也能通过 Router 成功调用模型。

## 结论（一句话）
**关键是让 Codex CLI 读取到正确的 provider 配置 + 能拿到 `ROUTER_API_KEY`（注意非交互 shell 不会读取 `.bashrc`）。**

---

## 最小配置（写入 config.toml）
路径：`/root/.codex/config.toml`

```toml
preferred_auth_method = "apikey"
model_provider = "router"
model = "gpt-5.2-codex"
model_reasoning_effort = "xhigh"
disable_response_storage = true
approval_policy = "never"
sandbox_mode = "danger-full-access"

[features]
web_search_request = true
apply_patch_freeform = true
view_image_tool = true
rmcp_client = true
remote_models = false

[model_providers.router]
name = "Router"
base_url = "https://router.yeying.pub/v1"
wire_api = "responses"
env_key = "ROUTER_API_KEY"
```

说明：
- `model_provider = "router"` 避免 OpenAI provider 的额外逻辑。
- `wire_api = "responses"` 与 Codex 模型路径一致。
- `remote_models = false` 避免 CLI 启动前刷新模型列表超时卡住。

---

## 环境变量（关键点）
**不要只写 `.bashrc`**，因为非交互 shell 会直接 return，导致 `ROUTER_API_KEY` 读不到。

推荐写到系统级 profile：

```bash
cat > /etc/profile.d/router_api_key.sh <<'EOF2'
# Codex Router API key
export ROUTER_API_KEY="sk-你的key"
EOF2
chmod 644 /etc/profile.d/router_api_key.sh
```

然后新开 shell 或：

```bash
source /etc/profile
```

---

## 验证（直接读配置）
在工作目录执行：

```bash
codex exec "只回答 ok"
```

预期输出：
- `provider: router`
- 返回 `ok`

---

## 常见坑
1. **只写 `.bashrc` 导致 key 缺失**
   - 现象：`ERROR: Missing environment variable: ROUTER_API_KEY`
   - 解决：写入 `/etc/profile.d/` 或在启动前显式 `export`。

2. **IPv6 失败导致请求不出本地**
   - 现象：CLI 长时间超时、服务端无日志。
   - 解决：`/etc/hosts` 强制 IPv4 或禁用 IPv6（若有权限）。

3. **Token 无权使用模型**
   - 现象：`403 该令牌无权使用模型 gpt-5.2-codex`
   - 解决：后台为 token 增加模型权限（或清空 models allowlist）。

---

## 快速排错（最小命令）
```bash
# 1) 确认 env
echo "ROUTER_API_KEY=${ROUTER_API_KEY:0:6}******"

# 2) 直接走 router
curl -sS https://router.yeying.pub/v1/models \
  -H "Authorization: Bearer $ROUTER_API_KEY"

# 3) Codex
codex exec "只回答 ok"
```

