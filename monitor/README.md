Monitoring

1) Setting up Prometheus
Use the monitoring server.

Create a user for Prometheus:

sudo useradd --no-create-home --shell /bin/false prometheus
Create the needed directories:

sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
Set the ownership of the /var/lib/prometheus directory:

sudo chown prometheus:prometheus /var/lib/prometheus
Download Prometheus:

cd /tmp/
wget https://github.com/prometheus/prometheus/releases/download/v2.7.1/prometheus-2.7.1.linux-amd64.tar.gz
Extract the files:

tar -xvf prometheus-2.7.1.linux-amd64.tar.gz
Move the configuration file and set the owner to the prometheus user:

cd prometheus-2.7.1.linux-amd64/
sudo mv console* /etc/prometheus
sudo mv prometheus.yml /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus
Move the binaries and set the owner:

sudo mv prometheus /usr/local/bin/
sudo mv promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
Create the service file:

sudo vi /etc/systemd/system/prometheus.service
Add:

[Unit]
Description=Prometheus
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
Save and exit.

Start Prometheus and make sure it automatically starts on boot:

sudo systemctl start prometheus
sudo systemctl enable prometheus

2) Set up Alertmanager.

Use the monitoring server.

Create the alertmanager system user:

cd
sudo useradd --no-create-home --shell /bin/false alertmanager
Create needed directories:

sudo mkdir /etc/alertmanager
Download Alertmanager:

cd /tmp/
wget "https://github.com/prometheus/alertmanager/releases/download/v0.16.1/alertmanager-0.16.1.linux-amd64.tar.gz"
Extract the files:

tar -xvf alertmanager-0.16.1.linux-amd64.tar.gz
cd alertmanager-0.16.1.linux-amd64/
Move the binaries:

sudo mv alertmanager /usr/local/bin/
sudo mv amtool /usr/local/bin/
Set the ownership of the binaries:

sudo chown alertmanager:alertmanager /usr/local/bin/alertmanager
sudo chown alertmanager:alertmanager /usr/local/bin/amtool
Move the configuration file into the /etc/alertmanager directory:

sudo mv alertmanager.yml /etc/alertmanager/
Set the ownership of the /etc/alertmanager directory:

sudo chown -R alertmanager:alertmanager /etc/alertmanager/
Create the alertmanager.service file for systemd:

sudo vi /etc/systemd/system/alertmanager.service

[Unit]
Description=Alertmanager
Wants=network-online.target
After=network-online.target

[Service]
User=alertmanager
Group=alertmanager
Type=simple
WorkingDirectory=/etc/alertmanager/
ExecStart=/usr/local/bin/alertmanager \
    --config.file=/etc/alertmanager/alertmanager.yml

[Install]
WantedBy=multi-user.target
Save and exit.

Stop Prometheus, then update the Prometheus configuration file to use Alertmanager:

sudo systemctl stop prometheus
sudo vi /etc/prometheus/prometheus.yml

alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - localhost:9093
Reload systemd, then start the prometheus and alertmanager services:

sudo systemctl start prometheus
sudo systemctl start alertmanager
Make sure alertmanager starts on boot:

sudo systemctl enable alertmanager


3) Use the grafana server.

Install the prerequisite package:

sudo apt-get install libfontconfig
Download and install Grafana using the .deb package provided on the Grafana download page:

cd /tmp/
wget https://dl.grafana.com/oss/release/grafana_5.4.3_amd64.deb
sudo dpkg -i grafana_5.4.3_amd64.deb
Start Grafana:

sudo systemctl start grafana-server
Ensure Grafana starts at boot:

sudo systemctl enable grafana-server
Access Grafana's web UI by going to IPADDRESS:3000.

Log in with the username admin and the password admin. Reset the password when promted.

Click Add data source on the home page.

Select Prometheus.

Set the URL to http://MONITORINGIP:9090

Click Save & Test


4) Add Alertmanager and Grafana endpoints to Prometheus.
Use the monitoring server.

Open the Prometheus configuration file:

sudo $EDITOR /etc/prometheus/prometheus.yml
Add the Alertmanager endpoint:

   - job_name: 'alertmanager'
     static_configs:
     - targets: ['localhost:9093']
Add the Grafana endpoint:

   - job_name: 'grafana'
     static_configs:
     - targets: ['GRAFANAIP:3000']
Save and exit.

Restart Prometheus:

sudo systemctl restart prometheus
Check that the endpoint exist on the Targets page on the Prometheus web UI.
