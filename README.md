# 🔧 Gensyn Node Setup – Made by **Thakur Saab** 🇮🇳🚩 (Gujarat)

> One‑click style guide to spin‑up your **Gensyn Testnet Node** with a custom banner, tunnelling, and two proven fixes for the “RL‑Swarm terminated” error.

---

## ⚠️ Reset Old Install

```bash
rm -rf rl-swarm
screen -S gensyn -X quit
```

---

## 🚀 Fresh Installation

### 1️⃣  System update + deps + banner  

```bash
sudo apt update && sudo apt install -y \
python3 python3-venv python3-pip curl wget screen git lsof nano unzip \
iproute2 build-essential gcc g++ figlet && echo "$(figlet 'Thakur Saab')"
```

### 2️⃣  Node.js (for tunnelling)

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
export NVM_DIR="$HOME/.nvm" && source "$NVM_DIR/nvm.sh"
nvm install 20 && nvm use 20
```

### 3️⃣  Start a screen session  

```bash
screen -S gensyn
```

### 4️⃣  Clone repo  

```bash
git clone https://github.com/gensyn-ai/rl-swarm.git
cd rl-swarm            # ⬅️  copy your .pem here if you’re an old user
```

### 5️⃣  Virtual‑env + launch node  

```bash
python3 -m venv .venv
source .venv/bin/activate
./run_rl_swarm.sh      # Y → A → 32 when prompted
```

### 6️⃣  Expose port 3000 (choose one)

**A) LocalTunnel**  
```bash
npx localtunnel --port 3000
```
Paste VPS IP when asked, verify e‑mail → done.

**B) Cloudflared (fallback)**  
```bash
wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb \
&& sudo dpkg -i cloudflared-linux-amd64.deb \
&& sudo apt-get install -f -y \
&& cloudflared tunnel --url http://localhost:3000
```

---

## 🛠️  RL‑Swarm “Terminated” Fixes

### 🥇 Method 1 – Tweak config values

```bash
cd ~/rl-swarm/hivemind_exp/configs/mac/
nano grpo-qwen-2.5-0.5b-deepseek-r1.yaml
```

Replace the block with:

```yaml
torch_dtype: float32
gradient_checkpointing: false
per_device_train_batch_size: 1
```

Save (`Ctrl+X` → `Y` → `Enter`) and restart:

```bash
python3 -m venv .venv
source .venv/bin/activate
./run_rl_swarm.sh      # choose 3 at W&B prompt
```

---

### 🥈 Method 2 – Patch `debug_utils.py`  
_Use **only if Method 1 didn’t solve the termination issue._)

```bash
nano ~/rl-swarm/hivemind_exp/debug_utils.py
```

1. Press `Ctrl+K` repeatedly to delete everything.  
2. Paste the entire code block below, save, and exit.

```python
import subprocess, sys, logging, platform
from pathlib import Path
from shutil import which
import psutil, colorlog

DIVIDER = "[---------] SYSTEM INFO [---------]"

