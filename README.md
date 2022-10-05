# Crowd Control Cardchain Testnet2 Kurulum Rehberi 

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
ver="1.18.4"
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
echo "export CROWD_CHAIN_ID=Cardchain" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Örnek
Node ve Cüzdan adımızın Mehmet olduğunu varsayalım. Kod aşağıdaki şekilde düzenlenecektir. 
```shell
echo "export CROWD_NODENAME=Mehmet"  >> $HOME/.bash_profile
echo "export CROWD_WALLET=Mehmet" >> $HOME/.bash_profile
echo "export CROWD_PORT=18" >> $HOME/.bash_profile
echo "export CROWD_CHAIN_ID=Cardchain" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Cardchain Kurulumu
```shell
curl https://get.ignite.com/DecentralCardGame/Cardchain@latest! | sudo bash
```

## Uygulamayı Yapılandırma
```shell
Cardchain config chain-id $CROWD_CHAIN_ID
Cardchain config keyring-backend test
```

## Uygulamayı Başlatma
```shell
Cardchain init $CROWD_NODENAME --chain-id $CROWD_CHAIN_ID
```

## Testnet Klasörünün İndirilmesi ve Genesis Dosyasının Kopyalanması
```shell
git clone https://github.com/DecentralCardGame/Testnet
cp $HOME/Testnet/genesis.json $HOME/.Cardchain/config/genesis.json
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
sed -i -e "/persistent_peers =/ s/= .*/= \"752cfbb39a24007f7316725e7bbc34c845e7c5f1@45.136.28.158:26658\"/" $HOME/.Cardchain/config/config.toml
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
```

## Zincir Verilerini Sıfırlama
```shell
Cardchain tendermint unsafe-reset-all --home $HOME/.Cardchain 
```

## Servis Dosyası Oluşturma
```shell
tee <<EOF >/dev/null /etc/systemd/system/Cardchaind.service
[Unit]
Description=Cardchain Daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$(which Cardchain) start
Restart=always
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Servisi Başlatma
```shell
systemctl daemon-reload
systemctl enable Cardchaind
systemctl restart Cardchaind
```

## Logları Kontrol Etme
```shell
journalctl -u Cardchaind -f -o cat
```  

## Cüzdan Oluşturma

### Yeni Cüzdan Oluşturma
`$CROWD_WALLET` bölümünü değiştirmiyoruz kurulumun başında cüzdanımıza değişkenler ile isim belirledik.
```shell 
Cardchain keys add $CROWD_WALLET
```  

### Var Olan Cüzdanı İçeri Aktarma
```shell
Cardchain keys add $CROWD_WALLET --recover
```

* BU AŞAMADAN SONRA NODE'UMUZUN EŞLEŞMESİNİ BEKLİYORUZ.

