# Server Monitoring Setup with Node Exporter, Prometheus, and Grafana

This guide explains how to set up a monitoring stack using Node Exporter, Prometheus, and Grafana on a Linux server.

---

## Prerequisites

- A Linux server (Ubuntu/Debian-based systems recommended).
- Root or sudo privileges.
- Basic understanding of Linux commands.

---

## Node Exporter Installation

### Step 1: Download Node Exporter
```bash
cd /tmp
curl -LO https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
```

### Step 2: Extract the Files
```bash
tar xvf node_exporter-1.8.2.linux-amd64.tar.gz
```

### Step 3: Move the Binary to `/usr/local/bin`
```bash
sudo mv node_exporter-1.8.2.linux-amd64/node_exporter /usr/local/bin/
```

### Step 4: Create a User for Node Exporter
```bash
sudo useradd -rs /bin/false node_exporter
```

### Step 5: Create a Systemd Service File
Create and edit `/etc/systemd/system/node_exporter.service`:

```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

### Step 6: Start and Enable Node Exporter
```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
```

---

## Prometheus Installation

### Step 1: Download Prometheus
```bash
cd /tmp
curl -LO https://github.com/prometheus/prometheus/releases/download/v3.1.0/prometheus-3.1.0.linux-amd64.tar.gz
```

### Step 2: Extract the Files
```bash
tar xvf prometheus-3.1.0.linux-amd64.tar.gz
```

### Step 3: Set Up Directories and Move Files
```bash
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
sudo mv prometheus-3.1.0.linux-amd64/prometheus /usr/local/bin/
sudo mv prometheus-3.1.0.linux-amd64/promtool /usr/local/bin/
sudo mv prometheus-3.1.0.linux-amd64/consoles /etc/prometheus
sudo mv prometheus-3.1.0.linux-amd64/console_libraries /etc/prometheus
```

### Step 4: Create a User for Prometheus
```bash
sudo useradd -rs /bin/false prometheus
```

### Step 5: Set Permissions
```bash
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus
```

### Step 6: Configure Prometheus
Edit `/etc/prometheus/prometheus.yml`:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```

### Step 7: Create a Systemd Service File
Create and edit `/etc/systemd/system/prometheus.service`:

```ini
[Unit]
Description=Prometheus
After=network.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/var/lib/prometheus \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

### Step 8: Start and Enable Prometheus
```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
```

---

## Grafana Installation

### Step 1: Add Grafana Repository and Install
```bash
sudo apt-get install -y apt-transport-https software-properties-common wget
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
sudo apt-get update
sudo apt-get install grafana
```

### Step 2: Start and Enable Grafana
```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

---

## Firewall Configuration
If you are using UFW, allow the necessary ports:

```bash
sudo ufw allow 3000/tcp  # Grafana
sudo ufw allow 9090/tcp  # Prometheus
sudo ufw allow 9100/tcp  # Node Exporter
```

---

## Access and Configure Grafana

1. Open a browser and go to `http://<SERVER_IP>:3000`.
2. Log in with default credentials:
   - **Username**: `admin`
   - **Password**: `admin`
3. Change the default password when prompted.

### Add Prometheus as a Data Source
1. Click the gear icon (⚙️) in the sidebar.
2. Select **Data Sources**.
3. Click **Add data source**.
4. Choose **Prometheus**.
5. Set the URL to `http://localhost:9090`.
6. Click **Save & Test**.

### Import Dashboard
1. Click the **+** icon in the sidebar.
2. Select **Import**.
3. Enter ID `1860` (Node Exporter Full dashboard).
4. Choose the Prometheus data source created earlier.
5. Click **Import**.

---

## Summary

- **Node Exporter** runs on port `9100`.
- **Prometheus** runs on port `9090`.
- **Grafana** runs on port `3000`.

You now have a monitoring stack that can visualize server metrics like CPU, memory, disk usage, and network traffic.

### Best Practices

- Update the default Grafana password.
- Configure the firewall appropriately.
- Use SSL/TLS for external access.
- Backup configurations and data regularly.

