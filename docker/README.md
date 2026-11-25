# Docker 部署文件

镜像：`gas0324/ascms`（Docker Hub，公开）

| 文件 | 用途 |
|---|---|
| `Dockerfile` | 标准镜像，`INSTALLED=false`，启动后进安装向导 |
| `Dockerfile.demo` | 演示站镜像，写入 `.installed` 跳过向导；不参与 CI |
| `docker-compose.yml` | 官方推荐部署配置 |
| `README.md` | 本文件 |

## 构建

构建上下文必须是发布包目录，不能用仓库源码（包内 `backend/dist`、`frontend/` 才是构建结果）：

```bash
node build-release.js 1.0.8
unzip releases/ascms-1.0.8.zip -d pkg-1.0.8
cp docker/Dockerfile pkg-1.0.8/
docker build -t gas0324/ascms:1.0.8 pkg-1.0.8
```

CI 也是这个流程：从 Release 附件解压 → 放入本目录的 `Dockerfile` → 构建 → 推送。

## 运行

```bash
docker compose up -d
# 打开 http://<host>:9000 走进安装向导
```

MySQL 需要提前准备好，推荐独立实例。数据库主机填法与网络位置有关：

| MySQL 位置 | `DB_HOST` 填什么 |
|---|---|
| 同一 compose 网络里的服务 | 服务名，如 `mysql` |
| Linux 宿主机 | `host.docker.internal`，并解开 compose 里的 `extra_hosts` |
| 另一台机器 | 那台机器的 IP，且账号允许远程连接 |

## 升级

在后台「系统 → 系统升级」完成，**不要用替换镜像的方式升级**。

`/app/backend`、`/app/frontend` 是命名卷，卷里已有代码时镜像内代码不生效；换镜像只会改变全新安装的起点，已装好的站点仍跑卷里的代码。

## 备份

```bash
docker run --rm -v ascms_ascms_backend:/data -v "$PWD":/out alpine \
  tar czf /out/backend-$(date +%F).tgz -C /data .
```

数据库单独备份。恢复时把归档解回卷，再 `docker compose up -d`。

## 注意

- 卷名带项目名，换目录或加 `-p` 会得到新空卷，等于重装
- `docker compose down -v` 会删除卷，代码与安装配置一起丢失
- 仅支持 `linux/amd64`；ARM 平台（Apple Silicon、树莓派、鲲鹏）暂不支持
- 本目录以 GitHub 仓库为准，修改后请同步 Gitee 仓库同名文件

完整说明见官网文档「安装部署 → Docker 部署」。