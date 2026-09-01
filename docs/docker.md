# Docker 部署教程

本教程介绍如何使用 Docker 部署 `douyin-auto-fire`。

项目提供两套公共镜像：

- 国内环境推荐：`docker.cnb.cool/1mev/douyin-auto-fire:latest`
- 海外环境推荐：`ghcr.io/unmev/douyin-auto-fire:latest`

两套镜像内容一致，只是镜像仓库不同。Python、Playwright、Chromium 和项目代码都已经封装进 Docker 镜像，**Docker 部署不需要下载完整源码仓库**。

## 1. 准备部署目录

```bash
mkdir -p ~/douyin-auto-fire/artifacts
cd ~/douyin-auto-fire
```

## 2. 下载 Compose 文件

### 国内服务器（推荐）

只从 CNB 下载一个 `docker-compose.yml`，不需要 `git clone` 整个仓库：

```bash
curl -fL https://cnb.cool/1mev/douyin-auto-fire/-/git/raw/main/docker-compose.yml -o docker-compose.yml
```

国内版默认使用：

```text
docker.cnb.cool/1mev/douyin-auto-fire:latest
```

因此国内部署时：

- Compose 文件从 CNB 下载；
- Docker 镜像从 CNB 拉取；
- 不需要访问 GitHub Raw；
- 不需要下载完整源码仓库。

### 海外服务器

海外环境只下载海外 Compose 文件：

```bash
curl -fL https://raw.githubusercontent.com/unmev/douyin-auto-fire/main/docker-compose.global.yml -o docker-compose.yml
```

海外版默认使用：

```text
ghcr.io/unmev/douyin-auto-fire:latest
```

## 3. 准备任务配置

创建 `config.json`：

```bash
nano config.json
```

也可以使用配置生成器生成内容：

https://douyin-config.pages.dev/

第一次建议只配置 1 个好友和 1 条文字消息，并先确认配置正确。

## 4. 准备 Cookie

在电脑浏览器登录抖音网页版，然后使用 Cookie-Editor 导出完整 JSON。

在部署目录创建：

```bash
nano cookie.json
```

将导出的完整 Cookie JSON 粘贴进去并保存。

> Cookie 相当于账号登录凭证，请勿上传到 GitHub、CNB 或公开分享。

## 5. 创建 `.env`

```bash
nano .env
```

推荐内容：

```env
TZ=Asia/Shanghai
CRON_SCHEDULE=30 0 * * *
RUN_ON_START=false
HEADLESS=true
DOUYIN_COOKIE=/data/cookie.json
TASK_CONFIG=/app/config.json
ARTIFACTS_DIR=/app/artifacts

DINGTALK_WEBHOOK=
DINGTALK_SECRET=
```

`CRON_SCHEDULE=30 0 * * *` 表示每天 00:30 执行。

例如每天 08:00：

```env
CRON_SCHEDULE=0 8 * * *
```

例如每天 20:00：

```env
CRON_SCHEDULE=0 20 * * *
```

如果希望容器每次启动时先立即执行一次任务：

```env
RUN_ON_START=true
```

正常长期运行建议保持：

```env
RUN_ON_START=false
```

## 6. 拉取镜像并启动

```bash
docker compose pull
docker compose up -d
```

查看容器状态：

```bash
docker compose ps
```

查看日志：

```bash
docker compose logs -f
```

国内版会拉取：

```text
docker.cnb.cool/1mev/douyin-auto-fire:latest
```

海外版会拉取：

```text
ghcr.io/unmev/douyin-auto-fire:latest
```

## 7. 手动执行一次任务

无需等待定时任务，可以直接在运行中的容器中执行：

```bash
docker exec -it douyin-auto-fire python run.py
```

Dry Run：

```bash
docker exec -it douyin-auto-fire python run.py --dry-run
```

第一次部署强烈建议先运行 Dry Run。

## 8. 宿主机目录

Docker 部署只需要这些文件：

```text
douyin-auto-fire/
├── docker-compose.yml
├── .env
├── config.json
├── cookie.json
└── artifacts/
```

其中：

- `docker-compose.yml`：容器配置；
- `config.json`：发送任务配置；
- `cookie.json`：抖音 Cookie；
- `.env`：时区、定时规则等；
- `artifacts/`：日志、截图、Trace、历史记录等运行产物。

修改 `config.json` 或 `cookie.json` 后，下次执行任务会直接读取最新内容。

修改 `.env` 后建议重新创建容器：

```bash
docker compose up -d
```

## 9. 更新

正常更新程序只需要拉取最新版镜像：

```bash
docker compose pull
docker compose up -d
```

不需要重新下载整个仓库。

如果以后 Compose 文件本身有更新，可以重新下载一次。

国内：

```bash
curl -fL https://cnb.cool/1mev/douyin-auto-fire/-/git/raw/main/docker-compose.yml -o docker-compose.yml
docker compose pull
docker compose up -d
```

海外：

```bash
curl -fL https://raw.githubusercontent.com/unmev/douyin-auto-fire/main/docker-compose.global.yml -o docker-compose.yml
docker compose pull
docker compose up -d
```

本地的 `config.json`、`cookie.json`、`.env` 和 `artifacts/` 不会因为更新镜像而丢失。

## 10. 常用 Docker 命令

```bash
# 启动
docker compose up -d

# 查看状态
docker compose ps

# 查看实时日志
docker compose logs -f

# 重启
docker compose restart

# 停止
docker compose stop

# 启动已经停止的容器
docker compose start

# 拉取最新版
docker compose pull

# 更新并启动
docker compose pull && docker compose up -d

# 停止并删除容器
docker compose down
```

也可以直接操作容器：

```bash
docker logs -f douyin-auto-fire
docker restart douyin-auto-fire
docker stop douyin-auto-fire
docker start douyin-auto-fire
```

## 11. 查看运行产物

程序写入宿主机：

```text
./artifacts/
```

可能包含：

```text
run.log
result.json
history.json
screenshots/
traces/
```

如果发送失败，优先查看：

```bash
cat artifacts/run.log
```

或者：

```bash
docker compose logs --tail=200
```

## 12. 国内 / 海外镜像说明

国内镜像：

```text
docker.cnb.cool/1mev/douyin-auto-fire:latest
```

海外镜像：

```text
ghcr.io/unmev/douyin-auto-fire:latest
```

GitHub Actions 会在 `main` 分支相关代码更新时自动构建，并同时推送到 GHCR 和 CNB。CNB 代码仓库作为国内镜像源的辅助仓库，用于提供 `docker-compose.yml` 等小文件的国内下载地址；Docker 用户无需克隆完整源码。

如需锁定某次构建，也可以使用对应的 SHA 标签，而不是一直使用 `latest`。

## 注意事项

- Cookie 不要写进 Docker 镜像，也不要提交到公开仓库。
- 同一个抖音账号不要同时运行多个实例，避免重复发送。
- 服务器网络环境可能触发抖音安全验证。
- Cookie 失效后需要重新导出并替换本地 `cookie.json`。
- `artifacts/` 中的日志、截图和 Trace 可能包含聊天相关信息，请勿随意公开。
