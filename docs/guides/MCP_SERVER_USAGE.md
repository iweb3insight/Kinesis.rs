# MCP 服务器使用指南

**文档版本**: 1.0  
**创建日期**: 2026-03-13  
**功能**: AI Agent 集成接口

---

## 📖 概述

MCP (Model Context Protocol) 服务器提供基于 JSON-RPC 2.0 的标准接口，允许 AI Agent（如 Gemini、Claude 等）与 Kinesis.rs 进行交互。

---

## 🚀 快速开始

### 启动 MCP 服务器

```bash
# 启动 MCP stdio 服务器
./target/release/solana_claw_coin_cli mcp
```

服务器启动后，会通过 stdio 接收和发送 JSON-RPC 消息。

---

## 📡 协议格式

### JSON-RPC 2.0 消息格式

#### 请求格式

```json
{
  "jsonrpc": "2.0",
  "id": "request-id",
  "method": "method-name",
  "params": {}
}
```

#### 响应格式

```json
{
  "jsonrpc": "2.0",
  "id": "request-id",
  "result": {},
  "error": null
}
```

---

## 🔧 可用方法

### 1. initialize

初始化 MCP 连接。

**请求**:
```json
{
  "jsonrpc": "2.0",
  "id": "init-1",
  "method": "initialize",
  "params": {
    "protocol_version": "1.0",
    "client_info": {
      "name": "gemini-agent",
      "version": "1.0.0"
    }
  }
}
```

**响应**:
```json
{
  "jsonrpc": "2.0",
  "id": "init-1",
  "result": {
    "protocol_version": "1.0",
    "server_info": {
      "name": "kinesis-rs",
      "version": "0.7.0",
      "tier": "free"
    },
    "capabilities": {
      "tools": {
        "listChanged": true
      }
    }
  }
}
```

---

### 2. tools/list

列出所有可用工具。

**请求**:
```json
{
  "jsonrpc": "2.0",
  "id": "list-1",
  "method": "tools/list",
  "params": {}
}
```

**响应**:
```json
{
  "jsonrpc": "2.0",
  "id": "list-1",
  "result": {
    "tools": [
      {
        "name": "kinesis_buy",
        "description": "买入代币",
        "inputSchema": {
          "type": "object",
          "properties": {
            "token": {"type": "string", "description": "代币合约地址"},
            "amount": {"type": "number", "description": "买入金额"},
            "slippage": {"type": "number", "default": 15},
            "chain": {"type": "string", "enum": ["bsc", "solana"]}
          },
          "required": ["token", "amount"]
        }
      },
      {
        "name": "kinesis_sell",
        "description": "卖出代币"
      },
      {
        "name": "kinesis_quote",
        "description": "获取报价"
      },
      {
        "name": "kinesis_balance",
        "description": "查询余额"
      },
      {
        "name": "kinesis_detect",
        "description": "检测代币路径"
      },
      {
        "name": "kinesis_config",
        "description": "查看配置"
      },
      {
        "name": "kinesis_wallet",
        "description": "查看钱包地址"
      }
    ]
  }
}
```

---

### 3. tools/call

调用工具。

**请求示例 - 查询余额**:
```json
{
  "jsonrpc": "2.0",
  "id": "call-1",
  "method": "tools/call",
  "params": {
    "name": "kinesis_balance",
    "arguments": {
      "chain": "solana"
    }
  }
}
```

**响应**:
```json
{
  "jsonrpc": "2.0",
  "id": "call-1",
  "result": {
    "success": true,
    "data": {
      "balance": "3675360878",
      "balance_formatted": "3.675",
      "asset": "SOL",
      "owner": "88DqDXNAQZHWscK5HjPavDkBCvsfzmUrDvAV9ZTY5jMv"
    }
  }
}
```

---

**请求示例 - 买入代币**:
```json
{
  "jsonrpc": "2.0",
  "id": "call-2",
  "method": "tools/call",
  "params": {
    "name": "kinesis_buy",
    "arguments": {
      "token": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
      "amount": 0.1,
      "chain": "solana",
      "slippage": 15,
      "dry_run": true
    }
  }
}
```

**响应**:
```json
{
  "jsonrpc": "2.0",
  "id": "call-2",
  "result": {
    "success": true,
    "data": {
      "tx_hash": "SIMULATED",
      "amount_in": "0.1",
      "amount_out": "86788",
      "token": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
      "chain": "solana",
      "dry_run": true,
      "gas_estimate": "5000"
    }
  }
}
```

---

## ❌ 错误处理

### 错误响应格式

```json
{
  "jsonrpc": "2.0",
  "id": "call-1",
  "error": {
    "code": -32602,
    "message": "Invalid params",
    "data": {
      "reason": "Missing 'token' parameter"
    }
  }
}
```

### 常见错误码

| 错误码 | 说明 |
|--------|------|
| -32700 | Parse error |
| -32600 | Invalid Request |
| -32601 | Method not found |
| -32602 | Invalid params |
| -32603 | Internal error |
| -32000 | Unauthorized |
| -32003 | Pro feature required |
| -32100 | Insufficient funds |
| -32101 | Slippage exceeded |
| -32102 | Honeypot detected |
| -32103 | Transaction failed |

---

## 🔐 权限说明

### 免费版工具

以下工具在免费版中可用：
- kinesis_buy
- kinesis_sell
- kinesis_quote
- kinesis_balance
- kinesis_detect
- kinesis_config
- kinesis_wallet

