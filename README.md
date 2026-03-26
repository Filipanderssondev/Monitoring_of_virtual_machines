# Monitoring of virtual machines

<!--
<img src="https://github.com/Filipanderssondev/Monitoring_of_virtual_machines/blob/main/Extra/monitoring-of-virtual-machines.png">

<br>
Monitoring of virtual machines<br>
Författare<br>
Publiceringsdatum<br>

<br>

## Abstract
Kort sammanfattning av dokumentet
-->

<div>
  <img src="https://github.com/Filipanderssondev/Monitoring_of_virtual_machines/blob/main/Extra/monitoring-of-virtual-machines.png" width="300" align="left" />
  <h2>Abstract</h2>
  <p> 
  Implementation of monitoring in a virtual environment using Prometheus and Grafana. This includes scraping every host and container for metrics, visualize collected data in a meaningful way, and create alerts that can automatically notify us when certain events occur. 
  <br>
	<br>
	<br>
	<br>
	<br>
    <br>
    <strong>Authors:</strong>
    <i><a href="https://github.com/Filipanderssondev">Filip Andersson</a></i>
    and
    <i><a href="https://github.com/JonatanHogild">Jonatan Högild</a></i>
    <br>
    26-03-2026
    <br clear="left"/>
  </p>
</div>
<br>

## Table of Contents

