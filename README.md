# Crowd Control Cardchain Testnet1 Kurulum Rehberi 

![image](https://user-images.githubusercontent.com/102043225/179354101-a597974d-793b-4201-8e38-364a4ba13a3f.png)

## Gereksinimler
  * 2 CPU
  * 4GB RAM
  * 50 GB SSD

## Yükleme
Testnet1'e katılmak için aşağıdaki komutu kullanmanız yeterli. yükleme sırasında sizden node adınızı soracaktır onu girerek işleme devam edebilirsiniz.

```
git clone https://github.com/DecentralCardGame/Testnet1 && chmod +x ./Testnet1/Cardchain_install.sh && chmod +x ./Testnet1/Cardchain_remove.sh && ./Testnet1/Cardchain_install.sh
```

## Node'u Silme
Aşağıdaki komut dosyası systemd hizmetini durdurarak, Cardchain klasörünü ve Cardchain kütüphanelerini içeren dosyaları kaldırır.
```
./Testnet1/Cardchain_remove.sh
```

## Senkronizasyonu Kontrol Etme

```
curl -s localhost:26657/status | jq .result.sync_info
```
