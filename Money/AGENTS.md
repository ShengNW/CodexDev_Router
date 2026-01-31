# Money handoff (billing/usage logic) - router_new

This file captures the current billing logic and runtime facts discovered so a new Codex session can continue immediately. It is read-only analysis; no code changes were made.

## Scope
- Repo: /root/code/router/router_new
- Goal: explain current billing/quota logic, where it is implemented, and how logs/usage map to quota.
- Online instance: https://llm.yeying.pub/ (public reverse proxy to local 3011).
- DB: PostgreSQL DSN provided by user (read-only queries already performed).

## Runtime DB snapshot (PG, read-only)
- Tables: abilities, channels, logs, options, redemptions, tokens, users
- options table has ONLY:
  - WalletAutoRegisterEnabled=true
  - WalletLoginEnabled=true
  => No billing-related overrides in DB (ModelRatio/GroupRatio/CompletionRatio/PreConsumedQuota/QuotaPerUnit/etc). So billing uses code defaults.
- channels table (id, name, type, group, model_mapping):
  - 1, yeying, 1, default, NULL
  - 2, deepseek, 36, default, NULL
  - 3, claude, 14, default, NULL
  - 4, Gemini, 24, default, NULL
  - 5, QinngYun, 1, default, NULL
- logs table exists with data (~121 rows from pg_stat_user_tables).

Channel type mapping (internal/relay/channeltype/define.go):
- 1 = openai
- 14 = anthropic
- 24 = gemini
- 36 = deepseek

## Core billing flow (text/chat/completions/embeddings/moderations)
Key files:
- internal/relay/controller/text.go
- internal/relay/controller/helper.go
- internal/relay/billing/ratio/model.go
- internal/relay/adaptor/openai/token.go
- internal/relay/adaptor/openai/main.go

High-level flow (RelayTextHelper):
1) Parse/validate request.
2) Count prompt tokens via openai.CountTokenMessages / CountTokenInput.
3) Compute ratios:
   - modelRatio = billingratio.GetModelRatio(model, channelType)
   - groupRatio = billingratio.GetGroupRatio(user.group)
   - completionRatio = billingratio.GetCompletionRatio(model, channelType)
   - total ratio = modelRatio * groupRatio
4) Pre-consume quota (helper.go preConsumeQuota):
   - preConsumedTokens = PreConsumedQuota + promptTokens (+ MaxTokens if provided)
   - preConsumedQuota = preConsumedTokens * ratio
   - if userQuota > 100 * preConsumedQuota: skip pre-consume (trusted)
   - otherwise deduct from token or user (PreConsumeTokenQuota / DecreaseUserQuota)
5) Send upstream request and parse usage.
6) Post-consume quota (helper.go postConsumeQuota):
   - quota = ceil((promptTokens + completionTokens * completionRatio) * ratio)
   - if ratio != 0 && quota <= 0, set quota=1
   - quotaDelta = quota - preConsumedQuota
   - adjust user/token quota by delta
   - update cache
7) Record consume log + update usage counters.

Important details:
- ApproximateTokenEnabled=true uses len(text)*0.38 in CountTokenText (default false).
- If upstream returns no usage, completion tokens are calculated from response text.
- completionRatio defaults by model family if not explicitly set in CompletionRatio map.

## Image billing
Key files:
- internal/relay/controller/image.go
- internal/relay/billing/ratio/image.go

Formula:
- quota = int64(modelRatio * groupRatio * imageCostRatio * 1000) * n
- imageCostRatio depends on size, and dalle-3 hd multiplies further.
- Logs: PromptTokens=0, CompletionTokens=0, Quota=computed.

## Audio billing
Key files:
- internal/relay/controller/audio.go
- internal/relay/billing/billing.go

Behavior:
- AudioSpeech (TTS): quota = len(input) * ratio (preConsumedQuota=quota)
- Transcription/translation: preConsumedQuota = PreConsumedQuota*ratio, then actual quota = CountTokenText(transcribed text)
  NOTE: actual quota does NOT multiply by ratio here (current code behavior).
- billing.PostConsumeQuota writes logs with PromptTokens = totalQuota, CompletionTokens=0.
  => Audio logs use PromptTokens field to store quota, not actual token counts.

## Logs, quota, and usage mapping
Key files:
- internal/admin/model/log.go
- internal/admin/repository/log/repository.go
- internal/admin/controller/billing/handler.go
- internal/admin/model/token.go
- internal/admin/repository/token/repository.go
- internal/admin/repository/user/repository.go

Logs table fields:
- quota: actual billed quota (core accounting)
- prompt_tokens/completion_tokens: token stats (text=real, image=0, audio=quota)

Usage endpoints:
- /dashboard/billing/usage and /dashboard/billing/subscription
- Use token.used_quota/remain_quota if DisplayTokenStatEnabled and token present.
- Otherwise use user.used_quota/quota.
- NOT derived from logs directly.

Sum stats:
- SumUsedQuota = sum(logs.quota) for LogTypeConsume
- SumUsedToken = sum(prompt_tokens + completion_tokens)

## Quota unit and money display
Key files:
- common/config/config.go
- common/utils.go
- internal/admin/controller/billing/handler.go

Defaults (no DB override detected):
- QuotaPerUnit = 500 * 1000  (i.e. $0.002 per 1K tokens maps to 1 quota)
- DisplayInCurrencyEnabled = true
- DisplayTokenStatEnabled = true

Conversion:
- Display amount = quota / QuotaPerUnit

## Ratios and overrides
Key files:
- internal/relay/billing/ratio/model.go
- internal/relay/billing/ratio/group.go
- internal/admin/model/option.go

Defaults:
- ModelRatio, GroupRatio, CompletionRatio are all in code.
- GroupRatio default map: default/vip/svip => 1

Overrides:
- options table keys ModelRatio/GroupRatio/CompletionRatio/PreConsumedQuota/QuotaPerUnit can override defaults.
- Current DB has none of those, so defaults apply.

## Token counting specifics
Key files:
- internal/relay/adaptor/openai/token.go

Notes:
- Uses tiktoken-go with model-specific encoders; defaults to gpt-3.5-turbo encoder.
- For chat messages, counts role/name tokens and supports image_url token costs for vision.

## Log DB
Key files:
- internal/admin/model/main.go

Notes:
- LOG_DB can be a separate DSN via LOG_SQL_DSN, else uses main DB.
- For PG, sequence setup for logs.id is handled in migrateLOGDB.

## Operational context
- Public domain: https://llm.yeying.pub/ (reverse proxy to local 3011).
- SESSION_SECRET set by user (do not change).

## Prior commands executed (read-only)
- PG read-only queries with default_transaction_read_only=on.
- No code changes, no services started.

## Next suggested checks (if needed, still read-only)
- Optional: sample a few logs rows (anonymize) to verify quota vs token fields.
- Optional: check if LOG_SQL_DSN is set in runtime environment.

