---
title: 一行命令找回丢失的 Git 提交？reflog 没你想的那么简单
date: 2026-07-06 18:00
tags: [git, 效率, 命令行]
---

前两天朋友一个误操作 `git reset --hard HEAD~3`，丢了三个 commit 的代码，差点心态崩了。

其实根本不用慌。Git 有个被低估的神器——`git reflog`，它记录了你所有 HEAD 移动的历史，**哪怕 commit 被 reset、rebase、甚至删了分支，只要还在 GC 周期内**，都能找回来。

---

## 核心命令

```bash
# 查看最近所有 HEAD 移动记录
git reflog

# 找到丢失的 commit hash 后，恢复
git checkout <lost-hash>

# 或者直接重置过去
git reset --hard HEAD@{2}
```

`git reflog` 默认保存 **90 天**内的操作记录，远超你的想象。

---

## 实操场景

### 场景 1：reset 之后后悔了

```bash
# 手滑了！想回到 reset 之前
git reflog
# 输出类似：
# abc1234 HEAD@{0}: reset: moving to HEAD~3
# def5678 HEAD@{1}: commit: 重要的提交 🚨

git reset --hard def5678
```

搞定，三个 commit 全部复活。

### 场景 2：rebase 搞砸了

```bash
# rebase 冲突太多想反悔
git rebase --abort  # 这是标准操作
# 但如果 rebase 完成了呢？
git reflog | grep rebase
git reset --hard HEAD@{5}
```

### 场景 3：删了分支才发现有用代码

```bash
# 分支被删，但 commit 还在 reflog 里
git reflog --all
# 找到 commit 后直接创建新分支
git branch recover-branch <hash>
```

**注意：`--all` 会显示包括 detached HEAD 在内的所有引用移动记录。**

---

## 进阶：reflog + bisect 组合拳

如果你的代码从某个时刻开始坏了，但不知道是哪个 commit 引入的：

```bash
# bisect 开始二分查找
git bisect start
git bisect bad        # 当前版本是坏的
git bisect good v1.0  # v1.0 是好的

# Git 自动切到中间 commit，你判断好坏后：
git bisect good       # 或 git bisect bad
# 重复几次，定位到罪魁祸首

# 完事清理
git bisect reset
```

结合 reflog，你可以在 bisect 过程中随时回到任何检查点：

```bash
git reflog -- bisect
```

---

## 一句话总结

> 在 Git 里，**只要你 commit 过，基本没有真正"丢"掉这件事**。reflog 就是你的后悔药专柜。

一行命令安利给团队：

```bash
alias git-undo="git reflog | head -20 && echo '找到 hash 后: git reset --hard <hash>'"
```
