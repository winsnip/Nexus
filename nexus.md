# Nexus-Beta

![ZenRock Banner](https://pbs.twimg.com/profile_banners/1508993482655866881/1723065167/1500x500)

## Links
- [Discord](https://discord.gg/tzTYqBCx)
- [Web](https://beta.nexus.xyz/)
- [Twitter](https://x.com/NexusLabsHQ/status/1800324588116860933)

## System Requirements

- **RAM**: 8 GB
- **CPU**: 4 cores
- **Disk Space**: 100 GB

## Installation ⚙️

# Auto Install

```bash
bash <(curl -s https://file.winsnip.xyz/file/uploads/zenrock.sh)
```

# Manual Install

```bash
# install go, if needed
cd $HOME
VER="1.23.1"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin

# set vars
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="test"" >> $HOME/.bash_profile
echo "export ZENROCK_CHAIN_ID="gardia-2"" >> $HOME/.bash_profile
echo "export ZENROCK_PORT="56"" >> $HOME/.bash_profile
source $HOME/.bash_profile

# download binary
cd $HOME
curl -o zenrockd https://releases.gardia.zenrocklabs.io/zenrockd-latest
chmod +x $HOME/zenrockd
mv $HOME/zenrockd $HOME/go/bin/

# config and init app
zenrockd init $MONIKER --chain-id $ZENROCK_CHAIN_ID
zenrockd config set client chain-id $ZENROCK_CHAIN_ID
zenrockd config set client node tcp://localhost:${ZENROCK_PORT}657

# download genesis and addrbook
wget -O $HOME/.zrchain/config/genesis.json https://server-5.itrocket.net/testnet/zenrock/genesis.json
wget -O $HOME/.zrchain/config/addrbook.json  https://server-5.itrocket.net/testnet/zenrock/addrbook.json

# set seeds and peers
SEEDS="50ef4dd630025029dde4c8e709878343ba8a27fa@zenrock-testnet-seed.itrocket.net:56656"
PEERS="5458b7a316ab673afc34404e2625f73f0376d9e4@zenrock-testnet-peer.itrocket.net:11656,e0b3dca981c062de699402ce56f56b6adea6a286@194.163.178.114:13656,a75e49f5185b6b31f0f14cc1385960347a369b2b@159.69.109.34:56656,ace905113977a21ff5e6f134ab95b959d95482ed@144.217.68.182:18256,a412c87699b151a02d1385cbb68b6df396a81c3f@95.217.196.224:26656,5d3e4d6fcb0a20bc3d627092d94bc3ab6bf64cea@109.199.109.133:56656,3e68a389ea37f829f8e2b78170deb1993a9e112e@135.181.139.249:20656,39bf4210b1e47b9df1de0d30a6ab94e18b7c4e9f@[2a03:cfc0:8000:13::b910:277f]:14156,6ef43e8d5be8d0499b6c57eb15d3dd6dee809c1e@52.30.152.47:26656,10100d10c3bbbbfc4bd9c607b24802657cbd5709@69.67.150.107:56656,129c7d519999863cf214e9cb8172d910bffca7e8@213.199.43.242:13656,fdf8971eaef4a440394dd6e123128ff953630775@161.97.130.234:656"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.zrchain/config/config.toml


# set custom ports in app.toml
sed -i.bak -e "s%:1317%:${ZENROCK_PORT}317%g;
s%:8080%:${ZENROCK_PORT}080%g;
s%:9090%:${ZENROCK_PORT}090%g;
s%:9091%:${ZENROCK_PORT}091%g;
s%:8545%:${ZENROCK_PORT}545%g;
s%:8546%:${ZENROCK_PORT}546%g;
s%:6065%:${ZENROCK_PORT}065%g" $HOME/.zrchain/config/app.toml

# set custom ports in config.toml file
sed -i.bak -e "s%:26658%:${ZENROCK_PORT}658%g;
s%:26657%:${ZENROCK_PORT}657%g;
s%:6060%:${ZENROCK_PORT}060%g;
s%:26656%:${ZENROCK_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${ZENROCK_PORT}656\"%;
s%:26660%:${ZENROCK_PORT}660%g" $HOME/.zrchain/config/config.toml

# config pruning
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.zrchain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.zrchain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.zrchain/config/app.toml

# set minimum gas price, enable prometheus and disable indexing
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0urock"|g' $HOME/.zrchain/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.zrchain/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.zrchain/config/config.toml

# create service file
sudo tee /etc/systemd/system/zenrockd.service > /dev/null <<EOF
[Unit]
Description=Zenrock node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.zrchain
ExecStart=$(which zenrockd) start --home $HOME/.zrchain
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

# reset and download snapshot
zenrockd tendermint unsafe-reset-all --home $HOME/.zrchain
if curl -s --head curl https://server-5.itrocket.net/testnet/zenrock/zenrock_2024-10-09_465456_snap.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl https://server-5.itrocket.net/testnet/zenrock/zenrock_2024-10-09_465456_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.zrchain
    else
  echo "no snapshot founded"
fi

# enable and start service
sudo systemctl daemon-reload
sudo systemctl enable zenrockd
sudo systemctl restart zenrockd && sudo journalctl -u zenrockd -f
```

## Service operations ⚙️

Check logs

```bash
sudo journalctl -u zenrockd -f
```

Start service

```bash
sudo systemctl start zenrockd
```

Stop service

```bash
sudo systemctl stop zenrockd
```

Restart service

```bash
sudo systemctl restart zenrockd
```

Check service status

```bash
sudo systemctl status zenrockd
```

Reload services

```bash
sudo systemctl daemon-reload
```

Enable Service

```bash
sudo systemctl enable zenrockd
```

Disable Service

```bash
sudo systemctl disable zenrockd
```

Node info

```bash
zenrockd status 2>&1| jq
```



## Key management

Add New Wallet

```bash
zenrockd keys add$WALLET
```

Restore executing wallet

```bash
zenrockd keys add$WALLET --recover
```

List All Wallets

```bash
zenrockd keys list
```

Delete wallet

```bash
zenrockd keys delete $WALLET
```

Check Balance

```bash
zenrockd q bank balances $WALLET_ADDRESS
```

Export Key (save to wallet.backup)

```bash
zenrockd keys export$WALLET
```

View EVM Prived Key

```bash
zenrockd keys unsafe-export-eth-key $WALLET
```

Import Key (restore from wallet.backup)

```bash
zenrockd keys import$WALLET wallet.backup
```

## Tokens

To valoper address To wallet address Amount, urock

Withdraw all rewards

```bash
zenrockd tx distribution withdraw-all-rewards --from $WALLET --chain-id gardia-2 --fees 30urock
```

Withdraw rewards and commission from your validator

```bash
zenrockd tx distribution withdraw-rewards $VALOPER_ADDRESS --from $WALLET --commission --chain-id gardia-2 --fees 30urock -y
```

Check your balance

```bash
zenrockd query bank balances $WALLET_ADDRESS
```

Delegate to Yourself

```bash
zenrockd tx staking delegate $(zenrockd keys show $WALLET --bech val -a) 1000000urock --from $WALLET --chain-id gardia-2 --fees 30urock -y
```

Delegate

```bash
zenrockd tx staking delegate <TO_VALOPER_ADDRESS> 1000000urock --from $WALLET --chain-id gardia-2 --fees 30urock -y
```

Redelegate Stake to Another Validator

```bash
zenrockd tx staking redelegate $VALOPER_ADDRESS<TO_VALOPER_ADDRESS> 1000000urock --from $WALLET --chain-id gardia-2 --fees 30urock -y
```

Unbond

```bash
zenrockd tx staking unbond $(zenrockd keys show $WALLET --bech val -a) 1000000urock --from $WALLET --chain-id gardia-2 --fees 30urock -y
```

Transfer Funds

```bash
zenrockd tx bank send $WALLET_ADDRESS<TO_WALLET_ADDRESS> 1000000urock --fees 30urock -y
```

## Validator operations

Moniker Identity Details Amount, urock Commission rate Commission max rate Commission max change rate

Create New Validator

```bash
zenrockd tx staking create-validator \--amount 1000000urock \--from $WALLET\--commission-rate 0.1\--commission-max-rate 0.2\--commission-max-change-rate 0.01\--min-self-delegation 1\--pubkey $(zenrockd tendermint show-validator)\--moniker "$MONIKER"\--identity ""\--details "Winsnip team"\--chain-id gardia-2 \--fees 30urock \-y
```

Edit Existing Validator

```bash
zenrockd tx staking edit-validator \--commission-rate 0.1\--new-moniker "$MONIKER"\--identity ""\--details "Winsnip team"\--from $WALLET\--chain-id gardia-2 \--fees 30urock \-y
```

Validator info

```bash
zenrockd status 2>&1| jq
```

Validator Details

```bash
zenrockd q staking validator $(zenrockd keys show $WALLET --bech val -a)
```

Jailing info

```bash
zenrockd q slashing signing-info $(zenrockd tendermint show-validator)
```

Slashing parameters

```bash
zenrockd q slashing params
```

Unjail validator

```bash
zenrockd tx slashing unjail --from $WALLET --chain-id gardia-2 --fees 30urock -y
```

Active Validators List

```bash
zenrockd q staking validators -oj --limit=2000| jq '.validators[] | select(.status=="BOND_STATUS_BONDED")'| jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " 	 " + .description.moniker'|sort -gr |nl
```

Check Validator key

```bash
[[$(zenrockd q staking validator $VALOPER_ADDRESS -oj | jq -r .consensus_pubkey.key)=$(zenrockd status | jq -r .ValidatorInfo.PubKey.value)]]&&echo -e "Your key status is ok"||echo -e "Your key status is error"
```

Signing info

```bash
zenrockd q slashing signing-info $(zenrockd tendermint show-validator)
```

## Governance

Title Description Deposit, urock

Create New Text Proposal

```bash
zenrockd  tx gov submit-proposal \--title ""\--description ""\--deposit 1000000urock \--type Text \--from $WALLET\--fees 30urock \-y
```

Proposals List

```bash
zenrockd query gov proposals
```

Proposal ID Proposal option Yes No No with veto Abstain

View proposal

```bash
zenrockd query gov proposal 1
```

Vote

```bash
zenrockd tx gov vote 1yes --from $WALLET --chain-id gardia-2  --fees 30urock -y
```