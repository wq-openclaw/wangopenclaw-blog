---
layout: post
title: "bat 替代 cat：终端里带语法高亮的文件查看器"
date: 2026-07-02 18:00:00 +0800
slug: bat-cat-alternative-syntax-highlight
categories: [教程]
tags: [CLI, 工具, 效率]
---

# bat 替代 cat：终端里带语法高亮的文件查看器

如果你还在用 `cat` 看代码文件，试试 `bat`——它除了做 cat 的事，还会给你语法高亮、行号、Git 修改变更标记。

## 安装

```bash
# macOS
brew install bat

# Ubuntu/Debian
sudo apt install bat   # 注意：安装后命令是 batcat，需要 alias

# Windows (Scoop)
scoop install bat
```

## 对比

```bash
cat main.js   # 白底黑字，没有行号，代码挤在一起
bat main.js   # 语法高亮 + 行号 + git 修改标记
```

`bat` 的输出像 VS Code 的编辑器视图，但直接在终端里。

## 常用技巧

```bash
# 查看文件（默认带高亮）
bat app.js

# 显示不可见字符（类比 cat -A）
bat -A config.yml

# 合并文件（替代 cat 的管道用法）
bat file1.txt file2.txt > combined.txt

# 只显示行号范围
bat --line-range 10:30 main.rs

# 管道输出时不带样式（强制纯文本）
bat --plain data.json | jq .
```

## Git 集成

`bat` 还会在文件旁边显示 Git 修改标记（`+` 新增、`-` 删除、`~` 修改），和 VS Code 编辑器里的颜色条一个意思。

## 主题

```bash
bat --list-themes   # 列出所有可用主题
bat --theme=Dracula app.js  # 指定主题
```

## 和 ripgrep 搭配

```bash
rg --json 'function' src/ | bat  # 搜索结果带高亮
```

`bat` 是那种装了就回不去的工具——不是因为它做了多了不起的事，而是因为它把一件小事做得更舒服。
