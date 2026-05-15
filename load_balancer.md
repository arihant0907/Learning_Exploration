# What is HAProxy?

** HAProxy is a high-performance load balancer and reverse proxy that distributes incoming traffic across multiple backend servers. ** 

*What it does:
1.Distributes web traffic (HTTP/HTTPS)
2.Balances TCP traffic (databases, APIs, etc.)
3.Performs health checks
4.Improves scalability and reliability

### Simple idea:
```
Users → HAProxy → Multiple App Servers
```
### Combined Architecture:
```
            Virtual IP (192.168.1.100)
                        |
         --------------------------------
         |                              |
   HAProxy + Keepalived         HAProxy + Keepalived
     Server 1 (MASTER)          Server 2 (BACKUP)
                        |
                Backend App Servers
```

### 1.Install HAProxy

Ubuntu/Debian:
```
sudo apt update
sudo apt install haproxy -y
```
RHEL/CentOS:
```
sudo yum install haproxy -y
```
Enable service:
```
sudo systemctl enable haproxy
```

### 2.Configure HAProxy

Edit:
```Bash
sudo nano /etc/haproxy/haproxy.cfg
```
Example configuration:
```
global
    log /dev/log local0
    daemon

defaults
    mode http
    timeout connect 5s
    timeout client 30s
    timeout server 30s

frontend http_front
    bind *:80
    default_backend app_servers

backend app_servers
    balance roundrobin
    option httpchk GET /health

    server app1 192.168.1.10:80 check
    server app2 192.168.1.11:80 check
```
### 3.Optional: HAProxy Stats Page

Add:
```Conf
listen stats
    bind *:8404
    stats enable
    stats uri /stats
    stats refresh 10s
```
Access:
```
http://192.168.1.100:8404/stats
```

