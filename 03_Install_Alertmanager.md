# Install Alertmanager

What good is a monitoring system without alerts? That's what Alertmanager is for!

## Steps

1. On the server create a new user & group named `alertmanager`. Set the home directory to `/var/lib/alertmanager` and the shell to `/usr/sbin/nologin`.
2. Go to https://prometheus.io/download/. Copy the URL for the latest Alertmanager version for your platform. Download this to your server.
3. Extract the download tarball. Copy the `alertmanager`binary to `/usr/local/bin`. Copy `alertmanager.yml` to `/etc`. It's also a good idea to chown these to root.
4. Create a file at `/etc/systemd/system/alertmanager.service` with the contents:
    ```
    [Unit]
    Description=Prometheus Alertmanager
    After=network.target

    [Service]
    Type=simple
    PIDFile=/var/run/alertmanager.pid
    User=alertmanager
    Group=alertmanager
    ExecReload=/bin/kill -HUP $MAINPID
    ExecStart=/usr/local/bin/alertmanager \
      --config.file=/etc/alertmanager.yml \
      --storage.path=/var/lib/alertmanager \
      --web.listen-address=0.0.0.0:9093
    SyslogIdentifier=alertmanager
    Restart=always

    [Install]
    WantedBy=multi-user.target
    ```
5. Edit `/etc/alertmanager.yml`. Change the `receiver` line from `'web.hook'` to `'mattermost'`. Remove the receiver config for `web.hook` and add a new config for `mattermost` that looks like the following:
    ```
    - name: mattermost
      slack_configs:
      - api_url: <mattermost_webhook_url>
        send_resolved: true
        pretext: "<Your Name>'s Prometheus"
    ```
6. Reload `systemd`, then enable & start the `alertmanager` service.
7. Point your web browser to http://`<server>`:9093/ and make sure the alertmanager UI loads
8. Let's configure Prometheus with some alert rules. Create fhe file `/etc/prometheus_alerts.yml`:
    ```
    groups:
    - name: node rules
      rules:
      - alert: Node Down
        annotations:
          description: '{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 2
            minutes.'
          summary: 'Node {{ $labels.instance }} down'
        expr: up == 0
        for: 2m
        labels:
          severity: warning

      - alert: Node High CPU Usage
        annotations:
          description: '{{ $labels.instance }} has CPU usage > 95% for more than 10 minutes'
          summary: 'Instance {{ $labels.instance }} has high cpu usage'
        expr: >-
          (100 - (avg by (consul_node) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)) > 95
        for: 10m
        labels:
          severity: warning

      - alert: Node Out of Memory
        annotations:
          description: '{{ $labels.instance }} has < 5% free memory for more than 10 minutes'
          summary: 'Instance {{ $labels.instance }} is out of memory'
        expr: (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) < 0.05
        for: 10m
        labels:
          severity: warning

      - alert: Node Out of Disk Space
        annotations:
          description: '{{ $labels.instance }} has < 1% free disk space on a volume for more than 5
            minutes'
          summary: 'Instance {{ $labels.instance }} is out of disk space'
        expr: >-
          (node_filesystem_avail_bytes{fstype!="tmpfs",fstype!="rootfs"} /
          node_filesystem_size_bytes{fstype!="tmpfs",fstype!="rootfs"}) < 0.01
        for: 5m
        labels:
          severity: critical

      - alert: Node Almost Out of Disk Space
        annotations:
          description: '{{ $labels.instance }} has < 5% free disk space on a volume for more than 2
            minutes'
          summary: 'Instance {{ $labels.instance }} is almost out of disk space'
        expr: >-
          (node_filesystem_avail_bytes{fstype!="tmpfs",fstype!="rootfs"} /
          node_filesystem_size_bytes{fstype!="tmpfs",fstype!="rootfs"}) < 0.05
        for: 2m
        labels:
          severity: warning
    ```
9. Edit `/etc/prometheus.yml`. Add an alertmanager target to `http://localhost:9093`. Add a rule file `/etc/prometheus_alerts.yml`
10. Reload the prometheus service.
11. Go to the prometheus UI and click alerts tab. You should see your alerts. (`http://<server>:9090/alerts`). Click Stats -> Runtime & Build Information. You should see your alertmanager.
12. Generate an alert! Easiest is to just stop node_exporter service. After 2 minutes, an alert should go to mattermost. View the alert in the alertmanager UI.
13. Resolve the alert, and watch as things recover.

## Commands

