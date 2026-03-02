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
  Monitoring of virtual machines on a server, metrics extraction and visualization 
  <br>
	<br>
	<br>
	<br>
	<br>
    <br>
    <strong>Monitoring of Virtual machines</strong> <br>
    <strong>Authors:</strong>
    <i><a href="https://github.com/Filipanderssondev">Filip Andersson</a></i>
    and
    <i><a href="https://github.com/JonatanHogild">Jonatan Högild</a></i>
    <br>
    dd-mm-yyyy
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
10. [Implementaion](#inplementation)
11. [Conclusion](#conclusion)  
12. [References](#references)

## Introduction
Inledning
Bakgrund och syfte. Eventuell översiktbild här.

## Goals and Objectives
Mål och syften

## Method
Hur vi gjorde det. Tillvägagångssätt. Vårt sätt

## Target Audience
Målgrupp

## Document Status
I enlighet med ISO 9001 kan det vara _Utkast → Granskad → Godkänd → Publicerad → Arkiverad_

> [!NOTE]  
> My work here is not finished yet. I need, among other things, to supplement with instructions on how each component should be configured to work together as well supplement with an overview image that explains how the whole thing works.


## Disclaimer
Ansvarsfriskrivning. Tex:
> [!CAUTION]
> This is intended for learning, testing, and experimentation. The emphasis is not on security or creating an operational environment suitable for production.

## Scope and Limitations
Omfattning och begränsningar

## Environment
Miljö som användes

## Acknowledgments
Tack och erkännanden. Tex:
Big thanks to all the people involved in the material I refer to in my links! I would also like to express gratitude to everyone out there, including my colleagues and friends, who are creating things that help and inspire us to continue learning and exploring this never-ending world of computer technology.

## Inplementation

### Node exporter

Node exporter is [one of many exporters](https://prometheus.io/docs/instrumenting/exporters/) available to Prometheus. It's purpose is to collect hardware information, and Prometheus collects it. 

Node exporter will be deployed on all VMs as a Podman-created container. The following parameters are used:<br>
* `-d` - Detached mode, runs the container in the background. Without this, logs would attach to your terminal.<br>
* `--name node_exporter` - Name as an alternative to container ID. 
*  `--restart=always` - Tells Podman to restart the container if it crashes or if the host reboots. Only works with systemd integration though. <br>
* `--net=host` - Makes the container share the host's network namespace. Node_exporter listens directly on :9100 and Prometheus can reach it normally. This is preferred since node_exporter exposes system metrics, and monitoring occurs on the host itself, using the host network. <br>
* `--pid=host` - The container will share the host process ID namespace. Without this option, the container would only see its own processes, and none of the host processes. <br>
* `-v /:/host:ro,rslave` - Mounts the host filesystem `/` into the container at `/host`. `ro` means read-only and `rslave` is a mount propagation option, allowing mount events from the host to propagate into the host. If a new filesystem are mounted onto the host, node_exporter will now be able to cleanly spot it. <br>
* `-p 9100:9100` - Port mapping [host]:[container]. 
* `--path.rootfs=/host` - Is not a podman option. It's passed directly to node exporter, stating that the root filesystem is located at `/host` and not `/`. <br>

Recommended for Docker (converted to podman): https://github.com/prometheus/node_exporter?tab=readme-ov-file#docker

Instead of using Podman directly, we'll create a playbook that deploys node_exporter on all hosts.

> [PLACEHOLDER]
> <br>
> <br>
> <br>
> <br>
> <br>
> <br>
> <br>
> <br>


Test to see if the containers are running:
```
curl http://localhost:9090
```

#### Firewall configuration

The applications we containerize use host-bound ports to relay traffic. These need to be permitted in the firewall before they can travel over the network.

Add two new Proxmox firewall security-groups, *node-exporter-in* and *node-exporter-out*. Create a new rule for each port that is used. For example:

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


## Deploy Prometheus

If you want to assign port 9090 to Prometheus, be aware that this port is already occupied on Rocky Linux by a system-service called *cockpit*:
```
sudo ss -tunlp | grep 9090
```

We will consider cockpit an unnecessary feature and potential attack surface. Disable the service and purge it from all hosts:
```
sudo systemctl disable cockpit
sudo dnf remove cockpit-*
```

Create directory:
```bash
mkdir -p /opt/prometheus
```

Create `/opt/prometheus/prometheus.yml`:

```yaml
global:
  scrape_interval: 15s
scrape_configs:
  - job_name: "node_exporter"
    static_configs:
          - targets: ["10.208.12.100:9100"]
          - targets: ["10.208.12.102:9100"]
          - targets: ["10.208.12.103:9100"]
          - targets: ["10.208.12.104:9100"]
          - targets: ["10.208.12.105:9100"]
```

Prometheus run command:
```bash
podman run -d \
  --name prometheus \
  --restart=always \
  -p 9090:9090 \
  -v /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro \
  r-harbor.smhi.se/praktik-labb/prom/prometheus
```

Test that the container is running correctly:
```
curl http://localhost:9090
```

## Deploy Grafana

```bash
podman run -d \
  --name grafana \
  --restart=always \
  -p 3000:3000 \
  docker.io/grafana/grafana
```

## Showcase VM

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

## Grafana

Showcase-01 should be able to access all deployed services from a web-browser using `http://<metrics-01>:3000`. Default Grafana login is `admin / admin`.

#### Add Prometheus as Data Source

Settings > Data Sources > Add Prometheus

URL:
```
http://<metrics-01>:9090
```

### Add a dashboard

There already exists plenty of [ready-made dashboard templates](https://grafana.com/grafana/dashboards/) for Grafana we can use. For Node exporter, we'll use the [Node Exporter Full](https://github.com/rfmoz/grafana-dashboards?tab=readme-ov-file#node-exporter-full) template. Dashboards are written in JSON, so if we'd like, we can edit it on the command-line, or graphically in Grafana. 

## Conclusion
Slutsats

## References
Referenser (om det behövs)
