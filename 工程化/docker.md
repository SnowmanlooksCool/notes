一、基础镜像 vs 应用镜像
类型    作用    示例内容
基础镜像    提供通用的构建和运行环境    Node.js、Nginx、系统依赖、全局工具
应用镜像    基于基础镜像，添加具体应用代码和配置    项目代码、构建后的静态文件、应用配置
二、创建基础镜像（Base Image）
场景：为团队或项目预装通用环境（如 Node.js + Nginx）
dockerfile
Dockerfile.base
FROM node:18-alpine AS base
安装通用依赖（如构建工具、Nginx）
RUN apk add --no-cache nginx git
设置工作目录
WORKDIR /app
全局安装常用工具（可选）
RUN npm install -g pnpm
其他通用配置...
构建基础镜像：
bash
docker build -t my-frontend-base:latest -f Dockerfile.base .
三、创建应用镜像（Application Image）
方案 1：直接构建（单阶段）
dockerfile
Dockerfile.app
FROM my-frontend-base:latest
复制代码并构建
COPY package.json pnpm-lock.yaml ./
RUN pnpm install
COPY . .
RUN pnpm build
配置 Nginx
COPY nginx.conf /etc/nginx/conf.d/default.conf
暴露端口并启动
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
方案 2：多阶段构建（推荐，更安全、镜像更小）
dockerfile
Dockerfile.app
---------- 阶段1：构建 ----------
FROM my-frontend-base:latest AS builder
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile
COPY . .
RUN pnpm build
---------- 阶段2：运行 ----------
FROM nginx:alpine
从基础镜像复制构建产物
COPY --from=builder /app/dist /usr/share/nginx/html
复制 Nginx 配置
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
四、构建并运行镜像
bash
1. 构建应用镜像
docker build -t my-frontend-app:latest -f Dockerfile.app .
2. 运行容器
docker run -d -p 80:80 --name my-app my-frontend-app:latest
五、最佳实践
多阶段构建：
分离构建环境和运行环境，最终镜像仅保留运行所需内容（如 Nginx + 静态文件）。
减少镜像体积和潜在安全风险。
使用 .dockerignore：
text
node_modules
.git
Dockerfile
忽略无关文件，加速构建过程。
镜像标签管理：
bash
docker tag my-frontend-app:latest my-registry.com/frontend:v1.0
为镜像添加版本标签，便于回滚和追踪。
使用 Alpine 镜像：
选择 node:18-alpine 或 nginx:alpine 等轻量级基础镜像，减少体积。
六、完整流程示例
csharp
项目结构
├── Dockerfile.base     # 基础镜像定义
├── Dockerfile.app      # 应用镜像定义
├── nginx.conf          # Nginx 配置
└── src/                # 前端代码
构建基础镜像：
bash
docker build -t my-frontend-base -f Dockerfile.base .
构建应用镜像：
bash
docker build -t my-frontend-app -f Dockerfile.app .
验证运行：
bash
docker run -p 80:80 my-frontend-app
七、常见问题
Q1：是否需要为每个项目创建基础镜像？
团队协作：建议统一基础镜像，确保环境一致性。
个人项目：可直接使用官方镜像（如 node:alpine），无需自定义基础镜像。
Q2：如何更新基础镜像？
修改 Dockerfile.base。
重新构建并推送新版本：
bash
docker build -t my-frontend-base:v2.0 -f Dockerfile.base .
docker push my-registry.com/frontend-base:v2.0