1. [Introduction](#introduction)
2. [Goals and Objectives](#goals-and-objectives)
3. [Method](#method)
4. [Target Audience](#target-audience)
5. [Document Status](#document-status)
6. [Disclaimer](#disclaimer)
7. [Scope and Limitations](#scope-and-limitations)
8. [Environment](#environment)
9. [Acknowledgments](#acknowledgments)
10. [Implementaion](#implementation) <br>
    10.1 [Node Exporter](#node-exporter) <br>
    10.2 [Prometheus](#prometheus) <br>
    10.3 [Prometheus Podman Exporter](#prometheus-podman-exporter) <br>
    10.4 [Grafana](#grafana) <br>
    10.5 [Showcase VM](#showcase-vm) <br>
    10.6 [Firewall Configuration](#firewall-configuration) <br>
    10.7 [Grafana Dashbaords](grafana-dashbaords) <br>
    10.8 [Grafana Alerts](#grafana-alerts) <br>
 	10.9 [FreeIPA integration with Grafana](#freeipa-integration-with-grafana) <br>
11. [Conclusion](#conclusion)
12. [References](#references) <br>
	12.1 [Other projects in our virtual IT-enviroment](#other-projects-in-our-virtual-it-enviroment)


## Introduction
**Welcome!** <br>
In this project, we will build a modern monitoring stack using Prometheus for data collection and Grafana for visual analysis. By combining Prometheus’s data time-series database with Grafana’s dashboards, we gain a centralized view of our environment, and insights into it. With alerts, we can further automate our monitoring. With this project, we hope to demonstrate how monitoring can improve visibility into system performance and availability.
This is the sixth project <a href="https://github.com/rafaelurrutiasilva/Proxmox_on_Nuc/blob/main/Extra/Mermaid/Projects.md">in a series of projects</a>, with the goal of setting up a complete virtualized, automated, and monitored IT-Enviroment as a part of our internship at [The Swedish Meteorological and Hydrological Institute (SMHI)](https://www.smhi.se/en/about-smhi). <br>

_[Other projects in our virtual IT-enviroment](#other-projects-in-our-virtual-it-enviroment)_

## Goals and Objectives
The goal is to implement a monitoring solution for an IT environment using Prometheus and Grafana. The project aims to collect system metrics from multiple hosts and containers, store and process the data in Prometheus, visualize it through dashboards in Grafana, and notify users with alerts. 

## Method
Node Exporter and Prometheus Podman Exporter will be deployed on all VMs as containers using podman. Prometheus and Grafana will be deployed on the *metrics-01* VM as containers. Node exporter will monitor every virtual machine. Podman exporter will monitor every container running on that virtual machine. Prometheus scrape targets configurations along with ready-made dashbaords will be integrated and deployed together with grafana provisioning configurations as well as rootCA certificate automation. An alert rule will be created to help monitor the root filesystem usage on all VMs. A mail server will be created and used as a contact point for alerts. FreeIPA login will be integrated with Grafana.  

## Target Audience
This project is for anyone who wants to learn about monitoring and automation, and implementing it in a multi-client, multi-user environment. This repo is also part of a larger project aimed at people interested in learning about IT-infrastructure and production, and building such an environment from scratch.

## Document Status
This repository is considered complete and officially published.<br>
Future improvements, refinements, or corrections may be introduced through controlled updates. Any changes will be versioned and documented in the commit history.

## Disclaimer
> [!CAUTION]
> This is intended for learning, testing, and experimentation. The emphasis is not on security or creating an operational environment suitable for production.

## Scope and Limitations
* This guide is not intended for production-grade, multi-node clusters or advanced HA setups.
* Hardware compatibility varies; If unsure, check hardware requirements before proceeding.
* Instructions may become outdated as software updates; always verify with the official documentation.
* Sensetive information will be withheld. This will not hinder participation in the guide.

## Environment
 - Asus PN64 ax210NGW
   - Intel® Core™ i7-12700H 
   - 1TB disk
   - 64 GB memory
 - FreeIPA (4.12.2)
 - Proxmox VE (9.1.1)
 - Rocky Linux (10.1)
 - Ansible (core 2.16.14)
 - Node_exporter (1.9.1)
 - Prometheus Podman exporter (v.1.21.1)
 - Prometheus (3.9.1)
 - Grafana (12.3.1)
 - PostFix (3.11.0)

## Acknowledgments
We would like to thank <a href=https://github.com/rafaelurrutiasilva>Rafael Urrutia</a> for his continuous support and guidance.


## Implementation

#### Playbooks
For each step in this process we will have a _deploy_exaple.yaml_ and a _kill_example.yaml_ instead of stopping and removing running containers, because each time we have to test a new feature or solve issues we have to remove the old container and build a new one.
Every file is available under <a href=https://github.com/Filipanderssondev/Monitoring_of_virtual_machines/tree/main/Code/management-vm/ansible>Code</a>

#### Config files
Each additional configuration files to go with the containers will be loccated under <a href=https://github.com/Filipanderssondev/Monitoring_of_virtual_machines/tree/main/Code/management-vm/ansible/files>_ansible/files_</a> 

#### Overview
```yaml
├── ansible.cfg
├── files
│   ├── certificates
│   │   ├── app-01-certs
│   │   │   ├── 10.208.12.103+1-key.pem
│   │   │   └── 10.208.12.103+1.pem
│   │   ├── metrics-01-certs
│   │   │   ├── 10.208.12.102-key.pem
│   │   │   └── 10.208.12.102.pem
│   │   └── mkcert-v1.4.4-linux-amd64
│   ├── grafana
│   │   ├── dashboards
│   │   │   ├── applications
│   │   │   │   └── container-health.json
│   │   │   └── infrastructure
│   │   │       └── vm-infrastructure-overview.json
│   │   └── provisioning
│   │       ├── alerting
│   │       │   ├── alerting.yml
│   │       │   └── alert-rules.yml
│   │       ├── dashboards
│   │       │   └── dashboards.yml
│   │       └── datasources
│   │           └── datasource.yml
│   ├── nginx
│   │   └── nginx.conf
│   └── prometheus
│       └── prometheus.yml
├── inventory
│   ├── group_vars
│   │   └── all
│   │       └── vault.yml
│   └── hosts.ini
├── playbooks
│   ├── application
│   │   ├── deploy_app.yaml
│   │   ├── deploy_frontend.yaml
│   │   ├── kill_applications.yaml
│   │   ├── kill_backend.yaml
│   │   ├── kill_frontend.yaml
│   ├── mail
│   │   └── install_postfix_client.yml
│   └── monitoring
│       ├── deploy_grafana.yaml
│       ├── deploy_monitoring.yaml
│       ├── deploy_node-exporter.yaml
│       ├── deploy_podman-exporter.yaml
│       ├── deploy_prometheus.yaml
│       ├── kill_grafana.yaml
│       ├── kill_monitoring.yaml
│       ├── kill_node-exporter_on_ipa.yaml
│       ├── kill_node-exporter.yaml
│       └── kill_podman-exporter.yaml
└── roles
    └── containers
        ├── images
        │   └── pull
        │       ├── defaults
        │       │   └── main.yaml
        │       └── tasks
        │           └── main.yaml
        ├── install
        │   └── tasks
        │       └── main.yaml
        ├── kill
        │   └── tasks
        │       └── main.yaml
        ├── login
        │   ├── filip
        │   │   ├── defaults
        │   │   │   └── main.yaml
        │   │   └── tasks
        │   │       └── main.yaml
        └── run
            ├── defaults
            │   └── main.yaml
            └── tasks
                └── main.yaml
         
```


### Node Exporter playbooks

#### Node exporter
Node exporter is [one of many exporters](https://prometheus.io/docs/instrumenting/exporters/) available to Prometheus. Its purpose is to collect hardware information, and Prometheus collects it. 

Node exporter will be deployed on all VMs as a Podman-created container. The [following parameters](https://github.com/prometheus/node_exporter?tab=readme-ov-file#docker) are used:<br>
* `-d` - Detached mode, runs the container in the background. Without this, logs would attach to your terminal.<br>
* `--name node_exporter` - Name as an alternative to container ID. 
*  `--restart=always` - Tells Podman to restart the container if it crashes or if the host reboots. Only works with systemd integration though. <br>
* `--net=host` - Makes the container share the host's network namespace. Node_exporter listens directly on :9100 and Prometheus can reach it normally. This is preferred since node_exporter exposes system metrics, and monitoring occurs on the host itself, using the host network. <br>
* `--pid=host` - The container will share the host process ID namespace. Without this option, the container would only see its own processes, and none of the host processes. <br>
* `-v /:/host:ro,rslave` - Mounts the host filesystem `/` into the container at `/host`. `ro` means read-only and `rslave` is a mount propagation option, allowing mount events from the host to propagate into the host. If a new filesystem are mounted onto the host, node_exporter will now be able to cleanly spot it. <br>
* `-p 9100:9100` - Port mapping [host]:[container]. 
* `--path.rootfs=/host` - Is not a podman option. It's passed directly to node exporter, stating that the root filesystem is located at `/host` and not `/`. <br>

#### deploy_node-exporter.yaml
~~~yaml
---
- name: Deploy Node Exporter on all VMs
  hosts: all
  roles:
    - role: containers/install
    - role: containers/login/filip
  become: true
  tasks:
    - name: Pull Node Exporter image
      include_role:
        name: containers/images/pull
      vars:
        images_to_pull:
          - manufacturer: prom
            image_name: node-exporter
            tag: latest

    - name: Enable podman socket
      ansible.builtin.systemd:
        name: podman.socket
        state: started
        enabled: true

    - name: Run Node Exporter container
      include_role:
        name: containers/run
      vars:
        container_name: node_exporter
        manufacturer: prom
        image_name: node-exporter
        tag: latest
        container_ports:
          - "9100:9100"
        pid: host
        container_network: host
        container_state: started
        container_restart_policy: always
        container_volumes:
          - /:/host:ro,rslave
        container_cmd:
          - "--path.rootfs=/host"
~~~

Test to see if the containers are running:
```
curl http://localhost:9090
```

#### kill_node-exporter.yaml
~~~yaml
---
- name: Remove Node Exporter on all VMs
  hosts: all
  become: true
  tasks:
    - name: Stop and remove Node Exporter container
      containers.podman.podman_container:
        name: node_exporter
        state: absent

    - name: Verify Node Exporter is gone
      ansible.builtin.command:
        cmd: podman ps -a --filter name=node_exporter --format "{{ '{{' }}.Names{{ '}}' }}"
      register: result
      changed_when: false

    - name: Print verification
      ansible.builtin.debug:
        msg: "{{ 'Node Exporter is gone' if result.stdout == '' else 'Node Exporter still exists!' }}"
~~~

#### Node exporter firewall rules

The applications we containerize use host-bound ports to relay traffic. These need to be permitted in the firewall before they can travel over the network.

Add two new Proxmox firewall security-groups, *node-exporter-in* and *node-exporter-out*. Create a new rule:

<pre>
Direction: in
Action: ACCEPT
Enable: yes
Protocol: tcp
Dest. port: 9100
Log level: info
</pre>

Create the corresponding *out* rule in *node-exporter-out*. For every VM, apply the *node-exporter-out* security group. For the *metrics-01* VM, apply the *node-exporter-in* security group.
The *metrics-01* VM will run the Promeptheus server, and collect all node exporter data. 

### Podman exporter playbooks

#### Prometheus Podman exporter instead of cAdvisor
At first we were going to use cAdvisor as a container, but we encountered a lot of issues with cAdvisor since it’s mainly designed for Docker and doesn’t fully support Podman. cAdvisor relies on Docker-style metadata, so in our case some containers showed up with unclear IDs instead of names, and it was hard to distinguish between frontend, backend, and database. We also saw that some metrics, especially network and filesystem data, were missing or inconsistent, even with the correct mounts configured. Because of this, the monitoring wasn’t reliable enough, so we switched to a Podman exporter. That gave us more accurate container identification and more consistent metrics directly from the Podman engine.

#### deploy_podman-exporter.yaml
~~~yaml
- name: Deploy Podman Exporter on all VMs
  hosts: all
  become: true

  roles:
    - role: containers/login/filip
  tasks:
    - name: Pull Podman Exporter image
      include_role:
        name: containers/images/pull
      vars:
        images_to_pull:
          - image_name: prometheus-podman-exporter
            tag: patched

    - name: Enable podman socket
      ansible.builtin.systemd:
        name: podman.socket
        state: started
        enabled: true

    - name: Run Podman Exporter container
      include_role:
        name: containers/run
      vars:
        container_name: podman_exporter
        image_name: prometheus-podman-exporter
        tag: patched
        container_state: started
        container_ports:
          - "9882:9882"
        container_network: host
        container_volumes:
          - "/run/podman/podman.sock:/run/podman/podman.sock:ro"
        container_env_vars:
          CONTAINER_HOST: "unix:///run/podman/podman.sock"
        container_user: root
        container_restart_policy: always
        container_security_opt:
          - "label=disable"
~~~

#### kill_podman-exporter.yaml
~~~yaml
---
- name: Remove Podman Exporter on application VM
  hosts: all
  become: true
  tasks:
    - name: Stop and remove Podman Exporter container
      containers.podman.podman_container:
        name: podman_exporter
        state: absent

    - name: Verify Podman Exporter is gone
      ansible.builtin.command:
        cmd: podman ps -a --filter name=podman_exporter --format "{{ '{{' }}.Names{{ '}}' }}"
      register: result
      changed_when: false

    - name: Print verification
      ansible.builtin.debug:
        msg: "{{ 'Podman Exporter is gone' if result.stdout == '' else 'Podman Exporter still exists!' }}"
~~~

#### Firewall rule for Prometheus Podman exporter

Create 2 new security groups, *podman-exporter-in* and *podman-exportr-out*, add inbound and outbound rules respectively.

On *app-01*, apply the *podman-exporter-in* secuirty-group. On *metrics-01*, apply the *podman-exportr-out* rule. 

<br>
<br>

### Prometheus

#### Scrape configs
Since both node exporter and podman exporter is deployed where they should be, we have to configure a prometheus.yml, configurations to be deployed together with the prometheus container and map it into the prometheus container. 

##### prometheus.yml
~~~yaml
global:
  scrape_interval: 15s

# VM Scrape config

scrape_configs:
  - job_name: "Management VM"
    static_configs:
      - targets:
          - "10.208.12.100:9100"
        labels:
          role: "Management"

  - job_name: "Application VM"
    static_configs:
      - targets:
          - "10.208.12.103:9100"
        labels:
          role: "Application"

  - job_name: "Monitoring VM"
    static_configs:
      - targets:
          - "10.208.12.102:9100"
        labels:
          role: "Monitoring/Metrics"

  - job_name: "IAM VM"
    static_configs:
      - targets:
          - "10.208.12.105:9100"
        labels:
          role: "IAM (FreeIPA)"

  - job_name: "Showcase VM"
    static_configs:
      - targets:
          - "10.208.12.104:9100"
        labels:
          role: "Showcase"

# Podman exporter, exporting metrics from containers on each vm

  - job_name: "Management Containers"
    static_configs:
      - targets: ["10.208.12.100:9882"]
        labels: {role: "Management"}

  - job_name: "Application Containers"
    static_configs:
      - targets: ["10.208.12.103:9882"]
        labels: {role: "Application"}

  - job_name: "Monitoring Containers"
    static_configs:
      - targets: ["10.208.12.102:9882"]
        labels: {role: "Monitoring"}

  - job_name: "IAM containers"
    static_configs:
      - targets: ["10.208.12.105:9882"]
        labels: {role: "IAM"}

  - job_name: "Showcase Containers"
    static_configs:
      - targets: ["10.208.12.104:9882"]
        labels: {role: "Showcase"}
~~~

#### deploy_prometheus.yaml
~~~yaml
- name: Deploy Prometheus
  hosts: monitoring
  become: true
  roles:
    - role: containers/login/filip
  tasks:
    - name: Ensure Prometheus config directory exists
      ansible.builtin.file:
        path: /home/Filip/prometheus
        state: directory
        mode: "0755"

    - name: Deploy Prometheus config
      ansible.builtin.copy:
        src: /opt/ansible/files/prometheus/prometheus.yml
        dest: /home/Filip/prometheus/prometheus.yml
        mode: "0644"

    - name: Run prometheus container
      include_role:
        name: containers/run
      vars:
        container_name: prometheus
        manufacturer: prom
        image_name: prometheus
        tag: main
        container_ports:
          - "9090:9090"
        container_volumes:
          - "/home/Filip/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:Z"
        container_state: started
        container_restart_policy: always
        container_network: host
~~~

If you want to assign port 9090 to Prometheus, be aware that this port may already be occupied on Rocky Linux by a system-service called *cockpit*:
```
sudo ss -tunlp | grep 9090
```

We will consider cockpit an unnecessary feature and potential attack surface. Disable the service and remove it from all hosts:
```
sudo systemctl disable cockpit
sudo dnf remove cockpit-*
```

#### Prometheus Firewall Configuration

Prometheus is simple enough, since we don't need to access it externally. Prometheus only needs to connect with Grafana on the local host, and for this, we need to add a firewall rule. Create a new security-group called *prometheus-in*:

<pre>
Direction: in
Action: ACCEPT
Enable: yes
Protocol: tcp
Dest. port: 9090
Log level: info
</pre>

Add this security-group on the *metrics-01* VM.


Test that the container is running correctly:
```
curl http://localhost:9090
```

#### kill_monitoring.yaml
~~~yaml
---
- name: Kill all monitoring Containers
  hosts: monitoring
  become: true
  tasks:

  #roles:
  #  - role: containers/kill
- name: Get all container names
  command: podman ps -a --format "{{'{{'}}.Names{{'}}'}}"
  register: container_names
  changed_when: false

- name: Show container names
  debug:
    msg: "Container found: {{ item }}"
  loop: "{{ container_names.stdout_lines }}"
  when: container_names.stdout != ""

- name: Stop all containers
  command: podman stop {{ item }}
  loop: "{{ container_names.stdout_lines }}"
  ignore_errors: true
  when: container_names.stdout != ""

- name: Remove all containers
  command: podman rm -f {{ item }}
  loop: "{{ container_names.stdout_lines }}"
  ignore_errors: true
  when: container_names.stdout != ""
~~~

<br>
<br>

### Grafana

#### Provisioning (Grafana configs)

##### dashboards

*showcase-01* should now be able to access Grafana from a web-browser using `http://<metrics-01>:3000`. Default Grafana login is `admin / admin`. 

There already exists plenty of [ready-made dashboard templates](https://grafana.com/grafana/dashboards/) for Grafana we can use. For Node exporter, we'll use the [Node Exporter Full](https://github.com/rfmoz/grafana-dashboards?tab=readme-ov-file#node-exporter-full) template. Dashboards are written in JSON, so if we'd like, we can edit them on the command-line, or graphically in Grafana. 

These are our custom made dashboards based on existing dashboards, node_exporter dashboard for example. These dashboard JSONs are extremely long so i will not post them here, instead i will link them:

###### The dashboard for our container monitoring on each virtual machine: <br>
<a href=https://github.com/Filipanderssondev/Monitoring_of_virtual_machines/blob/main/Code/management-vm/ansible/files/grafana/dashboards/applications/container-health.json>dashboards/applications/container-health.json</a>

###### The dashboard for our whole vm infrastructure, the general vm health, cpu usage, RAM usage etc: <br>
<a href=https://github.com/Filipanderssondev/Monitoring_of_virtual_machines/blob/main/Code/management-vm/ansible/files/grafana/dashboards/infrastructure/vm-infrastructure-overview.json>dashboards/infrastructure/vm-infrastructure-overview.json




<!--

##### alerting

###### alert-rules.yml
~~~yaml
apiVersion: 1
groups:
  - orgId: 1
    name: Infrastructure
    folder: Alerts
    interval: 1m
    rules:
      - uid: vm-down
        title: VM Down
        condition: C
        data:
          - refId: A
            relativeTimeRange:
              from: 600
              to: 0
            datasourceUid: Prometheus
            model:
              expr: up{job=~".*VM.*"}
              intervalMs: 1000
              maxDataPoints: 43200
          - refId: C
            relativeTimeRange:
              from: 600
              to: 0
            datasourceUid: __expr__
            model:
              type: threshold
              expression: A
              conditions:
                - evaluator:
                    type: lt
                    params: [1]
                  query:
                    params: [A]
        for: 1m
        annotations:
          summary: "VM {{ $labels.instance }} is down"
        labels:
          severity: critical

      - uid: high-cpu
        title: High CPU Usage
        condition: C
        data:
          - refId: A
            relativeTimeRange:
              from: 600
              to: 0
            datasourceUid: Prometheus
            model:
              expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
              intervalMs: 1000
              maxDataPoints: 43200
          - refId: C
            relativeTimeRange:
              from: 600
              to: 0
            datasourceUid: __expr__
            model:
              type: threshold
              expression: A
              conditions:
                - evaluator:
                    type: gt
                    params: [85]
                  query:
                    params: [A]
        for: 5m
        annotations:
          summary: "High CPU on {{ $labels.instance }}: {{ $values.A }}%"
        labels:
          severity: warning

      - uid: high-memory
        title: High Memory Usage
        condition: C
        data:
          - refId: A
            relativeTimeRange:
              from: 600
              to: 0
            datasourceUid: Prometheus
            model:
              expr: 100 * (1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)
              intervalMs: 1000
              maxDataPoints: 43200
          - refId: C
            relativeTimeRange:
              from: 600
              to: 0
            datasourceUid: __expr__
            model:
              type: threshold
              expression: A
              conditions:
                - evaluator:
                    type: gt
                    params: [90]
                  query:
                    params: [A]
        for: 5m
        annotations:
          summary: "High memory on {{ $labels.instance }}: {{ $values.A }}%"
        labels:
          severity: warning

      - uid: disk-filling
        title: Disk Filling Up
        condition: C
        data:
          - refId: A
            relativeTimeRange:
              from: 600
              to: 0
            datasourceUid: Prometheus
            model:
              expr: 100 - (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"} * 100)
              intervalMs: 1000
              maxDataPoints: 43200
          - refId: C
            relativeTimeRange:
              from: 600
              to: 0
            datasourceUid: __expr__
            model:
              type: threshold
              expression: A
              conditions:
                - evaluator:
                    type: gt
                    params: [80]
                  query:
                    params: [A]
        for: 5m
        annotations:
          summary: "Disk usage high on {{ $labels.instance }}: {{ $values.A }}%"
        labels:
          severity: warning

      - uid: container-down
        title: Container Down
        condition: C
        data:
          - refId: A
            relativeTimeRange:
              from: 600
              to: 0
            datasourceUid: Prometheus
            model:
              expr: podman_container_running_state == 0
              intervalMs: 1000
              maxDataPoints: 43200
          - refId: C
            relativeTimeRange:
              from: 600
              to: 0
            datasourceUid: __expr__
            model:
              type: threshold
              expression: A
              conditions:
                - evaluator:
                    type: gt
                    params: [0]
                  query:
                    params: [A]
        for: 1m
        annotations:
          summary: "Container {{ $labels.name }} is down on {{ $labels.instance }}"
        labels:
          severity: critical

      - uid: nginx-errors
        title: Nginx High Error Rate
        condition: C
        data:
          - refId: A
            relativeTimeRange:
              from: 600
              to: 0
            datasourceUid: Prometheus
            model:
              expr: rate(nginx_http_requests_total{status=~"5.."}[5m])
              intervalMs: 1000
              maxDataPoints: 43200
          - refId: C
            relativeTimeRange:
              from: 600
              to: 0
            datasourceUid: __expr__
            model:
              type: threshold
              expression: A
              conditions:
                - evaluator:
                    type: gt
                    params: [5]
                  query:
                    params: [A]
        for: 5m
        annotations:
          summary: "High Nginx error rate on {{ $labels.instance }}"
        labels:
          severity: warning

      - uid: postgres-connections
        title: Postgres High Connections
        condition: C
        data:
          - refId: A
            relativeTimeRange:
              from: 600
              to: 0
            datasourceUid: Prometheus
            model:
              expr: sum by(instance) (pg_stat_activity_count)
              intervalMs: 1000
              maxDataPoints: 43200
          - refId: C
            relativeTimeRange:
              from: 600
              to: 0
            datasourceUid: __expr__
            model:
              type: threshold
              expression: A
              conditions:
                - evaluator:
                    type: gt
                    params: [80]
                  query:
                    params: [A]
        for: 5m
        annotations:
          summary: "High Postgres connections on {{ $labels.instance }}: {{ $values.A }}"
        labels:
          severity: warning
~~~

###### alerting.yml
~~~yaml
apiVersion: 1
contactPoints:
  - orgId: 1
    name: ntfy
    receivers:
      - uid: ntfy-webhook
        type: webhook
        settings:
          url: http://10.208.12.102:9093/alerts
          httpMethod: POST

policies:
  - orgId: 1
    receiver: ntfy
    group_by: [severity]
    routes:
      - receiver: ntfy
        matchers:
          - severity = critical
        group_wait: 0s
      - receiver: ntfy
        matchers:
          - severity = warning
        group_wait: 30s
~~~



-->





#### datasource
##### datasource.yml
~~~yaml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    uid: Prometheus
    url: http://10.208.12.102:9090
    access: proxy
    isDefault: true
~~~

#### deploy_grafana.yaml
~~~yaml
---
- name: Deploy Grafana
  hosts: monitoring
  become: true
  tasks:
    - name: Create Grafana provisioning directories
      ansible.builtin.file:
        path: "{{ item }}"
        state: directory
        recurse: true
        mode: "0755"
      loop:
        - /home/Filip/grafana/provisioning/datasources
        - /home/Filip/grafana/provisioning/dashboards/infrastructure
        - /home/Filip/grafana/provisioning/dashboards/applications

    - name: Deploy Grafana datasource
      ansible.builtin.copy:
        src: /opt/ansible/files/grafana/provisioning/datasources/datasource.yml
        dest: /home/Filip/grafana/provisioning/datasources/datasource.yml
        mode: "0644"

    - name: Deploy dashboards provisioning config
      ansible.builtin.copy:
        src: /opt/ansible/files/grafana/provisioning/dashboards/dashboards.yml
        dest: /home/Filip/grafana/provisioning/dashboards/dashboards.yml
        mode: "0644"

    - name: Deploy infrastructure dashboard
      ansible.builtin.copy:
        src: /opt/ansible/files/grafana/dashboards/infrastructure/vm-infrastructure-overview.json
        dest: /home/Filip/grafana/provisioning/dashboards/infrastructure/vm-infrastructure-overview.json
        mode: "0644"

    - name: Deploy applications dashboard
      ansible.builtin.copy:
        src: /opt/ansible/files/grafana/dashboards/applications/container-health.json
        dest: /home/Filip/grafana/provisioning/dashboards/applications/container-health.json
        mode: "0644"

    - name: Create Grafana alerting provisioning directory
      ansible.builtin.file:
        path: /home/Filip/grafana/provisioning/alerting
        state: directory
        mode: "0755"

    - name: Deploy Grafana alerting config
      ansible.builtin.copy:
        src: /opt/ansible/files/grafana/provisioning/alerting/alerting.yml
        dest: /home/Filip/grafana/provisioning/alerting/alerting.yml
        mode: "0644"

    - name: Deploy alert rules
      ansible.builtin.copy:
        src: /opt/ansible/files/grafana/provisioning/alerting/alert-rules.yml
        dest: /home/Filip/grafana/provisioning/alerting/alert-rules.yml
        mode: "0644"

    - name: Run Grafana container
      include_role:
        name: containers/run
      vars:
        container_name: grafana
        image_name: grafana
        tag: "alpine-3.22.2"
        container_ports:
          - "3000:3000"
        container_volumes:
          - "/home/Filip/grafana/provisioning:/etc/grafana/provisioning:Z"
          - "/home/jonatan/grafana:/etc/grafana:Z"
          - "/etc/ipa:/etc/ipa:Z"
        container_state: started
        container_restart_policy: always
        container_network: host
~~~

#### kill_grafana.yaml
~~~yaml
---
- name: Kill Grafana
  hosts: monitoring
  become: true
  tasks:
    - name: Stop and remove Grafana
      ansible.builtin.shell: |
        podman stop grafana 2>/dev/null || true
        podman rm grafana 2>/dev/null || true
        echo "Done"
      register: result

    - name: Show result
      ansible.builtin.debug:
        msg: "{{ result.stdout }}"
~~~

#### Grafana Firewall Configuration

We want Grafana to be accessable from the *showcase-01* VM, so we'll make security-groups for both inbound and outbound rules:

<pre>
Direction: in
Action: ACCEPT
Enable: yes
Protocol: tcp
Dest. port: 3000
Log level: info
</pre>

<pre>
Direction: out
Action: ACCEPT
Enable: yes
Protocol: tcp
Dest. port: 3000
Log level: info
</pre>

Apply the *grafana-in* rule on *metrics-01* and apply the *grafana-out* rule on *showcase-01*. 

### Showcase VM

A new VM will be used to run web-applications from a browser. This is necessary for visualizing our data with Grafana. We will use a Rocky Linux image that comes with a preinstalled desktop environment.

Download the Rocky Linux default image Boot ISO from the [offcial site](https://rockylinux.org/download).

https://docs.rockylinux.org/guides/minimum_hardware_requirements/

Create a VM with the following settings:
<pre>
General:
	Name: showcase-01
	Add to HA: No
	Start at boot: No
	
OS:
	Use CD/DVD disc image file (iso)
	Storage: local
	ISO image: ubuntu-24.04.3-desktop-amd64.iso
	Type: Linux
	Version: 6.x - 2.6 kernel

System:
	Graphic Card: Default
	Machine: q35
	BIOS: OVMF (UEFI)
	Add EFI Disk: Yes
	EFI Storage: local-lvm
	Pre-Enroll keys: yes
	SCSI ontroller: VirtiO SCSI single
	
Disks:
	Bus/Device: SCSI 0
	Storage: local-lvm
	Disk size (GiB): 32
	Format: QEMU image format
	Cache: Default (No cache)
	Discard: No
	IO thread: Yes
	
CPU:
	Sockets: 1
	Cores: 14
	Type: host

Memory:
	Memory (MiB): 8000
	Ballooning Device: No
	Allow KSM: Yes

Network:
	Bridge: vmbr0
	Model: VirtIO (paravirtualized)
	Vlan Tag: No
	MAC address: Use the autogenerated address
	Firewall: Yes
</pre>

Configure the Proxmox Firewall on this VM so that it mirrors the other VMs. 

Start the VM. Configure IP-addresses, gateway and DNS, pick 'workstation' as software, and install.


### HTTPS / TLS Certificate for Grafana

> **Update:** After initial deployment, TLS was configured for Grafana to remove the insecure connection warning in Firefox on the Showcase VM.

#### How it works

Certificates were generated on Showcase-01 using a local mkcert binary. Since the rootCA from the same mkcert installation was already imported into Firefox on Showcase-01, any cert generated with it is automatically trusted — no extra imports needed.

#### Generate cert for Monitoring VM (on Showcase-01)
```bash
cd /home/Filip/certs
./mkcert 10.208.12.102
```

The generated certs are fetched back to Mgmt-01 automatically via the playbook and stored under:
```
/opt/ansible/files/certificates/metrics-01-certs/
```

#### certs-for-showcase.yaml

This playbook generates the cert on Showcase-01, fetches it back to Mgmt-01, and imports the rootCA into Firefox automatically:
```yaml
---
- name: Generate certs on Showcase VM
  hosts: desktop
  become: true
  tasks:
    - name: Ensure certs directory exists
      file:
        path: /home/Filip/certs
        state: directory

    - name: Copy mkcert binary to Showcase
      copy:
        src: /opt/ansible/files/certificates/mkcert-v1.4.4-linux-amd64
        dest: /home/Filip/certs/mkcert
        mode: '0755'

    - name: Generate cert for monitoring VM
      command: ./mkcert 10.208.12.102
      args:
        chdir: /home/Filip/certs

    - name: Fetch cert back to Mgmt-01
      fetch:
        src: /home/Filip/certs/10.208.12.102.pem
        dest: /opt/ansible/files/certificates/metrics-01-certs/
        flat: true

    - name: Fetch cert key back to Mgmt-01
      fetch:
        src: /home/Filip/certs/10.208.12.102-key.pem
        dest: /opt/ansible/files/certificates/metrics-01-certs/
        flat: true

    - name: Import rootCA into Firefox
      command: >
        certutil -A
        -n "mkcert"
        -t "CT,,"
        -i /home/Filip/.local/share/mkcert/rootCA.pem
        -d /home/Filip/.mozilla/firefox/4w84f1uv.default-default
      ignore_errors: true
```

#### Grafana container — additions to deploy_grafana.yaml

Certs are copied to the monitoring VM and mounted into the Grafana container:
```yaml
- name: Ensure certs directory exists on monitoring VM
  file:
    path: /home/Filip/certs
    state: directory

- name: Copy certs to monitoring VM
  copy:
    src: /opt/ansible/files/certificates/metrics-01-certs/
    dest: /home/Filip/certs/

- name: Run Grafana container
  include_role:
    name: containers/run
  vars:
    container_name: grafana
    image_name: grafana
    tag: "alpine-3.22.2"
    container_ports:
      - "3000:3000"
    container_volumes:
      - "/home/Filip/grafana/provisioning:/etc/grafana/provisioning:Z"
      - "/home/Filip/certs:/certs:Z"
    container_env_vars:
      GF_SERVER_PROTOCOL: https
      GF_SERVER_CERT_FILE: /certs/10.208.12.102.pem
      GF_SERVER_CERT_KEY: /certs/10.208.12.102-key.pem
    container_state: started
    container_restart_policy: always
    container_network: host
```

Access Grafana over HTTPS:
```
https://metrics-01.plab.internal:3000
```

### Grafana Alerts

A quick look in the Grafana dashboards shows us that several VMs have consumed more storage space than anticipated. We can trace a majority of the disk usage to /var/logs/. Then, we can determine that a lot of logs were generated by an already resolved issue. With automatic log rotation in place, the storage will be cleared up on its own in due time. Forcing log rotation will clear it immediately though. 

More importantly, this demonstrates how some issues can sneak up on you when you're not paying attention. Grafana alerts is a way of automating responses to monitoring insights. For this project we will keep it simple and create an alert for when the root filesystem usage exceeds 90%. This requires an alert rule, and at least one contact point for that rule. 

#### Create an alert rule

<!-- Alerts are written in PromQL (Prometheus Querying Language), -->

In Grafana, go to Alerting > Alert rules > New alert rule

Give it a name like *Root FS usage near full*

The alert condition will be a query very similar to the query used on the *full node exporter* dashboard. Go to the dashboard, open the menu for the *Root FS Used* element > Inspect > Query

For someone not entirely familiar with PromQL, this is appreciated, as making variants of existing queries is easier than making one from scratch. 

In the *New alert rule* window, paste the code, and make some adjustments:
```
( node_filesystem_size_bytes{mountpoint="/", fstype!= "rootfs"}
- node_filesystem_avail_bytes{mountpoint="/", fstype!= "rootfs"}
/ node_filesystem_size_bytes{mountpoint="/", fstype!= "rootfs"} )
```

We want this alert to apply to all hosts, so we do not specify any instances or jobs. 

Next, we'll set the alert condition as threshold with the *IS ABOVE* option, and a value of 0.9. This query can be tested to see if it's currently firing, which is useful for verifying the query. 

We'll place this in the *infrastructure* folder, where the *node exporter full* dashboard resides. Give it one or more labels, then define evaluation settings.

As for the notification configuration, we don't have any working contact points. Leave it as is, and create the alert rule. This will be edited later.

#### Create a mail server

Grafana offers a variety of contact points. Nowadays, enterprises favor solutions such as incident management systems or team communcation platforms. We'll go with something simple, email.

To receive emails we will need a mail server. We'll designate *mgmt-01* as a mail server and download postfix (https://www.postfix.org/):
```
sudo dnf install postfix
sudo systemctl start postfix
```

Next, make the following changes in the configuration */etc/postfix/main.cf*:
```
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
mynetworks = xxx.xxx.xxx.xxx/24
```

update:
```
sudo systemctl restart postfix
```

Note that this configuration may need to vary depending on network and domain characteristics. 

Next, on *metrics-01*, also install postfix, make the following changes to */etc/postfix/main.cf*:
```
relayhost = [xxx.xxx.xxx.xxx] #IP-address of the mail-server
```

Update:
```
sudo systemctl restart postfix
```

Update the firewall to allow SMTP over port 25.

Now you should have a simple but working mail server. Make sure you can send and receive emails.

On client:
```
echo "Hello!" | sendmail jonatan@plab.internal
```

On server:
```
cat /var/spool/mail/jonatan
```

The mail-addresses were actually made in the previous lab, when we created our FreeIPA-users. By default, the username and domain name gets combined into a mail-address when a user is made. We don't necessarily have to use these, but it's convinient. 

#### Update Grafana configuration to use email as a contact point

In order for Grafana to use email, some changes must be made in the <pre>/etc/grafana/grafana.ini</pre> file. Since Grafana is running in a container, we will copy over the currently running *grafana.ini* to our host system, make the changes there and then mount this file when running the container:
```
podman cp grafana:/etc/grafana/grafana.ini ./grafana.ini
```

Make the following changes:
```
[smtp]
enabled = true
host = <mail-server>.plab.internal:25
user =
password =
from_address = grafana@plab.internal
from_name = Grafana
skip_verify = true
```

SMTP configuration settings can be found here (https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/#smtp).

Next, we'll update the Ansible playbook pfreviously used to deploy Grafana, and add a new volume:
```yaml
container_volumes:
  - "/path/to/grafana.ini:/etc/grafana/grafana.ini:Z"
```

Redeploy Grafana, then run it again.


In the Grafana web UI, go to alerting, create a new contact point, or edit the default email one. Set mail addresses, and then test the contact point. Edit the alert rule and add the new contact point. 

> This is a crude mail system and not the most ideal way of recieving alerts.
> Postfix works mainly as a Mail Transmission Agent (MTA), and is a good choice as a delivery backbone.
> We could further improve on this by implementing Dovecot as a dedicated mail server, with email clients like thunderbird.

### FreeIPA integration with Grafana

Instead of using the default admin login in Grafana, we want to use our FreeIPA-users. Grafana has LDAP-support, which means, by extension, it has FreeIPA-support. We can specify which user-groups have what levels of access in Grafana as well. In Grafana, there are [three types of user-accounts](https://grafana.com/docs/grafana/latest/administration/roles-and-permissions/#organization-roles): administrators, editors and viewers. Following our current set of user-groups, sysadmins will be mapped to the admin role, the editor role will be skipped and regular users will be mapped to the viewer role. 


#### Create a Dedicated Bind User in FreeIPA

Grafana needs a read-only service account to query the LDAP directory.

On any VM:
```bash
kinit admin
ipa user-add grafana-bind --first=Grafana --last=Bind --password
```

#### Enable LDAP in Grafana

Go into the *grafana.ini* file on the host. 

Find the [auth.ldap] section and make these changes:
```ini
[auth.ldap]
enabled = true
port = 636
use_ssl = true
ssl_skip_verify = true
config_file = /etc/grafana/ldap.toml
allow_sign_up = true
```

#### LDAP configuration for Grafana

As with the grafana.ini file before, copy the *ldap.toml* file from the container to the host:
```
podman cp grafana:/etc/grafana/ldap.toml ./ldap.toml
```

For convenience, place ldap.toml in the same directory as grafana.ini, since it will also be mounted to /etc/grafana.

Make the following adjustments:
```
[[servers]]
host = "your-ipa-server.domain.com"
port = 636
use_ssl = true
ssl_skip_verify = false
root_ca_cert = /etc/ipa/ca.crt

# Search user bind dn
bind_dn = "uid=svc-grafana-bind,cn=users,cn=accounts,dc=domain,dc=com"
bind_password = 'grafana-bind-password'

# User search filter, for example "(cn=%s)" or "(sAMAccountName=%s)" or "(uid=%s)"
search_filter = "(uid=%s)"

# An array of base dns to search through
search_base_dns = ["cn=users,cn=accounts,dc=test,dc=loc"]

## An array of the base DNs to search through for groups. Typically uses ou=groups
group_search_base_dns = ["cn=groups,cn=accounts,dc=test,dc=loc"]

# Specify names of the ldap attributes your ldap uses
[servers.attributes]
name = "givenName"
surname = "sn"
username = "uid"
member_of = "memberOf"
email =  "mail"

[[servers.group_mappings]]
group_dn = "cn=sysadmins,cn=groups,cn=accounts,dc=plab,dc=internal"
org_role = "Admin"

[[servers.group_mappings]]
# If you want to match all (or no ldap groups) then you can use wildcard
group_dn = "*"
org_role = "Viewer"
```

#### Edit the deploy_grafana.yml playbook

Go into the deploy_grafana.yaml file.

Add a volume mount bind for */etc/ipa*:
```yaml
volumes:
  - "/etc/ipa/ca.crt:/etc/ipa/ca.crt:Z"
```

Stop and remove the current Grafana container, then redeploy using the playbook. 


## Conclusion
We concluded that the stack works reliably for our setup once we stopped forcing Docker-based tools and adapted it to Podman—switching from cAdvisor to Podman Exporter made a big difference in getting accurate, stable metrics.

Overall, we feel the project was challenging in the beginning, especially with compatibility issues, but it became much more straightforward once we understood the tooling better. In the end, we built something practical that we actually trust and could see being used in a real environment. We are very proud.

This concludes our internship. Thank you for reading!

## References
- [FreeIPA](https://www.freeipa.org/)
- [SMHI](https://www.smhi.se/en/about-smhi)
- [Ansible FreeIPA collection](https://github.com/freeipa/ansible-freeipa)
- [Ansible builtin module - Password parameter](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/user_module.html#parameter-password)
- [FreeIPA Replica Setup](https://www.freeipa.org/page/V4/Replica_Setup)
- [Grafana roles and permissions](https://grafana.com/docs/grafana/latest/administration/roles-and-permissions/#organization-roles)

### Other projects in our virtual IT-enviroment:
- Project 1 - [Proxmox on Nuc](https://github.com/rafaelurrutiasilva/Proxmox_on_Nuc/)
- Project 2 - [Rocky Linux golden image for cloning](https://github.com/Filipanderssondev/Rocky_Linux_OS_Base_for_VMs)
- Project 3 - [Ansible on management VM](https://github.com/JonatanHogild/Ansible_on_management_vm)
- Project 4 - [Container stack deployment and monitoring with ansible](https://github.com/Filipanderssondev/Container_Stack_Deployment_With_Ansible)
- Project 5 - [FreeIPA for a virtual environment](https://github.com/JonatanHogild/FreeIPA_for_virtual_environment)
