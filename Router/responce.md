# Responses 接口与测试调通记录（Codex/CCH）

目的：记录本次把 CCH 上游 `/v1/responses` 调通，并让 Router 的「测试通道」功能正确通过的关键经验，方便后续 Codex 快速复盘与复用。

## 指挥官视角：全局结论（可直接照做）
- 上游可用 ≠ Router 测试必然通过：测试失败多半是 **请求/解析不匹配**，而不是上游挂了。
- CCH 的 Codex 走的是 **/v1/responses**，请求体用 `input`，不是 `messages`。
- `responses` 的返回结果并不只有 `output_text`，也可能是 `content[].text`，**测试解析器必须兼容两种字段**。
- 只要三件事齐了，就能稳定通过：
  1) 路由命中 `/v1/responses`
  2) 请求体为 `input`
  3) 解析器支持 `text` / `output_text` 双格式

## 教学视角：机制理解（为什么会错）
- Router 之前的测试逻辑是按 **ChatCompletions** 的习惯去解析，默认找 `choices` 或 `output_text`。
- CCH 的 Codex Responses 响应里常见结构：
  - `output[].content[].text`（有时 `type` 是 `output_text`）
  - **并不保证有 `output_text` 字段**
- 结果：上游明明返回了内容，测试却报：
  - `response has no output text`

## 关键改动点（仅测试解析器）
> 改动原则：**只改测试解析**，不影响旧逻辑。

- 解析 `output[].content[]` 时：
  - 先取 `content.text`
  - 若为空再取 `content.output_text`
  - 找到第一个非空即视为成功
- 若完全找不到文本：
  - 返回错误，并附带 `content.type` 列表，便于定位

## 最小验证路径（不改配置文件）
1) 用 UI 或测试按钮触发通道测试。
2) 测试参数要求：
   - `use_responses=true`
   - 路由路径 `/v1/responses`
   - 请求体 `input`
3) 预期：测试返回成功文本，不再出现 `response has no output text`。

## 常见误区与快速判断
- **误区 1**：认为“上游通=测试通”。
  - 事实：请求/解析不匹配，测试会失败。
- **误区 2**：`responses` 还能用 `messages`。
  - 事实：CCH 的 `/v1/responses` 在 `messages` 下会返回 `No available providers`。
- **误区 3**：只支持 `output_text`。
  - 事实：`content[].text` 是常见字段，必须兼容。

## 相关代码位置（便于复查）
- 测试逻辑与解析器：
  - `/root/code/router/router_new/internal/admin/controller/channel/test.go`
- 路由与模式切换（已有支持）：
  - `/root/code/router/router_new/internal/transport/http/router/relay.go`
  - `/root/code/router/router_new/internal/transport/http/router/api.go`
  - `/root/code/router/router_new/internal/relay/meta/relay_meta.go`

## 复盘一句话
**本次测试失败不是上游故障，而是 Responses 响应解析器只认 `output_text`，未兼容 `content.text`。**

