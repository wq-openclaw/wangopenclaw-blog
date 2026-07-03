---
layout: post
title: "GitHub Actions + Docker + 阿里云：从零搭建个人 CI/CD 流水线"
date: 2026-07-02 08:00:00 +0800
slug: github-actions-docker-cicd-pipeline
categories: [教程]
tags: [CI/CD, GitHub Actions, Docker, DevOps]
---

# GitHub Actions + Docker + 阿里云：从零搭建个人 CI/CD 流水线

个人项目的 CI/CD，核心需求只有一个：**代码 push 后自动构建部署，我不用碰服务器**。

本文手把手搭一条最小可用流水线，全部免费（对个人项目来说），从 push 到上线不超过 3 分钟。

## 架构

```
Git push → GitHub Actions 构建 → Docker 打包 → 推送到阿里云镜像仓库 → SSH 登录服务器 → 拉取镜像 → 重启容器
```

## 1. Dockerfile

在项目根目录创建：

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

要点：
- **多阶段构建**：最终镜像只有运行所需文件，不含构建工具链，体积从 1.2GB 降到 150MB
- **`npm ci` 而不是 `npm install`**：lock 文件驱动，构建可复现

## 2. GitHub Actions 配置

创建 `.github/workflows/deploy.yml`：

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Login to Aliyun ACR
        uses: aliyun/acr-login@v1
        with:
          region-id: cn-hangzhou
          access-key-id: ${{ secrets.ALIYUN_ACCESS_KEY_ID }}
          access-key-secret: ${{ secrets.ALIYUN_ACCESS_KEY_SECRET }}

      - name: Build and Push
        run: |
          docker build -t registry.cn-hangzhou.aliyuncs.com/your-namespace/my-app:${{ github.sha }} .
          docker push registry.cn-hangzhou.aliyuncs.com/your-namespace/my-app:${{ github.sha }}

      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            docker pull registry.cn-hangzhou.aliyuncs.com/your-namespace/my-app:${{ github.sha }}
            docker stop my-app || true
            docker rm my-app || true
            docker run -d --name my-app --restart always \
              -p 3000:3000 \
              -e DB_URL=${{ secrets.DB_URL }} \
              registry.cn-hangzhou.aliyuncs.com/your-namespace/my-app:${{ github.sha }}
```

## 3. GitHub Secrets

在 Repo Settings → Secrets and variables → Actions 中添加：

| Secret | 说明 |
|--------|------|
| `ALIYUN_ACCESS_KEY_ID` | 阿里云 RAM 用户的 AccessKey |
| `ALIYUN_ACCESS_KEY_SECRET` | 对应的 Secret |
| `SERVER_HOST` | 服务器 IP |
| `SERVER_USER` | SSH 用户名（建议用非 root） |
| `SERVER_SSH_KEY` | SSH 私钥 |
| `DB_URL` | 数据库连接串 |

## 4. 初始化服务器

第一次需要在服务器上先做好基础设施：

```bash
# 安装 Docker（如果没装）
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER

# 拉取一次，确保后续 pull 不需要额外交互
docker login --username=your-username registry.cn-hangzhou.aliyuncs.com

# 创建目录
mkdir -p ~/app/data
```

## 5. 验证

push 到 main 后，去 GitHub 仓库的 Actions 标签页看流水线状态。绿了就是部署成功：

```bash
# 验证服务是否正常
curl -I http://你的服务器IP:3000
# 应该返回 200 OK
```

## 6. 优化：只对真正需要部署的路径触发

```yaml
on:
  push:
    branches: [main]
    paths:
      - 'src/**'
      - 'Dockerfile'
      - 'package*.json'
      - '.github/workflows/deploy.yml'
```

这样改 README 或 docs 不会触发部署，省 Actions 分钟数。

## 7. 优化：用 Docker Compose 管理多服务

当项目从 1 个服务变成 3 个时，`docker run` 变成噩梦。用 `docker-compose.yml` 替代：

```yaml
services:
  app:
    image: registry.cn-hangzhou.aliyuncs.com/your-namespace/my-app:latest
    ports:
      - "3000:3000"
    environment:
      - DB_URL=${DB_URL}
    restart: always

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
```

部署脚本改成：

```bash
docker compose pull
docker compose up -d --remove-orphans
```

## 几点经验

1. **不要在 Actions 里写死密码**——用 Secrets，哪怕项目不公开也要用
2. **不要用 latest 标签**——用 commit SHA 或版本号，方便回滚
3. **第一次手动部署一次**——验证服务器环境和网络没问题，再上自动流水线
4. **Actions 免费额度**：每个月 2000 分钟，个人项目根本用不完

这条流水线我从 2023 年用到现在，在三个不同项目上跑，没出过问题。够简单、够稳定，需要时再加 slack 通知等花活。
