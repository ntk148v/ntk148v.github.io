---
title: "Ansitheus"
date: 2020-05-05T13:51:59+07:00
lastmod: 2020-05-05T13:51:59+07:00
tags: ["tech", "ansible", "prometheus"]
toc: true
comments: true
draft: false
---

```bash
Ansitheus: Ansible + Prometheus
```

## 1. Prometheus overview

> **NOTE**: Checkout the [Prometheus official documentation](https://prometheus.io/docs/introduction/overview/).

[Prometheus](https://github.com/prometheus) is an open-source systems monitoring & alerting toolkit originally built at SoundCloud.

### 1.1. Features

- a multi-dimensional data model with time series data identified by metric name & key/value pairs
- PromQL, a flexible query language to leverage this dimensionality
- no reliance on distributed storage; single server nodes are autonomous
- time series collection happens via a pull model over HTTP
- pushing time series is supported via an intermediary gateway
- targets are discovered via service discovery or static configuration
- multiple modes of graphing & dashboarding support

### 1.2. Architecture & components

Prometheus scrapes metrics from instrumented jobs, either directly or via an intermediary push gateway for short-lived jobs. It stores all scraped samples locally & runs rules over this data to either aggregate & record new time series from existing data or generate alerts. Grafana or other API consumers can be used to visualize the collected data.

![](https://prometheus.io/assets/architecture.png)

- Prometheus server: scrapes & stores time series data.
- Prometheus alertmanager: handle alerts.
- Special-purpose exporters.
- Push-gateway: support short-lived jobs.
- client libraries: instrument application code.
- Various support tools: Grafana,...

## 2. Ansitheus

### 2.1. Why Ansitheus?

As you can see that, Prometheus ecosystem consists of multiple components. The operator may need a lot of efforts to configure, deploy & maintain these components. To make life easier, it is necessary to enter the world of automation, using modern tools of configuration management, provisioning & orchestration. [Ansible](https://ansible.com) is one of them. It is [simple, agentless IT automation that anyone can use](https://www.ansible.com/overview/how-ansible-works). My team decided to choose it as the automation solution, & [Ansitheus](https://github.com/ntk148v/ansitheus) is the result.

### 2.2. Features

The idea using Ansible to deploy Prometheus is not new. There are many existing solutions:

- [cloudalchemy/ansible-prometheus](https://github.com/cloudalchemy/ansible-prometheus)
- [ernestas-poskus/ansible-prometheus](https://github.com/ernestas-poskus/ansible-prometheus)
- ...

So what makes `Ansitheus` be different with others?

- **Deploy, configure & maintain the Prometheus ecosystem easily**.
- **Allow users to configure, deploy the system from scratch**:
  - Prepare, configure the local repository.
  - Install the necessary packages.
  - Configure Docker private registry.
  - Configure Docker daemon.
- **Containerize Prometheus components**:
  - [Docker](https://docker.com) is hotter than hot because it makes it possible to get far more apps running on the same old servers & it also makes it very easy to package & ship programs.
  - You can easily find the advantages of Docker & container through internet.
- **High availability** with HAProxy & Keepalived.
- **Support centralized Docker logging** with Fluentd.
- **Support Ansible vault** to work with sensitive data.

### 2.3. Components

- [Prometheus Server](https://github.com/prometheus/prometheus)
- [Prometheus Alertmanager](https://github.com/prometheus/alertmanager)
- [Prometheus Node-exporter](https://github.com/prometheus/node_exporter)
- [Google Cadvisor](https://github.com/google/cadvisor)
- [Prometheus SNMP exporter](https://github.com/prometheus/snmp_exporter)
- [Haproxy](http://www.haproxy.org/)
- [Keepalived](https://www.keepalived.org/)
- [Fluentd](https://github.com/fluent/fluentd)
- [Grafana](https://github.com/grafana/grafana)
- Other Prometheus exporters - **TODO**

### 2.4. Getting started

1. Install Ansible in deployment node.

2. Clone this repostiory.

3. Create configuration directory, default path `/etc/ansitheus`.

   ```bash
   sudo mkdir -p /etc/ansitheus
   sudo chown $USER:$USER /etc/ansitheus
   ```

4. Copy `config.yml` to `/etc/ansitheus` directory - this is the main configuration for Ansible monitoring tool.

   ```bash
   cp /path/to/ansitheus/repository/etc/ansitheus/config.yml \
       /etc/ansitheus/config.yml
   ```

5. Copy inventory files to the current directory.

   ```bash
   cp /path/to/ansitheus/repository/ansible/inventory/* .
   ```

6. Modify inventory & `/etc/ansitheus/config.yml`.
7. Run [tools/ansitheus](./tools/ansitheus), figure out yourself:

   ```bash
   Usage: ./tools/ansitheus COMMAND [option]

   Options:
       --inventory, -i <inventory_path> Specify path to ansible inventory file
       --configdir, -c <config_path>    Specify path to directory with config.yml
       --verbose, -v                    Increase verbosity of ansible-playbook
       --tags, -t <tags>                Only run plays & tasks tagged with these values
       --help, -h                       Show this usage information
       --skip-common                    Skip common role
       --ask-vault-pass                 Ask for vault password

   Commands:
       precheck                         Do pre-deployment checks for hosts
       deploy                           Deploy & start all ansitheus containers
       pull                             Pull all images for containers (only pull, no running containers)
       destroy                          Destroy Prometheus containers & service configuration
                                           --include-images to also destroy Prometheus images
                                           --include-volumes to also destroy Prometheus volumes

   ```

### 2.5. Contributors

1. [Kien Nguyen](https://github.com/ntk148v)
2. [Dat Vu](https://github.com/vtdat)
3. [Duc Nguyen](https://github.com/vanduc95)
4. [Long Cao](https://github.com/LongCaoBK)
