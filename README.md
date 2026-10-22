# Mook for LazyCat

将 [Mook](https://github.com/NikoYomi/mook) 打包为 LazyCat LPK v2。Mook 是一个支持 AI 的自托管服务器运维工作台：把 Web SSH 终端、服务器监控、SFTP 文件管理和 AI 辅助融为一体，部署后一个网页入口集中管理所有 VPS。

## 镜像来源

上游在打 `v*` tag 时由 GitHub Actions 自动构建并推送镜像（GHCR 与 Docker Hub 双推，amd64/arm64 双平台），因此直接使用官方镜像，不需要重新构建：

- 上游镜像：`nikoyomi/mook:0.4.6`（Docker Hub；GHCR `ghcr.io/nikoyomi/mook` 为主要发布地）
- 运行镜像：`docker.1ms.run/nikoyomi/mook:0.4.6`（Docker Hub 默认加速）
- 单容器，监听容器内 `5866`

本仓库跟踪上游的三段式版本 tag（`0.4.6` 这类），发布前校验 `linux/amd64` 源镜像与加速地址 digest 一致。

## 部署参数与安全

| 参数 | 说明 |
| --- | --- |
| `admin_password` | 控制台登录密码，至少 8 位，默认随机生成；以 bcrypt 哈希存储 |

- SSH 密码、私钥与 AI Key 的加密密钥使用 `{{ stable_secret "credentials_key" }}`，随应用持久化保存。
- 密码登录失败 5 次锁定 15 分钟，会话有效期 6 小时。
- 容器 `cap_drop: ALL` + `no-new-privileges`，仅监听容器内 `5866`，外部访问建议叠加 HTTPS 反向代理。
- 上游当前版本暂不校验 SSH 主机指纹（known_hosts）。

## 数据与运行方式

- 持久化：`/lzcapp/var/mook` → `/data`（SQLite 数据库、加密后的凭据、AI 配置）
- 服务名：`mook`

## 本地构建

```bash
lzc-cli project build -o dist/application.lpk
lzc-cli lpk info dist/application.lpk
```

## 自动发布

`.github/workflows/lazycat.yml` 使用 `ca-x/lazycat-github-action` 跟踪上游版本 tag，创建版本化 GitHub Release Asset 并只发布到喵喵私有商店。需要为仓库配置 `APPSTORE_URL`、`APPSTORE_TOKEN`，以及可选的 `APP_ID`、`PRIVATE_STORE_GROUP_CODES`。

## 许可

MIT（上游）