
# Microsoft-Rewards-Script 镜像

---

<div align="center">
  <img src="https://img.shields.io/github/last-commit/daitcl/Microsoft-Rewards-Script" alt="最后提交">
  <img src="https://img.shields.io/github/actions/workflow/status/daitcl/Microsoft-Rewards-Script/check-version.yml" alt="构建状态">
  <a href="https://github.com/daitcl/Microsoft-Rewards-Script/blob/main/LICENSE">
    <img src="https://img.shields.io/badge/License-GPL--3.0--or--later-blue?style=flat-square" alt="GPL-3.0 许可证">
  </a>
  <a href="https://github.com/daitcl/Microsoft-Rewards-Script/pkgs/container/microsoft-rewards-script">
    <img src="https://img.shields.io/badge/GHCR.io-Package-blue?logo=github" alt="GHCR Package">
  </a>
</div>

本项目通过 GitHub Actions 自动构建 [chiihero/Microsoft-Rewards-Script](https://github.com/chiihero/Microsoft-Rewards-Script)（`V4-china` 分支）的 Docker 镜像并推送至 GitHub Container Registry (GHCR)。镜像基于 TypeScript + Patchright(Playwright) 新架构，针对国内用户深度本地化，集成中国热搜查询源、日志中文化、PushPlus / 微信 ClawBot 推送等特性。

> 本仓库仅用于**构建与分发镜像**，不包含任何源码修改。所有源码版权归原作者所有。

---

## 📦 镜像地址

[ghcr.io/daitcl/microsoft-rewards-script:latest](https://ghcr.io/daitcl/microsoft-rewards-script:latest)

拉取示例：

```bash
docker pull ghcr.io/daitcl/microsoft-rewards-script:latest
```

------

## 🔧 触发构建

1. 打开仓库 **Actions** 标签页
2. 左侧选择 **Build and Push Docker Image**
3. 点击 **Run workflow**
   - `tag`：自定义镜像标签（默认 `latest`）
4. 等待构建完成（首次约 3–8 分钟，之后走缓存更快）

构建产物会自动推送至 GHCR，并附带 `sha-<commit>` 标签便于追溯源码版本。

------

## 🚀 快速开始

### 1. 准备账号配置（`.env`）

```bash
cp env.example .env
```

编辑 `.env`，至少填一个账号：

```dotenv
ACCOUNT_1_EMAIL=you@example.com
ACCOUNT_1_PASSWORD=your_password
```

多账号按 `ACCOUNT_2_*`、`ACCOUNT_3_*` 递增（编号无需连续，按编号升序运行）。

### 2. 编写 compose.yaml

```yaml
services:
  microsoft-rewards-script:
    image: ghcr.io/daitcl/microsoft-rewards-script:latest
    container_name: microsoft-rewards-script
    restart: unless-stopped
    env_file:
      - path: .env
    environment:
      TZ: Asia/Shanghai
      CRON_SCHEDULE: "0 7 * * *"
      RUN_ON_START: "true"
      CONFIG_SEARCH_QUERY_ENGINES: "china,local"
      CONFIG_PUSHPLUS_ENABLED: "false"
      CONFIG_PUSHPLUS_TOKEN: ""
    volumes:
      - ./config:/usr/src/microsoft-rewards-script/config
      - ./sessions:/usr/src/microsoft-rewards-script/sessions
```

> ⚠️ **重要提示**
>
> - `.env` 必须通过 `env_file` 传入容器，否则容器内读不到账号。
> - 列表式 `environment:`（`- KEY=value`）的值**不要加引号**，否则引号会成为值的一部分导致 cron 表达式失效；映射式（`KEY: value`）按 YAML 规则正常加引号。

### 3. 启动

```bash
docker compose up -d
```

数据持久化目录：

- `./config/`：配置文件（首次运行自动生成 `config.json`）
- `./sessions/`：登录会话，重建容器不丢数据（**请定期备份**）

常用命令：

```bash
docker compose logs -f     # 查看日志
docker compose down        # 停止并删除容器
docker compose restart     # 重启
docker compose pull        # 拉取最新镜像
docker compose up -d       # 使用新镜像重建容器
```

------

## ⚙️ 常用环境变量

| 变量                                                      | 说明                                         | 默认值                   |
| :-------------------------------------------------------- | :------------------------------------------- | :----------------------- |
| `TZ`                                                      | 时区                                         | `Asia/Shanghai`          |
| `CRON_SCHEDULE`                                           | 定时调度                                     | `0 7 * * *`（每天 7 点） |
| `RUN_ON_START`                                            | 容器启动立即跑一次                           | `true`                   |
| `CONFIG_SEARCH_QUERY_ENGINES`                             | 查询源（国内推荐）                           | `china,local`            |
| `CONFIG_CHINA_API_APPKEY`                                 | [gmya.net](https://gmya.net/) appkey，解限流 | 空（免费档）             |
| `CONFIG_PUSHPLUS_ENABLED` / `CONFIG_PUSHPLUS_TOKEN`       | PushPlus 微信推送                            | `false` / 空             |
| `CONFIG_SERVERCHAN_ENABLED` / `CONFIG_SERVERCHAN_SENDKEY` | Server酱推送                                 | `false` / 空             |
| `CONFIG_CLAWBOT_ENABLED`                                  | 微信 ClawBot 推送                            | `false`                  |
| `API_MODE`                                                | 开启控制 API                                 | `false`                  |
| `API_TOKEN`                                               | API 鉴权令牌                                 | 空                       |

完整 `CONFIG_*` 变量列表请参考[上游 README 配置参考](https://github.com/chiihero/Microsoft-Rewards-Script/tree/V4-china#️-配置参考)。

------

## 🔄 更新镜像

当上游 `V4-china` 分支更新后：

1. 在 **Actions** 页面重新触发 **Run workflow**
2. 构建完成后本地拉取新镜像并重建容器：

```bash
docker compose pull
docker compose up -d
```

------

## 🧩 关于镜像名大小写

Docker / GHCR 要求**仓库名必须小写**，因此镜像引用固定为 `microsoft-rewards-script`。若希望容器显示原样名称，可通过 `container_name: Microsoft-Rewards-Script` 指定容器名。

------

## 📄 许可证

> 协议：[GPL-3.0-or-later](LICENSE) — 本程序为自由软件，您可以依据自由软件基金会发布的 GNU 通用公共许可证（第 3 版或任何更新版本）的条款重新分发和/或修改它。分发本程序或其衍生作品时，必须同样以 GPL 授权，并提供完整对应源码。
>
> 免责声明：本程序按“原样”提供，不附带任何明示或暗示的保证，包括但不限于适销性和特定用途适用性的保证。在任何情况下，作者或版权持有人均不对任何索赔、损害或其他责任负责，无论是在合同诉讼、侵权诉讼或其他诉讼中，由于软件或软件的使用或其他交易引起的。
>
> **风险自负**：使用自动化脚本可能导致 Microsoft Rewards 账户被暂停或封禁。本项目仅供教育目的，作者不对 Microsoft 采取的任何账户操作承担责任。

------

## 📜 同步与致谢

| 项目     | 信息                                                         |
| :------- | :----------------------------------------------------------- |
| 上游仓库 | [chiihero/Microsoft-Rewards-Script](https://github.com/chiihero/Microsoft-Rewards-Script)（`V4-china` 分支） |
| 再上游   | [TheNetsky/Microsoft-Rewards-Script](https://github.com/TheNetsky/Microsoft-Rewards-Script)（v4 分支） |
| 本仓库   | [daitcl/Microsoft-Rewards-Script](https://github.com/daitcl/Microsoft-Rewards-Script)（镜像构建） |

本项目不修改源码，仅提供 CI 构建与镜像分发；分发镜像时同样遵循 GPL-3.0 条款，完整源码请向上游仓库获取。
