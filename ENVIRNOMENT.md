# Cybersecurity Study Environment Setup Guide

## Overview
A comprehensive guide for building a dedicated Ubuntu Server 24.04 LTS environment for blue team cybersecurity studies, with Docker containerization for security tools and lab environments. Includes architecture recommendations for integrating Kali Linux for offensive security modules.

---

## Core Technologies

### Operating System
- **Ubuntu Server 24.04 LTS**
  - Primary study environment
  - Matches industry-standard blue team deployments
  - Excellent package availability for security tools
  - Docker native support
  - Long-term support (5 years)

### Container Platform
- **Docker**
  - Container runtime for isolated lab environments
  - Simplifies tool deployment and cleanup
  - Enables reproducible lab scenarios
  - docker-compose for multi-container setups
  - Portainer for management GUI (optional)

---

## Security Tools & Containers

### Wazuh (Security Monitoring & Analytics)
- **Purpose:** Security Information and Event Management (SIEM)
- **Deployment:** Docker container
- **Key Features:**
  - Centralized log aggregation
  - Real-time threat detection
  - File integrity monitoring
  - Vulnerability assessment
  - Compliance monitoring (CIS, NIST, PCI-DSS)
- **Components:**
  - Wazuh Manager – Central server collecting and analyzing logs
  - Wazuh Agent – Deployed on monitored systems
  - Wazuh Dashboard – Web UI for visualization and alerting
- **Docker Setup:** Official Wazuh Docker images available
  - Typically deployed via docker-compose
  - Requires persistent volumes for data storage

### Additional Docker-Based Tools (Complementary)
- **Prometheus** – Metrics collection and monitoring
- **Grafana** – Visualization and dashboard creation
- **ELK Stack** – Elasticsearch, Logstash, Kibana (alternative to Wazuh)
- **osquery** – System monitoring and threat hunting
- **fail2ban** – Intrusion prevention

---

## Kali Linux Integration for Studies

### Setup Option 1: VirtualBox on Windows (Recommended)
**VirtualBox on Windows is a viable and practical option for running Kali Linux.**

#### Setup Steps:
1. **Install VirtualBox** on Windows
   - Download from virtualbox.org
   - Free and lightweight hypervisor
2. **Download Kali Linux VM** 
   - Official pre-built Kali VirtualBox images available
   - Or download ISO and create VM from scratch
3. **Import or Create VM**
   - Allocate adequate resources: 4GB RAM minimum, 20GB disk
   - Enable nested virtualization if needed
4. **Launch and Study**
   - Run labs isolated from the host system
   - Snapshot VM before each major lab (easy rollback)

**Advantages:**
- No need to reboot into Linux
- Easy switching between host OS and Kali studies
- VM snapshots enable rapid lab resets
- Portable VM can be moved to different machines

### Setup Option 2: VirtualBox on Ubuntu Server
**Not recommended** for server-based deployments.

Why:
- Ubuntu Server has no GUI, making VirtualBox difficult to manage
- VirtualBox requires X11/Wayland, which is heavyweight for a headless server
- Docker is better suited for containerized security tools on Ubuntu Server

### Setup Option 3: Dual Boot (Not Recommended)
- Kali as a second partition alongside a primary OS
- Loses the convenience of snapshots and lab isolation
- Introduces extra maintenance overhead with little benefit over a VM

---

## Recommended Study Environment Architecture

