# Kinesis.rs v0.6.0

> "Kinesis：市場の情緒を感知し、オンチェーンの生命を灯す。"

Kinesis.rs は、主に LLM エージェント向けに設計された、ステートレスで JSON ファーストなマルチチェーン暗号資産取引実行レイヤーです。

## セキュリティアーキテクチャ

Kinesis.rs は、秘密鍵の管理とオンチェーンのインタラクションに対して「ゼロトラスト」アプローチを採用しています。

### 1. 秘密鍵の保護
- **サーバーへのアップロードなし**: 秘密鍵がサーバーやクラウドにアップロードされることはありません。ローカルの環境変数からのみ読み込まれ、ローカルプロセス内でのみ使用されます。
- **実行コンテキストの分離**: 秘密鍵の明文がローカルバイナリプロセス外に出ることはありません。署名はメモリ内の保護された領域で行われ、エージェントのプロンプトや外部ログへの漏洩を防止します。
- **ローカル暗号化基準**: ローカル環境での保存には PBKDF2 + AES-256-GCM の使用を推奨します。
- **自動ロック機能**: タイムアウトによるメモリ内の解読済みキーの露出時間を最小限に抑えます。

### 2. オンチェーン保護
- **トランザクション期限**: BSC 側の `buy` と `sell` には強制的な Deadline が含まれ、古い価格での執行を防止します。
- **動的な承認先**: `approveTarget` は検証済みコントラクトから直接取得され、不正アドレスへの承認リスクを排除します。
- **透明性と監査可能性**: コードは完全にオープンソースであり、FreedomRouter は BscScan で検証済みです。

## 特徴
- **マルチチェーン対応**: BNB スマートチェーン (BSC) および Solana のネイティブ実行。
- **エージェントファースト**: JSON ファーストの通信プロトコル。
- **ハイパフォーマンス**: 並列 RPC レーシングとトランザクション事前構築。
- **全プラットフォーム対応**: Linux, macOS, Windows 用のビルド済みバイナリを提供。
- **スマートルーティング**: Pump.fun および Raydium V3 の自動検出と実行。

## クイックスタート
1. リポジトリをクローンします。
2. `.env.example` を `.env` にコピーし、秘密鍵を追加します。
3. ビルド: `cargo build --release`
4. 実行: `./target/release/kinesis-rs balance --chain solana`

## CLI 使用法

```text
Usage: kinesis-rs [OPTIONS] <COMMAND>

Commands:
  buy      Buy a token on a supported chain
  sell     Sell a token on a supported chain
  quote    Get a quote for a trade
  balance  Check balance of native or token
  approve  Approve a token for trading
  config   Display current configuration
  wallet   Display wallet address
  detect   Detect Solana token path (Pump.fun or Raydium)
  help     Print this message or the help of the given subcommand(s)

Options:
      --json             Global flag to force JSON output for agent consumption
      --dry-run          Global flag to simulate transactions without sending them
      --no-dry-run       Global flag to disable simulation and send the real transaction
      --wallet <WALLET>  Selects the wallet to use based on environment variable suffix (e.g., _1, _2) [default: 1]
  -h, --help             Print help (see more with '--help')
  -V, --version          Print version
```

## 寄付
Kinesis.rs は、実環境での継続的なテストのために寄付を歓迎しています：
- **SOL**: `UFePGDrDS8xmutWkLKKGfgKUvacvLLSyQZ66AacKYUZ`
- **BNB/ETH**: `0x1580b9604c47Dbef3A61ae5a3deFF7f6611f3C28`

## トラブルシューティング
- **macOS**: `killed` と表示される場合は、[macOS トラブルシューティングガイド](../docs/TROUBLESHOOTING_MACOS.md)を参照してください。
