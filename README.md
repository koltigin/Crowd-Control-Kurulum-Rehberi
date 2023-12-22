# Crowd Control Cardchain Testnet3 Kurulum Rehberi 

![image](https://user-images.githubusercontent.com/102043225/179354101-a597974d-793b-4201-8e38-364a4ba13a3f.png)

## Gereksinimler (Minimum)
  * 2 CPU
  * 4GB RAM
  * 100 GB SSD (benim önerim)
  
## Sistemi Güncelleme
```shell
sudo apt update && sudo apt upgrade -y
```

## Gerekli Kütüphanelerin Kurulması
```shell
sudo apt install curl make build-essential gcc tmux jq chrony htop -y < "/dev/null"
```

## Go Kurulumu
```shell
ver="1.21.4"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
rm -rf /usr/local/go
tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm -rf "go$ver.linux-amd64.tar.gz"
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

## Değişkenleri Yükleme
aşağıda değiştirmeniz gereken yerleri yazıyorum.
* `$CROWD_NODENAME` validator adınız
* `$CROWD_WALLET` cüzdan adınız
*  Eğer portu başka bir node kullanıyorsa aşağıdan değiştirebilirsiniz.
```shell
echo "export CROWD_NODENAME=$CROWD_NODENAME"  >> $HOME/.bash_profile
echo "export CROWD_WALLET=$CROWD_WALLET" >> $HOME/.bash_profile
echo "export CROWD_PORT=18" >> $HOME/.bash_profile
echo "export CROWD_CHAIN_ID=cardtestnet-6" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Örnek
Node ve Cüzdan adımızın Mehmet olduğunu varsayalım. Kod aşağıdaki şekilde düzenlenecektir. 
```shell
echo "export CROWD_NODENAME=Mehmet"  >> $HOME/.bash_profile
echo "export CROWD_WALLET=Mehmet" >> $HOME/.bash_profile
echo "export CROWD_PORT=18" >> $HOME/.bash_profile
echo "export CROWD_CHAIN_ID=cardtestnet-6" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Cardchain Kurulumu
```shell
git clone https://github.com/DecentralCardGame/Cardchain
wget https://github.com/DecentralCardGame/Cardchain/releases/download/v0.11.0/Cardchaind
chmod +x Cardchaind
mv $HOME/Cardchaind /usr/local/bin
```

## Uygulamayı Yapılandırma ve Başlatma
```shell
Cardchaind config chain-id $CROWD_CHAIN_ID
Cardchaind init $CROWD_NODENAME --chain-id $CROWD_CHAIN_ID
```

## Genesis ve addrbook Dosyasının Kopyalanması
```shell
wget -O $HOME/.Cardchain/config/genesis.json http://45.136.28.158:3000/genesis.json 
wget -O $HOME/.Cardchain/config/addrbook.json https://raw.githubusercontent.com/koltigin/Crowd-Control-Kurulum-Rehberi/main/addrbook.json
```

## Minimum GAS Ücretinin Ayarlanması
```shell
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0ubpf\"/" $HOME/.Cardchain/config/app.toml
```

## Indexer'i Kapatma (Opsiyonel)
```shell
sed -i 's|^indexer *=.*|indexer = "null"|' $HOME/.Cardchain/config/config.toml
```

## SEED ve PEERS Ayarlanması
```shell
SEEDS="947aa14a9e6722df948d46b9e3ff24dd72920257@cardchain-testnet-seed.itrocket.net:31656"
PEERS="99dcfbba34316285fceea8feb0b644c4dc67c53b@cardchain-testnet-peer.itrocket.net:31656,d0e4edcdd73a7578b10980b3739a5b7218b7e86f@212.23.222.109:26256,1f0a4eac263a6c77ec7020dcdde1547af473df4f@185.249.227.91:26656,5caecb793facd1605f3973397367bf61e9bebdc9@135.181.220.61:11656,78a2c6a4f6aeab1f24681ee0864f4546b615b48e@194.163.179.176:12356,542d3d320d50d7e4d4fe8abb8950f346f10fb106@142.132.202.86:16001,8d4bbea97ba2deccc21cc205284411b086de85b3@167.86.68.204:39656,443c0471d3bb10717c8e0df5b0171c87c1d6ed9d@142.132.152.46:22656,7dcbe1c7c24e849c0b89271ded17fb71dc61a7fe@95.216.35.51:21156,2fd09544afaaf5ea07ccea00f2e437d1cc283f51@185.202.236.103:26656,2433afb5d241e24a68f81b66be5f4db3c83dfec3@88.198.47.154:46656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.Cardchain/config/config.toml

```

## Prometheus'u Aktif Etme
```shell
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.Cardchain/config/config.toml
```

## Pruning'i Ayarlama
```shell
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.Cardchain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.Cardchain/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.Cardchain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.Cardchain/config/app.toml
```

## Portları Ayarlama
```shell
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CROWD_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CROWD_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CROWD_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CROWD_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CROWD_PORT}660\"%" $HOME/.Cardchain/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CROWD_PORT}317\"%; s%^address = \":8080\"%address = \":${CROWD_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CROWD_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CROWD_PORT}091\"%" $HOME/.Cardchain/config/app.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:${CROWD_PORT}657\"%" $HOME/.Cardchain/config/client.toml
```

External Adres Ekleme
```shell
PUB_IP=`curl -s -4 icanhazip.com`
sed -e "s|external_address = \".*\"|external_address = \"$PUB_IP:${CROWD_PORT}656\"|g" ~/.c4e-chain/config/config.toml > ~/.c4e-chain/config/config.toml.tmp
mv ~/.c4e-chain/config/config.toml.tmp  ~/.c4e-chain/config/config.toml
```

## Servis Dosyası Oluşturma
```shell
tee <<EOF >/dev/null /etc/systemd/system/Cardchaind.service
[Unit]
Description=Cardchain Daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$(which Cardchaind) start
Restart=always
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Servisi Aktive Etme
```shell
systemctl daemon-reload
systemctl enable Cardchaind
```

