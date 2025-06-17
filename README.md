# 🔧 Gensyn Node Setup – Made by Thakur Saab 🇮🇳🚩 (Gujarat)

> One-click style full setup for your Gensyn Testnet Node with a 🔥 custom banner.

---

## ⚠️ For Old Users (Reset Steps)

```bash
rm -rf rl-swarm
screen -S gensyn -X quit
```

---

## 🚀 Fresh Installation Guide

### 🔹 Step 1: System Update + Install All Dependencies + Show Banner

```bash
sudo apt update && sudo apt install -y python3 python3-venv python3-pip curl wget screen git lsof nano unzip iproute2 build-essential gcc g++ figlet && echo "$(figlet 'Thakur Saab')"
```

---

### 🔹 Step 2: Install Node.js with NVM (for localtunnel)

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
export NVM_DIR="$HOME/.nvm"
source "$NVM_DIR/nvm.sh"
nvm install 20
nvm use 20
```

---

### 🔹 Step 3: Start Gensyn in a Screen Session

```bash
screen -S gensyn
```

---

### 🔹 Step 4: Clone Official Repo & Enter

```bash
git clone https://github.com/gensyn-ai/rl-swarm.git
cd rl-swarm
```

Old users: copy your `.pem` file here.

---

### 🔹 Step 5: Create Virtual Env & Start Node

```bash
python3 -m venv .venv
source .venv/bin/activate
./run_rl_swarm.sh
```

➡️ Choose:
- Press `Y`
- Then `A`
- Then `7`

---

### 🔹 Step 6: Open Tunnel (New Terminal)

```bash
npx localtunnel --port 3000
```

Paste your VPS IP as password on the website.  
Then verify email — you're done!

---

## 🛠️ Fix for Node Termination Issue

### A. Edit Config File

```bash
cd rl-swarm
nano hivemind_exp/configs/mac/grpo-qwen-2.5-0.5b-deepseek-r1.yaml
```

🔄 Change to:

```yaml
torch_dtype: float32
bf16: false
gradient_checkpointing: false
per_device_train_batch_size: 1
```

Save: `Ctrl+X`, then `Y`, then `Enter`.

---

### B. Re-run Node

```bash
python3 -m venv .venv
source .venv/bin/activate
./run_rl_swarm.sh
```

W&B prompt → Enter `3`

---

### 🧿 Reconnect to Screen (if disconnected)

```bash
screen -r gensyn
```

---

## ✅ DONE – Your Gensyn Node is Live 🚀

> Made with 💻 by Thakur Saab – Gujarat 🚩  
> 🔗 Telegram: [@kittu2141](https://t.me/kittu2141)  
> ❗ If you face any error, **message me on Telegram!**
