# Qingyun Top 渠道探查记录（Key 模型可用性）

更新时间：2026-01-22T10:40:54+00:00

## 结论（给实现同学）
- 该 key 可通过 `GET /v1/models` 拉取完整模型清单（OpenAI 风格 `data[]/id`）。
- 鉴权方式：`Authorization: Bearer <key>`；实测不需要额外 header。
- 缺少 token / 无效 token 都会返回 401。
- `/models`（无 `/v1`）返回的是前端页面 HTML；接口应使用 `/v1`。

## 探查说明（教师视角）
- 目标：验证“渠道是否支持用 key 拉模型列表”，并记录返回结构与失败行为，便于后续自动拉模型适配。
- 平台模型可能随时更新；**以 `/v1/models` 为实时权威**，价格页/教程页仅作参考入口。

## 可用接口（实测）
- **模型列表**
  - URL：`https://api.qingyuntop.top/v1/models`
  - Method：`GET`
  - Headers：
    - `Authorization: Bearer <key>`
    - `Content-Type: application/json`（可省略）
  - 返回结构：
    - 顶层：`{ data: [...], message: "", success: true }`
    - 模型对象：`{ id, object, created, owned_by, supported_endpoint_types }`
  - 兼容性提示：`supported_endpoint_types` / `message` / `success` 为非 OpenAI 标准字段，适配时注意。

- **价格页（JS 页面，仅参考）**
  - `https://api.qingyuntop.top/pricing`

- **教程页（JS 页面，仅参考）**
  - `https://api.qingyuntop.top/about`

## 示例响应（截断）
```json
{
  "data": [
    {
      "id": "o1",
      "object": "model",
      "created": 1626777600,
      "owned_by": "openai",
      "supported_endpoint_types": ["openai", "openai-response"]
    },
    {
      "id": "qwen3-235b-a22b",
      "object": "model",
      "created": 1626777600,
      "owned_by": "ali",
      "supported_endpoint_types": ["openai"]
    }
  ],
  "message": "",
  "success": true
}
```

## 失败行为（实测）
- 未带 token：HTTP 401 + `Token not provided`
- 无效 token：HTTP 401 + `Invalid token`

## 模型 ID 清单（快照）
- 拉取时间：2026-01-22T10:40:54+00:00
- 数量：485
- 说明：这是用该 key 拉取的快照；后续如需更新，重新调用 `/v1/models` 即可。

