---
title: "Docker Desktop 收费后，我花了一周测试了 5 个免费替代方案，最终选了它"
date: 2026-07-04
categories: ["开发工具", "Docker", "效率"]
tags: ["docker", "podman", "orbstack", "colima", "rancher-desktop", "开发环境"]
---

# Docker Desktop 收费后，我花了一周测试了 5 个免费替代方案，最终选了它

> 2024 年 11 月，Docker 把 Pro 版从 $5/月涨到 $9/月，Team 版从 $9/月涨到 $15/月。虽然个人开发者和小企业还能免费用，但这条政策已经让无数人开始寻找退路。
>
> 问题是：市面上的替代方案鱼龙混杂，到底哪个能用、哪个好用、哪个坑最多？
>
> 我用了一周时间，在 MacBook Pro M3 和一台 Windows 笔记本上实测了 5 个主流方案，结论是：**大多数人选错了**。

---

## 先泼一盆冷水：Docker Desktop 到底收不收费？

很多人被"Docker Desktop 收费"这个说法吓到了，其实政策是这样的：

| 用户类型 | 是否收费 |
|---------|---------|
| 个人开发者 | ✅ 永久免费 |
| 小企业（<250人，年收入<1000万美元） | ✅ 免费 |
| 大企业/政府 | ❌ 必须付费 |

所以如果你只是个人开发，**现在完全不用慌**。但问题是：

1. **企业用户没得选**，必须付费或迁移
2. **Docker 的涨价趋势很明显**（2021 年首次收费 → 2024 年涨价 67%-80%），谁知道下次会不会砍免费额度？
3. **厂商锁定**是个真实存在的风险，提前准备退路是明智的

我的建议是：**即使你现在免费，也该知道有哪些替代方案，以及迁移成本到底多高。**

---

## 5 个替代方案实测对比

我测试的环境：
- **Mac**: MacBook Pro M3, 18GB RAM, macOS 15
- **Windows**: ThinkPad T14, 16GB RAM, Windows 11
- **测试项目**: 一个包含 5 个服务的 Docker Compose 项目（Node.js + PostgreSQL + Redis + Nginx）

### 1. OrbStack —— Mac 用户的"真香"选择

```bash
# 安装
brew install orbstack

# 启动后直接使用
docker run -d -p 8080:80 nginx
docker-compose up -d
```

**优点：**
- 启动速度比 Docker Desktop **快 10 倍**（实测冷启动 2 秒 vs Docker Desktop 的 20 秒）
- 内存占用极低，我的 M3 上只占了 300MB 左右
- 完全兼容 Docker CLI 和 Compose，零学习成本
- **个人永久免费**，连商业用途都免费

**缺点：**
- 只支持 macOS，Windows 用户别想了
- 没有内置 Kubernetes（虽然大多数人用不上）
- 社区相对较小，遇到问题搜不到太多资料

**我的评价：** 如果你用 Mac，这几乎是零缺点的选择。我现在的主力开发机已经彻底卸载了 Docker Desktop。

---

### 2. Podman + Podman Desktop —— 最"正确"的选择，但不一定最舒服

```bash
# macOS 安装
brew install podman
podman machine init
podman machine start

# 设置别名（可选但推荐）
alias docker=podman
alias docker-compose='podman-compose'

# 使用
podman run -d -p 8080:80 nginx
```

**优点：**
- **完全开源免费**，无商业限制
- 安全架构更好：**无守护进程、默认 rootless**
- CLI 与 Docker 几乎 100% 兼容
- 跨平台（Windows/macOS/Linux）
- Podman Desktop 的 GUI 已经做得很好了

**缺点：**
- macOS 上需要先初始化虚拟机（`podman machine init`），第一次配置有点麻烦
- 某些边缘场景兼容性有问题（比如我测试时遇到 `host.docker.internal` 解析失败）
- Compose 支持需要额外安装 `podman-compose`
- 文件共享性能在 macOS 上不如 OrbStack

**我的评价：** 从"技术正确性"角度，Podman 是最好的选择。但从"开箱即用"角度，它对新手不够友好。适合有一定 Docker 基础、注重安全的开发者。

---

### 3. Colima —— 极客的最爱，但别推荐给同事

```bash
# 安装
brew install colima docker docker-compose

# 启动
colima start

# 可选：带 Kubernetes
colima start --kubernetes

# 使用（直接兼容 docker 命令）
docker run -d -p 8080:80 nginx
```

**优点：**
- 资源占用极低，比 Docker Desktop 轻量得多
- 支持多种运行时（Docker / containerd / k3s）
- 对 Apple Silicon 优化很好
- 完全免费开源

**缺点：**
- **没有 GUI**，纯命令行工具
- 需要手动管理虚拟机生命周期（start/stop/status）
- 配置文件是 YAML，调试起来有点烦
- 文件共享偶尔有权限问题

