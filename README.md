# Babylon

#### Discord https://discord.gg/babylonglobal
### Babylon - это новый проект Cosmos, целью которого является использование безопасности Bitcoin для повышения безопасности зон Cosmos и других цепочек PoS. 
#### Ссылка на команду https://babylonchain.io/about
#### Сайт: https://babylonchain.io/foundation
#### Твиттер: https://www.twitter.com/babylon_chain
#### GitHub https://github.com/babylonchain/babylonchain.github.io
#### Zealy https://zealy.io/c/babylonchain/invite/1zn87lyrLTOaCWHZgpHR8

### Требования к серверу 
#### Minimum 3CPU 4RAM 80GB
#### Recommended 4CPU 8RAM 160GB

## как вариант заказать здесь https://pq.hosting/?from=36405

## Обновление пакетов сервера и подготовка к развертыванию ноды

```
sudo apt update && sudo apt upgrade -y 
```
```
sudo apt install make clang pkg-config libssl-dev build-essential git gcc chrony curl jq ncdu bsdmainutils htop net-tools lsof fail2ban wget -y 
```
```
sudo apt update 
sudo apt install -y curl git jq lz4 build-essential unzip 
```
```
# Install Go
bash <(curl -s "https://raw.githubusercontent.com/nodejumper-org/cosmos-scripts/master/utils/go_install.sh") 
source .bash_profile 
```

```
# Clone project repository
cd $HOME
rm -rf babylon
git clone https://github.com/babylonchain/babylon
cd babylon
git checkout v0.7.2
```

```
# Build binary
make install
```

```
# Set node CLI configuration
babylond config chain-id bbn-test-2
babylond config keyring-backend test
```

# Задать имя ноды !!!!
```
# Initialize the node
babylond init "ЗАДАЕМ_ИМЯ_НОДЫ" --chain-id bbn-test-2
```

```
# Download genesis and addrbook files
curl -L https://snapshots-testnet.nodejumper.io/babylon-testnet/genesis.json > $HOME/.babylond/config/genesis.json
curl -L https://snapshots-testnet.nodejumper.io/babylon-testnet/addrbook.json > $HOME/.babylond/config/addrbook.json
```

```
# Set seeds and peers
sed -i \
  -e 's|^seeds *=.*|seeds = ""|' \
  -e 's|^peers *=.*|peers = "03ce5e1b5be3c9a81517d415f65378943996c864@18.207.168.204:26656,a5fabac19c732bf7d814cf22e7ffc23113dc9606@34.238.169.221:26656,ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:20656"|' \
  $HOME/.babylond/config/config.toml
```

```
# Set minimum gas price
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001ubbn"|' $HOME/.babylond/config/app.toml

# Set pruning
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.babylond/config/app.toml

# Set additional configs
sed -i 's|^network *=.*|network = "mainnet"|g' $HOME/.babylond/config/app.toml

# Download latest chain data snapshot
curl "https://snapshots-testnet.nodejumper.io/babylon-testnet/babylon-testnet_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.babylond"

```

```
# Create a service
sudo tee /etc/systemd/system/babylond.service > /dev/null << EOF
[Unit]
Description=Babylon node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which babylond) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```


## меняем порты на не стандартные
```
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:31658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:31657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:6560\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:31656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":31660\"%" $HOME/.babylond/config/config.toml && sed -i.bak -e "s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:9590\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:9591\"%; s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:1817\"%; s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:9045\"%; s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:9046\"%; s%^address = \"127.0.0.1:8545\"%address = \"127.0.0.1:9045\"%; s%^ws-address = \"127.0.0.1:8546\"%ws-address = \"127.0.0.1:9046\"%" $HOME/.babylond/config/app.toml && sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:31657\"%" $HOME/.babylond/config/client.toml 
```

```
sudo systemctl daemon-reload
sudo systemctl enable babylond.service
```

