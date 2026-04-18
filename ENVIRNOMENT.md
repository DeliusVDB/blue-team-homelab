# Cybersecurity Study Environment Setup

## Overview
A dedicated Ubuntu Server 24.04 LTS environment for blue team cybersecurity studies, with Docker containerization for security tools and lab environments. Includes guidance on setting up Kali Linux for offensive security modules.

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

## Kali Linux for Studies

### Setup Option 1: VirtualBox on Windows 11 (Recommended)
**Yes, you can absolutely do this from Windows 11.**

#### Setup Steps:
1. **Install VirtualBox** on Windows 11
   - Download from virtualbox.org
   - Free and lightweight
2. **Download Kali Linux VM** 
   - Official pre-built Kali VirtualBox images available
   - Or download ISO and create VM from scratch
3. **Import or Create VM**
   - Allocate adequate resources: 4GB RAM minimum, 20GB disk
   - Enable nested virtualization if needed
4. **Launch and Study**
   - Run labs isolated from your main system
   - Snapshot VM before each major lab (easy rollback)

**Advantages:**
- No need to reboot into Linux
- Easy switching between Windows work and Kali studies
- Snapshots for lab reset
- Portable (can move VM to different machines)

### Setup Option 2: VirtualBox on Ubuntu Server
**Not recommended** for your use case.

Why:
- Ubuntu Server has no GUI, difficult to manage VirtualBox
- VirtualBox requires X11/Wayland (heavyweight for a server)
- Better to use Docker for containerized tools on Ubuntu

### Setup Option 3: Dual Boot (Not Recommended)
- Kali as second partition on Linux
- Loses convenience of snapshots and isolation
- Extra maintenance overhead

---

## Recommended Study Environment Architecture

```
┌─────────────────────────────────────────────┐
│         Windows 11 Workstation              │
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

### Security+ Studies
- **Windows:** Kali VM for network analysis labs (wireshark, nmap)
- **Ubuntu:** Docker setup for understanding SIEM, log management, security controls
- **Wazuh:** Deploy container, learn log ingestion and threat detection

### CySA+ Studies (Vulnerability Management)
- **Windows Kali:** Vulnerability scanning (Nessus, OpenVAS in containers)
- **Ubuntu Wazuh:** Monitor scan results, correlate with log data, create detection rules

### BTL1 Studies (Threat Intelligence)
- **Ubuntu Wazuh:** Analyze indicators of compromise (IOCs), create detection rules
- **Kali:** Understand attack techniques (MITRE ATT&CK framework)
- **Docker:** Containerize threat intel tools (YARA, threat feeds)

---

## Initial Setup Checklist

### Windows 11
- [ ] Install VirtualBox
- [ ] Download Kali Linux ISO or pre-built VM image
- [ ] Create/import Kali VM with snapshots enabled
- [ ] Test network connectivity to Ubuntu server

### Ubuntu Server 24.04
- [ ] Install Docker and docker-compose
- [ ] Install Portainer (optional, for GUI management)
- [ ] Pull Wazuh Docker images
- [ ] Create docker-compose.yml for Wazuh stack
- [ ] Configure persistent volumes for data
- [ ] Test Wazuh dashboard access from Windows

### Initial Wazuh Deployment
```bash
# Create docker-compose.yml with:
# - Wazuh Manager
# - Wazuh Dashboard (web UI)
# - Elasticsearch (data storage)

docker-compose up -d

# Access dashboard at https://ubuntu-server-ip
# Default credentials provided in compose file
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

## Notes for Your Security+ Journey

This setup directly supports:
- **Blue team fundamentals** – Understanding detection and response
- **SIEM concepts** – Wazuh teaches real SIEM operations
- **Container security** – Docker knowledge increasingly required
- **Lab repeatability** – Snapshots and containers enable consistent learning
- **Career readiness** – Skills directly applicable to SOC analyst roles in South Africa and Netherlands

The combination of Kali (offense) + Wazuh (defense) gives you balanced knowledge for understanding security both ways.