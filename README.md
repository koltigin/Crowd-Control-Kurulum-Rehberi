# Crowd Control Cardchain Testnet1 Kurulum Rehberi 

![image](https://user-images.githubusercontent.com/102043225/179354101-a597974d-793b-4201-8e38-364a4ba13a3f.png)

## Gereksinimler (Minimum)
  * 2 CPU
  * 4GB RAM
  * 50 GB SSD
  
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
ver="1.18.3"
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
* '$NODENAME' validator adınız
* '$WALLET' cüzdan adınız
```shell
echo "export NODENAME=$NODENAME"  >> $HOME/.bash_profile
echo "export WALLET=$WALLET" >> $HOME/.bash_profile
echo "export CHAIN_ID=Cardchain" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Örnek
Node ve Cüzdan adımızın Mehmet olduğunu varsayalım. Kod aşağıdaki şekilde düzenlenecektir. 
```shell
echo "export NODENAME=Mehmet"  >> $HOME/.bash_profile
echo "export WALLET=Mehmet" >> $HOME/.bash_profile
echo "export CHAIN_ID="Cardchain >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Cardchain Kurulumu
```shell
curl https://get.ignite.com/DecentralCardGame/Cardchain@latest! | sudo bash
```

## Uygulamayı Yapılandırma
```shell
Cardchain config chain-id $CHAIN_ID
Cardchain config keyring-backend test
```

## Uygulamayı Başlatma
```shell
Cardchain init $NODENAME --chain-id $CHAIN_ID
```

## Genesis ve Addrbook Dosyalarının İndirilmesi
```shell
cp $HOME/Testnet1/genesis.json $HOME/.Cardchain/config/genesis.json
```

## Minimum GAS Ücretinin Ayarlanması
```shell
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0ubpf\"/" $HOME/.Cardchain /config/app.toml
```

## SEED ve PEERS Ayarlanması
```shell
sed -i -e "/persistent_peers =/ s/= .*/= \"61f05a01167b1aec59275f74c3d7c3dc7e9388d4@45.136.28.158:26658\"/"  $HOME/.Cardchain/config/config.toml
```

## Prometheus'u Aktif Etme
```shell
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.Cardchain /config/config.toml
```

## Pruning'i Ayarlama
```shell
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.Cardchain /config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.Cardchain /config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.Cardchain /config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.Cardchain /config/app.toml
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
`$WALLET` bölümünü değiştirmiyoruz kurulumun başında cüzdanımıza isim belirledik.
```shell 
Cardchain keys add $WALLET
```  

### Var Olan Cüzdanı İçeri Aktarma
```shell
Cardchain keys add $WALLET --recover
```

* BU AŞAMADAN SONRA NODE'UMUZUN EŞLEŞMESİNİ BEKLİYORUZ.

## Faucet
Aşağıdaki kodda hata alırsanız [buradan](http://dragonapi.space:5000/) token isteyebilirsiniz.
```shell
KEY=$(Cardchain keys show CUZDAN_ADRESINIZ --output=json | jq .address -r)
curl -X POST https://cardchain.crowdcontrol.network/faucet/ -d "{\"address\": \"$KEY\"}"
```

## Cüzdan Bakiyesini Kontrol Etme
```shell
Cardchain query bank balances CUZDAN_ADRESINIZ --chain-id $CHAIN_ID
```  

## Senkronizasyonu Kontrol Etme
`false` çıktısı almaldıkça bir sonraki yani validator oluşturma adımına geçmiyoruz.
```shell
Cardchain status 2&1  jq .SyncInfo
```

## Validator Oluşturma
 Aşağıdaki komutta aşağıda berlittiğim yerler dışında bir değişikli yapmanız gerekmez;
   'identity'  buraya `httpskeybase.io` sitesine üye olarak size verilen kimlik numaranızı yazıyorsunuz.
   'details'  kendiniz hakkında bilgiler verebilir ya da `Rues Community Supporter` yazabilirsiniz.
   'website'  Varsa bir siteniz yazabilirsiniz ya da `httpsforum.rues.info` olarak bırakabilirsiniz.
   'security-contact'  E-posta adresiniz.
```shell 
Cardchain tx staking create-validator 
 --commission-max-change-rate=0.01 
 --commission-max-rate=0.2 
 --commission-rate=0.05 
 --amount 9900000ubpf 
 --pubkey=$(Cardchain tendermint show-validator) 
 --moniker=$NODENAME 
 --chain-id=$CHAIN_ID 
 --details=Rues Community Supporter 
 --security-contact=E-POSTANIZ 
 --website=httpsforum.rues.info 
 --identity=XXXX1111XXXX1111 
 --min-self-delegation=1000000 
 --from=$WALLET
 ```  

## Validator Linkinizi Paylaşma
Crowd Control Discord [#validator](https://discord.gg/bVKJaqmURN) kanalından validatorumuze ait [explorer](https://explorers.acloud.pp.ua/cardchain/staking) linkini gönderiyoruz.

## DAHA FAZLA SORUNUZ VARSA CROWD CPNTROL TÜRKİYE TELEGRAM GRUBU

[Stride Türkiye Telegram Sayfası](https://t.me/CrowdControlTurkish)

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
Cardchain keys add $WALLET --recover
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
Cardchain tx gov vote 1 yes --from $WALLET --chain-id=CHAIN_ID 
```

### Validatore Stake Etme  Delegate Etme
```shell
Cardchain tx staking delegate $VALOPER_ADDRESS 100000000utoi --from=$WALLET --chain-id=C$HAIN_ID  --gas=auto
```

### Mevcut Validatorden Diğer Validatore Stake Etme  Redelegate Etme
```shell
Cardchain tx staking redelegate MevcutValidatorAdresi StakeEdilecekYeniValidatorAdresi 100000000ubpf --from=WALLET --chain-id=CHAIN_ID  --gas=auto
```

### Ödülleri Çekme
```shell
Cardchain tx distribution withdraw-all-rewards --from=$WALLET --chain-id=CHAIN_ID  --gas=auto
```

### Komisyon Ödüllerini Çekme
```shell
Cardchain tx distribution withdraw-rewards VALIDATOR_ADRESI --from=$WALLET --commission --chain-id=CHAIN_ID 
```

### Validator İsmini Değiştirme
```shell
Cardchain tx staking edit-validator 
--moniker=YENI_NODE_ADI 
--chain-id=$CHAIN_ID  
--from=$WALLET
```

### Validatoru Jail Durumundan Kurtarma 
```shell
Cardchain tx slashing unjail 
  --broadcast-mode=block 
  --from=$WALLET 
  --chain-id=$CHAIN_ID  
  --gas=auto
```

### Node'u Tamamen Silme 
```shell
systemctl stop Cardchaind && 
systemctl disable Cardchaind && 
rm etc/systemd/system/Cardchaind.service && 
systemctl daemon-reload && 
cd $HOME && 
rm -rf .Cardchain Testnet1 && 
rm -rf $(which Cardchain)
```

### Hesaplar

[Linktree](https://linktr.ee.mehmetkoltigin)

[Twitter](https://twitter.com.mehmetkoltigin)

### Rues Komunite 

[Forum Rues](https://forum.rues.infoindex.php)

[Telegram Rues Announcement](https://t.meRuesAnnouncement)

[Telegram Rues Chat](https://t.meRuesChat)

[Telegram Rues Node](https://t.meRuesNode)

[Telegram Rues Node Chat](https://t.meRuesNodeChat)
