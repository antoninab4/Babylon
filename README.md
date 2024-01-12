# Babylon

## Discord https://discord.gg/babylonglobal
## Babylon - это новый проект Cosmos, целью которого является использование безопасности Bitcoin для повышения безопасности зон Cosmos и других цепочек PoS. 
## Ссылка на команду https://babylonchain.io/about
## Сайт: https://babylonchain.io/foundation
## Твиттер: https://www.twitter.com/babylon_chain
## GitHub https://github.com/babylonchain/babylonchain.github.io

### Требования к серверу 
#### Minimum 3CPU 4RAM 80GB
#### Recommended 4CPU 8RAM 160GB

## как вариант заказать здесь https://pq.hosting/?from=36405

## Обновление пакетов сервера и подготовка к развертыванию ноды

```
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

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

## Следите за новостями в дискорде проекта, а так же вступайте в наш канал телеграм https://t.me/WingsNodeTeam
