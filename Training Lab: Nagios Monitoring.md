
# Nagios Monitoring Training
**Module 2 – Monitoring Servers and HAProxy**

---

## Objective

In this lab, trainees will:

- Install Nagios Core on Ubuntu
- Add Linux hosts to monitoring
- Install NRPE agents on servers
- Monitor system resources
- Monitor HAProxy service
- Understand NOC monitoring workflow

Official installation guide:

https://assets.nagios.com/downloads/nagioscore/docs/nagioscore/4/en/quickstart-ubuntu.html

---

## Lab Environment

| Server | Role | Example IP |
|---|---|---|
| nagios01 | Nagios Server | 192.168.1.50 |
| web01 | Web Server | 192.168.1.101 |
| web02 | Web Server | 192.168.1.102 |
| lb01 | HAProxy Master | 192.168.1.201 |
| lb02 | HAProxy Backup | 192.168.1.202 |

---

# Part 1 – Install Nagios Core

Follow the official guide step-by-step:

https://assets.nagios.com/downloads/nagioscore/docs/nagioscore/4/en/quickstart-ubuntu.html

After installation, access:

http://(nagios-ip)/nagios

---

# Part 2 – Install NRPE on All Servers

On the following servers:
- web01
- web02
- lb01
- lb02

## Install NRPE

```bash
sudo apt update
sudo apt install nagios-nrpe-server nagios-plugins -y
```

---

## Allow Nagios Server

Edit configuration:

```
/etc/nagios/nrpe.cfg
```

Find:

```
allowed_hosts=127.0.0.1
```

Change to:

```
allowed_hosts=127.0.0.1,192.168.1.50
```

Restart service:

```bash
sudo systemctl restart nagios-nrpe-server
```

---

# Part 3 – Test NRPE from Nagios Server

On nagios01:

```bash
/usr/lib/nagios/plugins/check_nrpe -H 192.168.1.101
```

Expected output:

```
NRPE vX.X
```

---

# Part 4 – Add Hosts in Nagios

Create directory if not exists:

```bash
mkdir /usr/local/nagios/etc/servers
```

Create host configuration:

```
/usr/local/nagios/etc/servers/web01.cfg
```

Example:

```
define host {
    use             linux-server
    host_name       web01
    alias           Web Server 01
    address         192.168.1.101
}
```

Repeat for:
- web02
- lb01
- lb02

---

## Include Directory

Edit:

```
/usr/local/nagios/etc/nagios.cfg
```

Add:

```
cfg_dir=/usr/local/nagios/etc/servers
```

Restart Nagios:

```bash
sudo systemctl restart nagios
```

---

# Part 5 – Monitor System Resources

Example service configuration:

```
define service {
    use                 generic-service
    host_name           web01
    service_description CPU Load
    check_command       check_nrpe!check_load
}
```

Common checks:

| Resource | NRPE Command |
|---|---|
| CPU | check_load |
| Disk | check_disk |
| Users | check_users |

---

# Part 6 – Monitor HAProxy Service

## On lb01 and lb02

Edit:

```
/etc/nagios/nrpe.cfg
```

Add:

```
command[check_haproxy]=/usr/lib/nagios/plugins/check_procs -c 1: -C haproxy
```

Restart NRPE:

```bash
sudo systemctl restart nagios-nrpe-server
```

---

## Add Service in Nagios

Example:

```
define service {
    use                 generic-service
    host_name           lb01
    service_description HAProxy Service
    check_command       check_nrpe!check_haproxy
}
```

Repeat for lb02.

Restart Nagios:

```bash
sudo systemctl restart nagios
```

---

# Part 7 – Validation

## Stop HAProxy

On lb01:

```bash
sudo systemctl stop haproxy
```

Nagios should show:

CRITICAL – HAProxy not running

Start again:

```bash
sudo systemctl start haproxy
```

---

# Part 8 – Trainee Tasks

### Task 1
Add monitoring for:
- Ping
- Disk usage

---

### Task 2
Create failure scenario:
- Stop nginx on web01
- Observe alert in Nagios

---

### Task 3
Check Nagios logs:

```
/usr/local/nagios/var/nagios.log
```

---

### Task 4 – Advanced (Real Operations Scenario)

Add HAProxy configuration validation:

In NRPE:

```
command[check_haproxy_cfg]=/usr/sbin/haproxy -c -f /etc/haproxy/haproxy.cfg
```

Then create a service in Nagios to monitor configuration validity.

---

# Learning Outcomes

After this lab, trainees will understand:

- NOC monitoring concepts
- Agent-based monitoring (NRPE)
- Host and service monitoring
- HAProxy health monitoring
- Alert troubleshooting
- Real production monitoring workflow

---

# Instructor Notes

This lab simulates real Operations scenarios:

- Service failure detection
- Load balancer monitoring
- Infrastructure visibility
- Incident response workflow