### Pro 版工具 (未来)

以下工具将在 Pro 版中提供：
- kinesis_scan_risk (Honeypot 检测)
- kinesis_set_tp_sl (设置止盈止损)
- kinesis_notify_telegram (Telegram 通知)
- kinesis_notify_discord (Discord 通知)

---

## 💡 使用示例

### Python Agent 示例

```python
import subprocess
import json

# 启动 MCP 服务器
process = subprocess.Popen(
    ['./target/release/solana_claw_coin_cli', 'mcp'],
    stdin=subprocess.PIPE,
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE,
    text=True
)

def send_request(request):
    """发送 JSON-RPC 请求"""
    process.stdin.write(json.dumps(request) + '\n')
    process.stdin.flush()
    response = process.stdout.readline()
    return json.loads(response)

# 1. 初始化
init_request = {
    "jsonrpc": "2.0",
    "id": "1",
    "method": "initialize",
    "params": {
        "protocol_version": "1.0",
        "client_info": {"name": "my-agent", "version": "1.0.0"}
    }
}
response = send_request(init_request)
print("初始化完成:", response['result']['server_info'])

# 2. 列出工具
list_request = {
    "jsonrpc": "2.0",
    "id": "2",
    "method": "tools/list",
    "params": {}
}
response = send_request(list_request)
print("可用工具:", [t['name'] for t in response['result']['tools']])

# 3. 查询余额
balance_request = {
    "jsonrpc": "2.0",
    "id": "3",
    "method": "tools/call",
    "params": {
        "name": "kinesis_balance",
        "arguments": {"chain": "solana"}
    }
}
response = send_request(balance_request)
print("余额:", response['result']['data']['balance_formatted'], response['result']['data']['asset'])

# 4. 买入代币 (dry-run)
buy_request = {
    "jsonrpc": "2.0",
    "id": "4",
    "method": "tools/call",
    "params": {
        "name": "kinesis_buy",
        "arguments": {
            "token": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
            "amount": 0.1,
            "chain": "solana",
            "dry_run": True
        }
    }
}
response = send_request(buy_request)
if response['result']['success']:
    print("买入模拟成功:", response['result']['data'])
else:
    print("买入失败:", response['result'].get('error'))

# 关闭连接
process.terminate()
```

---

### Node.js Agent 示例

```javascript
const { spawn } = require('child_process');

// 启动 MCP 服务器
const mcp = spawn('./target/release/solana_claw_coin_cli', ['mcp']);

function sendRequest(request) {
    return new Promise((resolve) => {
        mcp.stdout.once('data', (data) => {
            resolve(JSON.parse(data.toString()));
        });
        mcp.stdin.write(JSON.stringify(request) + '\n');
    });
}

async function main() {
    // 1. 初始化
    const initResponse = await sendRequest({
        jsonrpc: '2.0',
        id: '1',
        method: 'initialize',
        params: {
            protocol_version: '1.0',
            client_info: { name: 'my-agent', version: '1.0.0' }
        }
    });
    console.log('初始化完成:', initResponse.result.server_info);

    // 2. 查询余额
    const balanceResponse = await sendRequest({
        jsonrpc: '2.0',
        id: '2',
        method: 'tools/call',
        params: {
            name: 'kinesis_balance',
            arguments: { chain: 'solana' }
        }
    });
    console.log('余额:', balanceResponse.result.data.balance_formatted);

    // 3. 买入代币
    const buyResponse = await sendRequest({
        jsonrpc: '2.0',
        id: '3',
        method: 'tools/call',
        params: {
            name: 'kinesis_buy',
            arguments: {
                token: 'EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v',
                amount: 0.1,
                chain: 'solana',
                dry_run: true
            }
        }
    });
    console.log('买入结果:', buyResponse.result);

    mcp.kill();
}

main();
```

---

## 📊 性能指标

| 指标 | 目标 | 实际 |
|------|------|------|
| 启动延迟 | <2 秒 | ~1 秒 |
| 工具调用延迟 | <3 秒 | ~1-2 秒 |
| 并发 QPS | 5+ | 5+ |
| 内存占用 | <500MB | ~200MB |

---

## 🔧 故障排除

### MCP 服务器无法启动

**检查点**:
1. 环境变量是否正确设置
2. .env 文件是否存在
3. RPC URL 是否可访问

```bash
# 检查配置
./target/release/solana_claw_coin_cli config

# 检查钱包
./target/release/solana_claw_coin_cli wallet
```

### 工具调用失败

**常见原因**:
1. 参数缺失或类型错误
2. RPC 连接问题
3. 余额不足

**解决方法**:
1. 检查错误消息中的详细信息
2. 使用 dry_run=true 先模拟
3. 检查日志输出

---

## 📚 相关文档

| 文档 | 说明 |
|------|------|
| [MCP_OPEN_SOURCE_FEATURES.md](docs/public/guides/MCP_OPEN_SOURCE_FEATURES.md) | MCP 开源版功能清单 |
| [MCP_DEV_TASKS_AND_ACCEPTANCE.md](docs/internal/MCP_DEV_TASKS_AND_ACCEPTANCE.md) | 开发任务与验收清单 |
| [POSITIONING_MANIFESTO.md](docs/internal/POSITIONING_MANIFESTO.md) | 产品定位宣言 |

---

**文档状态**: 已完成  
**版本**: 1.0  
**最后更新**: 2026-03-13  
**维护者**: Kinesis Team
