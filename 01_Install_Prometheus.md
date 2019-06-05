# Install Prometheus

## Installation Steps

Let's install Prometheus from a pre-built binary

1. On the server create a new user & group named `prometheus`. Set the home directory to `/var/lib/prometheus` and the shell to `/usr/sbin/nologin`.
2. Go to https://prometheus.io/download/. Copy the URL for the latest prometheus version for your platform. Download this to your server.
3. Extract the download tarball. Copy the `prometheus` & `promtool` binaries to `/usr/local/bin`. Copy the `prometheus.yml` file to /etc. It's also a good idea to chown these to root.
4. Create a file in `/etc/systemd/system/prometheus.service` with the contents:
    ```
    [Unit]
    Description=Prometheus
    After=network-online.target

    [Service]
    Type=simple
    User=prometheus
    Group=prometheus
    ExecReload=/bin/kill -HUP $MAINPID
    ExecStart=/usr/local/bin/prometheus \
    --config.file=/etc/prometheus.yml \
    --storage.tsdb.path=/var/lib/prometheus \
    --storage.tsdb.retention.time=30d \
    --storage.tsdb.retention.size=0 \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.console.templates=/etc/prometheus/consoles \
    --web.listen-address=0.0.0.0:9090
    SyslogIdentifier=prometheus
    Restart=always

    [Install]
    WantedBy=multi-user.target
    ```
5. Reload `systemd`, then enable & start the `prometheus` service.
6. Point your web browser to http://<server>:9090/.
    * Navigate to Status -> Targets. You should see prometheus scraping itself.
    * Go to the Graph tab. Enter `scrape_duration_seconds` in the query field. Click Execute. The Console tab will show the current value. The Graph tab will show a graph of the data.

## Installation Commands

```
# Step 1
mike@prometheus:~$ sudo useradd -r -m -d /var/lib/prometheus -s /usr/sbin/nologin -k /dev/null prometheus
mike@prometheus:~$ id prometheus
uid=999(prometheus) gid=999(prometheus) groups=999(prometheus)

# Step 2
mike@prometheus:~$ curl -L https://github.com/prometheus/prometheus/releases/download/v2.10.0/prometheus-2.10.0.linux-amd64.tar.gz -o prometheus.tgz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   625    0   625    0     0   1509      0 --:--:-- --:--:-- --:--:--  1506
100 46.2M  100 46.2M    0     0  7836k      0  0:00:06  0:00:06 --:--:-- 9586k

# Step 3
mike@prometheus:~$ tar zxf prometheus.tgz
mike@prometheus:~$ sudo cp prometheus-*/prometheus /usr/local/bin
mike@prometheus:~$ sudo cp prometheus-*/promtool /usr/local/bin
mike@prometheus:~$ sudo chown root: /usr/local/bin/prometheus /usr/local/bin/promtool
mike@prometheus:~$ sudo cp prometheus-*/prometheus.yml /etc
mike@prometheus:~$ sudo chown root: /etc/prometheus.yml

# Step 4
mike@prometheus:~$ cat << EOF | sudo tee /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP \$MAINPID
ExecStart=/usr/local/bin/prometheus \\
--config.file=/etc/prometheus.yml \\
--storage.tsdb.path=/var/lib/prometheus \\
--storage.tsdb.retention.time=30d \\
--storage.tsdb.retention.size=0 \\
--web.console.libraries=/etc/prometheus/console_libraries \\
--web.console.templates=/etc/prometheus/consoles \\
--web.listen-address=0.0.0.0:9090
SyslogIdentifier=prometheus
Restart=always

[Install]
WantedBy=multi-user.target
EOF

# Step 5
mike@prometheus:~$ sudo systemctl daemon-reload
mike@prometheus:~$ sudo systemctl enable prometheus.service
Created symlink /etc/systemd/system/multi-user.target.wants/prometheus.service → /etc/systemd/system/prometheus.service.
mike@prometheus:~$ sudo systemctl start prometheus.service
mike@prometheus:~$ sudo systemctl status prometheus.service
● prometheus.service - Prometheus
   Loaded: loaded (/etc/systemd/system/prometheus.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2019-06-04 18:59:01 UTC; 1min 28s ago
 Main PID: 20765 (prometheus)
    Tasks: 7 (limit: 1110)
   CGroup: /system.slice/prometheus.service
           └─20765 /usr/local/bin/prometheus --config.file=/etc/prometheus.yml --storage.tsdb.path=/var/lib/prometheus --storage.tsdb.retention.time=30d --storage.tsdb.retention.size=0 --web.

Jun 04 18:59:01 prometheus prometheus[20765]: level=info ts=2019-06-04T18:59:01.807Z caller=main.go:324 host_details="(Linux 4.15.0-51-generic #55-Ubuntu SMP Wed May 15 14:27:21 UTC 2019 x86_
Jun 04 18:59:01 prometheus prometheus[20765]: level=info ts=2019-06-04T18:59:01.807Z caller=main.go:325 fd_limits="(soft=1024, hard=4096)"
Jun 04 18:59:01 prometheus prometheus[20765]: level=info ts=2019-06-04T18:59:01.807Z caller=main.go:326 vm_limits="(soft=unlimited, hard=unlimited)"
Jun 04 18:59:01 prometheus prometheus[20765]: level=info ts=2019-06-04T18:59:01.812Z caller=main.go:645 msg="Starting TSDB ..."
Jun 04 18:59:01 prometheus prometheus[20765]: level=info ts=2019-06-04T18:59:01.813Z caller=web.go:417 component=web msg="Start listening for connections" address=0.0.0.0:9090
Jun 04 18:59:01 prometheus prometheus[20765]: level=info ts=2019-06-04T18:59:01.822Z caller=main.go:660 fs_type=EXT4_SUPER_MAGIC
Jun 04 18:59:01 prometheus prometheus[20765]: level=info ts=2019-06-04T18:59:01.829Z caller=main.go:661 msg="TSDB started"
Jun 04 18:59:01 prometheus prometheus[20765]: level=info ts=2019-06-04T18:59:01.830Z caller=main.go:730 msg="Loading configuration file" filename=/etc/prometheus.yml
Jun 04 18:59:01 prometheus prometheus[20765]: level=info ts=2019-06-04T18:59:01.846Z caller=main.go:758 msg="Completed loading of configuration file" filename=/etc/prometheus.yml
Jun 04 18:59:01 prometheus prometheus[20765]: level=info ts=2019-06-04T18:59:01.846Z caller=main.go:614 msg="Server is ready to receive web requests."