def print_system_info():
    lines = ['\n', DIVIDER, ""]
    lines.append("Python Version:\n  " + sys.version)
    lines.append("\nPlatform Information:")
    lines += [f"  {k}: {v}" for k, v in {
        "System": platform.system(),
        "Release": platform.release(),
        "Version": platform.version(),
        "Machine": platform.machine(),
        "Processor": platform.processor(),
    }.items()]
    lines.append("\nCPU Information:")
    lines += [f"  {k}: {v}" for k, v in {
        "Physical cores": psutil.cpu_count(logical=False),
        "Total cores": psutil.cpu_count(logical=True),
        "Max Frequency": f"{psutil.cpu_freq().max:.2f} MHz",
        "Current Frequency": f"{psutil.cpu_freq().current:.2f} MHz",
    }.items()]
    lines.append("\nMemory Information:")
    vm = psutil.virtual_memory()
    lines += [f"  {k}: {v:.2f} GB" for k, v in {
        "Total": vm.total / 2**30,
        "Available": vm.available / 2**30,
        "Used": vm.used / 2**30,
    }.items()]
    lines.append("\nDisk Information (>80 % used):")
    for p in psutil.disk_partitions():
        try:
            du = psutil.disk_usage(p.mountpoint)
            if du.used / du.total > .8:
                lines.append(f"  {p.device} {du.percent}% at {p.mountpoint}")
        except (PermissionError, FileNotFoundError):
            lines.append(f"  Skipped {p.mountpoint}")
    lines.append("")

    if which('nvidia-smi'):
        try:
            lines.append("\nNVIDIA GPU Information:")
            n_out = subprocess.check_output(
                ['nvidia-smi', '--query-gpu=name,memory.total,memory.used,memory.free,temperature.gpu,utilization.gpu',
                 '--format=csv,noheader,nounits'], encoding='utf-8')
            for l in n_out.strip().split('\n'):
                name, total, used, free, temp, util = l.split(', ')
                lines += [f"  GPU: {name}",
                          f"    Memory Total: {total} MB",
                          f"    Memory Used:  {used} MB",
                          f"    Memory Free:  {free} MB",
                          f"    Temperature:  {temp}°C",
                          f"    Utilization:  {util}%"]
        except (subprocess.CalledProcessError, FileNotFoundError):
            lines.append("  Error getting NVIDIA info")

    if which('rocm-smi'):
        try:
            lines.append("\nAMD GPU Information:")
            lines += ["  " + l for l in subprocess.check_output(
                ['rocm-smi', '--showproductname', '--showvbios', '--showtemp'],
                encoding='utf-8').strip().split('\n')]
        except (subprocess.CalledProcessError, FileNotFoundError):
            lines.append("  Error getting AMD info")

    if platform.system() == 'Darwin' and platform.machine() == 'arm64':
        try:
            lines.append("\nApple Silicon:")
            brand = subprocess.check_output(
                ['sysctl', '-n', 'machdep.cpu.brand_string'],
                encoding='utf-8').strip()
            lines.append(f"  Processor: {brand}")
            try:
                import torch
                lines.append("  MPS: " +
                             ("Available" if torch.backends.mps.is_available() else "Not available"))
            except ImportError:
                lines.append("  PyTorch not installed → cannot check MPS")
        except (subprocess.CalledProcessError, FileNotFoundError):
            lines.append("  Error getting Apple Silicon info")

    lines += ["", DIVIDER]
    return "\n".join(lines)


class TeeHandler(logging.Handler):
    def __init__(self, filename, mode='a', console_level=logging.INFO, file_level=logging.DEBUG):
        super().__init__()
        self.console_handler = colorlog.StreamHandler()
        self.console_handler.setLevel(console_level)
        self.console_handler.setFormatter(
            colorlog.ColoredFormatter("%(green)s%(levelname)s:%(name)s:%(message)s"))
        Path(filename).parent.mkdir(parents=True, exist_ok=True)
        self.file_handler = logging.FileHandler(filename, mode=mode)
        self.file_handler.setLevel(file_level)
        self.file_handler.setFormatter(
            logging.Formatter("%(asctime)s - %(levelname)s - %(name)s:%(lineno)d - %(message)s"))

    def emit(self, record):
        if record.levelno >= self.console_handler.level:
            self.console_handler.emit(record)
        if record.levelno >= self.file_handler.level:
            self.file_handler.emit(record)


class PrintCapture:
    def __init__(self, logger):
        self.logger = logger
        self.original_stdout = sys.stdout

    def write(self, buf):
        self.original_stdout.write(buf)
        for line in buf.rstrip().splitlines():
            if line.strip():
                self.logger.debug(f"[PRINT] {line.rstrip()}")

    def flush(self):
        self.original_stdout.flush()

    def __getattr__(self, attr):
        return getattr(self.original_stdout, attr)
```

_Save & exit (`Ctrl+X`, `Y`, `Enter`)_

Restart node:

```bash
cd ~/rl-swarm
python3 -m venv .venv
source .venv/bin/activate
./run_rl_swarm.sh
```

---

## 🧿  Re‑attach if screen drops

```bash
screen -r gensyn
```

---

## ✅  All set – Node is live! 🚀

Made with 💻 by **Thakur Saab** – Gujarat 🚩  
Telegram support: **[@kittu2141](https://t.me/kittu2141)** – DM for help if anything breaks.
