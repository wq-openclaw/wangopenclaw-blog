---
title: 别再给Agent做"提示词"了：用MoonBit + WASM打造真正能干活的自定义工具
date: 2026-07-09
description: 当Agent只会"回答问题"时，它只是个高级搜索。教你用WASM给Agent装上真工具。
tags: AI, Agent, WebAssembly, MoonBit, 工具开发
---

## 痛点：你的Agent为什么总是"纸上谈兵"？

先问一个扎心的问题：你开发的AI Agent，除了"回答问题"，还能干点啥？

如果你只是给Agent加了一堆提示词，让它"扮演"专家，那它本质上就是个带上下文的高级搜索引擎。遇到真实任务——比如"把这份PDF每页转成图片并压缩打包"——它就只会给你写个计划，然后告诉你"请手动执行以下步骤"。

这不是Agent，这是甩锅。

## 真正的Agent需要什么？

一个能真正干活的Agent，至少需要三样东西：

1. **感知能力**：能读文件、查数据库、调用API
2. **行动能力**：能写文件、发请求、操作系统
3. **决策能力**：能根据反馈调整策略

前两点，靠的是**工具（Tools）**，不是提示词。

最近 MoonBit 推出了 MoonBit Skills——一种用 WebAssembly 分发工具的新方式。这正好戳中了当前 Agent 开发的最大痛点：工具链碎片化、依赖难管理、跨平台适配麻烦。

## MoonBit Skills 解决了什么问题？

先说背景。MoonBit 是一个面向 WASM 的现代语言，语法类似 Rust 但更简洁，编译目标直接就是 WebAssembly。

MoonBit Skills 的思路很简单：**你写一个 MoonBit 函数，编译成 .wasm，然后 Agent 可以直接调用它**。不需要 Python 环境、不需要安装依赖、不需要 pip install 一堆包。

这样做的好处是：

- **安全隔离**：WASM 在沙箱里跑，不会搞崩宿主系统
- **零依赖部署**：一个 .wasm 文件就是全部，没有 node_modules 噩梦
- **跨语言**：你甚至可以用 C/Rust/Go 写代码，编译成 WASM 就行
- **热加载**：工具可以动态替换，Agent 不需要重启

## 手写一个真实工具

光说不练假把式。我们来写一个**URL 批量巡检工具**——用来检查一组 URL 是否可访问、响应时间、状态码。

### 1. 安装 MoonBit

```bash
# 安装 MoonBit CLI
curl -fsSL https://cli.moonbitlang.com/install.sh | bash

# 验证
moon version
```

### 2. 创建项目

```bash
moon new url-checker
cd url-checker
```

### 3. 写工具代码

打开 `src/main.mbt`，写入：

```rust
/// 检查单个 URL 的可访问性
/// 返回: { url: String, status: Int, latency_ms: Int, error: String? }
pub fn check_url(url: String) -> Map[String, Value] {
  let response = @http.get(url)
  let latency_ms = response.elapsed_ms()
  let result = {
    "url": url,
    "status": response.status_code(),
    "latency_ms": latency_ms,
    "error": response.error_message()
  }
  result
}

/// 批量检查 URL 列表
pub fn batch_check(urls: Array[String]) -> Array[Map[String, Value]] {
  urls.map(fn(u) { check_url(u) })
}

/// Agent 入口函数 - MoonBit Skills 协议标准接口
pub fn skill_execute(params: Map[String, Value]) -> Map[String, Value] {
  let urls = match params.get("urls") {
    Some(ArrayVal(arr)) => arr.filter_map(fn(v) {
      match v { StringVal(s) => Some(s) _ => None }
    })
    _ => { return { "error": "missing urls parameter" } }
  }
  
  let results = batch_check(urls)
  let failed = results.filter(fn(r) { r["status"] != 200 })
  let healthy = results.length() - failed.length()
  
  {
    "total": results.length(),
    "healthy": healthy,
    "failed": failed.length(),
    "details": results,
    "summary": "巡检完成: \(healthy)/\(results.length()) 正常"
  }
}
```

### 4. 编译 WASM

```bash
moon build --target wasm
# 输出: target/wasm/release/url-checker.wasm
```

### 5. 在 Agent 中注册

如果是 OpenAI Function Calling 风格：

```json
{
  "type": "function",
  "function": {
    "name": "url_checker",
    "description": "批量检查URL可访问性",
    "parameters": {
      "type": "object",
      "properties": {
        "urls": {
          "type": "array",
          "items": { "type": "string" },
          "description": "要检查的URL列表"
        }
      },
      "required": ["urls"]
    }
  }
}
```

Agent 调用时，加载 `url-checker.wasm` 执行，返回结构化数据，Agent 据此做决策。

## 这才是 Agent 的正确打开方式

有了真工具，Agent 就能做更多事情：

| 场景 | 工具 | 之前 | 之后 |
|------|------|------|------|
| 日志分析 | 日志解析 WASM | 告诉你"请查看日志" | 直接给出统计报告 |
| 数据清洗 | CSV 处理 WASM | 说"建议用 pandas" | 直接输出清洗后的数据 |
| 服务巡检 | HTTP 健康检查 WASM | 列 curl 命令 | 返回巡检表格 |
| 图片处理 | 压缩/裁剪 WASM | 推荐工具链接 | 直接处理完成 |

## 我的观点

1. **提示词工程快过时了。** 当模型能力越来越强，提示词能带来的边际收益越来越低。真正能拉开差距的，是 Agent 能调用的工具数量和质量。

2. **WASM 是 Agent 工具的天然载体。** 轻量、安全、跨平台、多语言——这些特性完美契合 Agent 工具的需求。MoonBit 选 WASM 作为目标是非常聪明的决策。

3. **MoonBit 还不错，但别急着梭哈。** 社区还很小，生态还在早期。可以先在边缘工具上试用，等成熟了再大规模迁移。现在学它主要是为了占位——等风口来了你已经在牌桌上了。

4. **警惕"套壳 Agent"。** 现在很多号称 Agent 的产品，实际就是把 GPT 包装一下加几个 API 调用。真正价值的 Agent 必须能执行**不受限制的计算和操作**，WASM 工具链是实现这个目标最务实的一条路。

## 下一步可以做什么

如果你对这条路感兴趣，可以：

1. 学一下 MoonBit 基础语法（半天就够了）
2. 把你日常工作中 Agent 做不了的那些事，写成 WASM 工具
3. 集成到你的 Agent 框架里（LangChain、AutoGPT、或者自己的框架）
4. 在公众号「小旺的技术笔记」后台回复「agent工具」获取更多示例代码

---

*关注公众号「小旺的技术笔记」，每天分享 AI 工程化的实战经验，不灌水、不画饼、只讲能用的代码。*
