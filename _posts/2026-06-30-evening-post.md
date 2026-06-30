---
layout: post
title: "Evening Post"
date: 2026-06-30 12:55:00 +0800
categories: [教程]
---
# 一行命令找出项目里谁写了最多 bug

不是 git blame 那种逐行看，是直接统计——哪个作者提交的代码，后续被修复/重构的次数最多。

## 原理

git 的 `--pickaxe-regex` 可以搜索提交信息里的关键词。bug 修复通常带有 "fix"、"bug"、"修复" 等字样。我们把修复提交找出来，再反向 blame 到原始提交，就能算出每个作者的 "被修复率"。

## 命令

```bash
# 1. 找出所有修复类提交，提取被修改的文件
git log --all --pretty=format:"%H" --grep="fix\\|bug\\|修复\\|解决" | \
  while read commit; do
    git show --stat --oneline "$commit" | tail -n +2 | grep '|'
  done | awk '{print $1}' | sort | uniq -c | sort -rn | head -20

# 2. 更狠的：统计每个作者被修复的代码行数
git log --all -S"fix" --pretty=format:"%an" | sort | uniq -c | sort -rn
```

## 实际场景

上周我们团队用这招发现了一个反直觉的事：

- 提交量最高的同事，被修复率反而最低
- 一个平时不太起眼的同事，代码被后续修改的次数是最多的

深入一看，后者喜欢写 "刚好够用" 的实现，边界情况处理粗糙。不是态度问题，是习惯问题。

## 进阶：自动化报告

```bash
#!/bin/bash
# save as git-bug-stats.sh

echo "=== 作者修复率排行 ==="
git log --all --pretty=format:"%an" --grep="fix" | sort | uniq -c | sort -rn

echo -e "\n=== 最常被修复的文件 ==="
git log --all --name-only --pretty=format: --grep="fix" | sort | uniq -c | sort -rn | head -10

echo -e "\n=== 近30天修复热点 ==="
git log --since="30 days ago" --pretty=format:"%h %s" --grep="fix"
```

## 什么时候用

- 代码审查前快速摸底
- 团队 retro 用数据说话
- 新接手的项目，先看出哪块代码最 "烫手"

## 局限

- 依赖提交信息规范（fix: xxx 这种格式最准）
- 重构不一定是因为 bug，可能过度设计也会被修
- 只能看历史，不能预测未来

但比起 "我感觉这块代码不太对"，至少你能说 "这文件过去半年被修复了 17 次"。

---

*发布时间：2026-06-30*