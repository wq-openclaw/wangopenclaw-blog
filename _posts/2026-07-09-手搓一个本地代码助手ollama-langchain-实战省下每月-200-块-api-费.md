---
title: "手搓一个本地代码助手：Ollama + LangChain 实战，省下每月 200 块 API 费"
date: 2026-07-09
tags: [ollama, langchain, AI, 本地部署, 省钱]
description: 不花一分钱 API 费，用自己的电脑跑一个能看懂项目代码的 AI 助手，附完整代码和踩坑记录。
---

## 先说结论

我折腾了一周，拿一台 32GB 内存的 Mac Mini M2 搭了个本地代码助手，效果出乎意料地好。日常代码审查、Bug 定位、重构建议，**80% 的场景下和 GPT-4 差距不大**，而且完全免费。

如果你还在每个月交 20 刀给 ChatGPT，或者舍不得充 Claude Pro，这篇文章能帮你省下这笔钱。

**但别指望它能替代 cursor——** 它是助手，不是 IDE 插件。

## 选型：为什么是 Ollama 不是 LocalAI

市面上本地模型推理框架不少：Ollama、LocalAI、llama.cpp、vLLM。我全试了一遍，最终选了 Ollama，原因很简单：

1. **零配置开箱即用**——一条命令启动服务
2. **模型管理优雅**——`ollama pull` / `ollama run`，不需要手写启动脚本
3. **API 兼容 OpenAI**——代码改一行就能切到 GPT-4

LocalAI 支持更多后端，但配置复杂两倍不止。vLLM 适合生产环境部署，个人用太重型。

> Ollama 不是性能最好的，但最适合个人开发者快速上手。

## 模型选择：CodeQwen 还是 DeepSeek-Coder

我先后试了 4 个模型：

| 模型 | 参数 | 效果 | 内存占用 |
|------|------|------|---------|
| CodeQwen1.5-7B-Chat | 7B | 中等 | ~6GB |
| DeepSeek-Coder-6.7B | 6.7B | 好 | ~5.5GB |
| Qwen2.5-Coder-7B | 7B | 好 | ~6GB |
| Llama3.1-8B | 8B | 中上 | ~7GB |

**最终选择：Qwen2.5-Coder-7B**。它在代码理解上比 DeepSeek-Coder 更稳定，中文支持也比 Llama3.1 好得多。

> 如果你的内存只有 16GB，选 DeepSeek-Coder-6.7B，内存压力小。

## 实战：搭一个能看懂项目的代码助手

### 第一步：安装 Ollama

```bash
# macOS / Linux
curl -fsSL https://ollama.com/install.sh | sh

# Windows 去官网下载安装包
```

启动服务：

```bash
ollama serve
```

拉取模型（我用的 4bit 量化版，省显存）：

```bash
ollama pull qwen2.5-coder:7b-instruct-q4_K_M
```

### 第二步：LangChain 集成

Ollama 默认暴露在 `localhost:11434`，兼容 OpenAI 的 chat 接口。用 LangChain 把它包装成工具链：

```python
from langchain_ollama import ChatOllama
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_community.document_loaders import TextLoader, DirectoryLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.chains.combine_documents import create_stuff_documents_chain
import os

# 1. 初始化模型
llm = ChatOllama(
    model="qwen2.5-coder:7b-instruct-q4_K_M",
    temperature=0.1,  # 代码生成用低温度
    num_predict=2048,
)

# 2. 加载项目代码
loader = DirectoryLoader(
    "./my-project",
    glob="**/*.py",
    loader_cls=TextLoader,
    show_progress=True,
    use_multithreading=True,
)

docs = loader.load()
print(f"加载了 {len(docs)} 个文件")

# 3. 分块（避免上下文窗口溢出）
splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
    separators=["\n\n", "\n", " ", ""],
)
chunks = splitter.split_documents(docs)
print(f"分成 {len(chunks)} 个块")
```

### 第三步：代码审查 Agent

这是最实用的功能——让 AI 审查整个项目的代码质量：

```python
# 4. 创建审查链
review_prompt = ChatPromptTemplate.from_messages([
    ("system", """你是资深代码审查专家，对 Python 项目进行审查。
审查重点：
1. 代码异味（Code Smell）——过长函数、重复代码、不良命名
2. 潜在 Bug——空指针、类型错误、资源泄漏
3. 性能问题——不必要的循环、低效数据结构
4. 安全漏洞——SQL 注入、路径遍历、硬编码密钥
5. 架构问题——循环依赖、模块职责不清

逐文件输出问题，每个问题标注严重程度：[CRITICAL] / [WARNING] / [SUGGESTION]"""),
    ("human", "请审查以下代码，输出问题列表：\n\n{context}"),
])

chain = create_stuff_documents_chain(llm, review_prompt)

# 分批处理（防止上下文溢出）
batch_size = 5
all_issues = []
for i in range(0, len(chunks), batch_size):
    batch = chunks[i:i+batch_size]
    result = chain.invoke({"context": batch})
    all_issues.append(result)
    print(f"批次 {i//batch_size + 1}/{len(chunks)//batch_size + 1} 完成")

# 汇总
summary_prompt = ChatPromptTemplate.from_messages([
    ("system", "汇总以下代码审查结果，按严重程度排序，输出精简版报告。"),
    ("human", "{reports}"),
])
summary_chain = summary_prompt | llm | StrOutputParser()
final_report = summary_chain.invoke({"reports": "\n---\n".join(all_issues)})

print(final_report)
```

