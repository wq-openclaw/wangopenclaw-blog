---
title: "PostgreSQL 被 Rust 重写了？pgrust 通过 100% 回归测试，性能暴增 300 倍"
date: 2026-07-13
description: "pgrust 用 Rust 重写了 PostgreSQL，已通过 46,000+ 回归测试，分析型查询比原版快 300 倍。本文带你从零部署体验。"
tags: [postgresql, rust, pgrust, 数据库, 性能优化]
---

# PostgreSQL 被 Rust 重写了？pgrust 通过 100% 回归测试，性能暴增 300 倍

如果你关注数据库领域，今天 GitHub Trending 上最炸裂的项目就是 **pgrust**——一个用 Rust 从头重写 PostgreSQL 的项目，刚刚宣布 **100% 通过 PostgreSQL 官方回归测试**。

你没看错。46,000+ 条回归查询，全部通过。而且这不是玩具——它磁盘兼容 Postgres 18.3，可以直接挂载你现有的 Postgres 数据目录跑起来。

## 这不是"又一个 Rust 重写"

我知道你肯定在想：又来了，又是"用 Rust 重写 XXX"，然后烂尾了对吧？

这次不太一样。

作者 Michael Malisper 是前 Heap 工程师，管理过 **PB 级 Postgres 集群**，写过几十篇 Postgres 内部分析文章。他不是那种"Hello World 之后就开坑"的人。

更吓人的是进度：从开始到通过 67% 测试用了两周，再到 **100% 通过只用了再一周多**。整个代码库 50 万+ 行 Rust。怎么做到的？8 个 Codex 账号、20 个 AI Agent 并行开发，每天合并 280 个 PR。

## 性能到底多猛？

直接上数字，不废话：

| 场景 | 对比 Postgres |
|------|-------------|
| 事务型负载 | **快 50%** |
| 分析型负载 | **快 300 倍** |
| 对比 ClickHouse (clickbench) | 慢 2x，但作者认为可以反超 |

**为什么快？** 核心原因就一个——**多线程 vs 多进程**。

Postgres 用进程模型，主要是历史包袱。每个连接一个进程，进程间通信开销大，并行查询只有在数据量够大时才触发，因为拉起进程太贵了。

pgrust 用 **thread-per-connection** 模型，线程切换成本低得多，可以更激进地做并行化。在内存数据集上，单查询就能快 3 倍。

另一个例子是正则表达式。Postgres 的正则引擎太古老了，而 Rust 的正则引擎用了 SIMD 指令集加速，简单测试中 pgrust 快了 **10 倍**。

## 直接上手体验

别光看，这东西现在就能跑。最简单的就是 Docker：

```bash
docker run -d --name pgrust \
  -e POSTGRES_PASSWORD=secret \
  malisper/pgrust:v0.1

# 等启动完成
until docker exec -e PGPASSWORD=secret pgrust psql \
  -h 127.0.0.1 -U postgres -c '\q' >/dev/null 2>&1; do
  sleep 1
done

# 连接
docker exec -it -e PGPASSWORD=secret pgrust \
  psql -h 127.0.0.1 -U postgres
```

连上以后所有 SQL 照常跑，它看起来就是 Postgres：

```sql
SELECT version(), 1 + 1 as two;
```

macOS 本地编译：

```bash
brew install icu4c openssl@3 libpq

export LIBRARY_PATH="$(brew --prefix openssl@3)/lib:${LIBRARY_PATH:-}"
export PKG_CONFIG_PATH="$(brew --prefix openssl@3)/lib/pkgconfig:$(brew --prefix icu4c)/lib/pkgconfig:${PKG_CONFIG_PATH:-}"
export PATH="$(brew --prefix libpq)/bin:$PATH"

# 编译
PGRUST_PGSHAREDIR="$PWD/vendor/postgres-18.3/share" \
  cargo build --release --locked --bin postgres

# 初始化
target/release/postgres --initdb \
  -D /tmp/pgrust-data \
  -L "$PWD/vendor/postgres-18.3/share" \
  --no-locale --encoding UTF8 -U postgres

# 启动
ulimit -s 65520
RUST_MIN_STACK=33554432 target/release/postgres \
  -D /tmp/pgrust-data -F \
  -c listen_addresses= -k /tmp -p 5432 \
  -c io_method=sync -c max_stack_depth=60000
```

也可以直接去 [pgrust.com](https://pgrust.com) 在浏览器里跑 WASM 版本，预置了不少例子，甚至可以跑一个 Lisp 解释器。

## 为什么这对你有影响？

再好的项目，如果只是 "Star 一下" 也没意义。pgrust 的意义在于它正在解决 Postgres **真正的痛点**：

**1. 350+ 配置项，调参调到吐**

pgrust 的目标是 auto-tuning，你不需要知道 `shared_buffers` 和 `work_mem` 应该设多少。

**2. JSONB 是二等公民**

Postgres 对 JSONB 不做统计，查询计划全凭猜。pgrust 计划原生优化 JSON 工作负载。

**3. 连接池是必需品**

Postgres 进程模型决定了你必须要一个 PgBouncer 在 front，不然连接风暴直接打挂。pgrust 内置连接池。

**4. VACUUM 是很多 DBA 的噩梦**

pgrust 的实验方向包括 no-vacuum 存储设计。这可是每个运维过 Postgres 的人做梦都想要的。

**5. AI 生成的 SQL 越来越多了**

AI Agent 写的 SQL 经常又烂又危险。pgrust 计划加入运行时护栏，拦截糟糕查询。

## 泼点冷水：现在能用吗？

**不能。** 作者明确说了 not production-ready。

100% 回归测试通过 ≠ 生产可用。回归测试测的是"同样的输入给同样输出"，但真实场景下的并发、持久化、崩溃恢复才是硬骨头。作者也正在找人帮忙在 hobby project 上跑 replica 测试。

另外，**现有 Postgres 扩展不兼容**。PL/Python、PL/Perl 这些都用不了。部分 contrib 模块已移植，但远没到全面兼容。

License 是 **AGPL-3.0**，如果你在商业产品里用，要注意合规。

## 我的观点

pgrust 是目前看到最靠谱的 Postgres Rust 重写项目，原因有三：

1. **作者是真懂 Postgres 的**，不是凭热情硬写
2. **AI 辅助开发的模式太猛了**，8 个账号 20 个 Agent 并行开发，这种产能是传统开发方式无法想象的
3. **目标务实**——保持 Postgres 的行为兼容，但改了底层架构。不是"重新发明数据库"，而是"让 Postgres 更好改"

如果你在跑 Postgres 或者做数据库相关开发，**现在就应该关注 pgrust**，fork 一份代码看看架构，跑跑测试。等它稳定了，直接上车。

GitHub: [malisper/pgrust](https://github.com/malisper/pgrust)
Discord: [pgrust Discord](https://discord.gg/FZZ4dbdvwU)
WASM Demo: [pgrust.com](https://pgrust.com)
