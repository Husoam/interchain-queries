<p style="font-size:14px" align="right">
<a href="https://t.me/StrideTurkish" target="_blank">Join Stride Turkish Official Telegram <img src="https://user-images.githubusercontent.com/50621007/183283867-56b4d69f-bc6e-4939-b00a-72aa019d1aea.png" width="30"/></a>
<a href="https://discord.gg/rwvZWAru" target="_blank">Join Stride discord <img src="https://user-images.githubusercontent.com/50621007/176236430-53b0f4de-41ff-41f7-92a1-4233890a90c8.png" width="30"/></a>
<a href="https://stride.zone/" target="_blank">Visit Stride Zone <img src="https://user-images.githubusercontent.com/50621007/168689709-7e537ca6-b6b8-4adc-9bd0-186ea4ea4aed.png" width="30"/></a>
</p>
<p align="center">
  <img height="500" height="auto" src="https://user-images.githubusercontent.com/98665693/185979661-5393a4cb-4f9e-4ee1-bd7c-437282817620.JPG">
</p>

# Relay interchain-queries using the new GO v2 Relayer

## 1. Download and build binaries
```
cd $HOME
git clone https://github.com/Stride-Labs/interchain-queries.git
cd interchain-queries
go build
sudo mv interchain-queries /usr/local/bin/icq
```
## 2. Make home dir for icq 
```
cd $HOME && mkdir .icq
```

## 3. Create configurations file
```
sudo tee $HOME/.icq/config.yaml > /dev/null <<EOF
default_chain: stride-testnet
chains:
  gaia-testnet:
    key: wallet                           # your wallet name
    chain-id: GAIA
    rpc-addr: http://127.0.0.1:23657      # use your own Gaia RPC endpoint here:  echo "$(curl -s ifconfig.me)$(grep -A 3 "\[rpc\]" ~/.gaia/config/config.toml | egrep -o ":[0-9]+")"
    grpc-addr: http://127.0.0.1:23090     # use your own Gaia GRPC endpoint here:  echo "$(curl -s ifconfig.me)$(grep -A 6 "\[grpc\]" ~/.gaia/config/app.toml | egrep -o ":[0-9]+")"
    account-prefix: cosmos
    keyring-backend: test
    gas-adjustment: 1.2
    gas-prices: 0.001uatom
    key-directory: /root/.icq/keys
    debug: false
    timeout: 20s
    block-timeout: ""
    output-format: json
    sign-mode: direct
  stride-testnet:
    key: wallet                           # your wallet name
    chain-id: STRIDE-TESTNET-4
    rpc-addr: http://127.0.0.1:16657      # use your own Stride RPC endpoint here (nano $HOME/.stride/config/config.toml)
    grpc-addr: http://127.0.0.1:16090     # use your own Stride GRPC endpoint here (nano $HOME/.stride/config/app.toml)
    account-prefix: stride
    keyring-backend: test
    gas-adjustment: 1.2
    gas-prices: 0.001ustrd
    key-directory: /root/.icq/keys
    debug: false
    timeout: 20s
    block-timeout: ""
    output-format: json
    sign-mode: direct
cl: {}
EOF
```
##  For Parsing Error
Make sure your .icq/config.yaml file is valid.Like this.

![image](https://user-images.githubusercontent.com/98665693/185976836-fc1783b7-c1e8-42fa-8e07-6cce9ee5f653.JPG)


## 3. Import wallets
> NOTE: Please use the same wallet you have used for relayer.
```
icq keys restore --chain stride-testnet <your_wallet_name>
icq keys restore --chain gaia-testnet <your_wallet_name>
```
 ## If you get "icq not found" error
  ### Delete all files of existing icq installation
  ```
cd $HOME
rm -rf interchain-queries
rm -rf /usr/local/bin/icq
rm -rf .icq
rm -rf /etc/systemd/system/icqd.service
```
  ### Install GO
  ```
wget -c https://go.dev/dl/go1.18.3.linux-amd64.tar.gz && rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.3.linux-amd64.tar.gz && rm -rf go1.18.3.linux-amd64.tar.gz
```
```
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
```
## And Install icq again and import wallet
 > NOTE:  In some cases, it was seen that the wallet was imported after trying several times, try it. 
 
## 4. Create icq service
```
sudo tee /etc/systemd/system/icqd.service > /dev/null <<EOF
[Unit]
Description=Interchain Query Service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which icq) run --debug
Restart=on-failure
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

## 5. Start icq service
```
sudo systemctl daemon-reload
sudo systemctl enable icqd
sudo systemctl restart icqd
```
## 6. Update Clients
```
rly transact update-clients stride-gaia
```

## 7. Check icq logs
  ### Screen(optional)
``` 
screen -S icq
``` 
```
journalctl -u icqd -f -o cat
```
You will have to wait some (20 min.) and you should see a log like this ;
```
ICQ RELAYER | query.Height= 0
ICQ RELAYER | res.Height= 62305
ICQ RELAYER | query.Height= 0
ICQ RELAYER | res.Height= 62305
Send batch of 4 messages
1 ClientUpdate message
1 SubmitResponse message

```

After that you can check you transaction in the explorer

![image](https://user-images.githubusercontent.com/98665693/185976366-330ba597-4314-4343-90b5-e24c672214de.JPG)

## 7. You can transfer and check explore hash
```
rly transact transfer gaia stride 1000uatom <yourstrideaddress> channel-0 --path stride-gaia
```
```
rly transact transfer stride gaia 1000ustrd <yourcosmosaddress> channel-0 --path stride-gaia
```
## ENJOY 
<p align="center">
  <img height="100" height="auto" src="https://user-images.githubusercontent.com/98665693/185981326-60dfb817-dc46-4273-8793-0ea3b7a6bae9.jpg">
</p>
