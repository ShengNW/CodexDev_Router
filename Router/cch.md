# CCH Codex CLI 连接验证记录

目的：记录如何用 Codex CLI 走 CCH 上游（`https://claude.hanbbq.top/v1`）并解释 `Client not allowed` 的根因与解决方式。

## 关键结论（已实测）
- CCH 服务器开启了 **客户端白名单（User-Agent）** 限制。
- 如果没有匹配的 `User-Agent`，会返回：`Client not allowed. Your client is not in the allowed list.`
- 带上 `User-Agent: codex-cli` 或 `User-Agent: codex-cli/0.80.0` 即可通过。

该逻辑来自上游实现（claude-code-hub）：
- 客户端限制逻辑：`/tmp/claude-code-hub/src/app/v1/_lib/proxy/client-guard.ts`
- 允许客户端列表里包含 `codex-cli`（UI preset）：`/tmp/claude-code-hub/src/app/[locale]/dashboard/_components/user/forms/access-restrictions-section.tsx`

## 快速 curl 验证
以下命令不修改任何配置，只验证链路是否可通：

```bash
curl -sS https://claude.hanbbq.top/v1/responses \
  -H "Authorization: Bearer $CCH_API_KEY" \
  -H "Content-Type: application/json" \
  -H "User-Agent: codex-cli" \
  -d '{"model":"gpt-5.2-codex","input":[{"role":"user","content":"ping"}]}'
```

预期：返回 `response` 对象（内容可为 `pong`）。

## CLI 临时验证（不改 config.toml）
避免改坏本机配置，可用命令行临时覆盖参数：

```bash
codex exec \
  -c 'model_provider="cch"' \
  -c 'model="gpt-5.2-codex"' \
  -c 'model_providers.cch.base_url="https://claude.hanbbq.top/v1"' \
  -c 'model_providers.cch.wire_api="responses"' \
  -c 'model_providers.cch.env_key="CCH_API_KEY"' \
  -c 'model_providers.cch.http_headers={"User-Agent"="codex-cli"}' \
  "只回答 ok"
```

预期：输出 `ok`。

## 常见报错与处理

### 1) `Client not allowed. Your client is not in the allowed list.`
原因：User-Agent 未匹配 allowlist。  
处理：带 `User-Agent: codex-cli`（无论 curl 还是 CLI）。

### 2) `authentication_error` / “未提供认证凭据”
原因：未传 key 或走了错误认证路径。  
处理：确保 `$CCH_API_KEY` 已导出；CLI 侧不要同时启用 `requires_openai_auth = true` 和 `env_key`。

## 注意
- 不要把真实 key 写入仓库文件。
- `/status` 只显示模型名，不显示 provider/base_url；判断是否走 CCH 以实测为准。

