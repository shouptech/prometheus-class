# Install Node Exporter

Exporters are the bread and butter of Prometheus. Information is gathered in Prometheus by scraping exporters. A common exporter used is Node Exporter.

## Installation Steps

Let's install Node Exporter on the same server as Prometheus.

1. On the server create a new user & group named `node-exp`. Set the home directory to `/` and the shell to `/usr/sbin/nologin`.
2. Go to https://prometheus.io/download/. Copy the URL for the latest Node Exporter version for your platform. Download this to your server.
3. Extract the download tarball. Copy the `node_exporter`binary to `/usr/local/bin`. It's also a good idea to chown this to root.
4. Create a file in `/etc/systemd/system/node_exporter.service` with the contents:
    ```
    [Unit]
    Description=Prometheus Node Exporter
    After=network.target

    [Service]
    Type=simple
    User=node-exp
    Group=node-exp
    ExecStart=/usr/local/bin/node_exporter \
        --collector.systemd \
        --web.listen-address=0.0.0.0:9100

    SyslogIdentifier=node_exporter
    Restart=always

    [Install]
    WantedBy=multi-user.target
    ```
5. Reload `systemd`, then enable & start the `node_exporter` service.
6. Point your web browser to http://<server>:9100/metrics and take a moment to read what is exported.
7. Edit `/etc/prometheus.yml`. Add a new job named `node_exporter`. Add a static target of `localhost:9100`. Reload the `prometheus` service.
8. Go to the Prometheus UI (http://<server>:9090), click Status -> Targets. Verify node_exporter is being scraped.
9. Go the graph tab in Prometheus and run the following queries:
    * Percentage of memory used: `1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)`
    * Filesystem free: `node_filesystem_avail_bytes{fstype!="rootfs",fstype!="tmpfs"} / node_filesystem_size_bytes{fstype!="rootfs",fstype!="tmpfs"}`
    * CPU usage percentage per CPU: `1 - irate(node_cpu_seconds_total{mode="idle"}[5m])`
    * Total CPU usage by instance: `1 - avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) by (instance)`

## Installation Commands

```
# Step 1
mike@prometheus:~$ sudo useradd -r -d / -s /usr/sbin/nologin -M node-exp
mike@prometheus:~$ id node-exp
uid=998(node-exp) gid=998(node-exp) groups=998(node-exp)

# Step 2
mike@prometheus:~$ curl -L https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz -o node-exp.tgz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   628    0   628    0     0   1341      0 --:--:-- --:--:-- --:--:--  1341
100 7893k  100 7893k    0     0  4662k      0  0:00:01  0:00:01 --:--:-- 8096k

# Step 3
mike@prometheus:~$ tar zxf node-exp.tgz
mike@prometheus:~$ sudo cp node_exporter-*/node_exporter /usr/local/bin
mike@prometheus:~$ sudo chown root: /usr/local/bin/node_exporter

# Step 4
mike@prometheus:~$ cat << EOF | sudo tee /etc/systemd/system/node_exporter.service
[Unit]
Description=Prometheus Node Exporter
After=network.target

[Service]
Type=simple
User=node-exp
Group=node-exp
ExecStart=/usr/local/bin/node_exporter --collector.systemd --web.listen-address=0.0.0.0:9100

SyslogIdentifier=node_exporter
Restart=always

[Install]
WantedBy=multi-user.target

# Step 5
mike@prometheus:~$ sudo systemctl daemon-reload
mike@prometheus:~$ sudo systemctl enable node_exporter.service
Created symlink /etc/systemd/system/multi-user.target.wants/node_exporter.service → /etc/systemd/system/node_exporter.service.
mike@prometheus:~$ sudo systemctl start node_exporter.service
mike@prometheus:~$ sudo systemctl status node_exporter.service
● node_exporter.service - Prometheus Node Exporter
   Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2019-06-05 16:56:43 UTC; 17s ago
 Main PID: 1918 (node_exporter)
    Tasks: 3 (limit: 1109)
   CGroup: /system.slice/node_exporter.service
           └─1918 /usr/local/bin/node_exporter --collector.systemd --web.listen-address=0.0.0.0:9100

Jun 05 16:56:43 prometheus node_exporter[1918]: time="2019-06-05T16:56:43Z" level=info msg=" - stat" source="node_exporter.go:104"
Jun 05 16:56:43 prometheus node_exporter[1918]: time="2019-06-05T16:56:43Z" level=info msg=" - systemd" source="node_exporter.go:104"
Jun 05 16:56:43 prometheus node_exporter[1918]: time="2019-06-05T16:56:43Z" level=info msg=" - textfile" source="node_exporter.go:104"
Jun 05 16:56:43 prometheus node_exporter[1918]: time="2019-06-05T16:56:43Z" level=info msg=" - time" source="node_exporter.go:104"
Jun 05 16:56:43 prometheus node_exporter[1918]: time="2019-06-05T16:56:43Z" level=info msg=" - timex" source="node_exporter.go:104"
Jun 05 16:56:43 prometheus node_exporter[1918]: time="2019-06-05T16:56:43Z" level=info msg=" - uname" source="node_exporter.go:104"
Jun 05 16:56:43 prometheus node_exporter[1918]: time="2019-06-05T16:56:43Z" level=info msg=" - vmstat" source="node_exporter.go:104"
Jun 05 16:56:43 prometheus node_exporter[1918]: time="2019-06-05T16:56:43Z" level=info msg=" - xfs" source="node_exporter.go:104"
Jun 05 16:56:43 prometheus node_exporter[1918]: time="2019-06-05T16:56:43Z" level=info msg=" - zfs" source="node_exporter.go:104"
Jun 05 16:56:43 prometheus node_exporter[1918]: time="2019-06-05T16:56:43Z" level=info msg="Listening on 0.0.0.0:9100" source="node_exporter.go:170"

# Step 6
mike@prometheus:~$ sudo vi /etc/prometheus.yml
# In the `scrape_configs` section add the following:
  - job_name: 'node_exporter'
    static_configs:
    - targets: ['localhost:9100']

```
