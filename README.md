<img src="https://github.com/mART321/story/blob/main/img/1story.png" alt="Grafa banner" style="width: 100%; height: 100%; object-fit: cover;" />

# Story Grafana Dashboard by ITRocket
> This dashboard includes blockchain overview, validator data (uptime, missing blocks, etc), block parameters, transactions & gas data, staking parameters, Geth overview, system health monitoring. Custom Prometheus exporter was developed as well to display additional validator metrics. Finally, alerts can be easily configured if needed.

🔗 **Grafana Dashboard:** https://grafana-story.itrocket.net/d/fe12cg1xxqhhca/story-dashboard?orgId=1  
🔗 **Json data:** link

- [Grafana Dashboard Functionality](#grafana-dashboard-functionality)
  - [Validator Data](#validator-data)
  - [Block Parameters](#block-parameters)
  - [Transactions & Gas](#transactions-gas)
  - [Staking](#staking)
  - [GETH Overview](#geth-overview)
  - [System Health](#system-health)
- [Installation Guide](#installation-guide)
  - [System Requirements](#system-requirements)
  - [Installation](#ㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤㅤINSTALLATION)
  - [Install Node Exporter](#install-node-exporter)
  - [Download and Unpack Prometheus](#download-and-unpack-prometheus)
  - [Install Grafana](#install-grafana)

<img src="https://github.com/mART321/story/blob/main/img/main.png" alt="Grafa banner x" style="width: 100%; height: 100%; object-fit: cover;" />


# Grafana Dashboard Functionality

###### 💡 Click ► to open/close image  

### Validator Data 👨‍💻  <a id="validator-data"></a>
<details open>
  <summary>A quick overview of validator performance and health: validator's bond and jail status, addresses, unbonding time, as well as missed blocks and uptime, both in real-time and for the last 24 hours and 5 minutes. </summary>
  <img src="https://github.com/mART321/story/blob/main/img/vd.png" style="width: 100%; height: 100%; object-fit: cover;" />
</details>

### Block parameters ⛓️  <a id="block-parameters"></a>
<details open>
  <summary>Key block parameters like block size, interval times, and block time rate. It also shows the timing metrics for various ABCI methods, including commit, extend_vote, and finalize_block, with mean, last, max, and min values.</summary>
  <img src="https://github.com/mART321/story/blob/main/img/bp.png" style="width: 100%; height: 100%; object-fit: cover;" />
</details>

### Transactions and Gas 🔄  <a id="transactions-gas"></a>
<details open>
  <summary>This block provides statistics on transactions and gas usage, showing the total number of transactions, their success rate, gas fees, and the state of the transaction pool.</summary>
  <img src="https://github.com/mART321/story/blob/main/img/tg.png" style="width: 100%; height: 100%; object-fit: cover;" />
</details>

### Staking 🪙 <a id="staking"></a>
<details open>
  <summary>This block displays metrics related to staking, such as minted tokens, delegate and undelegate transactions, and the staking queue status.</summary>
  <img src="https://github.com/mART321/story/blob/main/img/s.png" style="width: 100%; height: 100%; object-fit: cover;" />
</details>

### Geth Overview 🔍
<details id="geth-overview" open>
  <summary>GETH Overview: This block contains information about the GETH client version, network traffic, peers, and block processing, helping to monitor node status and network interaction.</summary>
  <img src="https://github.com/mART321/story/blob/main/img/go.png" style="width: 100%; height: 100%; object-fit: cover;" />
</details>

### System Health 📉
<details id="system-health" open>
  <summary>System Health: This block provides system health metrics, including CPU load, memory usage, disk usage, and uptime, helping to monitor system performance and stability.</summary>
  <img src="https://github.com/mART321/story/blob/main/img/sh.png" style="width: 100%; height: 100%; object-fit: cover;" />
</details>

# Installation  
<details id="system-requirements" open>  

### System Requirements ⚙️
```
- System: Ubuntu 20.04 or newer
- RAM: 4GB or more
- CPU: 2 cores or more
- Storage: 20GB
```

Enable prometheus on story node config.toml
~~~
prometheus = true
prometheus_listen_addr = ":26660"
~~~

Enable geth metric to adding `--metrics --metrics.addr 0.0.0.0 --metrics.port 6060` on geth start command

### Install Node Exporter ⬇️
<details id="install-node-exporter" open>  
❗ Install on the validator node server
```
cd $HOME
rm -rf node_exporter-*.*-amd64
curl -s https://api.github.com/repos/prometheus/node_exporter/releases/latest | grep "browser_download_url.*linux-amd64" | cut -d '"' -f 4 | wget -qi -
tar xvfz node_exporter-*.*-amd64.tar.gz
cd node_exporter-*.*-amd64
chmod +x node_exporter
sudo cp node_exporter /usr/local/bin/
node_exporter --version
```
<img src="https://github.com/mART321/story/blob/main/img/2story.png" alt="Grafa banner 1" style="width: 100%; height: 100%; object-fit: cover;" />


Create service file
```
sudo tee /etc/systemd/system/node-exporter.service <<EOF
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=$USER
ExecStart=$(which node_exporter) --web.listen-address=:9200
Restart=on-failure
RestartSec=3

[Install]
WantedBy=default.target
EOF
```
<img src="https://github.com/mART321/story/blob/main/img/3story.png" alt="Grafa banner 2" style="width: 100%; height: 100%; object-fit: cover;" />


Enable and start Node Exporter
```
sudo systemctl daemon-reload
sudo systemctl enable node-exporter
sudo systemctl restart node-exporter && sudo journalctl -u node-exporter -f
```

### Install Prometheus 🛰️
💡 Recommended to install on a separate server  
Create user  
```
sudo useradd --no-create-home --shell /bin/false prometheus
sudo groupadd --system prometheus
sudo usermod -aG prometheus prometheus
```
<img src="https://github.com/mART321/story/blob/main/img/5story.png" alt="Grafa banner 4" style="width: 100%; height: 100%; object-fit: cover;" />

Download and unpack Prometheus  
```
cd $HOME
[ ! -d /etc/prometheus ] && mkdir -p /etc/prometheus
[ ! -d /var/lib/prometheus ] && mkdir -p /var/lib/prometheus
rm -rf tmp/prometheus /etc/prometheus/consoles /etc/prometheus/console_libraries
mkdir -p tmp/prometheus && cd tmp/prometheus
curl -s https://api.github.com/repos/prometheus/prometheus/releases/latest | grep browser_download_url | grep linux-amd64 | cut -d '"' -f 4 | wget -qi -
tar xvf prometheus*.tar.gz
rm prometheus*.tar.gz
cd prometheus*
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles console_libraries /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus /usr/local/bin/prometheus /usr/local/bin/promtool /var/lib/prometheus
prometheus --version
promtool --version
```
<img src="https://github.com/mART321/story/blob/main/img/4story.png" alt="Grafa banner 3" style="width: 100%; height: 100%; object-fit: cover;" />

Create prometheus.yml file
Your story node ip address
```
NODE_IP=xxx.xxx.xxx.xx
```

Create prometheus.yml
```
cat <<EOF | sudo tee /etc/prometheus/prometheus.yml
global:
  scrape_interval: 5s
  scrape_timeout: 3s

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['localhost:9093']

scrape_configs:
  # The job name is added as a label \`job=<job_name>\` to any timeseries scraped from this config.
  - job_name: 'prometheus'
    scrape_interval: 5s
    scrape_timeout: 3s
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    scrape_interval: 5s
    scrape_timeout: 3s
    static_configs:
      - targets: ['${NODE_IP}:9200']
  
  - job_name: 'Story'
    scrape_interval: 5s
    scrape_timeout: 3s
    static_configs:
      - targets: ['${NODE_IP}:26660']

  - job_name: 'geth_node'
    scrape_interval: 5s
    scrape_timeout: 3s
    metrics_path: /debug/metrics/prometheus
    static_configs:
      - targets: ['${NODE_IP}:6060']
EOF
```
<img src="https://github.com/mART321/story/blob/main/img/7story.png" alt="Grafa banner 5" style="width: 100%; height: 100%; object-fit: cover;" />

Create service file
```
sudo tee /etc/systemd/system/prometheus.service<<EOF
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP \$MAINPID
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.listen-address=0.0.0.0:9099 

Restart=on-failure
RestartSec=5
TimeoutStopSec=20s
LimitNOFILE=8192
ProtectSystem=full
ProtectHome=yes
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
EOF
```
<img src="https://github.com/mART321/story/blob/main/img/6story.png" alt="Grafa banner 6" style="width: 100%; height: 100%; object-fit: cover;" />

Enable and start Prometheus
```
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl restart prometheus && sudo journalctl -u prometheus -f
```

### Install Grafana 🛠️ 
💡 Recommended to install on a separate server  
```
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
sudo apt update
sudo apt install grafana
sudo systemctl start grafana-server
sudo systemctl status grafana-server
```
<img src="https://github.com/mART321/story/blob/main/img/8story.png" alt="Grafa banner 7" style="width: 100%; height: 100%; object-fit: cover;" />

Enable and start Prometheus

Configure grafana server
Open browser and and navigate to http://<your_server_ip>:3000
>Default admin/admin then change your password

login: admin \
pass: ITRocket123 

[GO UP](#grafana-dashboard-functionality)

<img src="https://itrocket.net/logo.svg" style="width: 100%; fill: white" />

[X](https://twitter.com/itrocket_team)  \
[GITHUB](https://github.com/itrocket-am) \
[TELEGRAM](https://linktr.ee/itrocket_team) 