## StateSync ile Sistemi Başlatma ([Stavr](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Crowd_Control))
```shell
SNAP_RPC=http://crowd.rpc.t.stavr.tech:21207
PEERS="ec585d7fb38b67619dcb79aad90722f0eaf0faa3@crowd.peer.stavr.tech:21206"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.Cardchain/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height) \
&& BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)) \
&& TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash); \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.Cardchain/config/config.toml; \
Cardchaind tendermint unsafe-reset-all
wget -O $HOME/.Cardchain/config/addrbook.json "https://raw.githubusercontent.com/koltigin/Crowd-Control-Kurulum-Rehberi/main/addrbook.json"
systemctl restart Cardchaind && journalctl -u Cardchaind -f -o cat
```


## Cüzdan Oluşturma

### Yeni Cüzdan Oluşturma
`$CROWD_WALLET` bölümünü değiştirmiyoruz kurulumun başında cüzdanımıza değişkenler ile isim belirledik.
```shell 
Cardchaind keys add $CROWD_WALLET
```  

### Var Olan Cüzdanı İçeri Aktarma
```shell
Cardchaind keys add $CROWD_WALLET --recover
```

## Cüzdan ve Valoper Bilgileri
Burada cüzdan ve valoper bilgilerimize değişkene ekliyoruz.

```shell
CROWD_WALLET_ADDRESS=$(Cardchaind keys show $CROWD_WALLET -a)
CROWD_VALOPER_ADDRESS=$(Cardchaind keys show $CROWD_WALLET --bech val -a)
```

```shell
echo 'export CROWD_WALLET_ADDRESS='${CROWD_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export CROWD_VALOPER_ADDRESS='${CROWD_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

* BU AŞAMADAN SONRA NODE'UMUZUN EŞLEŞMESİNİ BEKLİYORUZ.

## Faucet
Aşağıdaki kodda hata alırsanız [buradan](https://crowdcontrol.exploreme.pro/utilities) cüzdanımızı bağlayıp token isteyebilirsiniz.

## Cüzdan Bakiyesini Kontrol Etme
```shell
Cardchaind query bank balances $CROWD_WALLET_ADDRESS --chain-id $CROWD_CHAIN_ID
```  

## Senkronizasyonu Kontrol Etme
`false` çıktısı almaldıkça bir sonraki yani validator oluşturma adımına geçmiyoruz.
```shell
Cardchaind status 2>&1 | jq .SyncInfo
```

## Validator Oluşturma
 Aşağıdaki komutta aşağıda berlittiğim yerler dışında bir değişikli yapmanız gerekmez;
   - `identity`  burada `XXXX1111XXXX1111` yazan yere `https://keybase.io` sitesine üye olarak size verilen kimlik numaranızı yazıyorsunuz.
   - `details` `Always forward with the Anatolian Team 🚀` yazan yere kendiniz hakkında bilgiler yazabilirsiniz.
   - `website`  `https://anatolianteam.com` yazan yere varsa bir siteniz ya da twitter vb. adresinizi yazabilirsiniz.
   - `security-contact`  E-posta adresiniz.
 ```shell 
Cardchaind tx staking create-validator \
--amount=1000000ubpf \
--pubkey=$(Cardchaind tendermint show-validator) \
--moniker=$CROWD_NODENAME \
--chain-id=$CROWD_CHAIN_ID \
--commission-rate="0.1" \
--commission-max-rate="0.5" \
--commission-max-change-rate="0.1" \
--min-self-delegation="1" \
--from=$CROWD_WALLET \
--details="Always forward with the Anatolian Team 🚀" \
--security-contact="xxxxxxx@gmail.com" \
--website="https://anatolianteam.com" \
--identity="XXXX1111XXXX1111" \
--yes
 ``` 

