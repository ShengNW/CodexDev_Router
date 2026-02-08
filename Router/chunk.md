# Router /v1/responses 503 诊断与修复（chunked vs Content-Length）

目的：记录一次真实排障过程，明确“503 与流式无关，根因是 chunked 传输被上游拒绝”，并给出最小修复方案，避免重复踩坑。

## 关键结论（已复现）
- 上游对 **Transfer-Encoding: chunked** 的 `/v1/responses` 请求会返回 503。
- **流式本身没问题**；只要使用 Content-Length（非 chunked），`stream=true` 也能 200。
- Router 直接 `return c.Request.Body` 时，Go 会使用 chunked；改为 `bytes.NewBuffer(rawBody)` 会自动带 Content-Length。

## 关键证据链
1) Router 侧抓到 raw body，回放到上游：
   - Content-Length 方式 → 200
   - chunked 方式 → 503
2) `stream=true` 且 Content-Length → 200
3) 同一 key / UA / body，仅改变传输方式即可复现 200 vs 503

## 最小修复方案（已生效）
位置：`/root/code/router/router_new/internal/relay/controller/text.go`

在 `getRequestBody(...)` 的 Responses 分支中：
- 读取 raw body
- 返回 `bytes.NewBuffer(rawBody)`（而不是 `c.Request.Body`）
这样保证请求走 Content-Length，不再 chunked。

## 排查思路（给后续同事）
1) 在 `/v1/responses` 打日志或 dump raw body（只用于排障）
2) 用 `curl --data-binary @dump.json` 回放上游，确认上游可用
3) 若回放 200、Router 503，优先检查传输方式是否 chunked

## 额外注意
- 如果上游报 `Client not allowed`，请看 `cch.md`（UA allowlist）
- 如果上游报 “encrypted content could not be verified”，请检查上游侧密钥/验签链路
