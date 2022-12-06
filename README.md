<img src="https://miro.medium.com/max/1200/1*zjuHnPveUr7ZlLmj9lbGQw.jpeg" width="auto">
<p align="center">

[Website](https://uptick.network/ ) \
[Explorer](https://explorer.secardnode.com/uptick/staking)
=
- **Minimum Sistem Gereksinimleri**:

| Durum |CPU | RAM  | HDD  | 
|-----------|----|------|----------|
| Testnet   |   4| 8GB  | 150GB    |

### Sunucu İlk Hazırlık
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 18.3 Yükleme ( Tek Komut ile Yükleme )
```bash
ver="1.18.3" && \
cd $HOME && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Build 28.11.22
```bash
cd $HOME
git clone https://github.com/UptickNetwork/uptick.git
cd uptick
git checkout v0.2.4
make install
```

`uptickd version`
+ 0.2.4

## 1. Adım Moniker ve Chain id Tanıtımı (Düzenlenecek bölümleri kendinize göre düzenleyin)
```bash
uptickd init "Senin Validator İsmin" --chain-id uptick_7000-2
uptickd config chain-id uptick_7000-2
```

## Cüzdan Oluşturma veya Mevcut Cüzdanı İçeri Alma
```bash
uptickd keys add <cüzdanismin>                          (Yeni Cüzdan Oluşturma)
uptickd keys add <cüzdanismin> --recover                (Mevcut Cüzdanı İçeri Alma)
```

## Genesis
```bash
curl -o $HOME/.uptickd/config/genesis.json https://raw.githubusercontent.com/UptickNetwork/uptick-testnet/main/uptick_7000-2/genesis.json
```

## Peer-Seed Ayarları
```bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0auptick\"/;" ~/.uptickd/config/app.toml
external_address=$(wget -qO- eth0.me)
peers="eecdfb17919e59f36e5ae6cec2c98eeeac05c0f2@peer0.testnet.uptick.network:26656,178727600b61c055d9b594995e845ee9af08aa72@peer1.testnet.uptick.network:26656,f97a75fb69d3a5fe893dca7c8d238ccc0bd66a8f@uptick-seed.p2p.brocha.in:30554,94b63fddfc78230f51aeb7ac34b9fb86bd042a77@uptick-testnet-rpc.p2p.brocha.in:30556,902a93963c96589432ee3206944cdba392ae5c2d@65.108.42.105:27656"
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.uptickd/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.uptickd/config/config.toml
```

# Servis Dosyası Oluşturma
```bash
sudo tee /etc/systemd/system/uptickd.service > /dev/null <<EOF
[Unit]
Description=uptick
After=network-online.target

[Service]
User=$USER
ExecStart=$(which uptickd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Node Başlatma
```bash
sudo systemctl daemon-reload && \
sudo systemctl enable uptickd && \
sudo systemctl restart uptickd && sudo journalctl -u uptickd -f -o cat
```

## Validator Oluşturma
```bash
uptickd tx staking create-validator \
--chain-id uptick_7000-2 \
--commission-rate=0.1 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.1 \
--min-self-delegation="1" \
--amount=1000000auptick \
--pubkey $(uptickd tendermint show-validator) \
--moniker "Validatorİsmin" \
--from=<walletismin> \
--gas="auto" \
--fees 555auptick
```

## Node Silme
```bash
sudo systemctl stop uptickd && \
sudo systemctl disable uptickd && \
rm /etc/systemd/system/uptickd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .uptickd && \
rm -rf uptick-v0.2.3 && \
rm -rf $(which uptickd)
```
