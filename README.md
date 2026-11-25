# ASCMS

ASCMS 是一套可自托管的建站系统（CMS）：NestJS 11 后端 + Vue 3 管理端，内置安装向导、栏目/内容/扩展模型、表单、模板机制、备份还原与在线升级。

本仓库是**官方发布仓库**，只存放发布包，不包含源码。

- 官网：<https://ascms.cmsuse.com>
- 演示站：<https://demo.cmsuse.com>
- 最新版本：[Gitee Releases](https://gitee.com/gas0324/ascms-releases/releases/latest)
- GitHub 镜像：<https://github.com/Gas0515/ascms-releases>

## 下载

打开最新 Release 下载附件：

| 附件 | 用途 |
| --- | --- |
| `ascms-<version>.zip` | **推荐**。完整程序包，包含后端 dist、前端构建产物与 `init.sql`；用于手动部署，1.0.3 起也用于在线升级 |
| `program-<version>.zip` | 旧版在线升级包。1.0.0–1.0.2 的升级器只识别该前缀；1.0.3 额外提供兼容包，1.0.4 起不再发布 |
| `upgrade-<version>.sql` | 增量数据库脚本。仅在该版本有表结构变更时提供，在线升级会自动执行 |

> ⚠️ Release 页面中平台自动生成的 `v<version>.zip` / `v<version>.tar.gz` 是源码快照（本仓库只包含说明文档），**不能部署，也不能用于升级**。请只下载上表中的附件。

## 在线升级（推荐）

1. 登录后台，进入「系统 → 系统升级」
2. 点击「检测更新」，有新版本时点击「立即升级」
3. 等待程序自动下载、备份数据库、执行增量 SQL、替换程序文件并重启

升级只替换 `dist`、`package.json`、`frontend` 三处，`.env`、`.installed`、`data`、`logs`、`backup`、`backend/public`、`backend/sql` 不会被覆盖；私有模板会在替换前备份、替换后还原。

注意事项：

- 当前版本以数据库 `as_system.version` 为准（1.0.2 起），其次才是 `data/version` 与包内 `package.json`
- 升级前会自动整库备份；数据库结构变更失败时依赖备份回滚
- 按版本号顺序升级，不要跳过中间版本，升级器会收集区间内所有 SQL
- **Docker 部署**：官方镜像 `gas0324/ascms`（[Docker Hub](https://hub.docker.com/r/gas0324/ascms)）。`/app/backend`、`/app/frontend` 用命名卷持久化，代码与配置都在卷里，重建容器不丢；升级请在后台「系统 → 系统升级」完成，**不要替换镜像**

## 手动部署

环境要求：

| 项目 | 要求 |
| --- | --- |
| Node.js | 20 LTS 及以上 |
| MySQL | 5.7 及以上，推荐 8.0 |
| 系统 | Linux x64 / Windows |
| 内存 | 建议 1 GB 以上 |
| 磁盘 | 建议 2 GB 以上 |

步骤：

```bash
# 1. 解压
unzip ascms-<version>.zip -d /www/ascms
cd /www/ascms/backend

# 2. 安装生产依赖
npm install --omit=dev

# 3. 启动
NODE_ENV=production node dist/main.js
```

默认监听 `9000` 端口（可用 `PORT` 覆盖）。首次启动时 `.installed` 不存在，浏览器打开 `http://<host>:9000/install` 进入安装向导；在向导中填写数据库、站点信息与后台路径，系统会自动建表。安装完成后，后台地址是 `.env` 中的 `FRONTEND_ROUTE`。

生产环境建议用 PM2 或 systemd 托管进程，并在前面加 Nginx/OpenResty 反向代理（上传大小 `client_max_body_size 50m`，升级进度需要 WebSocket `/socket.io/`）。

> Docker 部署、反向代理与 HTTPS 的完整说明见官网「部署与运维」：<https://ascms.cmsuse.com/deploy>

## 版本历史

| 版本 | 日期 | 主要变更 |
| --- | --- | --- |
| [1.0.8](https://gitee.com/gas0324/ascms-releases/releases/tag/v1.0.8) | 2026-11-24 | 修复静态模式文章页 404；访问统计记录真实 IP，来源判断按请求域名 |
| [1.0.7](https://gitee.com/gas0324/ascms-releases/releases/tag/v1.0.7) | 2026-11-24 | 新增邮件设置：SMTP 主机/端口/加密方式/账号/授权码、测试连接与测试发信；发件地址固定使用登录账号；密码不回显、留空保存不覆盖；`as_system` 增加 9 个邮件字段（需增量 SQL） |
| [1.0.6](https://gitee.com/gas0324/ascms-releases/releases/tag/v1.0.6) | 2026-11-23 | 静态生成跳过未配置模板的栏目；静态/远程模式首页 302 到 `index.html`、缺失静态页直接 404；升级弹窗重做并显示下载速度与剩余时间 |
| [1.0.5](https://gitee.com/gas0324/ascms-releases/releases/tag/v1.0.5) | 2026-11-22 | 修复静态生成不可用（栏目链接规则缺省兜底、空栏目出页、链接栏目内容页）；网站运行模式真正生效（动态/静态/远程切换链接方案，静态文件缺失时按规则兜底渲染） |
| [1.0.4](https://gitee.com/gas0324/ascms-releases/releases/tag/v1.0.4) | 2026-11-22 | 修复自定义模板静态资源（如 `/yujiekt`）被前台 404 兜底误伤；优化系统升级进度弹窗；1.0.4 起在线升级只发 `ascms-` 包 |
| [1.0.3](https://gitee.com/gas0324/ascms-releases/releases/tag/v1.0.3) | 2026-11-21 | 产品功能站并入内置默认模板（浅色主题）；新增 Gzip 压缩开关与静态资源长缓存；修复搜索 XSS、前台 404、安装向导；在线升级包改用 `ascms-` 前缀（保留 `program-` 兼容包） |
| [1.0.2](https://gitee.com/gas0324/ascms-releases/releases/tag/v1.0.2) | 2026-11-18 | 修复在线升级（卷挂载 EBUSY、进度订阅、版本记录）、登录保存密码、访问统计与后台顶栏 |
| [1.0.1](https://gitee.com/gas0324/ascms-releases/releases/tag/v1.0.1) | 2026-11-18 | 修复在线升级依赖、标签重名、登录自动填充与工作台卡片 |
| [1.0.0](https://gitee.com/gas0324/ascms-releases/releases/tag/v1.0.0) | 2026-11-17 | 首个正式版本：安装向导、内容/栏目/表单、备份还原、在线升级、模板机制 |

完整更新说明见各版本 Release。

## 反馈

- 问题与建议：<https://gitee.com/gas0324/ascms-releases/issues>
- 其他联系：<https://ascms.cmsuse.com/contact>

---

© ASCMS. 本仓库仅用于官方版本分发。
