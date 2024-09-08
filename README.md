# Story_Validator_Setup_Guide
## Hướng dẫn Cài đặt và Chạy Node cho Dự án Story đầy đủ (Story_Protocol_Validator Node Setup Guide Full)
Story raised $140M from Tier1 investors. Story is a blockchain making IP protection and licensing programmable and efficient. It automates IP management, allowing creators to easily license, remix, and monetize their work. With Story, traditional legal complexities are replaced by on-chain smart contracts and off-chain legal agreements, simplifying the entire process.

## 1. **Thông số Phần cứng anh em mua VPS** (System Requirements)

Mọi người có thể mua ở đây bằng USDT, ETH, BTC... 
https://pq.hosting/?from=719019

| **Hardware** | **Minimum Requirement** |
|--------------|-------------------------|
| **CPU**      | 4 Cores                 |
| **RAM**      | 8 GB                    |
| **Disk**     | 200 GB                  |
| **Bandwidth**| 10 MBit/s               |

Follow our TG : https://t.me/Crypto_Confessions

## **Ghi chú**

- Hệ điều hành: Ubuntu 22.04
- Công cụ cài đặt trên Windown: Bitvise => tải tại đây => https://bitvise.com/ssh-client-download
- Các công cụ cơ bản: `curl`, `git`, `make`, `jq`, `build-essential`, `gcc`, `unzip`, `wget`, `lz4`, `aria2`
  
## 2. Cài đặt (Install)

### 2.1. Cài đặt các công cụ cần thiết (Install dependencies)
```
sudo apt update
sudo apt-get update
sudo apt install curl git make jq build-essential gcc unzip wget lz4 aria2 pv -y
```

## Tạo thư mục $HOME/go/bin (Install Go)
```
cd $HOME && \
ver="1.22.0" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
source ~/.bash_profile && \
go version
```

## Tải xuống và cài đặt story-geth (Download Story-Geth binary)

```
wget https://story-geth-binaries.s3.us-west-1.amazonaws.com/geth-public/geth-linux-amd64-0.9.2-ea9f0d2.tar.gz
tar -xzvf geth-linux-amd64-0.9.2-ea9f0d2.tar.gz
[ ! -d "$HOME/go/bin" ] && mkdir -p $HOME/go/bin
if ! grep -q "$HOME/go/bin" $HOME/.bash_profile; then
  echo 'export PATH=$PATH:$HOME/go/bin' >> $HOME/.bash_profile
fi
sudo cp geth-linux-amd64-0.9.2-ea9f0d2/geth $HOME/go/bin/story-geth
source $HOME/.bash_profile
story-geth version
```

## Tải xuống và cài đặt story (Download Story binary)

```
wget https://story-geth-binaries.s3.us-west-1.amazonaws.com/story-public/story-linux-amd64-0.9.11-2a25df1.tar.gz
tar -xzvf story-linux-amd64-0.9.11-2a25df1.tar.gz
[ ! -d "$HOME/go/bin" ] && mkdir -p $HOME/go/bin
if ! grep -q "$HOME/go/bin" $HOME/.bash_profile; then
  echo 'export PATH=$PATH:$HOME/go/bin' >> $HOME/.bash_profile
fi
sudo cp story-linux-amd64-0.9.11-2a25df1/story $HOME/go/bin/story
source $HOME/.bash_profile
story version
```

## Khởi tạo Story (Initiate Iliad node)

Sữa lại "Your_moniker_name" theo tên bạn mong muốn 
(Ví dụ: story init --network iliad --moniker alberthung_story)

```
story init --network iliad --moniker "Your_moniker_name"
```

## 3. Cấu hình Dịch vụ
## Cấu hình dịch vụ story-geth (Create story-geth service file)
```
sudo tee /etc/systemd/system/story-geth.service > /dev/null <<EOF
[Unit]
Description=Story Geth Client
After=network.target

[Service]
User=root
ExecStart=/root/go/bin/story-geth --iliad --syncmode full
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```
## Cấu hình dịch vụ story (Create story service file)

