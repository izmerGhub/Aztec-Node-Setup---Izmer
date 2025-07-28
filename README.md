# 🧪 Aztec Network Sequencer Node Guide

A step-by-step guide on how to run a **Sequencer Node** on Aztec Network Testnet and earn the **Apprentice Role**.

---

## 🌐 What types of nodes can participate?

- **Sequencer**: Proposes blocks, validates blocks from others, and votes on upgrades.
- **Prover**: Generates ZK proofs that attest to roll-up integrity.

> ⚠️ *Only Sequencer nodes are needed for this guide.*

## 🎖 Roles Info

Running a Sequencer node gives you a chance to earn a **role** on the Aztec Discord.

Once synced, follow the "Get Role" steps below.

---

## 🖥 Hardware Requirements

| Node Type      | RAM     | CPU       | Disk                |
| -------------- | ------- | --------- | ------------------- |
| Sequencer Node | 8-16 GB | 4-9 cores | 100+ GB SSD         |
| Prover Node    | 128 GB  | 16 cores  | - (\~40x machines!) |

> ⚠️ *This guide skips Prover setup. It’s for data center usage.*

- **Windows Users**: Install Ubuntu using [this guide](https://learn.microsoft.com/en-us/windows/wsl/install).
- **VPS Users**: Use VPS with **4 cores CPU**, **8GB RAM** (Recommended).

**💰 Budget VPS (Crypto Accepted)**
1. **SpaceCore** - Contabo alternative, crypto payments
   [CLICK HERE](https://billing.spacecore.pro/billmgr?from=63882)

2. **RackNerd** - From $10.96/year, 30+ crypto options  
   [CLICK HERE](https://my.racknerd.com/aff.php?aff=14994)

3. **HostVDS** - Russian servers, crypto payments  
   [CLICK HERE](https://hostvds.com/?affiliate_uuid=f3d517f2-6e58-4549-9ecd-d280fa8cea3c)

---

## ✅ Step-by-Step Setup

### 1. Install Dependencies

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip -y
```

### 2. Install Docker

```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
sudo apt-get install ca-certificates curl gnupg -y
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```bash
sudo apt update -y && sudo apt upgrade -y
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo docker run hello-world
sudo systemctl enable docker
sudo systemctl restart docker
```

### 3. Install Aztec Tools

```bash
bash -i <(curl -s https://install.aztec.network)
echo 'export PATH="$HOME/.aztec/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Check:

```bash
aztec
```

### 4. Update Aztec

```bash
aztec-up 1.1.2
```

### 5. Get RPC URLs

- **RPC URL**: alchemy.com
  e.g -https://sepolia.infura.io/v3/YOUR_API_KEY
     
- **BEACON URL**: drpc.org
  e.g -https://lb.drpc.org/rest/YOUR_API_KEY/eth-beacon-chain-sepolia
     
You may also run your own Geth & Prysm node (600-1000 GB SSD required)(NEED 1 TB SPACE)

### 6. Stanby Ethereum Wallet

Get a public/private key pair. Save both.

### 7. Fund with Sepolia ETH

Use faucet to get Sepolia ETH.
[QuickNode Sepolia Faucet](https://faucet.quicknode.com/ethereum/sepolia) – get ~0.2 ETH every 12h

### 8. Get IP Address

```bash
curl ipv4.icanhazip.com
e.g 123.45.67.89
```

Save this IP.

### 9. Enable Firewall & Open Ports

```bash
ufw allow 22
ufw allow ssh
ufw allow 40400
ufw allow 8080
ufw enable
```

---

## 🚀 Run Sequencer Node

### Method 1: Docker

1. **Prepare Folder & Files**:

```bash
mkdir aztec && cd aztec
nano .env
```

Paste & update:

```
ETHEREUM_RPC_URL=YOUR_RPC
CONSENSUS_BEACON_URL=YOUR_BEACON
VALIDATOR_PRIVATE_KEY=0xYourPrivateKey
COINBASE=0xYourAddress
P2P_IP=YourIP
```

2. **Create Docker Compose File**:

```bash
nano docker-compose.yml
```

Paste config:

```yaml
services:
  aztec-node:
    container_name: aztec-sequencer
    network_mode: host
    image: aztecprotocol/aztec:latest
    restart: unless-stopped
    environment:
      ETHEREUM_HOSTS: ${ETHEREUM_RPC_URL}
      L1_CONSENSUS_HOST_URLS: ${CONSENSUS_BEACON_URL}
      DATA_DIRECTORY: /data
      VALIDATOR_PRIVATE_KEY: ${VALIDATOR_PRIVATE_KEY}
      COINBASE: ${COINBASE}
      P2P_IP: ${P2P_IP}
      LOG_LEVEL: debug
    entrypoint: >
      sh -c 'node --no-warnings /usr/src/yarn-project/aztec/dest/bin/index.js start --network alpha-testnet --node --archiver --sequencer'
    ports:
      - 40400:40400/tcp
      - 40400:40400/udp
      - 8080:8080
    volumes:
      - /root/.aztec/alpha-testnet/data/:/data
```

3. **Start Node**:

```bash
docker compose up -d
docker compose logs -fn 1000
```

### Method 2: CLI

```bash
screen -S aztec
aztec start --node --archiver --sequencer \
  --network alpha-testnet \
  --l1-rpc-urls RPC_URL \
  --l1-consensus-host-urls BEACON_URL \
  --sequencer.validatorPrivateKey 0xYourPrivateKey \
  --sequencer.coinbase 0xYourAddress \
  --p2p.p2pIp IP
```

---

## 🔁 Sync & Verify

Check synced block:

```bash
curl -s -X POST -H 'Content-Type: application/json' \
-d '{"jsonrpc":"2.0","method":"node_getL2Tips","params":[],"id":67}' \
http://localhost:8080 | jq -r ".result.proven.number"
```

Compare with: [https://aztecscan.xyz](https://aztecscan.xyz)

---

## 📝 Register as Validator

1. Ensure node is synced
2. Visit: [https://testnet.aztec.network/add-validator](https://testnet.aztec.network/add-validator)
3. Verify with ZKPassport & follow steps
4. Wait to earn Discord role

---

## 📊 Aztec Dashboard

Track your validator:

- Visit the [Aztec Dashboard](https://testnet.aztec.network/dashboard)
- Connect your X & Discord

---

## 🔄 Updating Node

### Docker Method

```bash
docker compose down -v
source ~/.bashrc
aztec-up 1.1.2
rm -rf ~/.aztec/alpha-testnet/data/
docker compose up -d
```

### CLI Method

```bash
screen -XS aztec quit
source ~/.bashrc
aztec-up 1.1.2
rm -rf ~/.aztec/alpha-testnet/data/
# Then rerun your CLI start command
```

## 🛠 Troubleshooting

**Error:** `block_stream: Error processing block stream...`

**Fix:** Stop node, delete data, restart.

---

## 🛡 Claim Guardian Role

1. Go to Discord > `#upgrade-role`
2. Type `/checkip`
3. Enter your **IP & Node Address**
4. If not eligible, wait for the next snapshot.

---

**Made by @izmer** — Feel free to fork and star if this helps.

