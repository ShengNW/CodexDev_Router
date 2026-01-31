# local 启动记录（Router）

## 适用场景
- 在本地仓库 `/home/snw/Codex/Router/Router_new` 启动后端，并使用最新前端构建产物。
- 访问地址为 `http://localhost:3011`。

## 启动步骤（生产态前端）
在 Router 根目录执行：
```bash
cd /home/snw/Codex/Router/Router_new
npm install --prefix web
npm run build --prefix web

mkdir -p logs
nohup go run ./cmd/router --log-dir ./logs > logs/router-dev.log 2>&1 &
```

## 验证与访问
```bash
ss -ltnp | rg ':3011\b'
```
浏览器访问：`http://localhost:3011`

## 常见问题
- **SQLite 迁移报错（duplicate column name: model_ratio）**
  - 说明本地 `one-api.db` 结构旧或不一致。
  - 处理：备份后重建
  ```bash
  cd /home/snw/Codex/Router/Router_new
  ts=$(date +%Y%m%d-%H%M%S)
  mv one-api.db one-api.db.bak-$ts
  ```
  - 再次启动后端即可生成新库。

## 停止进程
```bash
ss -ltnp | rg ':3011\b'
# 找到 pid 后 kill
kill <pid>
```

## 默认账户
- 管理员：`root / 123456`
