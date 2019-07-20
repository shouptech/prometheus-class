# Automation Resources

Doing all this by hand, while fun, is not a great thing to do long term. Below are some resources you might find helpful to automate. Most of these resources I have done myself.

## Ansible

I highly recommend [Cloud Alchemy](https://github.com/cloudalchemy)'s Ansible roles for installing all of these tools.

* [cloudalchemy.prometheus](https://github.com/cloudalchemy/ansible-prometheus)
* [cloudalchemy.node-exporter](https://github.com/cloudalchemy/ansible-node-exporter)
* [cloudalchemy.grafana](https://github.com/cloudalchemy/ansible-grafana)

And many more...

## Kubernetes

CoreOS has excellent resources for running Prometheus on your K8S clusters:

* [prometheus-operator](https://github.com/coreos/prometheus-operator)
* [kube-prometheus](https://github.com/coreos/kube-prometheus)

## Service Discovery

Seriously, who wants to manually change a config file everytime a new system is deployed? Don't do that. Use service discovery!

* [File based service discovery](https://prometheus.io/docs/guides/file-sd/)
* [Monitor containers via cAdvisor](https://prometheus.io/docs/guides/cadvisor/)

Just look through the documentation for everything you can do:

* [Consul](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#consul_sd_config)
* [EC2](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#ec2_sd_config)

And so much more...

## Grafana Provisioining

Grafana provisioning is an excellent way to keep dashboards consistent across environments.

* [Grafana Provisioning](https://grafana.com/docs/administration/provisioning/)


## Tips welcome!

Have any tips or other things to help with automation? Submit a merge request!
