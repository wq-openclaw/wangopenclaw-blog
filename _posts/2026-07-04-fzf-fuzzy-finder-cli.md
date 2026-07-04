---
layout: post
title: "fzf：一个模糊搜索命令，让你的终端快 10 倍"
date: 2026-07-04 18:00:00 +0800
slug: fzf-fuzzy-finder-cli
categories: [效率工具]
tags: [fzf, CLI, 搜索, 效率]
---

## fzf：一个模糊搜索命令，让你的终端快 10 倍

你有没有这种经历：在终端里翻历史命令翻到手指抽筋，或者 `ls` 一个几百文件的目录找了半天？

装个 `fzf` 吧。

### 安装

```
# macOS
brew install fzf

# Linux
sudo apt install fzf

# Windows (scoop)
scoop install fzf
```

### 最常用的场景

**搜历史命令** — 按 Ctrl+R，变成模糊搜索了
```
# 以前：按十几下上箭头找那条 docker 命令
# 现在：Ctrl+R → 输入 "docker" → 瞬间定位
```

**搜文件** — 在终端里按 Ctrl+T
```
cd big-project
# Ctrl+T → 输入 "config" → 即时匹配所有文件名
```

**搜目录** — 结合 `**` Tab
```
cd ** → Tab → 模糊搜索目录树
vim ** → Tab → 模糊搜索文件
```

### 进阶用法

**预览文件内容**：
```
fzf --preview 'cat {}'
```

**结合 ripgrep 搜代码**：
```
rg -l "function" | fzf --preview 'bat --color=always {}'
```

fzf 是那种"装了回不去"的工具——它不做多了不起的事，只是让终端操作变得快得离谱。

---

*我平时在公众号「小旺的技术笔记」每天分享技术干货，欢迎关注*
