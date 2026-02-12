---
title: Docker 容器化入门：从零开始
date: 2025-02-05
tags:
  - Docker
  - 运维
  - 容器化
categories:
  - 后端技术
cover: https://images.unsplash.com/photo-1605745341112-85968b19335b?w=800&q=80
excerpt: 从零开始学习 Docker，了解镜像、容器、Dockerfile 的核心概念，以及如何用 Docker Compose 编排多容器应用。
---

## 什么是 Docker？

Docker 是一个开源的容器化平台，可以将应用程序及其依赖打包到一个轻量级的容器中运行，实现"一次构建，到处运行"。

## 核心概念

- **镜像 (Image)**：应用的只读模板
- **容器 (Container)**：镜像的运行实例
- **Dockerfile**：构建镜像的脚本
- **Docker Compose**：多容器编排工具

## Dockerfile 示例

```dockerfile
# 使用 Node.js 官方镜像
FROM node:18-alpine

# 设置工作目录
WORKDIR /app

# 先复制依赖文件（利用缓存层）
COPY package*.json ./
RUN npm install --production

# 复制源代码
COPY . .

# 暴露端口
EXPOSE 3000

# 启动命令
CMD ["node", "server.js"]
```

## 常用命令

```bash
# 构建镜像
docker build -t my-app .

# 运行容器
docker run -d -p 3000:3000 --name my-app my-app

# 查看运行中的容器
docker ps

# 查看日志
docker logs my-app

# 停止容器
docker stop my-app
```

## Docker Compose

对于多容器应用，使用 `docker-compose.yml`：

```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - db
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: myapp
      POSTGRES_PASSWORD: secret
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```

## 最佳实践

1. **使用 Alpine 镜像**减小体积
2. **多阶段构建**分离构建和运行环境
3. **合理利用缓存层**，把不常变化的步骤放前面
4. **不要在容器中存储数据**，使用 Volume
5. **使用 .dockerignore** 排除无关文件

## 总结

Docker 大大简化了应用的部署流程，是现代开发运维的必备工具。从简单的单容器应用开始，逐步过渡到 Docker Compose 多容器编排，你会发现 Docker 比想象中简单得多。