```
sudo tee /etc/systemd/system/story.service > /dev/null <<EOF
[Unit]
Description=Story Consensus Client
After=network.target

[Service]
User=root
ExecStart=/root/go/bin/story run
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
Nếu ko check được bằng lệnh trên thì check bằng lệnh này:
```
curl -s localhost:26657/status | jq
```
### Chú ý: nếu 2 lệnh trên không chạy được thì cài cái "jp" bằng lệnh này "sudo apt install jq" => chọn "Y" => chạy lại lệnh "curl -s localhost:26657/status | jq" là ok

# Sync đến block gần nhất (SYNC using snapshot File)

### By Joseph Tran

### 1. Install tool
```
sudo apt-get install wget lz4 aria2 pv -y
```
### 2. Stop node
```
sudo systemctl stop story
sudo systemctl stop story-geth
```
### 3. Download Geth-data
```
cd $HOME
rm -f Geth_snapshot.lz4
if curl -s --head https://vps6.josephtran.xyz/Story/Geth_snapshot.lz4 | head -n 1 | grep "200" > /dev/null; then
    echo "Snapshot found, downloading..."
    aria2c -x 16 -s 16 https://vps6.josephtran.xyz/Story/Geth_snapshot.lz4 -o Geth_snapshot.lz4
else
    echo "No snapshot found."
fi
```
### 4. Download Story-data
```
cd $HOME
rm -f Story_snapshot.lz4
if curl -s --head https://vps6.josephtran.xyz/Story/Story_snapshot.lz4 | head -n 1 | grep "200" > /dev/null; then
    echo "Snapshot found, downloading..."
    aria2c -x 16 -s 16 https://vps6.josephtran.xyz/Story/Story_snapshot.lz4 -o Story_snapshot.lz4
else
    echo "No snapshot found."
fi
```
### Backup priv_validator_state.json:
```
mv $HOME/.story/story/data/priv_validator_state.json $HOME/.story/priv_validator_state.json.backup
```
### Remove old data
```
rm -rf ~/.story/story/data
rm -rf ~/.story/geth/iliad/geth/chaindata
```
### Extract Story-data
```
sudo mkdir -p /root/.story/story/data
```
```
lz4 -d Story_snapshot.lz4 | pv | sudo tar xv -C /root/.story/story/
```
### Extract Geth-data
```
sudo mkdir -p /root/.story/geth/iliad/geth/chaindata
```
```
lz4 -d Geth_snapshot.lz4 | pv | sudo tar xv -C /root/.story/geth/iliad/geth/
```

### Move priv_validator_state.json back
```
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
story validator export --export-evm-key
```
### 2. Private key preview
```
sudo nano ~/.story/story/config/private_key.txt
```
Hoặc dùng lệnh này cũng lấy đc private key: 
```
cat /root/.story/story/config/private_key.txt
```
### 3. Import Priavte Key to Metamask 

Get the wallet address for faucet

### 4. You need at least have 10 IP on wallet

Get it from faucet : https://faucet.story.foundation/

## Chờ khi nào trạng thái "catchin_up": false' (Check the sync the catching up must be 'false')
Dùng lệnh này để check Sync
```
curl -s localhost:26657/status | jq
```
Chỉ stake khi trạng thái "catching_up": false

### 5. Validator registering

Replace "your_private_key" with your key from the step2

```
story validator create --stake 1000000000000000000 --private-key "your_private_key"
```

### 6. Check your validator INFO
```
curl -s localhost:26657/status | jq -r '.result.validator_info' 
```

### 7. check your validator

Explorer: https://testnet.story.explorers.guru/
Paste HEX Validator Address to search

# BACK UP FILE

### 1. Wallet private key:
```
sudo nano ~/.story/story/config/private_key.txt
```
### 2. Validator key:

```
sudo nano ~/.story/story/config/priv_validator_key.json
```

# 7. Quản lý Dịch vụ
### 7.1. Khởi động dịch vụ
```
sudo systemctl daemon-reload
sudo systemctl start story-geth.service
sudo systemctl start story.service
````
### 7.2. Dừng dịch vụ
```
sudo systemctl stop story-geth.service
sudo systemctl stop story.service
````
### 7.3. Kích hoạt dịch vụ tự động khởi động cùng hệ thống
```
sudo systemctl enable story-geth.service
sudo systemctl enable story.service
````
### 8. Khắc phục sự cố 
Lỗi khi khởi động dịch vụ: Kiểm tra logs dịch vụ bằng lệnh 
```
journalctl -u story-geth.service
journalctl -u story.service
```

# 9. Delete node
WARNING: Backup your data, private key, validator key before remove node.
```
sudo systemctl stop story-geth
sudo systemctl stop story
sudo systemctl disable story-geth
sudo systemctl disable story
sudo rm /etc/systemd/system/story-geth.service
sudo rm /etc/systemd/system/story.service
sudo systemctl daemon-reload
sudo rm -rf $HOME/.story
sudo rm $HOME/go/bin/story-geth
sudo rm $HOME/go/bin/story
```
# Join our TG : https://t.me/Crypto_Confessions
