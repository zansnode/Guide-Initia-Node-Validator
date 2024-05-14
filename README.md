## Guide-Initia-Node-Validator
## Testnet Instruction Node Validator

### Required installations
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget build-essential jq make lz4 gcc unzip -y
```

### Install Go
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

### Install And Build Binary
```
git clone https://github.com/initia-labs/initia
cd initia
git checkout v0.2.11
make install
```

### Moniker 
```
initiad init (Ur-Moniker) --chain-id initiation-1
```

### Install Genesis.json
```
rm ~/.initia/config/genesis.json
wget https://initia.s3.ap-southeast-1.amazonaws.com/initiation-1/genesis.json
cp genesis.json ~/.initia/config/genesis.json
```

### After your used command above, use command below
```
sed -i -e 's/external_address = \"\"/external_address = \"'$(curl httpbin.org/ip | jq -r .origin)':26656\"/g' ~/.initia/config/config.toml
```

### Set Minim Gas Price
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.15uinit,0.01uusdc\"/" $HOME/.initia/config/app.toml
```

### Seeds Peers
```
PEERS="40d3f977d97d3c02bd5835070cc139f289e774da@168.119.10.134:26313,841c6a4b2a3d5d59bb116cc549565c8a16b7fae1@23.88.49.233:26656,e6a35b95ec73e511ef352085cb300e257536e075@37.252.186.213:26656,2a574706e4a1eba0e5e46733c232849778faf93b@84.247.137.184:53456,ff9dbc6bb53227ef94dc75ab1ddcaeb2404e1b0b@178.170.47.171:26656,edcc2c7098c42ee348e50ac2242ff897f51405e9@65.109.34.205:36656,07632ab562028c3394ee8e78823069bfc8de7b4c@37.27.52.25:19656,028999a1696b45863ff84df12ebf2aebc5d40c2d@37.27.48.77:26656,140c332230ac19f118e5882deaf00906a1dba467@185.219.142.119:53456,1f6633bc18eb06b6c0cab97d72c585a6d7a207bc@65.109.59.22:25756,065f64fab28cb0d06a7841887d5b469ec58a0116@84.247.137.200:53456,767fdcfdb0998209834b929c59a2b57d474cc496@207.148.114.112:26656,093e1b89a498b6a8760ad2188fbda30a05e4f300@35.240.207.217:26656,12526b1e95e7ef07a3eb874465662885a586e095@95.216.78.111:26656" && \
SEEDS="2eaa272622d1ba6796100ab39f58c75d458b9dbc@34.142.181.82:26656,c28827cb96c14c905b127b92065a3fb4cd77d7f6@testnet-seeds.whispernode.com:25756" && \
sed -i \
    -e "s/^seeds *=.*/seeds = \"$SEEDS\"/" \
    -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" \
    "$HOME/.initia/config/config.toml"
```

### Set Up Pruning
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.initia/config/app.toml
```

### Create A Service
```
sudo tee /etc/systemd/system/initiad.service > /dev/null <<EOF
[Unit]
Description=initiad
After=network-online.target

[Service]
User=$USER
ExecStart=$(which initiad) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### Start Service & Reload System Config
```
sudo systemctl daemon-reload
sudo systemctl enable initiad  
sudo systemctl restart initiad  && sudo journalctl -fu initiad  -o cat
```

## Manage Wallet

### Make New Wallet
```
initiad keys add wallet
```

### Recovery Your Wallet
```
initiad keys add wallet --recover
```

### Get Your Wallet List
```
initiad keys list
```

### Check Balance of Your Wallet
```
initiad q bank balances $(initiad keys show wallet -a)
```

## Validator 

### Node Info
before Create Validator, you have to make sure that blocks have syncs. use command below to check.
```
initiad status 2>&1 | jq .SyncInfo
```
If it shows false, the sign is that you can create a validator

### Create Validator
```
initiad tx mstaking create-validator \
    --amount=5000000uinit \
    --pubkey=$(initiad tendermint show-validator) \
    --moniker="(Your-Moniker)" \
    --chain-id=initiation-1 \
    --from=(Name-of-Your-wallet) \
    --commission-rate="0.10" \
    --commission-max-rate="0.20" \
    --commission-max-change-rate="0.01" \
    --identity=(Your-Keybase-id)
```
after created validator, Dont forget to save your priv_validator_key.json its only way to recover your validator 

### Validator Info
```
initiad q staking validator $(initiad keys show $WALLET --bech val -a)
```

### Faucet 
https://faucet.testnet.initia.xyz/

### Delegate Token To Your Validator
```
initiad tx staking delegate $(initiad keys show wallet --bech val -a) 1000000uinit --from wallet -y
```

### Unjail
```
initiad tx slashing unjail --from wallet --fees=0.025uinit -y
```

### Restart System
```
sudo systemctl restart initiad
```

### Stopping System
```
sudo systemctl stop initiad
```

### Check Log
```
sudo journalctl -u initiad -f -o cat
```

### Remove System
Reminder before remove your system, make sure you have to backup all important file!!!
```
sudo systemctl stop initiad.service
sudo systemctl disable initiad.service
sudo rm /etc/systemd/system/initiad.service
rm -rf $HOME/.initia $HOME/initia
```

# CONGRATS YOU HAVE RUN NODE AND CREATE VALIDATOR ON INITIA CHAIN
