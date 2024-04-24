# okayama_power_demand_logging
岡山工場電力監視システムセットアップ内容記録

# システム構成
工場WiFi接続用ゲートウェイ:Raspberry Pi 4B
ウェブロガー:DL30-G-R  
ワイヤレスゲートウェイ(ether側):WL40EW2  
ワイヤレスゲートウェイ(modubs側):WL5MW1  
電流マルチ変換：M5XWT-113  


# Raspberr Pi 4B WiFiゲートウェイ
WiFi側
IP:192.168.101.250

eth0側
IP:192.168.10.1

## セットアップ
OS:Bookworm 64bit


```
sudo apt update
sudo apt upgrade -y
sudo apt install iptables-persistent

nmcli con mod "wlao0接続名" ipv4.addresses "192.168.101.250/24" ipv4.gateway "192.168.101.1" ipv4.dns "8.8.8.8,8.8.4.4" ipv4.method manual
nmcli con up "wlan0接続名"

nmcli con mod "eth0接続名" ipv4.addresses '192.168.10.1/24' ipv4.method manual
nmcli con up "eth0接続名"


echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# NAT設定
sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE

# HTTPおよびHTTPSのポートフォワーディング
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 80 -j DNAT --to-destination 192.168.10.2:80
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 443 -j DNAT --to-destination 192.168.10.2:443
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 81 -j DNAT --to-destination 192.168.10.3:80
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 444 -j DNAT --to-destination 192.168.10.3:443

# iptablesの設定を保存
sudo netfilter-persistent save

```


# DL30-G-R webロガー2
IP:192.168.10.2
デフォルトゲートウェイ:192.168.10.1
## セットアップ
manual:nmdl30_g.pdf
setup tool:DL30GCFG
通常はUSB miniBケーブル接続で設定。
ネットワーク経由での設定にも対応。
port:30341
ID:minoru
pass:minoru0869553434



# WL40EW2 920MHz ゲートウェイether
IP:192.168.10.3
## setup
manual:nmwl40ew2_b.pdf
設定はブラウザにて行う。
初期は192.168.0.5にブラウザでアクセスして設定。




# WL5MW1 920 ゲートウェイ modbus
## セットアップ
manual:numm5xwt_b
セットアップツール：W920CFG
1号機設定
PANID:0001
ch:1
short address:0001
network name:okayamaPSL
crypt:00000000000000000000000000000000
mode:v3
packet fillterling: porling


firmware:1.0.2
serial:EP018931
modelNo:WLMW1-M2/E/A

wireless module firm:4.1.1
MAC address:00:25:36:00:00:01:BE:E2

# M5XWT-113
## setup
setup tool:PMCFG
connection tool :PC CONFIGRATOR CABLE MODEL:COP-US
※ドライバインストール必要 nmcopum_us_c.pdf参照


# メモ
ModBus終端抵抗は110Ω 
問い合わせ　杉川氏

slave0:DA902 油圧
slave1:DA902 ヒーター・その他  
slave2:DA902 押出

測定項目