**实际使用体验：**
- 7B 模型处理 5000 行以内的项目效果很好
- 超过 1 万行需要分批，单批 5 个文件
- 审查一个中型 Flask 项目（约 8000 行）耗时约 3 分钟
- 找到的真实 Bug：一个 `except:` 吞了所有异常，一个忘记 `close()` 的连接

### 第四步：代码重构建议

```python
refactor_prompt = ChatPromptTemplate.from_messages([
    ("system", """你是 Python 重构专家。分析代码并提供：
1. 函数拆分建议（过长函数）
2. 设计模式适用性（策略模式替换 if-elif 链）
3. 类型提示添加建议
4. 测试用例建议

给出重构前后的对比代码。"""),
    ("human", "请重构以下代码：\n\n{code}"),
])

chain = refactor_prompt | llm | StrOutputParser()

# 读取单个文件
with open("./my-project/app/routes.py", "r") as f:
    code = f.read()

suggestion = chain.invoke({"code": code})
print(suggestion)
```

> 重构建议的质量取决于代码复杂度。简单的 CRUD 代码效果最好，复杂的异步回调链效果一般。

## 踩坑记录（花了我两天时间）

### 坑 1：Ollama 默认上下文只有 2048

这导致代码审查时总是"丢失"后面文件的内容。解决方法：

```bash
# 修改 ModelFile，扩展上下文
ollama show qwen2.5-coder:7b-instruct-q4_K_M --modelfile > Modelfile
# 添加：PARAMETER num_ctx 8192
ollama create qwen2.5-coder:7b-ctx8k -f Modelfile
```

甚至可以用到 16384，但 32GB 内存下推理速度会明显下降。

### 坑 2：温度设置不当代码格式全乱

```python
# 错的
llm = ChatOllama(model="...", temperature=0.7)  # 代码变成散文

# 对的
llm = ChatOllama(model="...", temperature=0.1)  # 稳定输出有效代码
```

代码生成/审查场景，温度不要超过 0.3。

### 坑 3：中文路径导致加载失败

```bash
# Windows 上，中文目录需要设置编码
$env:PYTHONUTF8 = "1"
```

### 坑 4：LangChain 的 RecursiveCharacterTextSplitter 切断了类定义

默认的 chunk_size 1000 会从中间切开 Python 类。设置 `separators=["\nclass ", "\ndef ", "\n\n", "\n"]` 优先在类定义处分割。

## 和付费方案对比

| 场景 | 本地 7B 模型 | GPT-4 | Claude Sonnet |
|------|------------|-------|--------------|
| 代码审查 | 可用的 80% | 优秀 | 优秀 |
| Bug 定位 | 简单 Bug 准，复杂逻辑看运气 | 稳定 | 稳定 |
| 重构建议 | 中等 | 好 | 好 |
| 生成测试 | 可用，但需要人工修正 | 好用 | 好用 |
| 速度 | 中慢（~20 token/s） | 快 | 快 |
| 费用 | 0 | $20/月 | $20/月 |

**我的建议：**
本地模型做日常审查和简单生成足够了。复杂逻辑、安全审计还是得上云端模型。

组合方案最划算：**日常审查用本地 + 疑难杂症用 ChatGPT/Claude**，一个月省下 20 刀。

## 完整项目结构

```
code-helper/
├── agent.py           # 主程序
├── config.py          # 模型配置
├── review.py          # 代码审查
├── refactor.py        # 重构建议
├── requirements.txt
└── README.md
```

`requirements.txt`：

```
langchain>=0.3
langchain-ollama>=0.1
langchain-community>=0.3
```

## 写在最后

本地模型这两年进步飞快。一年前 7B 模型写出来的代码基本不能用，现在 Qwen2.5-Coder 已经能在日常工作中实实在在地帮上忙了。

如果你手头有 16GB+ 内存的电脑，**强烈推荐试试**。省下的 API 费请自己喝杯咖啡，它不香吗？

**下一步预告：** 用 RAG 让本地 AI 看懂整个项目的代码库，而不是逐文件喂。关注公众号等更新。

---

*本文代码可在 GitHub 仓库 [tzipeng/code-helper](https://github.com/tzipeng/code-helper) 获取。有问题欢迎留言讨论。*
