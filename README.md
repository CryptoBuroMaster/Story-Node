# Story-Node

# Story Protocol Validator Node Setup Guide

Story raised $140M from Tier1 investors. Story is a blockchain making IP protection and licensing programmable and efficient. It automates IP management, allowing creators to easily license, remix, and monetize their work. With Story, traditional legal complexities are replaced by on-chain smart contracts and off-chain legal agreements, simplifying the entire process.

## System Requirements

| **Hardware** | **Minimum Requirement** |
|--------------|-------------------------|
| **CPU**      | 4 Cores                 |
| **RAM**      | 8 GB                    |
| **Disk**     | 200 GB                  |
| **Bandwidth**| 10 MBit/s               |


Follow our TG : https://t.me/CryptoBuroOfficial



## Make Folder Story-Node

```
sudo mkdir ~/Story-Node
cd ~/Story-Node
```


## Install dependencies

```
sudo apt update
sudo apt-get update
sudo apt install curl git make jq build-essential gcc unzip wget lz4 aria2 pv -y
```

## Install Go

```
cd ~/Story-Node && \
ver="1.22.0" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf ./go && \
sudo tar -C . -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:$(pwd)/go/bin" >> ~/.bash_profile && \
source ~/.bash_profile && \
go version
```

## Download Story-Geth binary

```
cd ~/Story-Node && \
wget https://story-geth-binaries.s3.us-west-1.amazonaws.com/geth-public/geth-linux-amd64-0.9.2-ea9f0d2.tar.gz && \
tar -xzvf geth-linux-amd64-0.9.2-ea9f0d2.tar.gz && \
[ ! -d "$HOME/Story-Node/go/bin" ] && mkdir -p $HOME/Story-Node/go/bin && \
if ! grep -q "$HOME/Story-Node/go/bin" $HOME/.bash_profile; then
  echo 'export PATH=$PATH:$HOME/Story-Node/go/bin' >> $HOME/.bash_profile
fi && \
sudo cp geth-linux-amd64-0.9.2-ea9f0d2/geth $HOME/Story-Node/go/bin/story-geth && \
source $HOME/.bash_profile && \
story-geth version
```

## Download Story binary

```
cd ~/Story-Node && \
wget https://story-geth-binaries.s3.us-west-1.amazonaws.com/story-public/story-linux-amd64-0.9.11-2a25df1.tar.gz && \
tar -xzvf story-linux-amd64-0.9.11-2a25df1.tar.gz && \
[ ! -d "$HOME/Story-Node/go/bin" ] && mkdir -p $HOME/Story-Node/go/bin && \
if ! grep -q "$HOME/Story-Node/go/bin" $HOME/.bash_profile; then
  echo 'export PATH=$PATH:$HOME/Story-Node/go/bin' >> $HOME/.bash_profile
fi && \
sudo cp story-linux-amd64-0.9.11-2a25df1/story $HOME/Story-Node/go/bin/story && \
source $HOME/.bash_profile && \
story version
```

## Initiate Iliad node

Replace "Your_moniker_name" with any name you want 
(Ex: story init --network iliad --moniker Buro)

```
story init --network iliad --moniker "Your_moniker_name"
```
### Peers setup
```
cd ~/Story-Node && \
PEERS=$(curl -s -X POST https://rpc-story.josephtran.xyz -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"net_info","params":[],"id":1}' | jq -r '.result.peers[] | select(.connection_status.SendMonitor.Active == true) | "\(.node_info.id)@\(if .node_info.listen_addr | contains("0.0.0.0") then .remote_ip + ":" + (.node_info.listen_addr | sub("tcp://0.0.0.0:"; "")) else .node_info.listen_addr | sub("tcp://"; "") end)"' | tr '\n' ',' | sed 's/,$//' | awk '{print "\"" $0 "\""}') && \
sed -i "s/^persistent_peers *=.*/persistent_peers = $PEERS/" "$HOME/Story-Node/.story/story/config/config.toml" && \
if [ $? -eq 0 ]; then
    echo -e "Configuration file updated successfully with new peers"
else
    echo "Failed to update configuration file."
fi
```

## Create story-geth service file
```
cd ~/Story-Node && \
sudo tee /etc/systemd/system/story-geth.service > /dev/null <<EOF
[Unit]
Description=Story Geth Client
After=network.target

[Service]
User=root
ExecStart=$(pwd)/go/bin/story-geth --iliad --syncmode full
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```
## Create story service file

