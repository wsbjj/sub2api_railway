# Railway 部署指南

本分支已包含 Railway 配置：使用仓库根目录的 `Dockerfile` 构建，通过 `/health` 检查服务状态，并自动将 Railway 注入的 `PORT` 映射到 Sub2API 的 `SERVER_PORT`。

## 1. 创建服务

1. 在 Railway 新建项目，选择 **Deploy from GitHub repo**。
2. 选择仓库 `wsbjj/sub2api_railway`，部署分支选择 `codex/railway-deploy`。
3. 在项目中添加 **PostgreSQL** 和 **Redis** 服务。
4. 给 Sub2API 服务创建一个 Volume，挂载路径设置为 `/app/data`。

Volume 用于保存首次启动生成的 `config.yaml` 等运行数据。没有 Volume 时，重新部署可能导致登录会话或其他本地配置丢失。

## 2. 配置变量

在 Sub2API 服务的 **Variables** 中添加以下变量。`Postgres` 和 `Redis` 是 Railway 默认服务名；如果你修改过服务名，请同步修改引用前缀。

```dotenv
AUTO_SETUP=true
SERVER_HOST=0.0.0.0
SERVER_MODE=release
DATA_DIR=/app/data
TZ=Asia/Shanghai

DATABASE_HOST=${{Postgres.PGHOST}}
DATABASE_PORT=${{Postgres.PGPORT}}
DATABASE_USER=${{Postgres.PGUSER}}
DATABASE_PASSWORD=${{Postgres.PGPASSWORD}}
DATABASE_DBNAME=${{Postgres.PGDATABASE}}
DATABASE_SSLMODE=disable

REDIS_HOST=${{Redis.REDISHOST}}
REDIS_PORT=${{Redis.REDISPORT}}
REDIS_PASSWORD=${{Redis.REDISPASSWORD}}
REDIS_DB=0
REDIS_ENABLE_TLS=false

ADMIN_EMAIL=你的管理员邮箱
ADMIN_PASSWORD=请设置强密码
JWT_SECRET=请填写至少32字节的随机密钥
TOTP_ENCRYPTION_KEY=请填写32字节的随机密钥
```

不要手动设置 `PORT` 或 `SERVER_PORT`。Railway 会注入 `PORT`，本分支的入口脚本会自动适配。

可使用以下命令在本机生成两个独立密钥：

```powershell
1..2 | ForEach-Object { -join ((1..64) | ForEach-Object { '{0:x}' -f (Get-Random -Maximum 16) }) }
```

## 3. 验证部署

首次启动会自动连接 PostgreSQL 和 Redis、执行数据库迁移并创建管理员。部署状态变为 **Active** 后检查：

```text
https://你的域名/health
```

正常响应：

```json
{"status":"ok"}
```

随后访问服务首页，用 `ADMIN_EMAIL` 和 `ADMIN_PASSWORD` 登录。若部署未通过健康检查，优先查看 Deploy Logs 中的数据库或 Redis 连接错误，并核对变量引用的服务名。
