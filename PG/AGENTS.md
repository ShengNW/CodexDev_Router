# PG 探查记录

- 记录时间：2026-01-20T19:25:10+00:00
- 探查范围：只读（未做写操作）

## 1) SQL_DSN / 运行目录 / 日志确认
- SQL_DSN：存在，已脱敏：`postgres://router:***@51.75.133.235:5432/router?sslmode=disable`
- systemd WorkingDirectory：`/root/code/router/router_new`（与仓库根一致，`.env` 可自动加载）
- 启动日志：`openPostgreSQL`（确认当前使用 PostgreSQL）

## 2) PG 连接与版本
- 连接信息：db=router, user=router, host=51.75.133.235, port=5432
- 版本：PostgreSQL 16.11 (Alpine gcc 15.2.0, 64-bit)

## 3) 关键表结构核对
- `tokens.unlimited_quota`：`bigint`（非 boolean）
- `redemptions`：结构正常可读

## 4) 风险与注意点
- 结构一致性风险：当前 `tokens.unlimited_quota` 为 `bigint`；若未来代码/迁移期望 boolean，需要显式迁移或兼容处理。
- 连接注意点：SSL 为 `disable`，且 PG 为远程主机；网络/权限问题会直接影响启动与连接。


## 5) 修复记录（tokens.unlimited_quota 类型修正）
- 记录时间：2026-01-20T19:38:06Z
- 操作范围：结构写入（必要变更）
- SQL_DSN（脱敏）：`postgres://router:***@51.75.133.235:5432/router?sslmode=disable`

### 执行 SQL
- 预处理（为避免默认值类型转换失败）：
  `ALTER TABLE tokens ALTER COLUMN unlimited_quota DROP DEFAULT;`
- 迁移（按指定 SQL）：
  `ALTER TABLE tokens ALTER COLUMN unlimited_quota TYPE boolean USING (unlimited_quota <> 0);`
  `ALTER TABLE tokens ALTER COLUMN unlimited_quota SET DEFAULT false;`

### 结构变更
- 变更前：`tokens.unlimited_quota` 为 `bigint`，默认 `'0'::bigint`
- 变更后：`tokens.unlimited_quota` 为 `boolean`，默认 `false`

### 结果
- `\d+ tokens` 已显示 `unlimited_quota | boolean | default false`

## 6) 修复记录（tokens.id 默认序列）
- 记录时间：2026-01-20T20:00:39Z
- 操作范围：结构写入（必要变更）
- SQL_DSN（脱敏）：`postgres://router:***@51.75.133.235:5432/router?sslmode=disable`

### 结构检查
- 变更前：`tokens.id` 无默认值（`\d+ tokens` 默认列为空）

### 执行 SQL（事务）
```
BEGIN;
CREATE SEQUENCE IF NOT EXISTS tokens_id_seq OWNED BY tokens.id;
ALTER TABLE tokens
  ALTER COLUMN id SET DEFAULT nextval('tokens_id_seq');
SELECT setval('tokens_id_seq',
              COALESCE((SELECT MAX(id) FROM tokens), 0),
              true);
COMMIT;
```

### 结构变更
- 变更后：`tokens.id` 默认值为 `nextval('tokens_id_seq'::regclass)`

## 7) 修复记录（channels.id 默认序列）
- 记录时间：2026-01-20T20:14:26Z
- 操作范围：结构写入（必要变更）
- SQL_DSN（脱敏）：`postgres://router:***@51.75.133.235:5432/router?sslmode=disable`

### 结构检查
- 变更前：`channels.id` 无默认值（`\d+ channels` 默认列为空）

### 执行 SQL（事务）
```
BEGIN;
CREATE SEQUENCE IF NOT EXISTS channels_id_seq OWNED BY channels.id;
ALTER TABLE channels
  ALTER COLUMN id SET DEFAULT nextval('channels_id_seq');
SELECT setval('channels_id_seq',
              COALESCE((SELECT MAX(id) FROM channels), 0),
              true);
COMMIT;
```

### 结构变更
- 变更后：`channels.id` 默认值为 `nextval('channels_id_seq'::regclass)`

### 可选排查（同模板补齐）
- tokens：已有 `tokens_id_seq`（默认存在）
- users：发现缺默认值，已补齐 `users_id_seq` 并设置默认 nextval
- options：无 id 列（不适用）
- logs：已有 `logs_id_seq`（默认存在）
- redemptions：已有 `redemptions_id_seq`（默认存在）

### users 修复 SQL（事务）
```
BEGIN;
CREATE SEQUENCE IF NOT EXISTS users_id_seq OWNED BY users.id;
ALTER TABLE users
  ALTER COLUMN id SET DEFAULT nextval('users_id_seq');
SELECT setval('users_id_seq',
              COALESCE((SELECT MAX(id) FROM users), 0),
              true);
COMMIT;
```
