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

### Deploy Node exporter

Node exporter is [one of many exporters](https://prometheus.io/docs/instrumenting/exporters/) available to Prometheus. It's purpose is to collect hardware information, and Prometheus collects it. 

Node exporter will be deployed on all VMs as a Podman-created container. The [following parameters](https://github.com/prometheus/node_exporter?tab=readme-ov-file#docker) are used:<br>
* `-d` - Detached mode, runs the container in the background. Without this, logs would attach to your terminal.<br>
* `--name node_exporter` - Name as an alternative to container ID. 
*  `--restart=always` - Tells Podman to restart the container if it crashes or if the host reboots. Only works with systemd integration though. <br>
* `--net=host` - Makes the container share the host's network namespace. Node_exporter listens directly on :9100 and Prometheus can reach it normally. This is preferred since node_exporter exposes system metrics, and monitoring occurs on the host itself, using the host network. <br>
* `--pid=host` - The container will share the host process ID namespace. Without this option, the container would only see its own processes, and none of the host processes. <br>
* `-v /:/host:ro,rslave` - Mounts the host filesystem `/` into the container at `/host`. `ro` means read-only and `rslave` is a mount propagation option, allowing mount events from the host to propagate into the host. If a new filesystem are mounted onto the host, node_exporter will now be able to cleanly spot it. <br>
* `-p 9100:9100` - Port mapping [host]:[container]. 
* `--path.rootfs=/host` - Is not a podman option. It's passed directly to node exporter, stating that the root filesystem is located at `/host` and not `/`. <br>

Though, instead of using Podman directly, we'll create a playbook that deploys node_exporter on all hosts.

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


### Deploy Prometheus

If you want to assign port 9090 to Prometheus, be aware that this port may already be occupied on Rocky Linux by a system-service called *cockpit*:
```
sudo ss -tunlp | grep 9090
```

We will consider cockpit an unnecessary feature and potential attack surface. Disable the service and remove it from all hosts:
```
sudo systemctl disable cockpit
sudo dnf remove cockpit-*
```

Create directory:
```
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

### Deploy Grafana

```bash
podman run -d \
  --name grafana \
  --restart=always \
  -p 3000:3000 \
  docker.io/grafana/grafana
```

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

### Firewall Configuration

In the Proxmox Firewall, we will now set up rules for Prometheus and Grafana. Prometheus is simple enough, since we don't need to access it externally. Prometheus only needs to connect with Grafana on the local host, and for this, we need to add a firewall rule. Create a new security-group called *prometheus-in*:

<pre>
Direction: in
Action: ACCEPT
Enable: yes
Protocol: tcp
Dest. port: 9090
Log level: info
</pre>

Add this security-group on the *metrics-01* VM.

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

### Grafana Web UI

Showcase-01 should now be able to access Grafana from a web-browser using `http://<metrics-01>:3000`. Default Grafana login is `admin / admin`. 

#### Add Prometheus as Data Source

Settings > Data Sources > Add Prometheus

URL:
```
http://<metrics-01>:9090
```

#### Add a dashboard

There already exists plenty of [ready-made dashboard templates](https://grafana.com/grafana/dashboards/) for Grafana we can use. For Node exporter, we'll use the [Node Exporter Full](https://github.com/rfmoz/grafana-dashboards?tab=readme-ov-file#node-exporter-full) template. Dashboards are written in JSON, so if we'd like, we can edit them on the command-line, or graphically in Grafana. 

<!--
### cAdvisor

Next on the monitoring stack is cAdvisor (https://github.com/google/cadvisor) which help provide insight into the containers themselves. 
-->

### Prometheus Podman Exporter

#### Firewall rule for Prometheus Podman exporter

Create 2 new security groups, *podman-exporter-in* and *podman-exportr-out*, add inbound and outbound rules respectively.

On *app-01*, apply the *podman-exporter-in* secuirty-group. On *metrics-01*, apply the *podman-exportr-out* rule. 

### Grafana Alerts

A quick look in the Grafana dashboards shows us that several VMs have consumed more storage space than anticipated. We can trace a majority of the disk usage to /var/logs/. Then, we can determine that a lot of logs were generated by an already resolved issue. With automatic log rotation in place, the storage will be cleared up on its own in due time. Forcing log rotation will clear it immediately though. 

This demonstrates how some issues can sneak up on you when you're not paying attention. Grafana alerts is a way of automating responses to monitoring insights. For this project we will keep it simple and create an alert for when the root filesystem usage exceeds 90%. This requires an alert rule, and at least one contact point for that rule. 

#### Create an alert rule

In Grafana, go to Alerting > Alert rules > New alert rule

Give it a name like rootfs-near-full

The alert condition will be a query very similar to the query used on the dashboard. Instead of writing this query from scratch, go to the *full node exporter* dashboard, open the menu for the *Root FS Used* element > Inspect > Query

In the *New alert rule* window, paste the code, and make some adjustments:
```
(
node_filesystem_size_bytes{mountpoint="/", fstype!= "rootfs"}
- node_filesystem_avail_bytes{mountpoint="/", fstype!= "rootfs"}
)
/ node_filesystem_size_bytes{mountpoint="/", fstype!= "rootfs"}
```

Here, we do not specify any instances, so the rule applies to all monitored hosts. 

Next, we'll set a alert condition to *IS ABOVE* with a value of 0.9, which will work as the condition. This query can be tested to see if it's currently firing, which is useful for verifying the query. 

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

> This is a crude mail system and not the most ideal way of recieving alerts. We wanted to create a functioning alert pipeline, first and foremost.
> Postfix works mainly as a Mail Transmission Agent (MTA), and is a good choice as a delivery backbone.
> We could further improve on this by implementing Dovecot as a dedicated mail server, with email clients like thunderbird.

## Conclusion
Slutsats

## References
Referenser (om det behövs)
