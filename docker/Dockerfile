# ASCMS 标准镜像
#
# 构建上下文必须是 build-release.js 的产物目录——包内 backend/dist 是编译结果、
# frontend/ 是前端构建结果，不能用仓库根目录的源码：
#
#   node build-release.js 1.0.8
#   unzip releases/ascms-1.0.8.zip -d pkg-1.0.8
#   cp docker/Dockerfile pkg-1.0.8/
#   docker build -t gas0324/ascms:1.0.8 pkg-1.0.8
#
# 默认按首次安装处理，容器启动后进入安装向导。
# 数据库已初始化的预装镜像：docker build --build-arg INSTALLED=true ...
#
# 本文件是标准镜像的唯一来源（CI 从这里取），修改后请同步 Gitee 仓库同名文件。

FROM node:20-slim

ENV NODE_ENV=production \
    PORT=9000

WORKDIR /app/backend

# 依赖层：只复制 package.json，改动代码时可复用缓存
COPY backend/package.json ./package.json
RUN npm config set registry https://registry.npmmirror.com \
    && npm install --omit=dev --no-audit --no-fund \
    && npm cache clean --force

# 应用文件（来自发布包）
COPY backend/dist ./dist
COPY backend/public ./public
COPY backend/sql ./sql
COPY frontend /app/frontend

# 运行时目录、版本号；INSTALLED=true 表示数据库已初始化，跳过安装向导
ARG INSTALLED=false
RUN mkdir -p logs/mysql public/upload data \
    && if [ "$INSTALLED" = "true" ]; then touch .installed; fi \
    && node -p "require('./package.json').version" > data/version

# 健康检查：进程活着不等于服务可用（数据库连不上、端口没监听都会显示 Up）
HEALTHCHECK --interval=30s --timeout=5s --start-period=30s --retries=3 \
  CMD node -e "fetch('http://127.0.0.1:9000/health').then(r=>process.exit(r.ok?0:1)).catch(()=>process.exit(1))"

EXPOSE 9000

CMD ["node", "dist/main.js"]