```
# Start the service and check the logs
sudo systemctl start babylond.service
sudo journalctl -u babylond.service -f --no-hostname -o cat
```
## ожидаем коннекта и ждем полной синхронизации

### текущую высоту можно посмотреть здесь и сравнить с вашей в логах
```
https://babylon.explorers.guru/blocks
```

#### Если долго не может законнектиться к пирам, подкиньте новых пиров

```
PEERS="eb8ed707d57c8919624bdd9bd2d3d568e3f32a82@38.242.157.187:16456,5d3272610cb846d186d0876f1f20fb723299b5fb@62.171.169.28:16456,a05b93b6c0179484864db8c6462d1021d2bbf93b@75.119.153.199:16456,b6108bef2a77494c963379dc4c972fd5326c357c@45.88.188.32:16456,a98484ac9cb8235bd6a65cdf7648107e3d14dab4@116.202.231.58:16456,6c564cf63e0835a559ed4df9ee1ea79536abd1a9@45.85.147.23:26656,be2e243703485f58c3fa49d17fb503e4925aea2f@65.109.17.23:56114,e3972829d4403be7a9e9f31d19e741c0fd14a6bf@167.86.127.76:16456,a45344a36fa02187ddb6f3741e3c8425a19d07ba@62.171.186.40:31156,c113436716a5b5bb4057fd0d8791a87472d2b4c8@185.215.180.192:16456,90a04f4da53444fd65a2dbbb028ceca4418ab391@161.97.108.17:16456,cf8696279fa00c4c28d38cf93553b63837ba6e9f@38.242.195.229:16456,ac5ab2bf659cf55b8215f01943bdb89e1ebc4773@213.226.100.186:16456,1a03bfa239c7852fd71f2bea0b03c08245e1ee57@37.27.21.222:31156,6096d615b909df7ebb881d3a755eba1df5f59959@95.216.192.172:31656,2912157ed5a857b32f4fc52cf2c7f9bb714b0b00@92.50.136.246:16456,a34f6e8692f0aa1cefdadc079a0d7c72c641b286@109.228.160.58:16456,bbf8ef70a32c3248a30ab10b2bff399e73c6e03c@65.21.198.100:21756,24feb8c9c712796a6d0f40c65f5fbd1ed122f16a@95.216.241.246:43656,317cc19614c7f67bc3753756239e4531d081c1ea@164.68.126.242:16456,f3f13d544388ccf7eb8cfba0e7345332c35221b9@38.242.134.89:26656,79872752b7899f9598a43400bc3a9781800113c0@157.90.160.182:31156,87f942596810d8b2f8a86470caef0d7855813f89@65.109.48.247:09656,791fa91aab32a11100ebc5961d87d33455035146@88.204.124.44:16456,c626e138e60bbd75af8439c3bcb2cc415ff9671f@185.245.182.241:16456,0b3582f10e7d5d9b5df8b7dadb931489a5dc3283@161.97.151.149:16456,e50bab9869d4d200bbec113da8bbe39f1991096a@144.91.88.230:16456,85eebce42b8bdf6dd85b911a82958202110fa8c3@75.119.137.195:16456,1a62a090bbfbf8ea800706da09c6de35945f8259@86.48.0.190:12656,239b03e4917bf025592dddafbba5eb79e9f29cdb@213.133.100.172:26811,2e7773b7d57e38bb6d77131c8f5bd8d3839b923a@142.132.200.248:16456,dd99d99ed5c26a8ba3b1d2c64d82b5565c7bbce0@89.116.28.152:16456,d8a71c2b1901ee19c6cf8d2bbb14ee3dda5adab5@62.171.172.25:31156,2a2c298993c5f751acdc787edfd9659679b30f75@38.242.193.124:26656,3cacc1043c68467339b0212621c9888cee77623a@104.251.222.122:16456,21af6c66c8ab043e0d9b35bcbeb3ed7b12e96232@168.119.77.61:16456,4e0d48e4487a4672329c5ce13fa09681731cdcad@161.97.108.25:16456,dedf6b68fdc14e978c2b93553f42cdaa7f6e1fc2@174.2.80.247:16456,16f033e6a8ee599948f2ab9349899ef2bbded61a@65.109.70.45:27656,552993f488b6e8bb0ee2cd567903769cff99fb13@144.91.91.74:16456,b4abb45ce5d0367a192189207ee90ac51647bb2d@195.14.6.2:26656,99b32eeed7aadfb3ea2cdd85d4b2d801373b15b4@109.123.254.213:16456"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.babylond/config/config.toml

sudo systemctl restart babylond
sudo journalctl -u babylond -f --no-hostname -o cat
``` 