**我的评价：** 如果你是命令行控，Colima 很棒。但如果你团队里有设计师或产品经理偶尔需要跑一下环境，别用 Colima —— 你会变成"技术支持"。

---

### 4. Rancher Desktop —— Kubernetes 开发者的首选

```bash
# macOS 安装
brew install rancher
```

**优点：**
- 内置 Kubernetes（k3s），一键启用
- 支持切换 containerd / dockerd 运行时
- 封装了 nerdctl、kubectl、Helm 等工具
- 完全免费开源

**缺点：**
- 资源占用比 OrbStack/Colima 高
- 如果你不需要 Kubernetes，它有点"过度设计"
- 启动速度一般
- GUI 界面信息密度太高，有点乱

**我的评价：** 如果你每天和 Kubernetes 打交道，Rancher Desktop 是最佳选择。但如果你只是跑几个容器做 Web 开发，它太重了。

---

### 5. Container Desktop —— Windows 用户的救星

```powershell
# 通过 Chocolatey 安装
choco install container-desktop
```

**优点：**
- 基于 WSL2，与 Windows 集成很好
- 完全兼容 Docker 命令
- 比 Docker Desktop 更轻量
- 开源免费

**缺点：**
- 只支持 Windows
- 社区很小，文档不全
- 更新频率低

**我的评价：** Windows 用户如果企业要求替换 Docker Desktop，这是可行的方案。但说实话，Windows 上的容器体验整体不如 macOS/Linux。

---

## 实测数据对比

| 工具 | 冷启动时间 | 内存占用 | Compose 兼容性 | 学习成本 | 推荐指数 |
|-----|-----------|---------|--------------|---------|---------|
| **OrbStack** | 2s | 300MB | ✅ 完美 | 极低 | ⭐⭐⭐⭐⭐ |
| **Podman Desktop** | 15s | 800MB | ⚠️ 偶尔有问题 | 中 | ⭐⭐⭐⭐ |
| **Colima** | 5s | 400MB | ✅ 完美 | 高 | ⭐⭐⭐⭐ |
| **Rancher Desktop** | 20s | 1.2GB | ✅ 完美 | 中 | ⭐⭐⭐ |
| **Container Desktop** | 10s | 600MB | ✅ 完美 | 低 | ⭐⭐⭐ |

---

## 我的最终选择和建议

### 个人开发者（Mac）
**直接上 OrbStack。** 不要犹豫。它是唯一一个"比 Docker Desktop 更好用还免费"的工具。

### 个人开发者（Windows）
如果企业没强制要求，继续用 Docker Desktop 免费版。如果要替换，试试 Podman Desktop 或 Container Desktop。

### 团队/企业
**推荐 Podman。** 理由很简单：
1. 完全开源，没有厂商锁定风险
2. rootless 架构更安全
3. CLI 兼容，迁移成本低
4. SUSE/Red Hat 背书，企业级支持有保障

### 云原生/K8s 开发者
**Rancher Desktop。** 内置 k3s 省了你很多事情。

---

## 迁移实战：从 Docker Desktop 到 OrbStack

如果你决定迁移，步骤其实很简单：

```bash
# 1. 卸载 Docker Desktop
# macOS: 直接把 Docker.app 拖到废纸篓

# 2. 安装 OrbStack
brew install orbstack

# 3. 启动 OrbStack（会自动配置 docker CLI）
# 打开 OrbStack 应用即可

# 4. 验证
docker --version
docker-compose --version
docker run hello-world

# 5. 迁移镜像（可选）
# OrbStack 不共享 Docker Desktop 的镜像，需要重新拉取或导出导入
```

整个过程不到 5 分钟。我的 5 个服务 Compose 项目迁移后运行完全正常，没有任何兼容性问题。

---

## 说点得罪人的话

1. **Docker Desktop 的免费政策短期内不会变**，但如果你现在不准备退路，等它真收费那天你会很被动。

2. **不要无脑推荐 Podman 给所有人**。它确实"技术正确"，但配置复杂度会让很多开发者放弃。工具首先是给人用的，不是拿来炫技的。

3. **OrbStack 目前是最被低估的方案**。知道的人不多，但用过的人基本不会回去。我预测它两年内会被收购或者变成收费软件，趁现在免费赶紧用。

4. **Colima 适合你自己用，不适合团队推广。** 没有 GUI 这件事在团队协作里是硬伤。

---

## 总结

| 场景 | 推荐方案 |
|-----|---------|
| Mac 个人开发 | **OrbStack** |
| 企业迁移/安全优先 | **Podman** |
| 命令行极客 | **Colima** |
| K8s 开发 | **Rancher Desktop** |
| Windows 替代 | **Container Desktop** |

Docker Desktop 收费这件事，本质上是个**风险管理问题**。即使你现在免费，也该花 30 分钟测试一下替代方案，确认迁移成本。真到那天，你会感谢现在的自己。

---

*你目前在用哪个方案？欢迎在评论区分享你的迁移经历。*
