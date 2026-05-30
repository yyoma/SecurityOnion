# SecurityOnion

Bare-metal cybersecurity lab simulating an isolated corporate SOC network. Built headlessly via VirtualBox CLI on an Ubuntu Server host, it routes an enterprise SIEM (Security Onion 2) and a Linux analyst workstation through a private NAT network for safe malware traffic analysis, forensic hunting, and resource-constraint optimization testing.

## 📌 Project Overview

This repository serves as a playbook detailing the design, deployment, and optimization of a bare-metal **Security Onion (v3.x)** Network Security Monitoring (NSM) and SIEM environment. The goal of this project was to install and operate a modern security setup under real-world resource constraints. Standard installations of Security Onion expect either modern hardware or setups with a generous number of CPU cores and RAM. This project demonstrates how architectural modifications, strict component targeting(Ubuntu versions and Installation modes), and deep-dive kernel/JVM troubleshooting allowed an enterprise security tool to run on a low-spec, bare-metal node without massively compromising the reliability.

# 🛠️ Production Engineering Constraints

* **Physical Host CPU:** 4-Core Processing Limit

* **Physical Host RAM:** 12GB RAM, later upgraded to 16GB, the hardware's maximum.

* **Target Stack:** Security Onion 3.x (Elasticsearch, Logstash, Kibana, Fleet, Zeek, Suricata, SaltStack)

* **The Challengse:** Security Onion installation straight on the bare-metal desktop caused partial SaltStack deployment crashes and . Switching to a VM fixed these erro Network isolation failure and guest-to-guest connectivity issues when attempting to utilize standard static IP configurations.

## 🗂️ Repository Structure

To navigate the implementation details, this playbook is split into highly structured chapters within the `docs/` folder:
* **`docs/01-architecture-and-network.md`**: Physical bare-metal specifications, virtual memory budgeting, and software-defined network architecture topology map.

* **`docs/02-installation-and-setup.md`**: Step-by-step headless OS initialization, automated installation wrappers, and web console administrative configurations.

* **`docs/03-troubleshooting-and-optimization.md`**: The technical core. Detailed breakdowns of JVM heap exhaustion, disk-swapping mitigation, parsing broken SaltStack configuration state files (`Result: False`), failures to talk to the VM because of the bridged MAC address, and Docker daemon troubleshooting (`so-elasticsearch`, `so-fleet`)

* **`docs/04-analyst-playbook.md`**: Operational guidelines for traffic ingestion monitoring, log querying, and validating security telemetry.

* **`/scripts`**: Automation utilities, filesystem garbage collection, and configuration adjustment wrappers used to stabilize the node.


# Architecture Summary
*
                [ Physical Home Router (192.168.1.254) ]
                                     │
                                     │
            ┌────────────────────────┴────────────────────────┐
            │             Ubuntu Server Bare-Metal Host       │
            │                 IP: 192.168.1.227               │
            └────────────────────────┬────────────────────────┘
                                     │
                (VirtualBox CLI Layer / Headless VBoxManage)
                                     │
            ┌────────────────────────┴────────────────────────┐
            │        Security Onion VM (Static Management)    │
            │        Interface 1 (enp0s3): bond0 Sniffing     │
            │        Interface 2 (enp0s8): 192.168.1.200      │
            └─────────────────────────────────────────────────┘


# 1. Hardware Resource Allocation Model

This deployment utilizes Ubuntu as a server image, saving RAM that would be spent maintaining a graphical interface, and instead allocating it to the actual systems. This allows for smoother operations and remote access by using Remote Desktop Connection on another device. By stripping away virtualization hypervisor layers and heavy desktop environments, nearly 100% of the physical 12GB RAM allocation is delivered directly to the containerized security applications.



# 2. Operational Mode Optimization

EVAL mode was chosen as it comes with network security monitoring (NSM) and log management tools inside Docker containers, including:
The Elastic Stack / OpenSearch (for data ingestion, storage, and visualization)
Zeek or Suricata (for network traffic sniffing, signature matching, and protocol analysis)
Wazuh (for host-based intrusion detection)
SaltStack (for internal container and configuration management)
Stenographer (for full packet capture)

# Quick Start / Ingestion Verification
To spin up a diagnostic assessment or verify system health on the optimized cluster, run the following administrative operations:
```bash

# Verify all containerized subsystems are active and healthy
sudo so-status
# Check for resource exhaustion faults or failed SaltStack state maps
sudo salt-call state.highstate test=True
# Monitor live memory profiles and identify OOM kernel flags
sudo dmesg -T | grep -i -E 'oom|kill'