## Faucet
Aşağıdaki kodda hata alırsanız [buradan](http://dragonapi.space:5000/) token isteyebilirsiniz.
```shell
KEY=$(Cardchain keys show validator --output=json | jq .address -r)
curl -X POST https://cardchain.crowdcontrol.network/faucet/ -d "{\"address\": \"$KEY\"}"
```

## Cüzdan Bakiyesini Kontrol Etme
```shell
Cardchain query bank balances CUZDAN_ADRESINIZ --chain-id $CROWD_CHAIN_ID
```  

## Senkronizasyonu Kontrol Etme
`false` çıktısı almaldıkça bir sonraki yani validator oluşturma adımına geçmiyoruz.
```shell
Cardchain status 2>&1 | jq .SyncInfo
```

## Validator Oluşturma
 Aşağıdaki komutta aşağıda berlittiğim yerler dışında bir değişikli yapmanız gerekmez;
   - `identity`  burada `XXXX1111XXXX1111` yazan yere `httpskeybase.io` sitesine üye olarak size verilen kimlik numaranızı yazıyorsunuz.
   - `details` `Always forward with the Anatolian Team 🚀` yazan yere kendiniz hakkında bilgiler yazabilirsiniz.
   - `website`  `https://anatolianteam.com` yazan yere varsa bir siteniz ya da twitter vb. adresinizi yazabilirsiniz.
   - `security-contact`  E-posta adresiniz.
 ```shell 
Cardchain tx staking create-validator \
--amount=1000000ubpf \
--pubkey=$(Cardchain tendermint show-validator) \
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
Cardchain status 2>&1 | jq .SyncInfo
```

### Validator Bilgileri
```shell
Cardchain status 2>&1 | jq .ValidatorInfo
```

### Node Bilgileri
```shell
Cardchain status 2>&1 | jq .NodeInfo
```

### Node ID Öğrenme
```shell
Cardchain tendermint show-node-id
```

### Node IP Adresini Öğrenme
```shell
curl icanhazip.com
```

### Peer Adresinizi Öğrenme
```shell
echo $(Cardchain tendermint show-node-id)@$(curl ifconfig.me)16656
```

### Cüzdanların Listesine Bakma
```shell
Cardchain keys list
```

### Cüzdanı İçeri Aktarma
```shell
Cardchain keys add $CROWD_WALLET --recover
```

### Cüzdanı Silme
```shell
Cardchain keys delete CUZDAN_ADI
```

### Cüzdan Bakiyesine Bakma
```shell
Cardchain query bank balances CUZDAN_ADRESI
```

### Bir Cüzdandan Diğer Bir Cüzdana Transfer Yapma
```shell
Cardchain tx bank send CUZDAN_ADRESI GONDERILECEK_CUZDAN_ADRESI 100000000ubpf
```

### Proposal Oylamasına Katılma
```shell
Cardchain tx gov vote 1 yes --from $CROWD_WALLET --chain-id=$CROWD_CHAIN_ID 
```

### Validatore Stake Etme  Delegate Etme
```shell
Cardchain tx staking delegate $VALOPER_ADDRESS 100000000utoi --from=$CROWD_WALLET --chain-id=$CROWD_CHAIN_ID  --gas=auto
```

### Mevcut Validatorden Diğer Validatore Stake Etme  Redelegate Etme
```shell
Cardchain tx staking redelegate MevcutValidatorAdresi StakeEdilecekYeniValidatorAdresi 100000000ubpf --from=$CROWD_WALLET --chain-id=$CROWD_CHAIN_ID --gas=auto
```

### Ödülleri Çekme
```shell
Cardchain tx distribution withdraw-all-rewards --from=$CROWD_WALLET --chain-id=$CROWD_CHAIN_ID --gas=auto
```

### Komisyon Ödüllerini Çekme
```shell
Cardchain tx distribution withdraw-rewards VALIDATOR_ADRESI --from=$CROWD_WALLET --commission --chain-id=$CROWD_CHAIN_ID 
```

### Validator İsmini Değiştirme
```shell
Cardchain tx staking edit-validator 
--moniker=YENI_NODE_ADI 
--chain-id=$CROWD_CHAIN_ID  
--from=$CROWD_WALLET
```

### Validatoru Jail Durumundan Kurtarma 
```shell
Cardchain tx slashing unjail \
  --broadcast-mode=block \
  --from=$CROWD_WALLET \
  --chain-id=$CROWD_CHAIN_ID \
  --gas=auto \
  -y
```

### Node'u Tamamen Silme 
```shell
sudo systemctl stop Cardchaind
sudo rm /etc/systemd/system/Cardchaind.service -rf
sudo rm $HOME/.Cardchain/ -rf
sudo rm $HOME/Testnet1 -rf
sudo rm /usr/local/bin/Cardchain -rf
```

# Hesaplar

[Anatolian Team](https://anatolianteam.com)

[Twitter](https://twitter.commehmetkoltigin)

[Medium](https://medium.com/@mehmetkoltigin)

[YouTube](https://www.youtube.com/channel/UCmLgaftx5e38BE0E7gpY2dA)

[Discord](https://discordapp.com/users/837933958280904737)

[Telegram](https://t.me/mehmetkoltigin)