# Создаем кошелек в ноде с именем wallet  и сохраняем себе все данные, адрес и сид фразу !!!!!!!

```
# create wallet
babylond keys add wallet
```

## записываем валидатор ключ

```
cat $HOME/.babylond/config/priv_validator_key.json
```

## проверить статус синхронизации так же можно командой ниже, статус FALSE означает что синхронизация окончена, если TRUE - то еще идет. Ждем пока будет FALSE И ТОЛЬКО ПОСЛЕ ЭТОГО ПРИСТУПАЕМ К СОЗДАНИЮ ВАЛИДАТОРА

```
# wait util the node is synced, should return FALSE
babylond status 2>&1 | jq .SyncInfo.catching_up
```

## далее идем в дискорд проекта в ветку FAUCET и запрашиваем токены введя команду

```
!faucet адрес-вашего-кошелька от ноды
```

## Ждем пару минут и проверяем в ноде

```
# verify the balance
babylond q bank balances $(babylond keys show wallet -a)
```

## далее выполняем данную команду, рестартим ноду и смотрим логи, ожидаем пока нода снова законнектится и засинхронизируется

```
babylond create-bls-key $(babylond keys show wallet -a)
sudo systemctl restart babylond
```
## проверяем логи
```
sudo journalctl -u babylond.service -f --no-hostname -o cat
```

## Убеждаемся что нода синхронизирована, есть токены на кошельке приступаем к созданию валидатора, сначали измените ИМЯ НОДЫ!!!!

```
# create validator
babylond tx checkpointing create-validator \
--amount=1000000ubbn \
--pubkey=$(babylond tendermint show-validator) \
--moniker="ЗДЕСЬ УКАЗАТЬ ИМЯ ВАШЕЙ НОДЫ КАК ЗАДАВАЛИ ВЫШЕ" \
--details="https://t.me/WingsNodeTeam" \
--chain-id=bbn-test-2 \
--commission-rate=0.1 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.05 \
--min-self-delegation=1 \
--fees=2000ubbn \
--from=wallet \
-y
```

## статус cозданного валидатора можно проверить в ноде

```
babylond q staking validator $(babylond keys show wallet --bech val -a)
```


## Ждем до 1 часа и проверяем в эксплоере по имени ноды (номикера) здесь во вкладке неактив
```
https://babylon.explorers.guru/validators
```


### Токен для валидатора можно запросить в дискорде 1 раз в 24 часа. запрашивайте каждый день и валидируйте в вашего валидатора, продвигая его все выше. Чем больше вы застейкаете тем выше вы будете в списке валмдаторов и попадете в активные валидаторы

```
# Делегировать валидатору
    babylond tx epoching delegate $(babylond keys show wallet --bech val -a) 1000000ubbn --from wallet --chain-id bbn-test-2 --gas-adjustment 1.4 --gas auto --fees 200ubbn -y
```
### сумма 1000000 это 1 токен (шесть нулей)

## Удалить ноду

```
sudo systemctl stop babylond && sudo systemctl disable babylond && sudo rm /etc/systemd/system/babylond.service && sudo systemctl daemon-reload && rm -rf $HOME/.babylond && rm -rf babylon && sudo rm -rf $(which babylond)
```

## Следите за новостями в дискорде проекта, а так же вступайте в наш канал телеграм https://t.me/WingsNodeTeam
