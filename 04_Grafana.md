# Grafana

Prometheus gathers metrics, and Grafana visualizes them! Let's install Grafana and create some dashboards.

## Steps

1. Follow the [Installation steps](https://grafana.com/docs/installation/) for your particular distribution. Commands below are for Ubuntu 18.04
2. Point your web browser to `http://<server>:3000` and login with admin/admin. You will be forced to change the password.
3. Add a prometheus data source: Settings Cog -> Data Sources, click Add data source, select Prometheus. Set URL to `http://localhost:9090` then click "Save & Test".
4. Create a new dashboard. Change the dashboard's name to Nodes.
5. Add a panel displaying CPU usage, by instance.
    * Set the query to `1 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])))`
    * Set the Legend field for the query to `CPU {{instance}}`
    * Change the unit on the Left Y axis to Misc/percent (0.0-1.0)
    * Change the panel's title to "CPU Usage by instance"
6. Add a panel displaying mem usage, by instance.
    * Set the query to `avg by (instance) (node_memory_MemTotal_bytes-node_memory_MemAvailable_bytes) / avg by (instance) (node_memory_MemTotal_bytes)`
    * Set the Legend field to `Memory Usage {{instance}}`
    * Change the unit on the Left Y axis to Misc/percent (0.0-1.0)
    * Change the panel's title to "Memory Usage by instance"
7. *BONUS POINTS* Add a variable to your dashboard for instance, allowing you to choose which intance to view data for.
8. Also, you can import pre-built dashboards. Such as: https://grafana.com/grafana/dashboards/1860. There are tons of pre-built dashboards for Grafana.
9. *DOUBLE BONUS POINTS* Enable the metrics endpoint in Grafana. Configure Prometheus to scrape Grafana. Add the pre-built Grafana dashboard to your Grafana install.

## Commands

```
# Step 1
mike@prometheus:~$ curl -JLO https://dl.grafana.com/oss/release/grafana_6.2.5_amd64.deb
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 55.9M  100 55.9M    0     0   705k      0  0:01:21  0:01:21 --:--:--  555k
mike@prometheus:~$ sudo dpkg -i grafana_6.2.5_amd64.deb
Selecting previously unselected package grafana.
[...]
Errors were encountered while processing:
 grafana
mike@prometheus:~$ sudo apt-get install -f
Reading package lists... Done
Building dependency tree
Reading state information... Done
Correcting dependencies... Done
The following additional packages will be installed:
  fontconfig-config fonts-dejavu-core libfontconfig1
The following NEW packages will be installed:
  fontconfig-config fonts-dejavu-core libfontconfig1
0 upgraded, 3 newly installed, 0 to remove and 4 not upgraded.
1 not fully installed or removed.
Need to get 1,233 kB of archives.
After this operation, 4,015 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
[...]
Processing triggers for libc-bin (2.27-3ubuntu1) ...
Processing triggers for ureadahead (0.100.0-21) ...
Processing triggers for systemd (237-3ubuntu10.24) ...

mike@prometheus:~$ sudo systemctl disable grafana-server
Synchronizing state of grafana-server.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install disable grafana-server
Removed /etc/systemd/system/multi-user.target.wants/grafana-server.service.
mike@prometheus:~$ sudo systemctl enable grafana-server
Synchronizing state of grafana-server.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable grafana-server
Created symlink /etc/systemd/system/multi-user.target.wants/grafana-server.service → /usr/lib/systemd/system/grafana-server.service.
mike@prometheus:~$ sudo systemctl start grafana-server
mike@prometheus:~$ sudo systemctl status grafana-server
● grafana-server.service - Grafana instance
   Loaded: loaded (/usr/lib/systemd/system/grafana-server.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2019-07-18 18:34:24 UTC; 6s ago
     Docs: http://docs.grafana.org
 Main PID: 5789 (grafana-server)
    Tasks: 7 (limit: 1109)
   CGroup: /system.slice/grafana-server.service
           └─5789 /usr/sbin/grafana-server --config=/etc/grafana/grafana.ini --pidfile=/var/run/grafana/grafana-server.pid --pa

Jul 18 18:34:30 prometheus grafana-server[5789]: t=2019-07-18T18:34:30+0000 lvl=info msg="Executing migration" logger=migrator
Jul 18 18:34:30 prometheus grafana-server[5789]: t=2019-07-18T18:34:30+0000 lvl=info msg="Executing migration" logger=migrator
Jul 18 18:34:30 prometheus grafana-server[5789]: t=2019-07-18T18:34:30+0000 lvl=info msg="Executing migration" logger=migrator
Jul 18 18:34:30 prometheus grafana-server[5789]: t=2019-07-18T18:34:30+0000 lvl=info msg="Executing migration" logger=migrator
Jul 18 18:34:30 prometheus grafana-server[5789]: t=2019-07-18T18:34:30+0000 lvl=info msg="Executing migration" logger=migrator
Jul 18 18:34:30 prometheus grafana-server[5789]: t=2019-07-18T18:34:30+0000 lvl=info msg="Executing migration" logger=migrator
Jul 18 18:34:30 prometheus grafana-server[5789]: t=2019-07-18T18:34:30+0000 lvl=info msg="Executing migration" logger=migrator
Jul 18 18:34:30 prometheus grafana-server[5789]: t=2019-07-18T18:34:30+0000 lvl=info msg="Executing migration" logger=migrator
Jul 18 18:34:30 prometheus grafana-server[5789]: t=2019-07-18T18:34:30+0000 lvl=info msg="Executing migration" logger=migrator
Jul 18 18:34:31 prometheus grafana-server[5789]: t=2019-07-18T18:34:31+0000 lvl=info msg="Executing migration" logger=migrator

```
