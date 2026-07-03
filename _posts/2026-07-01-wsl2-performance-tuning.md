---
layout: post
title: "WSL 2 性能调优：从装好用到飞起"
date: 2026-07-01 08:00:00 +0800
slug: wsl2-performance-tuning
categories: [教程]
tags: [WSL2, Linux, Windows, 性能]
---

# WSL 2 性能调优：从装好用到飞起

WSL 2 装好就能用，但默认配置离"好用"还有一段距离。这几个调优步骤，能让你的体验从"能用"变成"真香"。

## 1. 限制内存占用（最关键）

WSL 2 的默认行为是：**宿主机有多少内存，它就敢用多少**。编译大型项目时，WSL 2 很容易把内存吃到 80% 以上，然后 Windows 开始卡。

修复方法：在 `%UserProfile%\.wslconfig` 里设置内存上限：

```ini
[wsl2]
memory=8GB
processors=4
swap=4GB
localhostForwarding=true
```

> ⚠️ `.wslconfig` 只影响 WSL 2 虚拟机级别的配置，需要对每个发行版单独生效。改完后运行 `wsl --shutdown` 重启 WSL。

**建议值：** 内存设为宿主机的一半。16GB 机器设 8GB，32GB 设 16GB。

## 2. 把项目搬到 WSL 2 文件系统

这是最大的坑：

```
❌ Windows 访问 /mnt/c/Users/xxx/project 慢 5-10 倍
✅ Linux 访问 ~/project 和原生 Linux 一样快
```

WSL 2 的 `/mnt/c` 走的是 `9p` 协议（网络文件系统），IO 性能远不如 ext4 原生分区。如果你在 Windows 上 clone 了代码，然后在 WSL 里编译——恭喜你，吃到了最慢的路径组合。

**正确做法：** 所有代码放到 WSL 内部（`~/projects/xxx`），在 WSL 里 clone、编译、运行。用 VS Code 的 Remote-WSL 插件连接进来编辑。

实测对比（编译一个 5 万行 C++ 项目）：

| 位置 | 首次编译 | 增量编译 |
|------|---------|---------|
| `/mnt/c/Users/xxx/project` | 4分20秒 | 52秒 |
| `~/project` | 1分05秒 | 11秒 |

差距接近 4-5 倍。

## 3. 配置 Docker 与 WSL 2 集成

Docker Desktop for Windows 默认使用 Hyper-V 虚拟机，资源占用高。更好的方式是用 WSL 2 后端：

1. Docker Desktop → Settings → General → 勾选 "Use WSL 2 based engine"
2. Resources → WSL Integration → 启用你的发行版

然后在 WSL 里直接：

```bash
docker ps  # 和原生 Linux 体验完全一致
```

更进一步，你可以完全抛弃 Docker Desktop，在 WSL 2 里安装 Docker CE：

```bash
# 在 WSL 2 里直接装 Docker
sudo apt update
sudo apt install docker.io
sudo service docker start
sudo usermod -aG docker $USER
```

这样省掉了 Docker Desktop 的 GUI 进程，大约节省 300-500MB 内存。

## 4. 配置 systemd（Ubuntu 22.04+ 默认开启）

老版本 WSL 没有 systemd，很多 Linux 工具（snap、systemctl）用不了。开启方法：

在 `/etc/wsl.conf` 中：

```ini
[boot]
systemd=true
```

然后 `wsl --shutdown` 重启。

## 5. 网络代理配置

在 WSL 里访问外网不走 Windows 的代理，需要手动配：

```bash
# 在 ~/.bashrc 或 ~/.zshrc 中
export host_ip=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}')
export ALL_PROXY=http://$host_ip:7890
export HTTP_PROXY=http://$host_ip:7890
export HTTPS_PROXY=http://$host_ip:7890
export NO_PROXY=localhost,127.0.0.1
```

端口号看你用的代理软件（Clash 默认 7890，v2ray 默认 10809）。

## 6. 硬盘碎片整理

WSL 2 的 ext4 虚拟硬盘（`ext4.vhdx`）不会自动收缩，用久了文件删了磁盘空间也不会还给 Windows。

手动压缩：

```powershell
# PowerShell (管理员)
wsl --shutdown
diskpart
# 在 diskpart 中：
select vdisk file="C:\Users\<你的用户名>\AppData\Local\Packages\...\LocalState\ext4.vhdx"
attach vdisk readonly
compact vdisk
detach vdisk
exit
```

找不到 vhdx 路径？在 PowerShell 里：

```powershell
Get-ChildItem -Recurse -Filter "ext4.vhdx" ~\AppData\Local\Packages\
```

## 总结

| 优化项 | 提升效果 | 难度 |
|--------|---------|------|
| 限制内存 | 防止 Windows 卡顿 | ⭐ |
| 项目搬到 ext4 | 编译快 4-5 倍 | ⭐ |
| Docker WSL 2 集成 | 省内存，体验一致 | ⭐⭐ |
| 开启 systemd | 兼容更多 Linux 工具 | ⭐ |
| 代理配置 | 翻墙不折腾 | ⭐⭐ |
| vhdx 压缩 | 回收磁盘空间 | ⭐⭐⭐ |

WSL 2 的核心优势是让你在 Windows 上获得接近原生 Linux 的开发体验。但这不等于装完就不用管了——这几个配置调一下，天差地别。
