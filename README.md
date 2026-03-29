# Push Chain Testnet Node Installation Guide

<p align="center">
  <img src="https://raw.githubusercontent.com/coinsspor/pushchain/refs/heads/main/push.jpg" width="150">
</p>

<p align="center">
  <a href="https://coinsspor.com/pushchain">Explorer</a> •
  <a href="https://coinsspor.com">Website</a> •
  <a href="https://twitter.com/coinsspor">Twitter</a> •
  <a href="https://t.me/coinsspor">Telegram</a>
</p>

---

## Chain Information

| Parameter | Value |
|-----------|-------|
| Chain ID | `push_42101-1` |
| Binary | `pchaind` |
| Denom | `upc` |
| Version | `v0.0.27` |
| Genesis | [Download](https://raw.githubusercontent.com/coinsspor/pushchain/refs/heads/main/genesis.json) |
| Addrbook | [Download](https://github.com/coinsspor/pushchain/blob/main/addrbook.json) |
| Faucet | https://faucet.push.org |

## Public Endpoints

| Service | URL |
|---------|-----|
| RPC | https://push-testnet-rpc.coinsspor.com |
| API | https://push-testnet-api.coinsspor.com |
| EVM JSON-RPC | https://push-testnet-evm.coinsspor.com |
| gRPC | `push-testnet-grpc.coinsspor.com:443` |
| Explorer | https://coinsspor.com/pushchain |

## Peer

```
0ee00aff1f2e10dc1a9ef2dd73ff6d4594c1f99e@65.108.65.23:61656
```

---

## Hardware Requirements

| Component | Minimum |
|-----------|---------|
| CPU | 4 Cores |
| RAM | 8 GB |
| Disk | 200 GB SSD |
| OS | Ubuntu 22.04+ |

---

## Installation

### 1. Install Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

### 2. Install Go

```bash
cd $HOME
VER="1.23.8"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

### 3. Install Cosmovisor

```bash
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
```

### 4. Set Variables

Replace `your-moniker` with your desired node name and set your preferred port prefix (2 digits).

```bash
echo "export WALLET=\"wallet\"" >> $HOME/.bash_profile
echo "export MONIKER=\"your-moniker\"" >> $HOME/.bash_profile
echo "export PUSH_CHAIN_ID=\"push_42101-1\"" >> $HOME/.bash_profile
echo "export PUSH_PORT=\"61\"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

> **Port Prefix:** The `PUSH_PORT` variable sets a custom port prefix to avoid conflicts with other nodes. For example, setting `61` will use ports `61657` (RPC), `61656` (P2P), `61317` (API), etc. Change this value if you have port conflicts.

### 5. Download Binary

```bash
cd $HOME
rm -rf ~/bin
wget -O push.tar.gz https://github.com/pushchain/push-chain-node/releases/download/v0.0.27/push-chain_0.0.27_linux_amd64.tar.gz
tar -xvf push.tar.gz
chmod +x ~/bin/pchaind
sudo mv ~/bin/pchaind $HOME/go/bin/
rm push.tar.gz
```

### 6. Initialize Node

```bash
pchaind config node tcp://localhost:${PUSH_PORT}657
pchaind config keyring-backend os
pchaind config chain-id push_42101-1
pchaind init "$MONIKER" --chain-id push_42101-1
```

### 7. Setup Cosmovisor

```bash
mkdir -p $HOME/.pchain/cosmovisor/genesis/bin
mkdir -p $HOME/.pchain/cosmovisor/upgrades
cp $(which pchaind) $HOME/.pchain/cosmovisor/genesis/bin/pchaind
```

### 8. Download Genesis & Addrbook

```bash
wget -O $HOME/.pchain/config/genesis.json https://raw.githubusercontent.com/coinsspor/pushchain/refs/heads/main/genesis.json
wget -O $HOME/.pchain/config/addrbook.json https://raw.githubusercontent.com/coinsspor/pushchain/main/addrbook.json
```

### 9. Set Seeds & Peers

```bash
SEEDS="a8d3377ef5f091980a425b84380655865c0f2320@push-testnet-seed.itrocket.net:30656"
PEERS="0ee00aff1f2e10dc1a9ef2dd73ff6d4594c1f99e@65.108.65.23:61656"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.pchain/config/config.toml
```

### 10. Set Custom Ports

```bash
# app.toml
sed -i.bak -e "s%:1317%:${PUSH_PORT}317%g;
s%:8080%:${PUSH_PORT}080%g;
s%:9090%:${PUSH_PORT}090%g;
s%:9091%:${PUSH_PORT}091%g;
s%:8545%:${PUSH_PORT}545%g;
s%:8546%:${PUSH_PORT}546%g;
s%:6065%:${PUSH_PORT}065%g" $HOME/.pchain/config/app.toml

# config.toml
sed -i.bak -e "s%:26658%:${PUSH_PORT}658%g;
s%:26657%:${PUSH_PORT}657%g;
s%:6060%:${PUSH_PORT}060%g;
s%:26656%:${PUSH_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${PUSH_PORT}656\"%;
s%:26660%:${PUSH_PORT}660%g" $HOME/.pchain/config/config.toml
```

### 11. Configure Pruning

```bash
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.pchain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.pchain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"19\"/" $HOME/.pchain/config/app.toml
```

### 12. Set Gas Price & Other Settings

```bash
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "1000000000upc"|g' $HOME/.pchain/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.pchain/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.pchain/config/config.toml
```

### 13. Enable API, gRPC & EVM

```bash
# Enable API
sed -i '/\[api\]/,/\[/{s/enable = false/enable = true/}' $HOME/.pchain/config/app.toml

# Enable gRPC
sed -i '/\[grpc\]/,/\[/{s/enable = false/enable = true/}' $HOME/.pchain/config/app.toml

# Enable EVM JSON-RPC
sed -i '/\[json-rpc\]/,/\[/{s/enable = false/enable = true/}' $HOME/.pchain/config/app.toml
```

### 14. Create Service File

```bash
sudo tee /etc/systemd/system/pchaind.service > /dev/null <<EOF
[Unit]
Description=Push Chain Node (Cosmovisor)
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/.pchain
ExecStart=$(which cosmovisor) run start --home $HOME/.pchain
Environment="DAEMON_NAME=pchaind"
Environment="DAEMON_HOME=$HOME/.pchain"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### 15. Start Node

```bash
sudo systemctl daemon-reload
sudo systemctl enable pchaind
sudo systemctl start pchaind
```

### 16. Check Logs

```bash
sudo journalctl -u pchaind -fo cat
```

---

## Create Wallet

### New Wallet

```bash
pchaind keys add $WALLET --node tcp://localhost:${PUSH_PORT}657
```

### Import Existing Wallet

```bash
pchaind keys add $WALLET --recover --node tcp://localhost:${PUSH_PORT}657
```

### Check Balance

```bash
pchaind query bank balances $(pchaind keys show $WALLET -a) --node tcp://localhost:${PUSH_PORT}657
```

---

## Create Validator

```bash
cd $HOME

echo "{\"pubkey\":{\"@type\":\"/cosmos.crypto.ed25519.PubKey\",\"key\":\"$(pchaind comet show-validator --home $HOME/.pchain | grep -Po '\"key\":\s*\"\K[^"]*')\"},
    \"amount\": \"1000000000000000000upc\",
    \"moniker\": \"$MONIKER\",
    \"identity\": \"\",
    \"website\": \"\",
    \"security\": \"\",
    \"details\": \"\",
    \"commission-rate\": \"0.1\",
    \"commission-max-rate\": \"0.2\",
    \"commission-max-change-rate\": \"0.01\",
    \"min-self-delegation\": \"1\"
}" > validator.json