```
┌─────────────────────────────────────────────┐
│      Workstation (Windows/macOS/Linux)      │
│                                             │
│  ┌───────────────────────────────────────┐  │
│  │      VirtualBox + Kali Linux VM       │  │
│  │   (Offensive security studies)        │  │
│  │   - Penetration testing labs          │  │
│  │   - Exploit development               │  │
│  │   - Red team scenarios                │  │
│  └───────────────────────────────────────┘  │
│                                             │
│  ┌───────────────────────────────────────┐  │
│  │  SSH/Network access to Ubuntu Server  │  │
│  │   (Blue team studies)                 │  │
│  │   - Wazuh monitoring                  │  │
│  │   - Log analysis                      │  │
│  │   - Security hardening                │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────────┐
│    Ubuntu Server 24.04 (Physical or VM)     │
│                                             │
│  ┌───────────────────────────────────────┐  │
│  │ Docker Containers                     │  │
│  │ ├── Wazuh Manager + Dashboard         │  │
│  │ ├── Prometheus + Grafana              │  │
│  │ ├── ELK Stack (optional)              │  │
│  │ └── Other security tools              │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

---

## Study Path Alignment

### Security+ Certification Path
- **Kali VM:** Network analysis labs (wireshark, nmap, packet analysis)
- **Ubuntu Docker:** Understanding SIEM architecture, log management, security controls
- **Wazuh Container:** Deploy and configure SIEM for log ingestion and threat detection

### CySA+ Certification Path (Vulnerability Management)
- **Kali VM:** Vulnerability scanning tools (Nessus, OpenVAS in containers)
- **Ubuntu Wazuh:** Monitor scan results, correlate with event logs, create detection rules
- **Integration:** Feed vulnerability data into SIEM for comprehensive risk assessment

### BTL1 Certification Path (Threat Intelligence)
- **Ubuntu Wazuh:** Analyze indicators of compromise (IOCs), develop detection rules
- **Kali VM:** Understand attack techniques using MITRE ATT&CK framework
- **Docker Containers:** Containerize threat intel tools (YARA, threat feeds, threat hunting)
- **Integration:** Correlate threat intelligence with detected events

---

## Initial Setup Checklist

### Host Operating System
- [ ] Install VirtualBox (or preferred hypervisor)
- [ ] Download Kali Linux ISO or pre-built VM image
- [ ] Create/import Kali VM with snapshots enabled
- [ ] Configure network settings for VM connectivity
- [ ] Test network connectivity between host and Ubuntu server

### Ubuntu Server 24.04
- [ ] Install Docker and docker-compose
- [ ] Install Portainer for container management (optional)
- [ ] Pull official Wazuh Docker images
- [ ] Create docker-compose.yml for Wazuh stack
- [ ] Configure persistent volumes for data storage
- [ ] Test Wazuh dashboard access from host machine

### Wazuh Deployment
```bash
# Create docker-compose.yml containing:
# - Wazuh Manager
# - Wazuh Dashboard (web UI)
# - Elasticsearch (data storage)

docker-compose up -d

# Access dashboard via HTTPS on specified port
# Configure with provided default credentials
```

---

## Key Differences: Wazuh vs Other Tools

| Tool | Purpose | Best For |
|------|---------|----------|
| **Wazuh** | SIEM + threat detection | Blue team, log analysis, compliance |
| **ELK Stack** | Log aggregation | Flexible, larger deployments |
| **Prometheus** | Metrics monitoring | Infrastructure health, performance |
| **Grafana** | Visualization | Creating dashboards from any data |

**For your studies:** Wazuh is excellent because it bundles SIEM, FIM (File Integrity Monitoring), and vulnerability assessment in one container stack.

---

## Resources for Getting Started

### Wazuh Documentation
- Official Docker deployment guide: https://documentation.wazuh.com
- Quick start: Search "Wazuh Docker compose"
- Community labs and tutorials

### Kali Linux
- Download pre-built VirtualBox images: https://www.kali.org/get-kali/
- VirtualBox documentation for VM management
- Kali docs for tool usage

### Docker
- Official Docker docs: https://docs.docker.com
- Docker Hub: Official Wazuh images available
- Compose reference: https://docs.docker.com/compose/

---

## Backup & Lab Snapshots

### Kali VM Strategy
```bash
# Before starting risky labs
Snapshot → Run lab → Experiment → Revert to snapshot
```
- Makes cleanup easy
- No persistent changes mess up future labs
- Fast reset between study sessions

### Ubuntu/Wazuh Data Persistence
- Use Docker named volumes for database storage
- Regular backups of Wazuh configuration and rules
- Test restore procedures (important for blue team!)

---

## Next Steps

1. **Set up Ubuntu Server** with Docker
2. **Deploy Wazuh container** following official docs
3. **Set up Kali VM** in VirtualBox on Windows
4. **Create lab scenario:** Feed Kali traffic into Wazuh for detection
5. **Document findings** in study notes for Security+ exams

---

## Career Readiness & Practical Skills

This setup directly supports:
- **Blue team fundamentals** – Understanding detection and response workflows
- **SIEM operations** – Wazuh teaches real-world SIEM architecture and rule management
- **Container security** – Docker knowledge increasingly required in modern environments
- **Lab repeatability** – Snapshots and containers enable consistent, reproducible learning
- **Industry relevance** – Skills directly applicable to SOC analyst and blue team roles
- **Balanced security knowledge** – Kali (offensive) + Wazuh (defensive) provides comprehensive perspective

The combination of offensive tools (Kali) and defensive tools (Wazuh) creates a holistic learning environment that mirrors real-world security operations.