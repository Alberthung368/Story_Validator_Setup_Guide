# Hướng dẫn Cài đặt và Chạy Node cho Dự án Story (Story_Protocol_Validator Node Setup Guide Full)

Info: Story raised $140M from Tier1 investors. Story is a blockchain making IP protection and licensing programmable and efficient. It automates IP management, allowing creators to easily license, remix, and monetize their work. With Story, traditional legal complexities are replaced by on-chain smart contracts and off-chain legal agreements, simplifying the entire process.

## 1. **Thông số Phần cứng anh em mua VPS** (System Requirements)

Mọi người có thể mua ở đây bằng USDT, ETH, BTC... 
https://pq.hosting/?from=719019

| **Hardware** | **Minimum Requirement** |
|--------------|-------------------------|
| **CPU**      | 4 Cores                 |
| **RAM**      | 8 GB                    |
| **Disk**     | 800 GB                  |
| **Bandwidth**| 10 MBit/s               |

Tham gia với chúng tôi trên group telegram => https://t.me/Crypto_Confessions

## **Ghi chú**

- Hệ điều hành trên VPS, chọn: Ubuntu 22.04
- Công cụ cài đặt trên Windown: Bitvise => tải tại đây => https://bitvise.com/ssh-client-download
- Điền thông tin khi mở phần mềm Bitvise trên PC như hình:
- ![image](https://github.com/user-attachments/assets/34ac61de-a8f5-4a78-b8fe-22610d100954)

- Host: điền VPS của bạn được gửi trong email
- Port: 22
- Username: root
- Initial method: pass được gửi trong mail 
- => Accept and Save
 
## 2. Cài đặt (Install)

### 2.1. Cài đặt các công cụ cần thiết (Install dependencies)
```
sudo apt update
sudo apt-get update
sudo apt install curl git make jq build-essential gcc unzip wget lz4 aria2 pv -y
```

## 2.2 Tạo thư mục $HOME/go/bin (Install Go)
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

## 2.3. Tải xuống và cài đặt story-geth (Download Story-Geth binary)

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

## 2.4. Tải xuống và cài đặt story (Download Story binary)

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

## 2.5. Khởi tạo Story (Initiate Iliad node)

Sữa lại "Your_moniker_name" theo tên bạn mong muốn 
(Ví dụ: story init --network iliad --moniker alberthung_story)

```
story init --network iliad --moniker "Your_moniker_name"
```

## 3. Cấu hình Dịch vụ
## 3.1. Cấu hình dịch vụ story-geth (Create story-geth service file)
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
## 3.2. Cấu hình dịch vụ story (Create story service file)

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

## 3.3. Reload and start story-geth
```
sudo systemctl daemon-reload && \
sudo systemctl enable story-geth && \
sudo systemctl enable story && \
sudo systemctl start story-geth && \
sudo systemctl start story && \
sudo systemctl status story-geth
```

# 4. Check logs

## 4.1 Geth logs
Mở tab riêng để check
```
sudo journalctl -u story-geth -f -o cat
```
Chờ hệ thống kết nối peers
- Nếu không kết nối đc thì chạy đoạn Peers sau: Live Peers
```
PEERS=$(curl -s -X POST https://rpc-story.josephtran.xyz -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"net_info","params":[],"id":1}' | jq -r '.result.peers[] | select(.connection_status.SendMonitor.Active == true) | "\(.node_info.id)@\(if .node_info.listen_addr | contains("0.0.0.0") then .remote_ip + ":" + (.node_info.listen_addr | sub("tcp://0.0.0.0:"; "")) else .node_info.listen_addr | sub("tcp://"; "") end)"' | tr '\n' ',' | sed 's/,$//' | awk '{print "\"" $0 "\""}')

sed -i "s/^persistent_peers *=.*/persistent_peers = $PEERS/" "$HOME/.story/story/config/config.toml"

if [ $? -eq 0 ]; then
    echo -e "Configuration file updated successfully with new peers"
else
    echo "Failed to update configuration file."
fi
```
- Chạy lại:
```
sudo journalctl -u story-geth -f -o cat
```
## 4.2 Story logs
Mở tab riêng để check
```
sudo journalctl -u story -f -o cat
```
Chờ hệ thống chạy
## 4.3 Check sync status

```
curl localhost:26657/status | jq
```
Nếu ko check được bằng lệnh trên thì check bằng lệnh này:
```
curl -s localhost:26657/status | jq
```
- Check được như hình này là đã chạy thành công.
- ![image](https://github.com/user-attachments/assets/76088368-5e0e-458a-99ab-79af783ac7e4)

### Note: nếu dùng 2 lệnh trên không chạy được thì cài "jp" bằng lệnh này ```sudo apt install jq``` => chọn "Y" => chạy lại lệnh ```curl -s localhost:26657/status | jq``` là thành công

## 4.4. Upgrade node
### Upgrade manually
- Download new binary
```
cd $HOME
wget https://story-geth-binaries.s3.us-west-1.amazonaws.com/story-public/story-linux-amd64-0.10.0-9603826.tar.gz
tar -xzvf story-linux-amd64-0.10.0-9603826.tar.gz
```
- Stop node
```
sudo systemctl stop story
```
- Copy binary to $HOME/go/bin to use it anywhere
```
cp $HOME/story-linux-amd64-0.10.0-9603826/story $HOME/go/bin
source $HOME/.bash_profile
story version
```
- Restart node
```
sudo systemctl daemon-reload && \
sudo systemctl start story && \
sudo systemctl status story
```

# 5. Sync đến block gần nhất trên hệ thống (SYNC using snapshot File)

## Tham khảo và trích lọc từ 'Joseph Tran': (https://service.josephtran.xyz/testnet/story/snapshot)

## 5.1. Cài đặt công cụ  (Install tool)
```
sudo apt-get install wget lz4 aria2 pv -y
```
## 5.2. Stop node
```
sudo systemctl stop story
sudo systemctl stop story-geth
```
## 5.3. Download Geth-data
```
cd $HOME
rm -f Geth_snapshot.lz4
wget --show-progress https://josephtran.co/Geth_snapshot.lz4
```
Chạy đoạn lệnh trên và đợi tải Geth-data về thành công 
## 5.4. Download Story-data
```
cd $HOME
rm -f Story_snapshot.lz4
wget --show-progress https://josephtran.co/Story_snapshot.lz4
```
Chạy đoạn lệnh trên và đợi tải Story-data về thành công.

## Backup priv_validator_state.json:
```
cp ~/.story/story/data/priv_validator_state.json ~/.story/priv_validator_state.json.backup
```
## Remove old data
```
rm -rf ~/.story/story/data
rm -rf ~/.story/geth/iliad/geth/chaindata
```
## 5.5. Extract Story-data
```
sudo mkdir -p /root/.story/story/data
```
```
lz4 -d Story_snapshot.lz4 | pv | sudo tar xv -C /root/.story/story/
```
Chờ giải nén xong rồi tiếp tục chạy lệnh dưới
## 5.6. Extract Geth-data
```
sudo mkdir -p /root/.story/geth/iliad/geth/chaindata
```
```
lz4 -d Geth_snapshot.lz4 | pv | sudo tar xv -C /root/.story/geth/iliad/geth/
```
Chờ chờ giải nén xong rồi chạy lệnh dưới 

## Move priv_validator_state.json back
```
cp ~/.story/priv_validator_state.json.backup ~/.story/story/data/priv_validator_state.json
```
## Restart node 
```
sudo systemctl start story
sudo systemctl start story-geth
```

# 6. Register your Validator
## 6.1. Export wallet:
```
story validator export --export-evm-key
```
## 6.2. Private key preview
```
sudo nano ~/.story/story/config/private_key.txt
```
Hoặc dùng lệnh này cũng lấy đc private key: 
```
cat /root/.story/story/config/private_key.txt
```
## 6.3. Import your Priavte Key to Metamask 

Get the wallet address for faucet

## 6.4. You need at least have 10 IP on wallet

Get it from faucet : https://faucet.story.foundation/

### Chú ý: Chờ khi nào trạng thái "catchin_up": chuyển từ "Truse" sang "false" rồi mới chạy lệnh stake IP ở mục 6.5 (Check the sync the catching up must be 'false')

Dùng lệnh này để check Sync
```
curl -s localhost:26657/status | jq
```
Chỉ stake khi trạng thái "catching_up": false

## 6.5. Validator registering
Replace "your_private_key" with your key from the step2

```
story validator create --stake 10000000000000000000 --private-key "your_private_key"
```
- 10000000000000000000 = 10 IP
- 
## 6.6. Check your validator INFO
```
curl -s localhost:26657/status | jq -r '.result.validator_info' 
```
## 6.7. check your validator

Explorer: https://testnet.story.explorers.guru/
Paste HEX Validator Address to search

### Các bạn làm hết đến bước 6.5 là đã chạy node thành công, tuy nhiên node chỉ chạy thành công thôi chứ chưa active node được, để active thì node phải stake số lượng IP lớn hơn validaytor của top 100.. Kiềm tra top 100 Validators active ở đây https://staking.story.foundation/ để biết chính xác số IP cần stake. 

## 6.8. Stake thêm IP trên node để node active (Validator Staking)
- Stake IP thêm vô node của các bạn (luu ý stake tối thiều 1024IP, còn để đủ cho node active thì stake số lượng phải lớn hon top 100 validator)

```
story validator stake \
   --validator-pubkey "VALIDATOR_PUB_KEY_IN_BASE64" \
   --stake 1024000000000000000000 \
   --private-key xxxxxxxxxx
```
- Ví dụ:
```
story validator stake \
 --validator-pubkey "AgmYC5fB37rzzu6e5TV4iHJOuGPSx5Pc/v6SFK1vpCKW" \
 --stake 1101000000000000000000 \
 --private-key 0xgsd.....1ukt4
```
- Đổi "VALIDATOR_PUB_KEY_IN_BASE64" thành của các bạn
![image](https://github.com/user-attachments/assets/6f4b91af-c72f-4bed-ac01-5df52a1641e4)
- Muốn xem thông tin như hình trên thì bạn dùng lệnh check Sync để lấy validator-pubkey-value
```
curl -s localhost:26657/status | jq
```
- Sau khi stake thêm 1101IP thì sẽ hiện thành công như hình này
- ![image](https://github.com/user-attachments/assets/2a586660-e3ec-4883-ac97-a846be03946a)

## 6.10. Unstake IP trên node của anh em: (làm giống mục 6.9)
```
story validator unstake \
   --validator-pubkey "VALIDATOR_PUB_KEY_IN_BASE64" \
   --unstake 1024000000000000000000 \
   --private-key xxxxxxxxxxxxxx
```
## 6.11. Unstake IP trên node của của người khác mà anh em đã stake vô:
**** Cách lấy thông tin của Validator:
- Top 100 Validator => https://testnet.story.explorers.guru/validators

- Ví dụ lấy thông tin của varlidator top 2 mà mình đã stake vào đây 1024IP.
1. Vào validator top 2 trên guru xem => https://testnet.story.explorers.guru/validator/storyvaloper1qzgsp7x8pgwm8gw42kq2w8v2tyn6egjcanaqp8

2. Xem và copy thông tin mục Operator (VALIDATOR_PUB_KEY_IN_BASE64): 
storyvaloper1qzgsp7x8pgwm8gw42kq2w8v2tyn6egjcanaqp8

3. Chạy lệnh trên terminal:
Ví dụ:
```
curl https://api-story-testnet.trusted-point.com/cosmos/staking/v1beta1/validators/storyvaloper1qzgsp7x8pgwm8gw42kq2w8v2tyn6egjcanaqp8 | jq .validator.consensus_pubkey.key
```

- Chú ý: "VALIDATOR_PUB_KEY_IN_BASE64" = "Operator info"
- Cách lấy Operator info: vào đây => https://testnet.story.explorers.guru/ rồi chọn validator mà anh em đã stake trong đó và xem (Xem hình)
- ![image](https://github.com/user-attachments/assets/fa8bd8a6-a7b2-4414-b184-823cf7073fde)

## Chúc các bạn thực hiện thành công

==========================================================================

# 7. MORE

## BACK UP FILE

## 7.1. Wallet private key:
```
sudo nano ~/.story/story/config/private_key.txt
```
## 7.2. Validator key:
```
sudo nano ~/.story/story/config/priv_validator_key.json
```
## 7.3. Quản lý Dịch vụ
### Khởi động dịch vụ
```
sudo systemctl daemon-reload
sudo systemctl start story-geth.service
sudo systemctl start story.service
````
### Dừng dịch vụ
```
sudo systemctl stop story-geth.service
sudo systemctl stop story.service
````
### Kích hoạt dịch vụ tự động khởi động cùng hệ thống
```
sudo systemctl enable story-geth.service
sudo systemctl enable story.service
````
### Khắc phục sự cố 
Lỗi khi khởi động dịch vụ: Kiểm tra logs dịch vụ bằng lệnh 
```
journalctl -u story-geth.service
journalctl -u story.service
```
- Nếu bạn muốn kiểm tra trạng thái của 'story' khi nó đang chạy, việc truy vấn nội bộ điểm cuối JSONRPC/HTTP sẽ rất hữu ích. Dưới đây là một số lệnh hữu ích để chạy:
```
curl localhost:26657/net_info | jq '.result.peers[].node_info.moniker'
```
Lệnh này sẽ cung cấp cho bạn danh sách các 'peer' đồng thuận mà node đang đồng bộ, được xác định bằng moniker.

```
curl localhost:26657/health
```
Lệnh này sẽ cho bạn biết node có đang hoạt động tốt hay không - dấu {} là đang hoạt động tốt.

# 8. Delete node
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
## Tham gia nhóm telegram cùng bọn mình để kiếm nhiều dự án hay hơn tại đây https://t.me/Crypto_Confessions
