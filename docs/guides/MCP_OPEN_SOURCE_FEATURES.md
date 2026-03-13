# Kinesis.rs MCP 开源版 (免费版) 功能清单

**文档版本**: 1.0  
**创建日期**: 2026-03-12  
**目标版本**: v0.7.0  
**状态**: 待实施

---

## 📋 目录

1. [核心目标](#核心目标)
2. [MCP 协议功能](#1-mcp-协议功能)
3. [基础工具 (7 个)](#2-基础工具)
4. [CLI 模式](#3-cli 模式)
5. [技术实现](#4-技术实现)
6. [实施计划](#5-实施计划)
7. [验收标准](#6-验收标准)

---

## 核心目标

### 定位

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   MCP 开源版 = 小散的自由交易工具                              │
│                                                                 │
│   免费 · 开源 · 高性能 · 易用                                   │
│                                                                 │
│   用户只需关注策略层                                            │
│   基础设施交给 Kinesis.rs                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 核心价值

| 价值 | 说明 | 目标 |
|------|------|------|
| **免费** | 完全免费使用 | 0 费用 |
| **开源** | 100% 源码透明 | GitHub 公开 |
| **高性能** | Rust 原生性能 | <1 秒执行 |
| **易用** | 一键安装启动 | curl \| bash |
| **灵活** | CLI + MCP 协议 | Agent 友好 |

---

## 1. MCP 协议功能

### 1.1 协议基础

| 功能 | 说明 | 状态 |
|------|------|------|
| **JSON-RPC 2.0** | 标准协议支持 | ⬜ 待实施 |
| **MCP 协议** | Model Context Protocol | ⬜ 待实施 |
| **stdio 传输** | CLI 模式 (标准输入输出) | ⬜ 待实施 |
| **工具发现** | tools/list 方法 | ⬜ 待实施 |
| **工具调用** | tools/call 方法 | ⬜ 待实施 |

### 1.2 不支持功能 (Pro 版独有)

| 功能 | 说明 |
|------|------|
| ❌ HTTP + SSE | 仅支持 stdio 传输 |
| ❌ 常驻模式 | 仅支持 CLI 按需启动 |
| ❌ 连接池 | 每次新建 RPC 连接 |
| ❌ Block 预处理 | 实时构造交易 |
| ❌ 内存缓存 | 无缓存机制 |

---

## 2. 基础工具

### 工具列表 (7 个)

| 工具 | 描述 | 输入参数 | 输出结果 |
|------|------|----------|----------|
| **kinesis_buy** | 买入交易 | token, amount, slippage, chain | tx_hash, amount_out |
| **kinesis_sell** | 卖出交易 | token, amount, slippage, chain | tx_hash, amount_out |
| **kinesis_quote** | 获取报价 | token, amount, action, chain | amount_out, path |
| **kinesis_balance** | 查询余额 | token (可选), chain | balance, asset |
| **kinesis_detect** | 路径检测 | token, chain | path, token_program |
| **kinesis_config** | 查看配置 | - | config (脱敏) |
| **kinesis_wallet** | 钱包地址 | wallet_index (可选) | bsc_address, sol_address |

---

### 2.1 kinesis_buy

#### 功能描述

买入代币 (支持 BSC 和 Solana)

#### 输入参数

```json
{
  "token": "string (required) - 代币合约地址",
  "amount": "number (required) - 买入金额 (SOL/BNB)",
  "slippage": "number (optional, default: 15) - 滑点容忍度 %",
  "chain": "string (optional, default: solana) - 链 (bsc/solana)",
  "dry_run": "boolean (optional, default: true) - 是否干跑"
}
```

#### 输出结果

```json
{
  "success": "boolean",
  "tx_hash": "string (optional)",
  "amount_out": "string (optional)",
  "gas_estimate": "number (optional)",
  "error": "object (optional)"
}
```

#### 使用示例

```bash
# CLI 模式
kinesis buy EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v 0.1 \
  --chain solana --slippage 15 --dry-run

# MCP 模式
{
  "method": "tools/call",
  "params": {
    "name": "kinesis_buy",
    "arguments": {
      "token": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
      "amount": 0.1,
      "chain": "solana"
    }
  }
}
```

---

### 2.2 kinesis_sell

#### 功能描述

卖出代币 (支持 BSC 和 Solana)

#### 输入参数

```json
{
  "token": "string (required) - 代币合约地址",
  "amount": "number (required) - 卖出数量",
  "slippage": "number (optional, default: 15) - 滑点容忍度 %",
  "chain": "string (optional, default: solana) - 链 (bsc/solana)",
  "dry_run": "boolean (optional, default: true) - 是否干跑"
}
```

#### 输出结果

```json
{
  "success": "boolean",
  "tx_hash": "string (optional)",
  "amount_out": "string (optional)",
  "gas_estimate": "number (optional)",
  "error": "object (optional)"
}
```

---

### 2.3 kinesis_quote

#### 功能描述

获取买入/卖出报价

#### 输入参数

```json
{
  "token": "string (required) - 代币合约地址",
  "amount": "number (required) - 数量",
  "action": "string (optional, default: buy) - 买卖方向 (buy/sell)",
  "chain": "string (optional, default: solana) - 链 (bsc/solana)"
}
```

#### 输出结果

```json
{
  "success": "boolean",
  "amount_out": "string",
  "path": "string (optional)",
  "price_impact": "number (optional)"
}
```

---

### 2.4 kinesis_balance

#### 功能描述

查询余额 (原生代币或 ERC20/SPL)

#### 输入参数

```json
{
  "token": "string (optional) - 代币合约地址 (空则查原生代币)",
  "chain": "string (optional, default: solana) - 链 (bsc/solana)"
}
```

#### 输出结果

```json
{
  "success": "boolean",
  "balance": "string",
  "balance_formatted": "string",
  "asset": "string",
  "owner": "string"
}
```

---

### 2.5 kinesis_detect

#### 功能描述

检测代币路径 (Solana 专用)

#### 输入参数

```json
{
  "token": "string (required) - 代币合约地址",
  "chain": "string (optional, default: solana) - 链 (仅支持 solana)"
}
```

#### 输出结果

```json
{
  "success": "boolean",
  "path": "string (Pump.fun/Raydium/RaydiumGraduated/Unknown)",
  "token_program_id": "string"
}
```

---

### 2.6 kinesis_config

#### 功能描述

查看当前配置 (脱敏)

#### 输入参数

```json
{}
```

#### 输出结果

```json
{
  "success": "boolean",
  "config": {
    "bsc_rpc_url": "string (脱敏)",
    "sol_rpc_url": "string (脱敏)",
    "wallet_index": "number"
  }
}
```

---

### 2.7 kinesis_wallet

#### 功能描述

查看钱包地址

#### 输入参数

```json
{
  "wallet_index": "number (optional, default: 1) - 钱包索引"
}
```

#### 输出结果

```json
{
  "success": "boolean",
  "bsc": "string (BSC 地址)",
  "solana": "string (Solana 地址)"
}
```

---

## 3. CLI 模式

### 3.1 安装方式

```bash
# 一键安装
curl -sL https://raw.githubusercontent.com/iweb3insight/Kinesis.rs/main/scripts/install.sh | bash

# 验证安装
kinesis --version
```

### 3.2 配置方式

```bash
# 创建 .env 文件
cat > .env << EOF
BSC_RPC_URL="https://bsc-dataseed.binance.org/"
SOL_RPC_URL="https://api.mainnet-beta.solana.com"
BSC_PRIVATE_KEY_1="your_bsc_private_key"
SOL_PRIVATE_KEY_1="your_sol_private_key"
EOF
```

### 3.3 使用示例

```bash
# 查询余额
kinesis balance --chain solana

# 获取报价
kinesis quote EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v 0.1 --action buy --chain solana

# 买入 (dry-run)
kinesis buy EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v 0.1 --chain solana --dry-run

# 卖出 (dry-run)
kinesis sell EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v 100 --chain solana --dry-run

# 检测路径
kinesis detect EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v --chain solana

# 查看配置
kinesis config

# 查看钱包
kinesis wallet
```

---

## 4. 技术实现

### 4.1 架构设计

```
┌─────────────────────────────────────────────────────────────────┐
│  MCP 开源版架构                                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────┐         ┌─────────────────┐               │
│  │   Agent Client  │         │   CLI Client    │               │
│  │   (Gemini/...)  │         │   (Command Line)│               │
│  └────────┬────────┘         └────────┬────────┘               │
│           │ MCP over stdio            │ CLI 命令                 │
│           ▼                           ▼                         │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   MCP Server                            │   │
│  │  ┌───────────────────────────────────────────────────┐  │   │
│  │  │  Protocol Layer                                   │  │   │
│  │  │  - JSON-RPC Parser                                │  │   │
│  │  │  - Request Router                                 │  │   │
│  │  │  - Response Formatter                             │  │   │
│  │  └───────────────────────────────────────────────────┘  │   │
│  │  ┌───────────────────────────────────────────────────┐  │   │
│  │  │  Tool Registry (7 tools)                          │  │   │
│  │  │  - kinesis_buy                                    │  │   │
│  │  │  - kinesis_sell                                   │  │   │
│  │  │  - kinesis_quote                                  │  │   │
│  │  │  - kinesis_balance                                │  │   │
│  │  │  - kinesis_detect                                │  │   │
│  │  │  - kinesis_config                                │  │   │
│  │  │  - kinesis_wallet                                │  │   │
│  │  └───────────────────────────────────────────────────┘  │   │
│  └─────────────────────────────────────────────────────────┘   │
│           │                                                     │
│           ▼                                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Core Services (共用)                                   │   │
│  │  - BSC Executor                                         │   │
│  │  - Solana Executor                                      │   │
│  │  - Path Detector                                        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 模块结构

```
src/
├── mcp/
│   ├── mod.rs              — MCP 模块入口
│   ├── protocol.rs         — JSON-RPC 解析
│   ├── tool_registry.rs    — 工具注册表 (7 个工具)
│   └── stdio_transport.rs  — stdio 传输层
├── cli.rs                  — CLI 参数解析
├── bsc/
│   └── executor.rs         — BSC 执行器
└── solana/
    ├── executor.rs         — Solana 执行器
    └── detector.rs         — 路径检测
```

### 4.3 依赖项

```toml
[dependencies]
# MCP 协议
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"

# CLI
clap = { version = "4", features = ["derive"] }

# 异步运行时
tokio = { version = "1", features = ["full"] }

# BSC
alloy = { version = "0.8.3", features = ["full"] }

# Solana
solana-sdk = "4.0.1"
spl-associated-token-account = "8.0.0"
borsh = { version = "1.6.0", features = ["derive"] }

# 网络
reqwest = { version = "0.12", features = ["json"] }

# 错误处理
thiserror = "2.0"
anyhow = "1.0"

# 日志
tracing = "0.1"
tracing-subscriber = "0.3"
```

---

## 5. 实施计划

### Phase 1: MCP 基础框架 (W1-W2)

| 任务 | 预计工时 | 交付物 | 状态 |
|------|----------|--------|------|
| JSON-RPC 解析器 | 2d | `src/mcp/protocol.rs` | ⬜ |
| stdio 传输层 | 2d | `src/mcp/stdio_transport.rs` | ⬜ |
| 工具注册表框架 | 2d | `src/mcp/tool_registry.rs` | ⬜ |
| MCP 模块集成 | 2d | `src/mcp/mod.rs` | ⬜ |

**里程碑**: MCP 基础框架完成 ✅

---

### Phase 2: 基础工具实现 (W3-W4)

| 任务 | 预计工时 | 交付物 | 状态 |
|------|----------|--------|------|
| kinesis_buy | 2d | 买入工具 | ⬜ |
| kinesis_sell | 2d | 卖出工具 | ⬜ |
| kinesis_quote | 1d | 报价工具 | ⬜ |
| kinesis_balance | 1d | 余额工具 | ⬜ |
| kinesis_detect | 1d | 路径检测工具 | ⬜ |
| kinesis_config | 0.5d | 配置工具 | ⬜ |
| kinesis_wallet | 0.5d | 钱包工具 | ⬜ |

**里程碑**: 7 个基础工具完成 ✅

---

### Phase 3: CLI 集成与测试 (W5)

| 任务 | 预计工时 | 交付物 | 状态 |
|------|----------|--------|------|
| CLI 命令集成 | 2d | `src/cli.rs` | ⬜ |
| 单元测试 | 2d | 测试用例 | ⬜ |
| 集成测试 | 1d | E2E 测试 | ⬜ |

**里程碑**: CLI 集成完成 ✅

---

### Phase 4: 文档与发布 (W6)

| 任务 | 预计工时 | 交付物 | 状态 |
|------|----------|--------|------|
| API 文档 | 1d | MCP API Reference | ⬜ |
| 使用指南 | 1d | 用户手册 | ⬜ |
| 发布准备 | 1d | v0.7.0 Release | ⬜ |

**里程碑**: v0.7.0 正式发布 🎉

---

## 6. 验收标准

### 功能验收

| 功能 | 验收标准 | 状态 |
|------|----------|------|
| **MCP 协议** | 支持 JSON-RPC 2.0 | ⬜ |
| **stdio 传输** | CLI 模式正常工作 | ⬜ |
| **工具发现** | tools/list 返回 7 个工具 | ⬜ |
| **工具调用** | 7 个工具均可调用 | ⬜ |
| **CLI 命令** | 7 个 CLI 命令正常工作 | ⬜ |

### 性能验收

| 指标 | 目标 | 状态 |
|------|------|------|
| **启动延迟** | <2 秒 | ⬜ |
| **执行延迟** | <3 秒 (dry-run) | ⬜ |
| **并发 QPS** | 5+ | ⬜ |

### 质量验收

| 指标 | 目标 | 状态 |
|------|------|------|
| **单元测试覆盖率** | >70% | ⬜ |
| **集成测试通过率** | 100% | ⬜ |
| **文档完整性** | 100% | ⬜ |

---

## 7. 与 Pro 版对比

| 功能 | 开源版 (免费) | Pro 版 ($49/月) |
|------|--------------|----------------|
| **MCP 模式** | stdio (CLI) | HTTP + SSE (常驻) |
| **启动延迟** | 500-2000ms | <10ms |
| **执行延迟** | 1-3 秒 | <100ms |
| **工具数量** | 7 个 | 13 个 (+6 个 Pro) |
| **Pro 工具** | ❌ | ✅ Honeypot/Auto TP/SL/通知 |
| **连接池** | ❌ | ✅ RPC 长连接 |
| **Block 预处理** | ❌ | ✅ 预构造交易 |
| **并发 QPS** | 5 | 50+ |
| **监控代币** | 10 | 1000+ |
| **钱包数量** | 1 | 5 |

---

## 8. 未来扩展

### 可选功能 (社区贡献)

| 功能 | 优先级 | 说明 |
|------|--------|------|
| **GUI 插件** | 🟢 低 | Chrome 侧边栏交易终端 |
| **密码锁** | 🟢 低 | AES-GCM 加密 + 超时锁定 |
| **暗色主题** | 🟢 低 | CLI 颜色主题切换 |
| **更多行情源** | 🟡 中 | Birdeye/DexTools 集成 |

### Pro 版升级路径

```
开源版用户 → Pro 版升级

升级理由:
- MCP 常驻模式 (快 100-200 倍)
- Honeypot 检测 (避免资金损失)
- Auto TP/SL (自动止盈止损)
- Telegram/Discord 通知
- 多钱包支持 (5 个)

价格: $49/月 或 $499/年
```

---

## 📚 相关文档

| 文档 | 说明 |
|------|------|
| [MCP_TECHNICAL_DESIGN.md](MCP_TECHNICAL_DESIGN.md) | MCP 技术设计总览 |
| [MCP_DETAILED_SPEC.md](MCP_DETAILED_SPEC.md) | MCP 详细技术规范 |
| [MCP_PRO_DESIGN.md](MCP_PRO_DESIGN.md) | Pro 版设计 |
| [POSITIONING_MANIFESTO.md](POSITIONING_MANIFESTO.md) | 产品定位宣言 |

---

**文档状态**: 已完成  
**版本**: 1.0  
**最后更新**: 2026-03-12  
**维护者**: Kinesis Team CTO  
**下一步**: 团队评审 → 开始实施 (Phase 1)
