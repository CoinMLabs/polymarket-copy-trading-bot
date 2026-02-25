# Polymarket Copy Trading Bot (Rust)

[![Rust](https://img.shields.io/badge/rust-1.70%2B-orange.svg)](https://www.rust-lang.org/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

A high-performance Rust implementation of a copy trading bot for [Polymarket](https://polymarket.com/). This bot automatically mirrors trades from selected traders to your wallet in real-time using WebSocket connections for minimal latency.

<img width="1545" height="911" alt="image (15)" src="https://github.com/user-attachments/assets/2adbd321-d8b7-453a-be84-c80b3083dca5" />
<img width="1280" height="772" alt="image (17)" src="https://github.com/user-attachments/assets/15eb7771-5c59-4c39-8054-36e7530fb253" />
<img width="1464" height="941" alt="image (16)" src="https://github.com/user-attachments/assets/127699f6-05e1-44f2-9c47-7140e559b43e" />

## 🚀 Features

### Lightning-fast identical copying: same entry price + exact share size for perfect mirroring

<img width="1272" height="281" alt="image (18)" src="https://github.com/user-attachments/assets/bd0d3496-3e6f-4bb3-8bb0-58a77f7f8f51" />
![photo_2026-02-26_09-02-09](https://github.com/user-attachments/assets/74f79c9c-c0ea-4a4e-85b1-6b27fcc55c21)

### Real-Time Trade Execution
- **WebSocket-Based Monitoring**: Connects to Polymarket's Real-Time Data Stream (RTDS) for instant trade detection
- **Zero Database Overhead**: Executes trades immediately upon detection without requiring MongoDB
- **Low Latency**: Direct WebSocket connection ensures minimal delay between trader action and your execution

### Advanced Copy Strategies
- **Percentage Strategy**: Copy a fixed percentage of each trader's position size
- **Fixed Strategy**: Execute trades with a fixed USD amount regardless of trader's position
- **Adaptive Strategy**: Dynamically adjust copy percentage based on trade size with configurable thresholds
- **Tiered Multipliers**: Apply different multipliers based on trade size ranges
- **Position Limits**: Set maximum position sizes and daily volume limits for risk management

### Risk Management
- **Balance Protection**: Automatically checks available USDC balance before executing trades
- **Order Size Limits**: Configurable minimum and maximum order sizes
- **Position Tracking**: Monitors your current positions to prevent over-exposure
- **Error Handling**: Robust retry logic and graceful error recovery

### Production Ready
- **Health Checks**: Built-in system health monitoring
- **Comprehensive Logging**: Detailed logs for debugging and monitoring
- **Configuration Validation**: Validates environment setup before execution
- **Graceful Shutdown**: Handles interrupts and cleanup properly

## 📋 Requirements

- **Rust**: Version 1.70 or higher
- **Polygon Network Access**: RPC endpoint for Polygon mainnet
- **USDC Contract**: Polygon USDC contract address
- **Polymarket Account**: Valid wallet with USDC balance
- **CLOB API Access**: Polymarket CLOB HTTP and WebSocket URLs

## 🛠️ Installation

### Prerequisites

1. Install Rust (if not already installed):
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

2. Verify installation:
```bash
rustc --version  # Should be 1.70 or higher
```

### Build from Source

```bash
# Clone the repository
git clone <repository-url>
cd polymarket-copy-trader-rust

# Build in release mode
cargo build --release

# The binary will be in target/release/polymarket-copy-rust
```

## ⚙️ Configuration

### Environment Variables

Create a `.env` file in the project root with the following required variables:

```env
# Trader addresses to copy (comma-separated or JSON array)
USER_ADDRESSES=0x1234...,0x5678...

# Your wallet address (proxy wallet for executing trades)
PROXY_WALLET=0xYourWalletAddress

# Private key
PRIVATE_KEY=your_private_key_hex

# Polymarket CLOB API endpoints
CLOB_HTTP_URL=https://clob.polymarket.com
CLOB_WS_URL=wss://clob.polymarket.com

# Polygon RPC endpoint
RPC_URL=https://polygon-rpc.com

# USDC contract address on Polygon
USDC_CONTRACT_ADDRESS=0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174
```

### Copy Strategy Configuration

#### Percentage Strategy
```env
COPY_STRATEGY=PERCENTAGE
COPY_SIZE=10.0  # Copy 10% of trader's position
MAX_ORDER_SIZE_USD=100.0
MIN_ORDER_SIZE_USD=1.0
```

#### Fixed Strategy
```env
COPY_STRATEGY=FIXED
COPY_SIZE=50.0  # Always copy $50 worth
MAX_ORDER_SIZE_USD=100.0
MIN_ORDER_SIZE_USD=1.0
```

#### Adaptive Strategy
```env
COPY_STRATEGY=ADAPTIVE
COPY_SIZE=10.0
ADAPTIVE_MIN_PERCENT=5.0
ADAPTIVE_MAX_PERCENT=20.0
ADAPTIVE_THRESHOLD_USD=500.0
```

#### Advanced Options
```env
# Position limits
MAX_POSITION_SIZE_USD=1000.0
MAX_DAILY_VOLUME_USD=5000.0

# Trade multiplier
TRADE_MULTIPLIER=1.5

# Tiered multipliers (JSON format)
TIERED_MULTIPLIERS=[{"min":0,"max":100,"multiplier":1.0},{"min":100,"max":500,"multiplier":1.5}]

# Network settings
REQUEST_TIMEOUT_MS=10000
NETWORK_RETRY_LIMIT=3
RETRY_LIMIT=3
```

## 🎯 Usage

### Quick Start

1. **Setup configuration**:
```bash
cp .env.example .env
# Edit .env with your configuration
```

2. **Run the bot**:
```bash
make run
# or
cargo run --release
```

### Development Mode

```bash
make dev
# or
cargo run
```

### Health Check

```bash
make health-check
# or
cargo run --release --bin health_check
```

### Available Commands

```bash
make help              # Show all available commands
make setup            # Interactive setup wizard
make health-check     # Run health check
make run              # Build and run in release mode
make dev              # Run in development mode
make build            # Build release binary
make clean            # Clean build artifacts
```

## 🏗️ Architecture

### Project Structure

```
polymarket-copy-trader-rust/
├── src/
│   ├── main.rs          # Application entry point
│   ├── config.rs        # Configuration and copy strategy logic
│   ├── monitor.rs       # RTDS WebSocket monitoring
│   ├── executor.rs      # Trade execution engine
│   ├── types.rs         # Shared data structures
│   └── utils/           # Utilities (logging, HTTP, health checks)
├── Cargo.toml           # Rust dependencies
├── Makefile             # Build automation
└── README.md            # This file
```

### Key Components

1. **Monitor (`monitor.rs`)**
   - Establishes WebSocket connection to Polymarket RTDS
   - Subscribes to trade activity for configured traders
   - Forwards detected trades to executor

2. **Executor (`executor.rs`)**
   - Receives trades from monitor
   - Calculates order size based on copy strategy
   - Executes orders via Polymarket CLOB API
   - Handles retries and error recovery

3. **Config (`config.rs`)**
   - Loads and validates environment configuration
   - Implements copy strategy calculations
   - Manages risk limits and position tracking

## 🔒 Security Considerations

⚠️ **Important Security Notes**:

- **Private Key Storage**: Never commit your `.env` file or private keys to version control
- **Wallet Security**: Use a dedicated trading wallet, not your main wallet
- **Balance Limits**: Set appropriate position and daily volume limits
- **Network Security**: Use secure RPC endpoints (consider private RPC providers)
- **Key Management**: Consider using hardware wallets or secure key management systems for production

## 🐛 Troubleshooting

### Debug Mode

Enable verbose logging by running in development mode:
```bash
RUST_LOG=debug cargo run
```

## 📊 Performance

- **Latency**: Sub-second trade execution from detection to order placement
- **Throughput**: Handles multiple traders simultaneously
- **Resource Usage**: Low memory footprint, efficient WebSocket handling
- **Reliability**: Automatic reconnection and retry logic

## How it works


## 📞 Support

For questions or issues, contact via Telegram: [@Vladmeer](https://t.me/vladmeer67) and Twitter: [@Vladmeer](https://x.com/vladmeer67)

**Built with ❤️ using Rust**