## Validator Linkinizi Paylaşma
Crowd Control Discord [#validator](https://discord.gg/bVKJaqmURN) kanalından validatorumuze ait [explorer](https://explorers.acloud.pp.ua/cardchain/staking) linkini gönderiyoruz.

## FAYDALI KOMUTLAR

### Logları Kontrol Etme 
```shell
journalctl -fu Cardchaind -o cat
```

### Sistemi Başlatma
```shell
systemctl start Cardchaind
```

### Sistemi Durdurma
```shell
systemctl stop Cardchaind
```

### Sistemi Yeniden Başlatma
```shell
systemctl restart Cardchaind
```

### Node Senkronizasyon Durumu
```shell
Cardchaind status 2>&1 | jq .SyncInfo
```

### Validator Bilgileri
```shell
Cardchaind status 2>&1 | jq .ValidatorInfo
```

### Node Bilgileri
```shell
Cardchaind status 2>&1 | jq .NodeInfo
```

### Node ID Öğrenme
```shell
Cardchaind tendermint show-node-id
```

### Node IP Adresini Öğrenme
```shell
curl icanhazip.com
```

### Peer Adresinizi Öğrenme
```shell
echo $(Cardchaind tendermint show-node-id)@$(curl ifconfig.me)18656
```

### Cüzdanların Listesine Bakma
```shell
Cardchaind keys list
```

### Cüzdanı İçeri Aktarma
```shell
Cardchaind keys add $CROWD_WALLET --recover
```

### Cüzdanı Silme
```shell
Cardchaind keys delete CUZDAN_ADI
```

### Cüzdan Bakiyesine Bakma
```shell
Cardchaind query bank balances $CROWD_WALLET_ADDRESS
```

### Bir Cüzdandan Diğer Bir Cüzdana Transfer Yapma
```shell
Cardchaind tx bank send $CROWD_WALLET_ADDRESS GONDERILECEK_CUZDAN_ADRESI 100000000ubpf
```

### Proposal Oylamasına Katılma
```shell
Cardchaind tx gov vote 1 yes --from $CROWD_WALLET --chain-id=$CROWD_CHAIN_ID 
```

### Validatore Stake Etme  Delegate Etme
```shell
Cardchaind tx staking delegate $CROWD_VALOPER_ADDRESS 100000000ubpf --from=$CROWD_WALLET --chain-id=$CROWD_CHAIN_ID  --gas=auto
```

### Mevcut Validatorden Diğer Validatore Stake Etme  Redelegate Etme
```shell
Cardchaind tx staking redelegate MevcutValidatorAdresi StakeEdilecekYeniValidatorAdresi 100000000ubpf --from=$CROWD_WALLET --chain-id=$CROWD_CHAIN_ID --gas=auto
```

### Ödülleri Çekme
```shell
Cardchaind tx distribution withdraw-all-rewards --from=$CROWD_WALLET --chain-id=$CROWD_CHAIN_ID --gas=auto
```

### Komisyon Ödüllerini Çekme
```shell
Cardchaind tx distribution withdraw-rewards $CROWD_VALOPER_ADDRESS --from=$CROWD_WALLET --commission --chain-id=$CROWD_CHAIN_ID 
```

### Validator İsmini Değiştirme
```shell
Cardchaind tx staking edit-validator 
--moniker=YENI_NODE_ADI 
--chain-id=$CROWD_CHAIN_ID  
--from=$CROWD_WALLET
```

### Validatoru Jail Durumundan Kurtarma 
```shell
Cardchaind tx slashing unjail \
  --broadcast-mode=block \
  --from=$CROWD_WALLET \
  --chain-id=$CROWD_CHAIN_ID \
  --gas=auto \
  -y
```

### Node'u Tamamen Silme 
```shell
systemctl stop Cardchaind && \
systemctl disable Cardchaind && \
rm /etc/systemd/system/Cardchaind.service && \
systemctl daemon-reload && \
cd $HOME && \
rm -rf .Cardchain Cardchain && \
rm -rf $(which Cardchaind)
sed -i '/CROWD_/d' ~/.bash_profile
```

# Hesaplar

[Anatolian Team](https://anatolianteam.com)

[Twitter](https://twitter.commehmetkoltigin)

[Medium](https://medium.com/@mehmetkoltigin)

[YouTube](https://www.youtube.com/channel/UCmLgaftx5e38BE0E7gpY2dA)

[Discord](https://discordapp.com/users/837933958280904737)

[Telegram](https://t.me/mehmetkoltigin)
