
# High Availability Web Environment Training
**For Trainees – Networking & Operations Lab**

---

## Objective

Build a High Availability Web Architecture that includes:

- Publishing a sample website
- Load balancing using HAProxy
- Failover using Keepalived (Virtual IP)
- Simulating real production behavior

---

## Lab Environment

| Server | Role | Example IP |
|---|---|---|
| web01 | Web Server | 192.168.1.101 |
| web02 | Web Server | 192.168.1.102 |
| lb01 | HAProxy + Keepalived (MASTER) | 192.168.1.201 |
| lb02 | HAProxy + Keepalived (BACKUP) | 192.168.1.202 |
| VIP | Virtual IP | 192.168.1.200 |

---

# Part 1 – Publish Sample Website

## On web01

```bash
sudo apt update
sudo apt install nginx -y
echo "This is WEB01" | sudo tee /var/www/html/index.html
sudo systemctl enable nginx
sudo systemctl start nginx
```

## On web02

```bash
sudo apt update
sudo apt install nginx -y
echo "This is WEB02" | sudo tee /var/www/html/index.html
sudo systemctl enable nginx
sudo systemctl start nginx
```

### Test
Open:
http://192.168.1.101  
http://192.168.1.102

---

# Part 2 – Install HAProxy (lb01 & lb02)

```bash
sudo apt update
sudo apt install haproxy -y
sudo systemctl enable haproxy
```

Edit:
```
/etc/haproxy/haproxy.cfg
```

Add:

```cfg
frontend http_front
    bind *:80
    default_backend web_servers

backend web_servers
    balance roundrobin
    option httpchk
    server web01 192.168.1.101:80 check
    server web02 192.168.1.102:80 check
```

Validate & restart:

```bash
haproxy -c -f /etc/haproxy/haproxy.cfg
sudo systemctl restart haproxy
```

Test:
http://192.168.1.201

---

# Part 3 – Install Keepalived

On both load balancers:

```bash
sudo apt install keepalived -y
sudo systemctl enable keepalived
```

## lb01 (MASTER)

```
/etc/keepalived/keepalived.conf
```

```cfg
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 101
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass 1234
    }

    virtual_ipaddress {
        192.168.1.200
    }
}
```

## lb02 (BACKUP)

```cfg
vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass 1234
    }

    virtual_ipaddress {
        192.168.1.200
    }
}
```

Start service:

```bash
sudo systemctl restart keepalived
ip a
```

VIP should appear on lb01.

---

# Part 4 – Final Test

Access:
http://192.168.1.200

You should see traffic alternating between WEB01 and WEB02.

---

# Part 5 – Failover Test

Stop MASTER:

```bash
sudo systemctl stop keepalived
```

Check lb02:

```bash
ip a
```

VIP should move to lb02.

Test again:
http://192.168.1.200

---

# Part 6 – Health Check Test

On web01:

```bash
sudo systemctl stop nginx
```

Traffic should go only to WEB02.

---

# Part 7 – Trainee Tasks

### Task 1
Change algorithm:
```
balance leastconn
```

### Task 2 – Logs

```bash
journalctl -u haproxy
journalctl -u keepalived
```

### Task 3 – Service Status

```bash
systemctl status haproxy
systemctl status keepalived
```

### Task 4 – Validate Config

```bash
haproxy -c -f /etc/haproxy/haproxy.cfg
```

---

# Learning Outcomes

- Networking fundamentals
- Load balancing concepts
- High Availability design
- Virtual IP failover
- Health checks
- Troubleshooting production-like environments

---

# Optional Advanced Tasks

- Add third backend server
- Configure HTTPS
- Integrate with Nagios
- Track HAProxy process using Keepalived