```
# Step 1
mike@prometheus:~$ sudo useradd -r -m -d /var/lib/alertmanager -s /usr/sbin/nologin alertmanager
mike@prometheus:~$ id alertmanager
uid=997(alertmanager) gid=997(alertmanager) groups=997(alertmanager)

# Step 2
mike@prometheus:~$ curl -L https://github.com/prometheus/alertmanager/releases/download/v0.18.0/alertmanager-0.18.0.linux-amd64.tar.gz -o alertmanager.tgz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   628    0   628    0     0   1546      0 --:--:-- --:--:-- --:--:--  1542
100 22.5M  100 22.5M    0     0  68287      0  0:05:46  0:05:46 --:--:-- 71247
mike@prometheus:~$ tar zxf alertmanager.tgz

# Step 3
mike@prometheus:~$ sudo cp alertmanager-*/alertmanager /usr/local/bin
mike@prometheus:~$ sudo chown root: /usr/local/bin/alertmanager
mike@prometheus:~$ sudo cp alertmanager-*/alertmanager.yml /etc
mike@prometheus:~$ sudo chown root: /etc/alertmanager.yml

# Step 4
mike@prometheus:~$ cat << EOF | sudo tee /etc/systemd/system/alertmanager.service
[Unit]
Description=Prometheus Alertmanager
After=network.target

[Service]
Type=simple
PIDFile=/var/run/alertmanager.pid
User=alertmanager
Group=alertmanager
ExecReload=/bin/kill -HUP \$MAINPID
ExecStart=/usr/local/bin/alertmanager \\
  --config.file=/etc/alertmanager.yml \\
  --storage.path=/var/lib/alertmanager \\
  --web.listen-address=0.0.0.0:9093
SyslogIdentifier=alertmanager
Restart=always

[Install]
WantedBy=multi-user.target
EOF

# Step 5
mike@prometheus:~$ sudo vi /etc/alertmanager.yml
## Change line 9 to be:
  receiver: 'mattermost'
## Remove the 'web.hook' receiver and add a new reciever that looks like:
- name: mattermost
  slack_configs:
  - api_url: <mattermost_webhook_url>
    send_resolved: true


# Step 6
mike@prometheus:~$ sudo systemctl daemon-reload
mike@prometheus:~$ sudo systemctl start alertmanager
mike@prometheus:~$ sudo systemctl status alertmanager
● alertmanager.service - Prometheus Alertmanager
   Loaded: loaded (/etc/systemd/system/alertmanager.service; disabled; vendor preset: enabled)
   Active: active (running) since Thu 2019-07-18 02:43:27 UTC; 4s ago
 Main PID: 3834 (alertmanager)
    Tasks: 9 (limit: 1109)
   CGroup: /system.slice/alertmanager.service
           └─3834 /usr/local/bin/alertmanager --config.file=/etc/alertmanager.yml --storage.path=/var/lib/alertmanager --web.li

Jul 18 02:43:27 prometheus systemd[1]: Started Prometheus Alertmanager.
Jul 18 02:43:27 prometheus alertmanager[3834]: level=info ts=2019-07-18T02:43:27.367Z caller=main.go:197 msg="Starting Alertman
Jul 18 02:43:27 prometheus alertmanager[3834]: level=info ts=2019-07-18T02:43:27.371Z caller=main.go:198 build_context="(go=go1
Jul 18 02:43:27 prometheus alertmanager[3834]: level=info ts=2019-07-18T02:43:27.392Z caller=cluster.go:161 component=cluster m
Jul 18 02:43:27 prometheus alertmanager[3834]: level=info ts=2019-07-18T02:43:27.418Z caller=cluster.go:623 component=cluster m
Jul 18 02:43:27 prometheus alertmanager[3834]: level=info ts=2019-07-18T02:43:27.434Z caller=coordinator.go:119 component=confi
Jul 18 02:43:27 prometheus alertmanager[3834]: level=info ts=2019-07-18T02:43:27.434Z caller=coordinator.go:131 component=confi
Jul 18 02:43:27 prometheus alertmanager[3834]: level=info ts=2019-07-18T02:43:27.436Z caller=main.go:429 msg=Listening address=
Jul 18 02:43:29 prometheus alertmanager[3834]: level=info ts=2019-07-18T02:43:29.418Z caller=cluster.go:648 component=cluster m

# Step 9
# In /etc/prometheus.yml, change the configuration for `alerting` and `rules_files` to be this:
```
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - localhost:9093

rule_files:
  - "/etc/prometheus_alerts.yml"
```

# Step 10
mike@prometheus:~$ sudo systemctl reload prometheus


# Step 12
mike@prometheus:~$ sudo systemctl stop node_exporter

```