```
ERNIE-3.5-8K
flux-pro
gpt-5-pro-2025-10-06
fal-ai/flux-pro/kontext/max
gpt-4-gizmo-*
kling-advanced-lip-sync
kling-effects
llama-3.1-405b-instruct
gemini-2.5-flash-lite-preview-06-17
glm-4.6
gpt-4o-2024-05-13
kling-image
mj_imagine
qwen3-30b-a3b-instruct-2507
ERNIE-4.0-8K
gpt-4.1-2025-04-14
gpt-4o-audio-preview
gpt-5.1-codex-mini
o1-preview
veo3.1-pro
fal-ai/bytedance/seedream/v4/text-to-image
deepseek-r1-250120
glm-4.5-x
gpt-5-chat
qwen2-vl-7b-instruct
veo3.1-pro-4k
qwen-vl-max-2025-08-13
qwen3-0.6b
qwen3-tts-flash
z-image-turbo
o1-preview-2024-09-12
claude-opus-4-5-20251101
gemma-2b-it
o1-pro-all
o3-pro-all
veo2-fast-frames
chatgpt-4o-latest
aigc-video-vidu
mj_blend
emini-3-pro-preview-11-2025
gpt-4o-mini-audio-preview
gpt-5-thinking-all
gpt-oss-20b
veo2-pro-components
longcat-flash-thinking
gemini-3-flash-preview
glm-4-flash
o1
o4-mini
qwen3-30b-a3b
gemini-2.5-flash-lite-preview-06-17-thinking
fal-ai/recraft/v3/text-to-image
kimi-k2-0711-preview
gpt-4-turbo-2024-04-09
claude-3-haiku-20240307
gpt-5.1-chat-latest
kling-video-extend
gpt-4-32K-0613
text-embedding-v1
kling-custom-voices
o3-pro-2025-06-10
gpt-4-1106-preview
gpt-4.1-mini
Dolphin3.0-R1-Mistral-24B
kling-motion-control
fal-ai/flux-1/schnell
gpt-oss-120b
qwen3-max-preview-n
deepseek-ocr
gpt-4-0613
gpt-4o-2024-11-20
fal-ai/qwen-image-edit
veo_3_1-fast-components-4K
doubao-seedance-1-5-pro-251215
gpt-4o-mini-tts
text-embedding-3-large
veo3.1-fast-components
gpt-4o-realtime-preview-2024-10-01
MiniMax-M2.1
SparkDesk-v2.1
gpt-4-32k-0613
gpt-5.1-2025-11-13
MiniMax-Hailuo-2.3-Fast
deepseek-v3
gpt-4o-mini-transcribe-2025-12-15
qwen2.5-vl-7b-instruct
veo_3_1
fal-ai/flux-1/dev
doubao-seedream-4-0-250828
gpt-4o-mini-audio-preview-2024-12-17
veo2-fast
gpt-4o-search-preview
deepseek-r1-0528
mistral-large-latest
qwen3-vl-8b-thinking
veo_3_1-fast-4K
veo2-pro
veo3-fast
davinci-002
gpt-4.1-mini-2025-04-14
gpt-5-nano-2025-08-07
doubao-seedance-1-0-pro-fast-251015
flux.1-kontext-dev
o3-mini-2025-01-31
qwq-plus
gpt-image-1-mini
claude-opus-4-20250514
deepseek-v3.1-fast
glm-4-air
o1-mini
gemini-pro-latest
doubao-seedance-1-0-pro-250528
BAAI/bge-reranker-v2-m3
deepseek-r1-distill-llama-70b
gpt-4o-mini-realtime-preview
llama-3.1-405b
gemini-2.5-pro-thinking-*
claude-3-5-sonnet-20241022
gpt-image-1-all
gpt-4o-transcribe
grok-4
kling-omni-image
mj_upscale
deepseek-r1-searching
qwen3-235b-a22b-instruct-2507
qwen3-coder-30b-a3b-instruct
gemini-2.5-flash-image-preview
claude-3-7-sonnet-20250219
gpt-4.5-preview
grok-4-fast-reasoning
qwen-plus-latest
fal-ai/flux-pro/kontext/max/text-to-image
gpt-5.1
qwen-vl-max-latest
ERNIE-Character-8K
mistral-small-latest
mj_modal
o3-mini-all
qwen3-tts-flash-2025-11-27
gemini-2.5-pro
flux-pro-max
gpt-4-turbo-preview
gpt-5
gpt-5-all
kimi-k2-250905
o3-pro
gemini-2.0-flash
glm-4.7
sora_image
text-embedding-ada-002
gemini-2.5-pro-nothinking
gemini-3-pro-preview
qwen-vl-max
gpt-4
gpt-5.2-chat
mj_pan
mj_video
gpt-5.1-codex
codex-mini-latest
kling-avatar-image2video
qwen3-30b-a3b-thinking-2507
gemini-2.5-flash
gpt-4o-image-vip
claude-3-5-sonnet-20240620
flux-2-flex
gpt-4o-audio-preview-2024-12-17
doubao-seedream-4-5-251128
flux-kontext-pro
mj_low_variation
qwq-plus-2025-03-05
veo3-pro
gemini-flash-lite-latest
fal-ai/flux-pro/kontext/text-to-image
llama-3-sonar-large-32k-chat
gemini-2.5-flash-preview-09-2025
fal-ai/recraft/vectorize
gpt-image-1.5-all
kling-video
o3-all
glm-4.5-air
mj_edits
gpt-4.1-nano-2025-04-14
Kimi-K2-Instruct
flux-kontext-max
gpt-4o-mini-transcribe
gpt-5.2-chat-2025-12-11
kimi-k2-0905
gpt-5.1-all
claude-3-opus-20240229
deepseek-reasoner
gpt-5-mini
qvq-max
qwen2.5-72b-instruct
gemini-2.0-flash-lite-001
claude-haiku-4-5-20251001-thinking
claude-sonnet-4-5-20250929
qwen3-vl-235b-a22b-thinking
ERNIE-Functions-8K
gpt-4o-2024-08-06
llama-2-13b
glm-4.5-airx
gpt-5-nano
o1-2024-12-17
o4-mini-all
deepseek-v3.2-speciale
gpt-4o-realtime-preview
qwen3-vl-plus
tts-1-hd-1106
o1-pro
deepseek-v3.2-exp-thinking
qwen-max
fal-ai/nano-banana
deepseek-r1-distill-qwen-32b
flux-pro-1.1-ultra
grok-3-reasoning
codex-mini-2025-05-16
deepseek-v3-0324
glm-4.6v
kling-custom-elements
veo3.1-components
fal-ai/qwen-image
gpt-4o-mini
gemini-2.5-flash-image
gpt-5.2
grok-3-mini
kimi-k2
wan2.5-i2v-preview
gemini-2.5-flash-lite-preview-09-2025
deepseek-v3.2-fast
grok-3-reasoner
llama-3.1-70b-instruct
Qwen/Qwen3-Reranker-0.6B
qwen3-coder
gemini-2.5-pro-thinking
aigc-image-qwen
kling-audio
gpt-4o-audio-preview-2025-06-03
Embedding-V1
o3-mini
claude-3-7-sonnet-20250219-thinking
gemini-2.5-pro-all
gemini-2.0-flash-lite
gpt-5-codex-2025-09-15
wen-max-2025-01-25
gpt-4o-realtime-preview-2024-12-17
mj_reroll
doubao-seedance-1-0-lite-t2v-250428
deepseek-v3.2-thinking
grok-3-image
qwen2.5-32b-instruct
tts-1-hd
claude-sonnet-4-20250514
claude-sonnet-4-20250514-thinking
qwen3-4b
qwen3-coder-flash
claude-3-sonnet-all
qwen3-rerank
qwen3-vl-8b-instruct
runwayml-gen3a_turbo-5
babbage-002
o3-mini-high-all
qwen-plus
qwen3-14b
tts-1-1106
aigc-video-hailuo
gpt-4-32k
kling-image-recognize
qwen-turbo
qwq-72b-preview
gemini-pro-latest-thinking-*
grok-3-deepsearch
grok-video-3
deepseek-r1-distill-qwen-7b
gemma-7b-it
veo3.1-components-4k
deepseek-math-v2
deepseek-r1-2025-01-20
gpt-5-2025-08-07
sora-2
minimax-m2
ERNIE-Tiny-8K
gpt-4o-realtime-preview-2025-06-03
qwen2.5-7b-instruct
SparkDesk-v3.1
gemini-3-pro-preview-11-2025
gpt-5-chat-2025-08-07
Pro/BAAI/bge-reranker-v2-m3
qvq-max-latest
qwen3-coder-480b-a35b-instruct
glm-4.5-flash
gpt-4-all
gpt-4-vision-preview
MAI-DS-R1
veo3
gemini-2.5-flash-lite-thinking
ERNIE-Speed-8K
gpt-4-0125-preview
qwen3-max-preview
suno_music
gemini-3-pro-preview-thinking
gpt-5-codex-medium
veo3.1
SparkDesk-v1.1
gpt-4o-mini-realtime-preview-2024-12-17
meta-llama/llama-4-scout
gpt-4o-search-preview-2025-03-11
gpt-4o-mini-search-preview
deepseek-v3-1-think-250821
llama-3-sonar-small-32k-chat
gemini-2.5-flash-all
gemini-2.5-pro-preview-tts
aigc-video-kling
deepseek-v3-fast
glm-4-airx
MiniMax-Hailuo-02
o1-mini-all
gemini-2.5-flash-preview-09-2025-thinking-*
deepseek-r1-250528
ERNIE-Speed-128K
gpt-5.1-codex-mini-2025-11-13
tts-1
gemini-2.5-flash-lite-nothinking
glm-4.6-thinking
gpt-4.1-nano
gpt-5.1-thinking-all
runwayml-gen4_turbo-5
sora-2-all
whisper-1
gpt-5.2-chat-latest
kimi-k2-thinking
veo_3_1-components-4K
flux.1-kontext-pro
gpt-5.2-2025-12-11
veo3-fast-frames
claude-opus-4-1-20250805-thinking
minimax-m2.1
gpt-4o-mini-search-preview-2025-03-11
gpt-3.5-turbo-1106
gpt-4o-mini-tts-2025-12-15
gpt-audio-2025-08-28
o1-preview-all
gpt-4.5-preview-2025-02-27
qwen3-32b
veo2-fast-components
gemini-3-pro-image-preview
dall-e-3
deepseek-v3-250324
deepseek-v3.1-thinking
glm-4
claude-sonnet-4-5-20250929-thinking
deepseek-chat
deepseek-v3-1-250821
flux-schnell
MiniMax-Hailuo-2.3
qwen3-next-80b-a3b-thinking
deepseek-v3.1
suno_lyrics
sora-2-pro-all
fal-ai/flux-pro/kontext
gpt-5.1-chat
claude-opus-4-5-20251101-thinking
fal-ai/flux-lora
deepseek-v3-1
glm-3-turbo
kling-omni-video
mj_high_variation
qwen3-coder-plus
suno_uploads
veo2
gpt-4o-audio-preview-2024-10-01
gpt-5-codex-low
text-curie-001
deepseek-v3-search
qwen3-vl-235b-a22b-instruct
gemini-2.5-flash-lite-preview-06-17-nothinking
grok-4-fast
o4-mini-2025-04-16
qwen3-next-80b-a3b-instruct
sora-2-characters
gemini-2.5-flash-preview-tts
fal-ai/flux-1/dev/image-to-image
gpt-realtime-2025-08-28
grok-4.1-fast
o1-mini-2024-09-12
qwen-plus-character
qwen3-vl-30b-a3b-instruct
MiniMax-M2
mj_zoom
doubao-seedance-1-0-lite-i2v-250428
deepseek-v3.2-exp
mj_custom_zoom
qwen-turbo-2025-07-15
qwen3-1.7b
fal-ai/flux-1/dev/redux
fal-ai/flux-1/schnell/redux
fal-ai/nano-banana/edit
grok-4-image
veo3-frames
qwen3-max
grok-3
longcat-flash-chat
o3-2025-04-16
qwen-image-max
qwen3-235b-a22b
gemini-2.5-flash-lite
gemini-2.5-flash-nothinking
fal-ai/flux-pro/kontext/max/multi
qwen-image-max-2025-12-30
qwen-plus-2025-12-01
qwen3-235b-a22b-thinking-2507
deepseek-r1
gpt-4.1
gpt-5-pro-all
qwen2-vl-72b-instruct
qwen3-30b-a3b-think
veo3-pro-frames
glm-4-long
glm-4.7-thinking
grok-4-deepsearch
kling-multi-elements
veo3.1-4k
gpt-4o
gpt-5.1-codex-max
mj_inpaint
veo_3_1-fast
gpt-image-1.5
text-babbage-001
meta-llama/llama-4-maverick
qwen2.5-vl-72b-instruct
veo_3_1-4K
gemini-2.5-flash-thinking-*
claude-haiku-4-5-20251001
glm-4.5
grok-4.1
qwen3-8b
fal-ai/bytedance/seedream/v4/edit
flux-2-pro
gpt-5.1-chat-2025-11-13
llama-2-70b
mj_variation
runwayml-gen4_turbo-10
SparkDesk-v3.5
deepseek-v3.2
gpt-5.2-all
mj_upload
o3
claude-opus-4-1-20250805
gemini-2.5-flash-thinking
ERNIE-Lite-8K
qwen-flash
qwen3-vl-flash
o1-pro-2025-03-19
flux.1.1-pro
gpt-5-codex
gpt-5-mini-2025-08-07
gpt-image-1
qwen3-vl-32b-thinking
netease-youdao/bce-reranker-base_v1
qwq-32b
text-embedding-3-small
claude-3-5-haiku-20241022
claude-opus-4-20250514-thinking
gpt-4o-all
gpt-5-pro
grok-4-fast-non-reasoning
veo3.1-fast
flux-dev
gpt-4-turbo
gpt-5-codex-high
qwen3-vl-32b-instruct
runwayml-gen3a_turbo-10
gemini-flash-latest
flux.1-dev
gpt-5-chat-latest
mj_describe
qwen2.5-vl-32b-instruct
qwen3-vl-30b-a3b-thinking
seed-oss-36b-instruct
veo_3_1-components
fal-ai/flux-pro/kontext/multi
aigc-image-gem
gpt-4o-mini-2024-07-18
o1-all
qwen-max-latest
wan2.6-i2v
```

## 复现方式（不放明文 key）
```bash
curl -sS -H "Authorization: Bearer <YOUR_KEY>"   https://api.qingyuntop.top/v1/models
```

## 安全说明
- 本文件 **不存放明文 key**；如需使用，请从安全位置读取并注入调用。
