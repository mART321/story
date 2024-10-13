# Wave-2: Story Validators Race
<img src="https://raw.githubusercontent.com/mART321/story/blob/main/img/1story.png" alt="Grafa banner" style="width: 100%; height: 100%; object-fit: cover;" />

# Task 3: Grafana dashboard

Enable prometheus on story node config.toml
~~~
prometheus = true
prometheus_listen_addr = ":26660"
~~~

Enable geth metric to adding `--metrics --metrics.addr 0.0.0.0 --metrics.port 6060` on geth start command

### Install Node Exporter
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

Enable and start Node Exporter
```
sudo systemctl daemon-reload
sudo systemctl enable node-exporter
sudo systemctl restart node-exporter && sudo journalctl -u node-exporter -f
```


### Install Prometheus
```
sudo useradd --no-create-home --shell /bin/false prometheus
sudo groupadd --system prometheus
sudo usermod -aG prometheus prometheus
```

Dounload and unpach prometheus
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

Create prometheus.yml file
Your story node ip address
```
NODE_IP=<IP_ADDRESS>
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

Enable and start Prometheus
```
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl restart prometheus && sudo journalctl -u prometheus -f
```

### Install Grafana
```
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
sudo apt update
sudo apt install grafana
sudo systemctl start grafana-server
sudo systemctl status grafana-server
```

Configure grafana server
Open browser and and navigate to http://<your_server_ip>:3000
>Default admin/admin then change your password

login: admin
pass: ITRocket123


