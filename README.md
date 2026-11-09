# Linux-cloud-server-build-hardening
AWS EC2 Ubuntu Linux server deployment, hardening, Nginx, automated backups and monitoring.
# Linux Cloud Server Build & Hardening

![AWS](https://img.shields.io/badge/Cloud-AWS-orange)
![Ubuntu](https://img.shields.io/badge/OS-Ubuntu%2024.04-E95420)
![Nginx](https://img.shields.io/badge/Web%20Server-Nginx-green)
![Bash](https://img.shields.io/badge/Scripting-Bash-black)
![Status](https://img.shields.io/badge/Project-Completed%20after%20verification-success)

## 1. Project Overview

This project demonstrates the deployment, configuration and security hardening of an Ubuntu Linux server hosted on Amazon Web Services (AWS) EC2.

The project was completed as a practical DevOps infrastructure assignment, focusing on secure remote administration, Linux user and group management, file permissions, firewall configuration, web server deployment, automated backups and basic server monitoring.

The objective was to build a secure and manageable Linux environment that could support internal operational tools for a growing UK online retailer.

## 2. Client Brief

A growing UK online retailer required a secure Linux server hosted on AWS to support its internal tools.

The requirements were to:

- Provision an Ubuntu EC2 instance.
- Restrict SSH access to authorised administrators.
- Update and maintain the Linux operating system.
- Create Linux users and groups with appropriate privileges.
- Configure project directory ownership and permissions.
- Harden SSH authentication.
- Configure the UFW firewall.
- Install and configure Nginx.
- Automate daily backups using Bash and cron.
- Monitor system health and inspect service logs.
- Produce a technical runbook and deployment evidence.

## 3. What I Built

I provisioned an Ubuntu Linux server on AWS EC2 and implemented the following configurations:

| Area | Implementation |
|---|---|
| Cloud infrastructure | Ubuntu EC2 instance |
| Remote access | SSH key-based authentication |
| Linux administration | Two users and a devops group |
| Privilege management | Sudo access for the administrator |
| File management | `/bmn` project directory |
| Permissions | Ownership and `chmod 2770` |
| SSH security | Root and password login disabled |
| Firewall | AWS Security Group and UFW |
| Web server | Nginx |
| Automation | Bash backup script |
| Scheduling | Daily cron job |
| Monitoring | Disk, memory, CPU and Nginx logs |

## 4. Technologies and Tools Used

- **Amazon Web Services (AWS):** Cloud infrastructure platform.
- **Amazon EC2:** Virtual server hosting.
- **Ubuntu Server 24.04 LTS:** Linux operating system.
- **SSH:** Secure remote server administration.
- **Linux:** User, group, process and file management.
- **UFW:** Host-based firewall.
- **Nginx:** Web server.
- **Bash:** Backup automation.
- **Cron:** Scheduled task management.
- **Git and GitHub:** Source control and project documentation.

## 5. Architecture Diagram

The following diagram illustrates the intended infrastructure architecture.

```mermaid
flowchart TD
    A[Administrator Laptop]
    B[Internet]
    C[AWS Security Group]
    D[EC2 Ubuntu Server]
    E[UFW Firewall]
    F[SSH Port 22]
    G[HTTP Port 80]
    H[Linux Users and Groups]
    I[Nginx Web Server]
    J[/bmn Project Directory]
    K[Bash Backup Script]
    L[Cron Scheduler]
    M[Local Backup Storage]

    A --> B
    B --> C
    C -->|Authorised SSH IP| F
    C -->|HTTP| G
    F --> D
    G --> D
    D --> E
    E --> H
    E --> I
    E --> J
    L --> K
    K --> J
    K --> M
```

**Note:** This is a logical representation. The local backup directory is hosted on the same EC2 instance. Production disaster recovery should include off-instance backup storage.

## 6. Repository Structure

```text
Linux-cloud-server-build-hardening/
│
├── README.md
├── .gitignore
│
├── docs/
│   ├── runbook.md
│   └── architecture.md
│
├── diagrams/
│   └── architecture-diagram.png
│
├── screenshots/
│   ├── 01-ec2-instance.png
│   ├── 02-ssh-connection.png
│   ├── 03-system-update.png
│   ├── 04-users-and-groups.png
│   ├── 05-directory-permissions.png
│   ├── 06-ssh-hardening.png
│   ├── 07-ufw-firewall.png
│   ├── 08-nginx-public-ip.png
│   ├── 09-backup-script.png
│   ├── 10-backup-evidence.png
│   ├── 11-cron-schedule.png
│   └── 12-server-monitoring.png
│
├── scripts/
│   └── bmn-backup.sh
│
└── config/
    ├── ssh-hardening.conf
    └── bmn-backup.cron
```

## 7. How to Reproduce the Deployment

### Prerequisites

Before beginning, you will need:

- An AWS account.
- An Ubuntu EC2 instance.
- An SSH key pair.
- A terminal application.
- An authorised public IP address.
- Git installed on your workstation.

### Step 1: Clone the repository

```bash
git clone https://github.com/YOUR-GITHUB-USERNAME/Linux-cloud-server-build-hardening.git

cd Linux-cloud-server-build-hardening
```

### Step 2: Connect to the EC2 instance

```bash
chmod 400 BMN-DevOps-Key.pem

ssh -i BMN-DevOps-Key.pem ubuntu@YOUR-EC2-PUBLIC-IP
```

Replace the placeholders with your own approved connection details. Keep private connection information out of the public repository.

### Step 3: Update Ubuntu

```bash
sudo apt update

sudo apt upgrade -y
```

### Step 4: Create Linux users and groups

```bash
sudo groupadd devops

sudo adduser devadmin

sudo adduser devuser

sudo usermod -aG devops devadmin

sudo usermod -aG devops devuser

sudo usermod -aG sudo devadmin
```

### Step 5: Create the project directory

```bash
sudo mkdir -p /bmn

sudo chown root:devops /bmn

sudo chmod 2770 /bmn
```

### Step 6: Configure SSH security

Create:

`/etc/ssh/sshd_config.d/99-bmn-hardening.conf`

Configuration:

```text
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
KbdInteractiveAuthentication no
```

Validate and reload:

```bash
sudo sshd -t

sudo systemctl reload ssh
```

Test a second SSH connection before closing your original session.

### Step 7: Configure UFW

Replace `YOUR_PUBLIC_IP` with your authorised public IP.

```bash
sudo ufw default deny incoming

sudo ufw default allow outgoing

sudo ufw allow from YOUR_PUBLIC_IP to any port 22 proto tcp

sudo ufw allow 80/tcp

sudo ufw enable

sudo ufw status verbose
```

The AWS Security Group must also have matching SSH and HTTP inbound rules.

### Step 8: Install Nginx

```bash
sudo apt install nginx -y

sudo systemctl enable nginx

sudo systemctl start nginx

sudo systemctl status nginx
```

Visit the EC2 public IPv4 address using HTTP to verify the Nginx welcome page.

### Step 9: Configure automated backups

Copy the backup script from `scripts/bmn-backup.sh` to:

`/usr/local/bin/bmn-backup.sh`

Make it executable:

```bash
sudo chmod 750 /usr/local/bin/bmn-backup.sh
```

Run a manual test:

```bash
sudo /usr/local/bin/bmn-backup.sh
```

Configure the root cron schedule:

```cron
0 2 * * * /usr/local/bin/bmn-backup.sh
```

Verify:

```bash
sudo crontab -l

sudo systemctl status cron
```

### Step 10: Monitor the server

```bash
df -h

free -m

top

sudo systemctl status nginx

sudo tail -n 20 /var/log/nginx/access.log

sudo tail -n 20 /var/log/nginx/error.log
```

## 8. Backup Strategy

The project includes a Bash script that:

- Backs up the `/bmn` project directory.
- Creates timestamped compressed archives.
- Stores archives in `/var/backups/bmn`.
- Records backup activity in a log.
- Removes old matching backup archives after the retention period.

The backup is scheduled to execute daily at 02:00 using cron.

For production environments, off-instance encrypted backups, retention policies and recovery testing should also be implemented.

## 9. Monitoring and Verification

The following Linux commands were used to inspect the server:

| Command | Purpose |
|---|---|
| `df -h` | Disk usage |
| `free -m` | Memory usage |
| `top` | Running processes and CPU |
| `systemctl status nginx` | Nginx service health |
| `journalctl` | System logs |
| `tail` | Inspect recent Nginx logs |
| `ufw status verbose` | Firewall configuration |

## 10. Screenshots and Evidence

Screenshots demonstrating the deployment and verification process are available in the `screenshots/` directory.

Key evidence includes:

- AWS EC2 running instance.
- Successful SSH connection.
- Ubuntu package updates.
- Linux users and groups.
- Project directory permissions.
- SSH hardening configuration.
- UFW firewall rules.
- Nginx welcome page accessed through the EC2 public IP.
- Backup script and generated archive.
- Cron schedule.
- Server health and Nginx logs.

### Nginx Web Server Evidence

![Nginx Welcome Page](screenshots/08-nginx-public-ip.png)

## 11. Technical Documentation

For detailed operational instructions, troubleshooting and maintenance procedures, see:

- [Deployment and Operations Runbook](docs/runbook.md)
- [Architecture Documentation](docs/architecture.md)

## 12. Security Considerations

Security controls implemented include:

- SSH access restricted by source IP.
- Key-based SSH authentication.
- Root SSH login disabled.
- SSH password authentication disabled.
- UFW firewall enabled.
- AWS Security Group configured.
- Linux user privileges separated.
- Restricted project directory permissions.

No private SSH keys, credentials, passwords or confidential client information should be committed to this repository.

## 13. Future Improvements

Potential improvements for a production deployment include:

- Amazon S3 for off-instance backup storage.
- AWS CloudWatch for metrics, alarms and log monitoring.
- AWS Systems Manager for managed server access.
- HTTPS using TLS certificates.
- Automated infrastructure provisioning using Terraform.
- Centralised log management.
- Regular security patching and vulnerability assessment.
- Automated backup restoration tests.

## 14. Project Status

The project should be marked complete only after all deployment tasks have been tested and screenshot evidence captured.

**Author:** Enyinna G. Anyanwu

**Project:** Linux Cloud Server Build and Hardening

**Platform:** AWS EC2

**Operating System:** Ubuntu Linux

---

*This repository is intended as a practical DevOps learning and portfolio project. Production deployment requires appropriate client security approval, access controls, backup policies and operational monitoring.*
