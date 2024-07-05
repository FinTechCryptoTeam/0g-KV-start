# 0g-KV-start: Setting Up 0g Storage KV on Linux

Welcome to the **ZeroG-KVLaunchpad** guide! This manual is designed for individuals looking to deploy the **0g storage kv** on their Linux server from scratch.

## What is 0g Storage KV?
0g Storage KV is a distributed key-value storage system, optimized for the zero-gravity conditions of cyberspace. It provides high availability and scalability for your applications.

## Prerequisites
Before you begin, ensure you have:
- A Linux server with SSH access.
- `git`, `docker`, and `docker-compose` installed.

## Installation and Launch

### Ensure Rust & GO are installed

```
rustc --version
go version
```

### Build Storage KV from source code

```
git clone -b v1.1.0-testnet https://github.com/0glabs/0g-storage-kv.git
cd 0g-storage-kv
git submodule update --init
cargo build --release
```

### Copy the example config_example.toml & rename to config.toml

```
STORAGE_PORT=$(grep -oP '(?<=rpc_listen_address = "0.0.0.0:)\d+(?=")' $HOME/0g-storage-node/run/config.toml)
ZGS_LOG_SYNC_BLOCK=$(grep -oP '(?<=log_sync_start_block_number = )\d+' $HOME/0g-storage-node/run/config.toml)
STORAGE_RPC_ENDPOINT=http://$(wget -qO- eth0.me):$STORAGE_PORT
BLOCKCHAIN_RPC_ENDPOINT=$(sed -n 's/blockchain_rpc_endpoint = "\([^"]*\)"/\1/p' $HOME/0g-storage-node/run/config.toml)
LOG_CONTRACT_ADDRESS=$(sed -n 's/log_contract_address = "\([^"]*\)"/\1/p' $HOME/0g-storage-node/run/config.toml)
MINE_CONTRACT_ADDRESS=$(sed -n 's/mine_contract_address = "\([^"]*\)"/\1/p' $HOME/0g-storage-node/run/config.toml)
JSON_PORT=$(sed -n '/\[json-rpc\]/,/^address/ s/address = "0.0.0.0:\([0-9]*\)".*/\1/p' $HOME/.0gchain/config/app.toml)
JSON_RPC_ENDPOINT=http://$(wget -qO- eth0.me):$JSON_PORT

echo -e "\nSTORAGE_RPC_ENDPOINT: $STORAGE_RPC_ENDPOINT\nLOG_CONTRACT_ADDRESS: $LOG_CONTRACT_ADDRESS\nMINE_CONTRACT_ADDRESS: $MINE_CONTRACT_ADDRESS\nBLOCKCHAIN_RPC_ENDPOINT: $BLOCKCHAIN_RPC_ENDPOINT\nJSON_RPC_ENDPOINT: $JSON_RPC_ENDPOINT\nZGS_LOG_SYNC_BLOCK: $ZGS_LOG_SYNC_BLOCK\n\nScript by \033[35mNodeCattel\033[0m"
```

```
sed -i "s|rpc_listen_address = .*|rpc_listen_address = \"0.0.0.0:6789\"|" $HOME/0g-storage-kv/run/config.toml
sed -i "s|zgs_node_urls = .*|zgs_node_urls = \"$STORAGE_RPC_ENDPOINT\"|" $HOME/0g-storage-kv/run/config.toml
sed -i "s|log_config_file = .*|log_config_file = \"$HOME/0g-storage-kv/run/log_config\"|" $HOME/0g-storage-kv/run/config.toml
sed -i "s|blockchain_rpc_endpoint = .*|blockchain_rpc_endpoint = \"$BLOCKCHAIN_RPC_ENDPOINT\"|" $HOME/0g-storage-kv/run/config.toml
sed -i "s|log_contract_address = .*|log_contract_address = \"$LOG_CONTRACT_ADDRESS\"|" $HOME/0g-storage-kv/run/config.toml
sed -i "s|log_sync_start_block_number = .*|log_sync_start_block_number = $ZGS_LOG_SYNC_BLOCK|" $HOME/0g-storage-kv/run/config.toml
```

### Create zgs-kv service (storage KV)  to run in the background

```
sudo tee /etc/systemd/system/zgs-kv.service > /dev/null <<EOF
[Unit]
Description=ZGS-KV Node
After=network.target

[Service]
User=$USER
WorkingDirectory=$HOME/0g-storage-kv/run
ExecStart=$HOME/0g-storage-kv/target/release/zgs_kv --config $HOME/0g-storage-kv/run/config.toml
Restart=on-failure
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### Start Storage KV node

```
sudo systemctl daemon-reload && \
sudo systemctl enable zgs-kv && \
sudo systemctl restart zgs-kv
```

### View Storage KV log

```
sudo journalctl -u zgs-kv.service -f
```

### Stop Storage KV node - (if you wish to stop the service)

```
sudo systemctl stop zgs-kv
```

By the end of this guide, you'll have a fully functional 0g storage kv system ready to handle your application's data needs. Let's get started!
