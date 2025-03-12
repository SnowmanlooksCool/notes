github action 可以在代码提交后进行一些后续的作业，可以在此来触发文件的打包和把镜像发送到docker hub的问题

但是如果对于一些依赖项目 每次进行打包太费时间 考虑使用缓存策略

1 使用docker的分层缓存策略 但是 每次进行action 都相当于是创建了新的上下文 
2 action 自带的 cache from  - 好处 免费 10G空间  在每次构建时候不需要从类似 docker hub上拉数据 但是缓存只有7天的有效期？ 看后续能否突突破这个问题

https://juejin.cn/post/7215967453914234938

dockerfile 
`
# 使用官方基础镜像
FROM python:3.10-slim as base

# 设置缓存目录
ENV PIP_CACHE_DIR=/root/.cache/pip
RUN mkdir -p ${PIP_CACHE_DIR}

# 优先安装系统依赖（利用Docker层缓存）
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# 安装Python依赖（利用PIP缓存）
COPY requirements.txt .
RUN pip install --user -r requirements.txt
`


 aciton yml
# .github/workflows/docker-build.yml
name: Docker Build with Cache

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and Push
        uses: docker/build-push-action@v4
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: your-image:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max

# .github/workflows/docker-build.yml
name: Docker Build with Explicit Cache

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Cache Docker layers
        uses: actions/cache@v3
        with:
          path: /tmp/.buildx-cache
          key: ${{ runner.os }}-buildx-${{ hashFiles('**/Dockerfile') }}
          restore-keys: |
            ${{ runner.os }}-buildx-

      - name: Setup Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Build and Push
        uses: docker/build-push-action@v4
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: your-image:latest
          cache-from: type=local,src=/tmp/.buildx-cache
          cache-to: type=local,dest=/tmp/.buildx-cache-new,mode=max

      - name: Update cache
        run: |
          rm -rf /tmp/.buildx-cache
          mv /tmp/.buildx-cache-new /tmp/.buildx-cache