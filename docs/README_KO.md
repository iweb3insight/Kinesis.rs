# Kinesis.rs v0.6.0

> "Kinesis: 시장의 정서를 감지하고, 온체인의 생명력을 밝히다."

Kinesis.rs는 LLM 에이전트를 위해 설계된 상태 비저장(stateless), JSON 우선, 멀티체인 암호화폐 거래 실행 레이어입니다.

## 보안 아키텍처

Kinesis.rs는 개인 키 관리 및 온체인 상호작용에 대해 "제로 트러스트" 접근 방식을 채택합니다.

### 1. 개인 키 보호
- **서버 업로드 없음**: 개인 키는 서버나 클라우드에 절대 업로드되지 않으며, 로컬 환경 변수에서만 로드되어 프로세스 내에서만 사용됩니다.
- **실행 컨텍스트 격리**: 개인 키 명문은 로컬 바이너리 프로세스를 벗어나지 않습니다. 서명은 메모리 내 보호된 영역에서만 수행되어 에이전트 프롬프트나 외부 로그로의 유출을 방지합니다.
- **로컬 암호화 표준**: 로컬 저장 시 PBKDF2 + AES-256-GCM 암호화 사용을 권장합니다.
- **자동 잠금 메커니즘**: 타임아웃을 지원하여 복호화된 키의 메모리 노출 시간을 최소화합니다.

### 2. 온체인 보호 장치
- **거래 기한 설정**: 모든 BSC `buy` 및 `sell` 작업에는 Deadline이 강제 포함되어 오래된 가격으로의 체결을 방지합니다.
- **동적 승인 대상**: `approveTarget`은 검증된 컨트랙트에서 직접 가져오며, 잘못된 주소로의 권한 부여 리스크를 제거합니다.
- **투명성 및 감사**: 모든 코드는 오픈 소스로 공개되며, FreedomRouter는 BscScan에서 검증되었습니다.

## 주요 기능
- **멀티체인 지원**: BNB 스마트 체인(BSC) 및 Solana 기본 실행.
- **에이전트 우선**: JSON 우선 통신 프로토콜.
- **고성능**: 병렬 RPC 레이싱 및 트랜잭션 사전 구축.
- **전 플랫폼 지원**: Linux, macOS, Windows용 사전 컴파일된 바이너리 제공.
- **스마트 라우팅**: Pump.fun 및 Raydium V3 자동 감지 및 실행.

## 시작하기
1. 저장소를 클론합니다.
2. `.env.example`을 `.env`로 복사하고 키를 추가합니다.
3. 빌드: `cargo build --release`
4. 실행: `./target/release/kinesis-rs balance --chain solana`

## CLI 사용법

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

## 후원
Kinesis.rs는 실전 환경에서의 지속적인 검증을 위해 후원을 환영합니다:
- **SOL**: `UFePGDrDS8xmutWkLKKGfgKUvacvLLSyQZ66AacKYUZ`
- **BNB/ETH**: `0x1580b9604c47Dbef3A61ae5a3deFF7f6611f3C28`

## 문제 해결
- **macOS**: `killed` 오류 발생 시 [macOS 문제 해결 가이드](../docs/TROUBLESHOOTING_MACOS.md)를 참조하세요.
