---
title: "AI Coding Agent 实战测评：我为什么从 Cursor 换到了 Claude Code"
date: "2026-07-08"
description: "三个月深度使用后的真实对比，附完整配置文件和跨项目迁移经验"
tags: ["AI", "工具评测", "Claude Code", "Cursor", "效率"]
category: "工具评测"
---

## 为什么写这个

先说结论：**Cursor 依然优秀，但 Claude Code 更适合真正的工程团队。**

我不是要吹谁踩谁。这几个月我分别在两个项目上各用了两个月，踩了不少坑，也找到了一些"真香"的用法。这篇文章会讲清楚什么时候该用哪个，以及怎么配置才能不浪费钱。

## 先上配置对比

先看一张速览表（数据来自实际使用，非官方）：

| 维度 | Claude Code | Cursor | GitHub Copilot |
|------|-------------|--------|----------------|
| 订阅价格 | $20/月（需 Claude Pro） | $20/月 | $10/月（个人） |
| 上下文窗口 | 200K tokens | 变长，取决于模型 | 约 16K tokens |
| 代码理解深度 | 整个代码库 | 当前文件 + 索引 | 仅当前上下文 |
| 自动执行命令 | ✅ 原生支持 | ❌ 需手动 | ❌ |
| Git 操作 | ✅ 原生集成 | ⚠️ 基础 | ❌ |
| 非代码文件编辑 | ✅ Markdown/JSON/YAML | ❌ | ❌ |

## 我的真实使用体验

### 场景一：重构遗留代码

这是我的主力痛点。接手了一个有 5 年历史的 Python 后端，代码风格混杂，测试覆盖率不到 20%。

**Cursor 的表现：** 它擅长单文件重构。打开一个文件，描述需求，改得挺准。但涉及跨文件调用链的时候，它就搞不清楚了。你需要自己把相关文件一个个打开喂给它。

**Claude Code 的表现：** 直接在终端跑：

```bash
cd legacy-project
claude "分析 project/api/ 目录下的所有路由，找出未使用的 endpoints 并删除"
```

它自己会遍历整个目录、理解路由注册逻辑、检查调用栈、删除无用代码，然后直接跑测试验证结果不报错。你没看错，**它真的会自己跑测试**。

### 场景二：写单元测试

这是两家的强项，但思路完全不同。

**Cursor 的 Composer：** 你写好一个函数，按 Ctrl+K，输入"写单元测试"，它生成。你需要手动复制到测试文件，然后自己跑。

**Claude Code 的做法：**

```bash
claude "为 src/services/order_service.py 写完整单元测试，覆盖所有分支和异常路径，使用 pytest + mock，直接写入 tests/ 目录"
```

它一口气做了四件事：
1. 扫描整个 service 文件的依赖和调用关系
2. 推断需要 mock 的外部服务
3. 生成全面的测试用例（正常路径 + 边界 + 异常）
4. **直接写入测试文件然后运行测试**

如果测试挂了，它会自动修复。最多迭代 2-3 轮后通过。这省掉了我"复制-粘贴-运行-修复"的循环时间。

### 场景三：Debug 一个诡异的线上 Bug

有个 bug：某个 API 在高并发下概率性返回 502，压测复现不稳定。

**Cursor：** 我把相关代码贴给它，它分析了一通，猜测是数据库连接池耗尽。方向对，但不精确。

**Claude Code：**

```bash
claude "调查线上 502 问题，先从 Nginx 日志入手"
```

它自己读了 Nginx 日志 → 发现是 upstream timeout → 追查应用层超时配置 → 发现异步任务队列等待锁超时 → nginx 先于应用超时断开。**最终定位到是 Redis 锁的 TTL 设置不合理**，这个推理链本地 IDE 根本做不到，因为它需要跨 Nginx、应用代码、异步队列、Redis 四个层面联查。

## 花活功能：自动化工作流

Claude Code 最被低估的功能是它的**自动化能力**。你可以写一个 prompt 让它在后台做多步操作：

```bash
claude "
1. 读 CHANGELOG.md 找到最新版本号
2. 更新 package.json 版本号
3. 生成 release notes
4. 打 git tag
5. 推送到 origin
"
```

这在 Cursor 里需要手搓多个操作，Claude Code 一条命令搞定。CI 提效明显。

现在它还有一个叫做 **Claude Cowork** 的功能，可以让 AI 在云端后台跑任务，关掉笔记本它也不会停。今天（7月7日）刚更新了云端运行能力。

## 讲讲坑

### Claude Code 的坑

1. **过度自信。** 它偶尔会编造不存在的 API，尤其是一些冷门库。必须让它先读文档再写代码，或者加 `--verbose` 看每个步骤确认。

2. **成本问题。** 它的高上下文模式消耗 token 很快。做一个大重构，可能跑掉 $5-10 的额度。建议在 `claude.json` 里限制 max_tokens：

```json
{
  "maxTokensPerStep": 4000,
  "maxSteps": 50,
  "allowedTools": ["Read", "Edit", "Bash", "Git"]
}
```

3. **Windows 兼容性。** 原生 Windows 上用 claude 命令需要注意 shell 路径的问题，建议用 WSL。

### Cursor 的坑

Cursor 最大的问题是**外部集成弱**。它不会帮你跑命令、不会操作 Git、不会读系统日志。如果你主要在 IDE 里"对话式编码"，Cursor 体验更好。但一旦需要跟操作系统交互，它就歇菜了。

## 我的选型建议

- **个人项目 / 小团队** → Cursor。上手快，交互自然，写新代码体验丝滑。
- **中大型项目 / 需要重构和运维** → Claude Code + 你喜欢 IDE。比如我现在的配法是：VS Code 写代码 + Claude Code 做重构和自动化。
- **预算有限** → VS Code + Copilot + Claude Code（免费版够用，付费按需）。
- **团队协作** → 推荐在 CI 里集成 Claude Code 做自动 code review。远比人的 review 全面且省时。

## 配置模板

这是我的 `claude.json`（放在项目根目录），给你抄作业：

```json
{
  "projectName": "my-service",
  "ignoredFiles": ["*.min.js", "dist/", "node_modules/", "__pycache__/"],
  "maxTokensPerStep": 6000,
  "maxSteps": 100,
  "allowedTools": ["Read", "Edit", "Bash", "Git", "Search"],
  "bashHistory": true,
  "autoRunTests": true,
  "hooks": {
    "preEdit": "npx eslint --fix",
    "postEdit": "npm run typecheck"
  }
}
```

装完 Claude Code 后在项目根目录建这个文件，以后每次 `claude` 都会自动加载配置。

## 最后几句实话

AI Coding Agent 不是银弹。它们擅长的是**执行已知模式的编码任务**，但不擅长**设计系统架构**。别让 AI 帮你做架构决策——让它帮你写已经决定好的代码。

Cursor 更适合"探索式编程"（边写边想），Claude Code 更适合"生产式编程"（想好了让它写）。**两个都要有，别只用一个。**

**周末用半天时间把两个都配好，你接下来半年省的时间会远超这半天。**

---

*题外话：本来想写 Claude Code 的详细配置教程，但发现每人配置差异太大。如果你有具体场景的配置需求，留言告诉我，我单独写一篇。*

*我是小旺，每天在这里分享技术干货。如果你觉得有用，欢迎关注。*
