# Linux Server Operations Runbook

## 1. Purpose

This runbook provides operational instructions for administering, maintaining, securing and monitoring the BMN Ubuntu EC2 server.

It is intended for authorised system administrators and DevOps engineers.

## 2. Server Information

| Item | Value |
|---|---|
| Cloud provider | AWS |
| Service | EC2 |
| OS | Ubuntu Server 24.04 LTS |
| Region | eu-west-2 |
| Instance name | BMN-Ubuntu-Server |
| Project directory | /bmn |
| Web server | Nginx |
| Backup directory | /var/backups/bmn |

Actual IP addresses and sensitive infrastructure identifiers should be recorded in the client's approved private inventory, not this public runbook.

## 3. Connecting to the Server

From an authorised administrator workstation:

```bash
chmod 400 BMN-DevOps-Key.pem

ssh -i BMN-DevOps-Key.pem ubuntu@YOUR-EC2-PUBLIC-IP
```

Replace the placeholder with the server's actual public IPv4 address.

If SSH access fails, verify:

1. The EC2 instance is running.
2. The administrator's public IP is authorised in the AWS Security Group.
3. The same IP is allowed in UFW.
4. The correct private key is being used.
5. The SSH service is running.

## 4. Linux User Administration

### List users

```bash
getent passwd
```

### Inspect the devops group

```bash
getent group devops
```

### Check user group memberships

```bash
groups devadmin

groups devuser
```

### Create a new user

```bash
sudo adduser NEW_USERNAME
```

### Add a user to the devops group

```bash
sudo usermod -aG devops NEW_USERNAME
```

### Grant sudo access

Only when approved:

```bash
sudo usermod -aG sudo NEW_USERNAME
```

### Remove sudo access

```bash
sudo gpasswd -d NEW_USERNAME sudo
```

### Remove a user from the server

Only after approval and confirmation that their files and responsibilities have been transferred:

```bash
sudo deluser NEW_USERNAME
```

## 5. Project Directory Management

Project directory:

```text
/bmn
```

Check ownership and permissions:

```bash
ls -ld /bmn
```

Expected ownership:

```text
root:devops
```

Expected permissions:

```text
2770
```

Restore the directory configuration if necessary:

```bash
sudo chown root:devops /bmn

sudo chmod 2770 /bmn
```

## 6. SSH Security Management

SSH configuration file:

```text
/etc/ssh/sshd_config.d/99-bmn-hardening.conf
```

Required configuration:

```text
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
KbdInteractiveAuthentication no
```

Validate configuration:

```bash
sudo sshd -t
```

Check effective settings:

```bash
sudo sshd -T | grep -E 'permitrootlogin|passwordauthentication|pubkeyauthentication|kbdinteractiveauthentication'
```

Reload SSH:

```bash
sudo systemctl reload ssh
```

Check service status:

```bash
sudo systemctl status ssh
```

Always retain an existing working administrative session until a second SSH connection has been successfully tested.

## 7. Firewall Management

Check firewall status:

```bash
sudo ufw status verbose
```

Required inbound access:

| Service | Port | Access |
|---|---|---|
| SSH | 22 | Authorised administrator IP |
| HTTP | 80 | Public for demonstration |

To inspect numbered rules:

```bash
sudo ufw status numbered
```

Do not remove or change the SSH allow rule without ensuring that another authorised management route is available.

## 8. Nginx Management

### Check service status

```bash
sudo systemctl status nginx
```

### Start Nginx

```bash
sudo systemctl start nginx
```

### Stop Nginx

```bash
sudo systemctl stop nginx
```

### Restart Nginx

```bash
sudo systemctl restart nginx
```

### Reload Nginx configuration

```bash
sudo systemctl reload nginx
```

### Check whether Nginx starts at boot

```bash
sudo systemctl is-enabled nginx
```

### Test the web server

```bash
curl -I http://localhost
```

## 9. Backup Operations

Backup script:

```text
/usr/local/bin/bmn-backup.sh
```

Backup location:

```text
/var/backups/bmn
```

### Run a manual backup

```bash
sudo /usr/local/bin/bmn-backup.sh
```

### List available backups

```bash
sudo ls -lh /var/backups/bmn/
```

### Inspect backup activity

```bash
sudo cat /var/log/bmn-backup.log
```

### Check cron schedule

```bash
sudo crontab -l
```

Expected schedule:

```cron
0 2 * * * /usr/local/bin/bmn-backup.sh
```

### Restore a backup

First, identify the correct archive:

```bash
sudo ls -lh /var/backups/bmn/
```

Inspect its contents:

```bash
sudo tar -tzf /var/backups/bmn/NAME_OF_BACKUP.tar.gz
```

For a safe recovery test, create a separate restoration directory:

```bash
sudo mkdir -p /tmp/bmn-restore-test
```

Extract the archive there:

```bash
sudo tar -xzf /var/backups/bmn/NAME_OF_BACKUP.tar.gz -C /tmp/bmn-restore-test
```

Inspect the restored files:

```bash
sudo ls -la /tmp/bmn-restore-test/bmn
```

Only restore over the original project directory after verifying the archive, obtaining approval and planning the recovery.

## 10. Server Health Monitoring

### Disk space

```bash
df -h
```

### Memory

```bash
free -m
```

### Processes and CPU

```bash
top
```

### System errors

```bash
sudo journalctl -p err -n 20
```

### Nginx access logs

```bash
sudo tail -n 20 /var/log/nginx/access.log
```

### Nginx error logs

```bash
sudo tail -n 20 /var/log/nginx/error.log
```

### SSH logs

```bash
sudo journalctl -u ssh -n 30
```

## 11. Troubleshooting

| Problem | Recommended action |
|---|---|
| SSH connection refused | Check EC2 status, SSH service and firewall rules |
| SSH permission denied | Verify username, private key and authorised public key |
| Nginx page unavailable | Check Nginx service, AWS Security Group and UFW |
| Nginx fails to start | Inspect Nginx error logs and configuration |
| Backup fails | Check source directory, disk space, script permissions and logs |
| Cron job does not execute | Check cron service, root crontab and backup logs |
| Disk space is low | Inspect filesystem usage and investigate large files |

## 12. Maintenance Recommendations

- Review user access periodically.
- Apply approved security updates.
- Monitor disk and memory utilisation.
- Review SSH authentication logs.
- Verify backup completion.
- Perform periodic restoration tests.
- Keep off-instance backups.
- Review firewall rules.
- Monitor Nginx availability.

## 13. Operational Handover

The receiving team should confirm:

- Administrative access has been tested.
- Linux user permissions are correct.
- SSH security controls are active.
- Firewall rules are correct.
- Nginx is running.
- Backup automation is operational.
- Restoration has been tested.
- Monitoring commands are understood.

**Runbook author:** Enyinna FOR DevOps Team

**Document status:** Update after deployment verification.