pchaind tx staking create-validator validator.json \
    --from $WALLET \
    --chain-id push_42101-1 \
    --gas auto --gas-adjustment 1.5 \
    --fees 500000000000000upc \
    --node tcp://localhost:${PUSH_PORT}657 \
    -y
```

---

## Useful Commands

### Check Sync Status

```bash
pchaind status --node tcp://localhost:${PUSH_PORT}657 | jq .sync_info
```

### Check Node Info

```bash
pchaind status --node tcp://localhost:${PUSH_PORT}657 | jq .node_info
```

### Check Validator Info

```bash
pchaind query staking validator $(pchaind keys show $WALLET --bech val -a --node tcp://localhost:${PUSH_PORT}657) --node tcp://localhost:${PUSH_PORT}657
```

### Delegate to Yourself

```bash
pchaind tx staking delegate $(pchaind keys show $WALLET --bech val -a --node tcp://localhost:${PUSH_PORT}657) AMOUNT_upc \
    --from $WALLET \
    --chain-id push_42101-1 \
    --gas auto --gas-adjustment 1.5 \
    --fees 500000000000000upc \
    --node tcp://localhost:${PUSH_PORT}657 \
    -y
```

### Unjail Validator

```bash
pchaind tx slashing unjail \
    --from $WALLET \
    --chain-id push_42101-1 \
    --gas auto --gas-adjustment 1.5 \
    --fees 500000000000000upc \
    --node tcp://localhost:${PUSH_PORT}657 \
    -y
```

### Withdraw Rewards

```bash
pchaind tx distribution withdraw-rewards $(pchaind keys show $WALLET --bech val -a --node tcp://localhost:${PUSH_PORT}657) \
    --commission \
    --from $WALLET \
    --chain-id push_42101-1 \
    --gas auto --gas-adjustment 1.5 \
    --fees 500000000000000upc \
    --node tcp://localhost:${PUSH_PORT}657 \
    -y
```

### Send Tokens

```bash
pchaind tx bank send $WALLET RECIPIENT_ADDRESS AMOUNT_upc \
    --chain-id push_42101-1 \
    --gas auto --gas-adjustment 1.5 \
    --fees 500000000000000upc \
    --node tcp://localhost:${PUSH_PORT}657 \
    -y
```

### Vote on Proposal

```bash
pchaind tx gov vote PROPOSAL_ID yes \
    --from $WALLET \
    --chain-id push_42101-1 \
    --gas auto --gas-adjustment 1.5 \
    --fees 500000000000000upc \
    --node tcp://localhost:${PUSH_PORT}657 \
    -y
```

---

## Service Management

### Restart Node

```bash
sudo systemctl restart pchaind
```

### Stop Node

```bash
sudo systemctl stop pchaind
```

### Check Service Status

```bash
sudo systemctl status pchaind
```

### View Logs

```bash
sudo journalctl -u pchaind -fo cat
```

---

## Remove Node

> ⚠️ **Warning:** This will completely remove the node and all data. Make sure to back up your keys first!

```bash
sudo systemctl stop pchaind
sudo systemctl disable pchaind
sudo rm -rf /etc/systemd/system/pchaind.service
sudo systemctl daemon-reload
sudo rm -rf $HOME/.pchain
sudo rm -f $(which pchaind)
sudo rm -f $(which cosmovisor)
```

---

<p align="center">
  <b>Coinsspor Node Center</b><br>
  Unlock the Power of Blockchain
</p>
