## 📈 Prometheus + Grafana Monitoring Setup

This guide helps you set up **Prometheus** and **Grafana** on a Linux server to monitor a **Windows Server** using the `windows_exporter`.

---

## 🧰 Requirements

- ✅ Ubuntu Server (for Prometheus + Grafana)
- ✅ Windows Server (as the monitored system)
- ✅ `windows_exporter` installed on the Windows Server

---

## 🔧 A. Install Prometheus on Ubuntu

Download and set up Prometheus:

```bash
sudo apt update
sudo useradd --system --no-create-home --shell /bin/false prometheus

wget https://github.com/prometheus/prometheus/releases/download/v2.51.2/prometheus-2.51.2.linux-amd64.tar.gz
tar -xvf prometheus-2.51.2.linux-amd64.tar.gz
cd prometheus-2.51.2.linux-amd64/

sudo mkdir -p /data /etc/prometheus
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml

sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
```

---

## ⚙️ B. Configure Prometheus as a Service

Create the systemd service file:

```bash
sudo nano /etc/systemd/system/prometheus.service
```

Paste the following:

```ini
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle

[Install]
WantedBy=multi-user.target
```

---

## ▶️ C. Start Prometheus

```bash
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```

Access Prometheus at:  
👉 `http://<your-server-ip>:9090`  
Example: `http://192.168.0.148:9090`

---

## 📊 D. Install Grafana on Ubuntu

```bash
sudo apt-get install -y apt-transport-https software-properties-common
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list

sudo apt update
sudo apt install -y grafana
```

---

## ▶️ E. Start Grafana

```bash
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
sudo systemctl status grafana-server
```

Access Grafana at:  
👉 `http://<your-server-ip>:3000`  
Example: `http://192.168.0.148:3000`  
(Default login: `admin` / `admin`)

---

## 🖥️ F. Install `windows_exporter` on Windows Server

1. Download the `.msi` installer from:  
   👉 [https://github.com/prometheus-community/windows_exporter/releases](https://github.com/prometheus-community/windows_exporter/releases)

2. Run the installer.

3. Confirm it's running:  
   - Open `services.msc` → check `windows_exporter`

4. Visit in browser:  
   👉 `http://localhost:9182/metrics`

---

## 🔗 G. Bind Windows Exporter to Prometheus

1. Edit `/etc/prometheus/prometheus.yml`  
2. Modify the `scrape_configs` section:

```yaml
scrape_configs:
  - job_name: 'windows'
    scrape_interval: 6s
    scrape_timeout: 5s
    static_configs:
      - targets: ["localhost:9090", "192.168.0.149:9182"]
```

> Replace `192.168.0.149` with the actual IP of your Windows Server.

3. Check configuration syntax:

```bash
promtool check config /etc/prometheus/prometheus.yml
```

4. Restart Prometheus:

```bash
sudo systemctl restart prometheus
```

---

## 📥 H. Import Grafana Dashboard

1. In Grafana → Go to **Dashboards** → **Import**  
2. Use Dashboard ID:  
   `14694` (Windows Node Dashboard)

---

## ✅ Done!

You now have Prometheus scraping metrics from a Windows Server and Grafana visualizing them.

---



