# Prometheus Class

## Goals

By the end of the class, the following should be working:

* Prometheus scraping itself and a Node Exporter endpoint.
* Prometheus sending alerts to Alert Manager.
* Alert Manager sending notifications to a webhook of your choice.
* A Grafana dashboard visualizing things in Prometheus.

This class will show how to manually do all of these things. T

## Requirements

To complete this class on your own, you will need a Linux server to play with. The class will be taught using Ubuntu 18.04, but anything with SystemD will likely work. This can be a virtual machine on your laptop.

If taking this class in person, I can optionally provide you with a cloud VM for the duration of the class.

## Sections

### [Install Prometheus](01_Install_Prometheus.md)

* Installing Prometheus from a pre-built binary
* Configuring SystemD to launch Prometheus on system startup
* Configuring Prometheus to scrape itself
* View the Prometheus UI

### [Install Node Exporter](02_Install_Node_Exporter.md)

* Install Node Exporter
* Configure Prometheus to scrape Node Exporter
* Run some queries to view Node Exporter metrics

### [Install Alertmanager](03_Install_Alertmanager.md)

* Install Alertmanager
* Configure Prometheus to send alerts to Alertmanager
* Configure an alert in Prometheus
* Configure Alertmanager to send alerts to Mattermost!

### [Grafana](04_Grafana.md)

* Install Grafan
* Add Prometheus as a data source
* Create a dashboard!
