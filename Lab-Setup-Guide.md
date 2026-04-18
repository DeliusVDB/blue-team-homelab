# Ubuntu Server 24.04 Cybersecurity Lab Setup Guide

A step-by-step guide for building a complete cybersecurity study lab on Ubuntu Server 24.04 LTS. This guide covers Docker, Portainer, Wazuh SIEM, and essential security tools for blue team studies.

**Target Audience:** Beginners with basic Linux command line knowledge.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Initial Server Hardening](#initial-server-hardening)
3. [Installing Docker](#installing-docker)
4. [Installing Docker Compose](#installing-docker-compose)
5. [Installing Portainer](#installing-portainer)
6. [Installing Wazuh SIEM](#installing-wazuh-siem)
7. [Additional Security Tools](#additional-security-tools)
8. [Network Configuration](#network-configuration)
9. [Verification & Testing](#verification--testing)
10. [Next Steps](#next-steps)

---

## Prerequisites

### System Requirements
- **OS:** Ubuntu Server 24.04 LTS (fresh install)
- **RAM:** Minimum 8GB (16GB recommended for Wazuh)
- **Storage:** Minimum 50GB (100GB+ recommended)
- **CPU:** 2+ cores (4+ recommended)
- **Network:** Static IP or DHCP reservation recommended

### Before Starting
- SSH access to the server configured
- User account with `sudo` privileges
- Basic familiarity with terminal commands

### Update the System First
Always start with a fully updated system:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y
```

Reboot if kernel updates were installed:

```bash
sudo reboot
```

---

## Initial Server Hardening

Before installing any tools, apply basic security hardening. This is good practice and relevant to cybersecurity studies.

### 1. Configure the Firewall (UFW)

```bash
# Enable UFW
sudo ufw enable

# Allow SSH (critical - don't lock yourself out!)
sudo ufw allow 22/tcp

# Check status
sudo ufw status verbose
```

### 2. Install Fail2Ban (Brute-force Protection)

```bash
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# Verify it's running
sudo systemctl status fail2ban
```

### 3. Configure Automatic Security Updates

```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

### 4. Disable Root SSH Login (Optional but Recommended)

```bash
sudo nano /etc/ssh/sshd_config
```

Find and set:
```
PermitRootLogin no
PasswordAuthentication no  # Only if using SSH keys
```

Restart SSH:
```bash
sudo systemctl restart ssh
```

---

## Installing Docker

Docker is the foundation for running containerized security tools.

### 1. Remove Any Old Docker Versions

```bash
sudo apt remove docker docker-engine docker.io containerd runc -y
```

### 2. Install Dependencies

```bash
sudo apt install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

### 3. Add Docker's Official GPG Key

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

### 4. Add Docker Repository

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 5. Install Docker Engine

```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

### 6. Verify Docker Installation

```bash
sudo docker run hello-world
```

A successful installation will display a welcome message.

### 7. Run Docker Without Sudo (Optional but Convenient)

```bash
sudo usermod -aG docker $USER
```

Log out and log back in for this to take effect, then verify:

```bash
docker ps
```

### 8. Enable Docker at Startup

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

---

## Installing Docker Compose

Docker Compose v2 is now included as a Docker plugin. Verify it's installed:

```bash
docker compose version
```

**Note:** Use `docker compose` (with a space), not `docker-compose` (with a hyphen — that was v1).

---

## Installing Portainer

Portainer provides a web-based GUI for managing Docker containers, perfect for beginners.

### 1. Create a Volume for Portainer Data

```bash
docker volume create portainer_data
```

### 2. Deploy Portainer Container

```bash
docker run -d \
  -p 8000:8000 \
  -p 9443:9443 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

### 3. Open Firewall Port

```bash
sudo ufw allow 9443/tcp
```

### 4. Access Portainer

Open a browser and navigate to:
```
https://<server-ip>:9443
```

- Accept the self-signed certificate warning
- Create an admin account on first login
- Select "Docker" as the environment to manage

---

## Installing Wazuh SIEM

Wazuh is a comprehensive SIEM platform — the centerpiece of the blue team lab.

### 1. Install Required Dependencies

```bash
sudo apt install -y git curl
```

### 2. Clone the Wazuh Docker Repository

```bash
cd ~
git clone https://github.com/wazuh/wazuh-docker.git -b v4.9.0
cd wazuh-docker/single-node
```

> **Note:** Check the [Wazuh GitHub releases](https://github.com/wazuh/wazuh-docker) for the latest stable version.

### 3. Generate Indexer Certificates

```bash
docker compose -f generate-indexer-certs.yml run --rm generator
```

### 4. Increase System Limits (Required for Elasticsearch)

Wazuh's indexer requires increased `max_map_count`:

```bash
sudo sysctl -w vm.max_map_count=262144
```

Make it persistent:

```bash
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

### 5. Start the Wazuh Stack

```bash
docker compose up -d
```

This deploys three containers:
- **wazuh.manager** – Core SIEM engine
- **wazuh.indexer** – Data storage (based on OpenSearch)
- **wazuh.dashboard** – Web UI

### 6. Open Firewall Ports

```bash
sudo ufw allow 443/tcp        # Wazuh Dashboard
sudo ufw allow 1514/tcp       # Agent communication
sudo ufw allow 1515/tcp       # Agent enrollment
sudo ufw allow 55000/tcp      # Wazuh API
```

### 7. Access the Wazuh Dashboard

Open a browser and navigate to:
```
https://<server-ip>
```

**Default credentials:**
- Username: `admin`
- Password: `SecretPassword`

> **Important:** Change the default passwords immediately after first login. Instructions are in the Wazuh documentation.

### 8. Verify Wazuh is Running

```bash
docker ps
```

All three Wazuh containers should show as "Up" and "healthy".

---

## Additional Security Tools

These complementary tools enhance the lab environment.

### Suricata (Network IDS/IPS)

Deploy as a Docker container for network intrusion detection:

```bash
docker run -d \
  --name suricata \
  --net=host \
  --cap-add=NET_ADMIN \
  --cap-add=SYS_NICE \
  -v /var/log/suricata:/var/log/suricata \
  jasonish/suricata:latest
```

### Grafana (Visualization)

For creating custom security dashboards:

```bash
docker run -d \
  --name=grafana \
  -p 3000:3000 \
  -v grafana-storage:/var/lib/grafana \
  --restart=always \
  grafana/grafana:latest
```

Access at `http://<server-ip>:3000` (default: admin/admin)

### TheHive & Cortex (Case Management)

For incident response case management:

```bash
# Create docker-compose.yml following TheHive's official documentation
# https://docs.strangebee.com/thehive/
```

### MISP (Threat Intelligence Platform)

```bash
docker run -d \
  --name misp \
  -p 80:80 \
  -p 443:443 \
  -e MYSQL_HOST=db \
  coolacid/misp-docker:core-latest
```

### CyberChef (Data Analysis)

Useful for CTF challenges and encoding/decoding:

```bash
docker run -d \
  --name cyberchef \
  -p 8080:80 \
  --restart=always \
  mpepping/cyberchef:latest
```

Access at `http://<server-ip>:8080`

### Kibana/Elasticsearch (Alternative SIEM Stack)

If exploring the ELK stack instead of Wazuh:

```bash
# Use the official Elastic docker-compose examples
# https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html
```

---

## Network Configuration

### Set a Static IP (Recommended)

Edit the Netplan configuration:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Example configuration:
```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [1.1.1.1, 8.8.8.8]
```

Apply the configuration:
```bash
sudo netplan apply
```

### Install Network Analysis Tools

```bash
sudo apt install -y \
    net-tools \
    tcpdump \
    nmap \
    wireshark-common \
    tshark \
    dnsutils
```

---

## Verification & Testing

### Check All Running Containers

```bash
docker ps
```

Expected containers:
- `portainer`
- `wazuh.manager`
- `wazuh.indexer`
- `wazuh.dashboard`
- Any additional tools installed

### Check Container Health

```bash
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

### Test Service Endpoints

```bash
# Portainer
curl -k https://localhost:9443

# Wazuh Dashboard
curl -k https://localhost

# Verify ports are listening
sudo ss -tulpn | grep LISTEN
```

### Monitor System Resources

```bash
# Check disk usage
df -h

# Check memory
free -h

# Check Docker resource usage
docker stats --no-stream
```

---

## Backup Strategy

### Backup Docker Volumes

```bash
# List all volumes
docker volume ls

# Backup a specific volume
docker run --rm \
  -v portainer_data:/source \
  -v $(pwd):/backup \
  alpine tar czf /backup/portainer_backup.tar.gz -C /source .
```

### Backup Wazuh Configuration

```bash
cd ~/wazuh-docker/single-node
tar czf wazuh-config-backup.tar.gz config/
```

### Schedule Regular Backups

Create a cron job for automated backups:

```bash
sudo crontab -e
```

Add a line for daily 2 AM backups:
```
0 2 * * * /path/to/backup-script.sh
```

---

## Useful Commands Reference

### Docker Management

```bash
# View all containers
docker ps -a

# Stop a container
docker stop <container_name>

# Start a container
docker start <container_name>

# View container logs
docker logs <container_name>

# Follow logs in real-time
docker logs -f <container_name>

# Remove a container
docker rm <container_name>

# Remove an image
docker rmi <image_name>

# Clean up unused resources
docker system prune -a
```

### Docker Compose Management

```bash
# Start services
docker compose up -d

# Stop services
docker compose down

# View logs
docker compose logs -f

# Restart services
docker compose restart

# Pull latest images
docker compose pull
```

### System Monitoring

```bash
# Check system logs
sudo journalctl -xe

# Check specific service
sudo systemctl status docker

# Monitor processes
htop  # (install with: sudo apt install htop)
```

---

## Troubleshooting

### Docker Permission Denied
```bash
# Ensure user is in docker group
sudo usermod -aG docker $USER
# Log out and back in
```

### Wazuh Dashboard Won't Start
```bash
# Check if vm.max_map_count is set
sysctl vm.max_map_count
# Should return: vm.max_map_count = 262144
```

### Port Already in Use
```bash
# Find what's using a port
sudo ss -tulpn | grep :<port>

# Kill the process or change the port in docker-compose.yml
```

### Container Keeps Restarting
```bash
# Check logs for errors
docker logs <container_name>

# Inspect the container
docker inspect <container_name>
```

---

## Next Steps

Once the lab is running, consider these progressive learning paths:

### Week 1-2: Foundation
- Explore the Wazuh dashboard interface
- Install a Wazuh agent on a test machine
- Generate test logs and observe detection

### Week 3-4: Configuration
- Customize Wazuh detection rules
- Create custom alerts
- Integrate with Kali Linux for attack simulation

### Month 2: Advanced Topics
- MITRE ATT&CK framework integration
- Threat intelligence feeds with MISP
- Network monitoring with Suricata
- Custom detection rule writing

### Month 3+: Specialization
- Incident response workflows
- Forensics tooling
- SOAR (Security Orchestration, Automation, and Response)
- Certification lab practice (Security+, CySA+, BTL1)

---

## Resources

### Official Documentation
- [Wazuh Documentation](https://documentation.wazuh.com/)
- [Docker Documentation](https://docs.docker.com/)
- [Portainer Documentation](https://docs.portainer.io/)
- [Ubuntu Server Documentation](https://ubuntu.com/server/docs)

### Learning Platforms
- TryHackMe – Hands-on cybersecurity labs
- HackTheBox – Advanced penetration testing
- Professor Messer – Free Security+ training
- Blue Team Labs Online – Defensive security training

### Communities
- r/cybersecurity
- r/AskNetsec
- r/Wazuh
- Wazuh Slack Community

---

## Security Reminders

- **Change default passwords** on all services immediately
- **Keep systems updated** regularly with `apt update && apt upgrade`
- **Monitor logs** proactively rather than reactively
- **Test backups** regularly — untested backups are not backups
- **Document changes** to the environment for audit purposes
- **Segment networks** when possible using VLANs or Docker networks
- **Use SSH keys** instead of password authentication
- **Enable MFA** on all accessible services that support it

---

*This guide is a living document — update it as tools and best practices evolve.*