# WX Farm Bot — QQ 农场多账号挂机 + Web 面板

基于 Node.js 的 QQ 农场自动化工具，提供多账号挂机、农场与好友自动化、作物与超变图鉴、活动中心、商城、数据分析和 Web 控制面板。

在原项目基础上，本仓库**整合了微信登录与微信自动刷新 Code 能力**，并在前端设置面板新增了微信自动刷新 Code 的可视化管理界面。

> [!IMPORTANT]
> 首次启动会创建默认管理员 `admin` / `admin`，Web 面板默认端口为 `3007`。对外部署后请立即修改密码，避免将未加防护的管理端口直接暴露到公网。

## 核心特性

### 登录方式
- 抓包 Code 登录
- 微信扫码登录（OAuth 2.0 + MMTLS 原生协议）
- QQ 扫码登录
- **微信 Code 自动刷新**：后台按配置间隔自动续期登录凭证，避免掉线

### 微信自动刷新 Code（本仓库新增）
在 Web 面板「设置 → 自动控制」中提供独立管理卡片，支持：
- 查看所有微信账号的自动刷新状态
- 按账号启用 / 禁用自动刷新
- 自定义刷新间隔（1–1440 分钟）
- 查看最后刷新时间、下次刷新时间、刷新结果
- 手动触发立即刷新

### 农场自动化
- 自动收获、种植、浇水、除草、除虫、铲除、土地升级
- 自动施肥、多季补肥、化肥自动购买
- 一键务农 / 种植 / 收获 / 铲除 / 全收
- 种植策略、背包种子优先级、作物黑名单、执行间隔配置

### 好友互动
- 自动访问、偷菜、帮忙浇水除草除虫、放草放虫
- 好友黑名单、安静时段、互动经验上限
- 单次访问合并处理，好友宠物信息按天缓存

### 图鉴 / 背包 / 商城 / 活动
- 作物图鉴与超变图鉴等级、进度、果实收藏
- 背包管理、批量出售、神秘商人、游戏商城购买
- 活动中心：千星游记、观星礼录、星砂商店、节令活动、鹊桥寄情、雨落成诗等

### Web 控制面板
- 概览、个人、活动、好友、分析、神秘商人、游戏商城、设置页面
- Socket.IO 实时状态与日志
- PC 固定侧栏，移动端顶部栏 + 底部导航，响应式适配
- 基于 `pushoo` 的事件推送与离线提醒

## 微信自动刷新 API

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| `GET` | `/api/wechat-auto-refresh-status` | 获取所有微信账号的刷新状态 |
| `POST` | `/api/wechat-auto-refresh/:accountId` | 更新指定账号的刷新配置（`enabled`、`intervalMinutes`） |
| `POST` | `/api/wechat-auto-refresh/:accountId/refresh-now` | 手动触发指定账号立即刷新 |

详细说明见 `WECHAT_AUTO_REFRESH_FEATURE.md`，Worker 集成方式见 `WORKER_INTEGRATION_PATCH.md`。

## 技术栈

| 模块 | 技术 |
| --- | --- |
| 后端 | Node.js 20、Express 4、Socket.IO 4、TypeScript 5.9 |
| 前端 | Vue 3.5、Naive UI 2、Vue Router、Pinia、Vite 7、UnoCSS |
| 工程 | pnpm 10、ESLint 9、Docker Compose |
| 协议 | WebSocket、Protocol Buffers、TSDK WASM、MMTLS（微信登录） |

## 快速开始

### 环境要求
- Node.js 20.19+
- pnpm 10+（推荐通过 Corepack 启用）

```bash
corepack enable
pnpm --version
```

### 源码运行

```bash
git clone https://github.com/H6beg/wx-farm-bot.git
cd wx-farm-bot
pnpm install
pnpm dev
```

`pnpm dev` 会先构建前端，再启动后端服务。启动后访问：
- 本机：`http://localhost:3007`
- 局域网：`http://<服务器 IP>:3007`

分别启动开发服务：

```bash
# 终端 1：前端开发服务
pnpm dev:web

# 终端 2：后端开发服务
pnpm dev:core
```

只运行 `pnpm dev:core` 时，需先执行一次 `pnpm build:web`。

### Linux 后台脚本

```bash
bash ./start.sh            # 启动
bash ./start.sh --rebuild  # 忽略缓存完整重建
tail -f app_dev.log        # 查看日志
bash ./stop.sh             # 停止
```

### Docker Compose

```bash
docker compose up -d --build
docker compose logs -f
docker compose down
```

默认访问 `http://<服务器 IP>:3007`。修改根目录 `.env` 中的 `PORT` 可调整宿主机映射端口。数据保存在 `qq-farm-data` 与 `qq-farm-logs` 命名卷中，`docker compose down` 不会删除这些卷（勿用 `-v`）。

## 常用脚本

| 命令 | 说明 |
| --- | --- |
| `pnpm dev` | 构建前端并启动后端 |
| `pnpm dev:web` | 前端开发服务 |
| `pnpm dev:core` | 后端开发服务 |
| `pnpm build` | 构建前端与后端 |
| `pnpm build:web` | 仅构建前端 |
| `pnpm build:core` | 仅编译后端 |
| `pnpm lint` | 前后端 ESLint |
| `pnpm typecheck:web` | 前端类型检查 |

## 项目结构

```
wx-farm-bot/
├── core/                          后端（Express + Socket.IO + TypeScript）
│   └── src/
│       ├── controllers/admin/     管理端路由（含微信自动刷新 API）
│       ├── services/wx-login/     微信登录服务（OAuth 2.0 + MMTLS 协议）
│       ├── services/              wx-login-adapter 登录适配层
│       └── runtime/               auto-code-refresh 自动刷新逻辑
├── web/                           前端（Vue 3 + Naive UI + Vite）
│   └── src/
│       ├── components/settings/   设置组件（含 WechatAutoRefreshSettings.vue）
│       └── views/                 页面视图
├── docs/                          协议与实现说明文档
├── tools/                         工具脚本
├── docker-compose.yml
├── start.sh / stop.sh
├── WECHAT_AUTO_REFRESH_FEATURE.md  微信自动刷新功能文档
└── WORKER_INTEGRATION_PATCH.md     Worker 集成补丁说明
```

## 安全提示

- 部署后立即修改默认管理员密码
- 不要将管理端口直接暴露到公网，建议置于反向代理或内网环境
- Web 面板只有单一超级管理员，不提供注册与多租户

## 许可

本项目仅供学习与个人使用，请遵守相关游戏服务条款，勿用于商业或违规用途。