```
cd ~/Story-Node && \
sudo tee /etc/systemd/system/story.service > /dev/null <<EOF
[Unit]
Description=Story Consensus Client
After=network.target

[Service]
User=root
ExecStart=$(pwd)/go/bin/story run
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```

## Reload and start story-geth
```
sudo systemctl daemon-reload && \
sudo systemctl enable story-geth && \
sudo systemctl enable story && \
sudo systemctl start story-geth && \
sudo systemctl start story && \
sudo systemctl status story-geth
```

# Check logs

### Geth logs
```
sudo journalctl -u story-geth -f -o cat
```
### Story logs
```
sudo journalctl -u story -f -o cat
```
### Check sync status

```
curl localhost:26657/status | jq
```

# SYNC using snapshot File

C2 Joseph Tran

### Stop node
```
sudo systemctl stop story
sudo systemctl stop story-geth
```
### Download Geth-data
```
cd ~/Story-Node && \
rm -f Geth_snapshot.lz4 && \
if curl -s --head https://vps6.josephtran.xyz/Story/Geth_snapshot.lz4 | head -n 1 | grep "200" > /dev/null; then
    echo "Snapshot found, downloading..." && \
    aria2c -x 16 -s 16 https://vps6.josephtran.xyz/Story/Geth_snapshot.lz4 -o Geth_snapshot.lz4
else
    echo "No snapshot found."
fi
```
### Download Story-data
```
cd ~/Story-Node && \
rm -f Story_snapshot.lz4 && \
if curl -s --head https://vps6.josephtran.xyz/Story/Story_snapshot.lz4 | head -n 1 | grep "200" > /dev/null; then
    echo "Snapshot found, downloading..." && \
    aria2c -x 16 -s 16 https://vps6.josephtran.xyz/Story/Story_snapshot.lz4 -o Story_snapshot.lz4
else
    echo "No snapshot found."
fi
```
### Backup priv_validator_state.json:
```
cd ~/Story-Node && \
mv $HOME/.story/story/data/priv_validator_state.json $HOME/.story/priv_validator_state.json.backup
```
### Remove old data
```
cd ~/Story-Node && \
rm -rf ~/.story/story/data && \
rm -rf ~/.story/geth/iliad/geth/chaindata
```
### Extract Story-data
```
cd ~/Story-Node && \
sudo mkdir -p /root/.story/story/data && \
lz4 -d Story_snapshot.lz4 | pv | sudo tar xv -C /root/.story/story/
```
### Extract Geth-data
```
cd ~/Story-Node && \
sudo mkdir -p /root/.story/geth/iliad/geth/chaindata && \
lz4 -d Geth_snapshot.lz4 | pv | sudo tar xv -C /root/.story/geth/iliad/geth/
```
### Move priv_validator_state.json back

```
cd ~/Story-Node && \
mv $HOME/.story/priv_validator_state.json.backup $HOME/.story/story/data/priv_validator_state.json
```
### Restart node 
```
sudo systemctl start story
sudo systemctl start story-geth
```

# Register your Validator

### 1. Export wallet:
```
cd ~/Story-Node && \
story validator export --export-evm-key
```
### 2. Private key preview
```
cd ~/Story-Node && \
sudo nano ~/.story/story/config/private_key.txt
```
### 3. Import Key to Metamask 

Get the wallet address for faucet

### 4. You need at least have 1 IP on wallet

Get it from faucet : https://faucet.story.foundation/

Check the sync, the catching up must be 'false'
```
cd ~/Story-Node && \
curl -s localhost:26657/status | jq
```
Stake only after "catching_up": false

### 5. Validator registering

Replace "your_private_key" with your key from the step2

```
cd ~/Story-Node && \
story validator create --stake 1000000000000000000 --private-key "your_private_key"
```

### 6. Check your validator INFO
```
cd ~/Story-Node && \
curl -s localhost:26657/status | jq -r '.result.validator_info' 
```

### 7. check your validator

Explorer: https://testnet.story.explorers.guru/

## BACK UP FILE

### 1. Wallet private key:
```
cd ~/Story-Node && \
sudo nano ~/.story/story/config/private_key.txt
```
### 2. Validator key:

```
cd ~/Story-Node && \
sudo nano ~/.story/story/config/priv_validator_key.json
```

Join our TG : https://t.me/CryptoBuroOfficial
