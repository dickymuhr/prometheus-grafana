# Prometheus Grafana
## Download Prometheus
1. Go to https://prometheus.io/download/ and copy download url 
2. In VM, download Prometheus using `wget`
```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.48.0-rc.2/prometheus-2.48.0-rc.2.linux-amd64.tar.gz
```
3. Also download node-exporter
```bash
https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
```

4. Extract Prometheus file
```bash
tar xvf prometheus-2.48.0-rc.2.linux-amd64.tar.gz
cd prometheus-2.48.0-rc.2.linux-amd64
```

## Create Prometheus User
1. Create User Group
```bash
sudo groupadd --system prometheus
``` 
2. Create User and Assign to Group
```bash
sudo useradd --system -s /sbin/nologin -g prometheus prometheus
```

## Move Binary File
1. Move File
```bash
sudo mv prometheus promtool /usr/local/bin/
```
2. Check Binary File
```bash 
which prometheus
which promtool
prometheus --version
```

## Create Prometheus Directory 
1. Create Folder for Config File
```bash
sudo mkdir /etc/prometheus
```
2. Create Folder for Saving Data
```bash
sudo mkdir /var/lib/prometheus
```
3. Change Permission
```bash
sudo chown -R prometheus:prometheus /var/lib/prometheus
ls -l /var/lib/prometheus
```
4. Move File
```bash
sudo mv consoles/ console_libraries/ prometheus.yml /etc/prometheus/
cd /etc/prometheus/
```

## Set Config File
```bash 
sudo nano prometheus.yml
```
```bash
global:
    scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
    evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
scrape_configs:
    - job_name: "prometheus"
    static_configs:
        - targets: ["Localhost:9090"]
```

## Create Service Daemon

Create system daemon service
```bash
sudo nano /etc/systemd/system/prometheus.service
```
```bash
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
--config.file /etc/prometheus/prometheus.yml \
--storage.tsdb.path /var/lib/prometheus/ \
--web.console.templates=/etc/prometheus/consoles \
--web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

Restart system daemon
```bash
sudo systemctl daemon-reload
```

Check prometheus system daemon service
```bash
sudo systemctl status prometheus
```

Run prometheus and activate auto enable
```bash
sudo systemctl enable --now prometheus
```

Check port running
```bash
sudo lsof -n -i | grep prometheus
```

## Run node-exporter on Service Daemon
Extract
```bash
tar xvf node_exporter-1.6.1.linux-amd64.tar.gz
cd node_exporter-1.6.1.linux-amd64
```
Move file
```bash
sudo mv node_exporter /usr/local/bin/
which node_exporter
```

Create service daemon
```bash
sudo nano /etc/systemd/system/node-exporter.service
```
```bash
[Unit]
Description=Prometheus exporter for machine metrics

[Service]
Restart=always
User=prometheus
ExecStart=/usr/local/bin/node_exporter
ExecReload=/bin/kill -HUP $MAINPID
TimeoutStopSec=20s
SendSIGKILL=no

[Install]
WantedBy=multi-user.target
```

Reload
```bash
sudo systemctl deamon-reload
```

Check prometheus system daemon service
```bash
sudo systemctl status node-exporter
```

Run
```bash
sudo systemctl enable --now node-exporter
```

Check port
```bash
sudo lsof -n -i | grep node
```

## Integrate Prometheus with node-exporter

```bash
sudo nano /etc/prometheus/prometheus.yml
```

```bash
global:
    scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
    evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
scrape_configs:
    - job_name: "prometheus"
    static_configs:
        - targets: ["localhost:9090"]
    - job_name: "node-exporter"
    static_configs:
        - targets: ["localhost:9100"]
```

Restart prometheus
```bash
sudo systemctl restart prometheus
```
```bash
sudo systemctl status prometheus
```