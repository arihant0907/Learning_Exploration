# What is Keepalived?

** Keepalived is a high availability (HA) tool used to assign a Virtual IP (VIP) and provide failover between servers.**
The below setup is known as MySQL dual-master replication setup or master - master replication setup 

### What it does:
1.Creates a floating Virtual IP
2.Ensures failover between servers
3.Uses VRRP (Virtual Router Redundancy Protocol)
4.Detects failures and switches active node

### Types of Master-Master Replication:
> 1. Active-Active
** Both servers accept writes.** 
```
App → DB1
App → DB2
```
Problem
* Can cause:
1.Split brain
2.Conflicts
3.Duplicate keys
4.Data inconsistency

> [!NOTE]
> Especially risky with only 2 nodes.

> 2. Active-Passive (Your Setup)

** Only one server accepts writes.**
```
App → DB1 (Active)
DB2 = Standby
```
> [!NOTE]
> B2 becomes active only during failover.

*Benefits
1.Safer
2.Easier to manage
3.Better consistency
4.Lower risk of conflicts

### Simple idea:
>> Server1 (MASTER) ← VIP → Server2 (BACKUP)

If Server1 fails → VIP moves to Server2 automatically.

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
### 1.Configure Keepalived (MASTER NODE): 

On Server 1:
```Bash
sudo nano /etc/keepalived/keepalived.conf
```

MASTER configuration:

```conf
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 101
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass mypassword
    }

    virtual_ipaddress {
        192.168.1.100
    }
}
```
### 2.Configure Keepalived (BACKUP NODE):
On Server 2:

```
vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass mypassword
    }

    virtual_ipaddress {
        192.168.1.100
    }
}
```

### 3.Start Services:
On both servers:

```Bash
sudo systemctl restart haproxy
sudo systemctl restart keepalived
```
Check status:
```
systemctl status haproxy
systemctl status keepalived
```
### 4.Verify Setup
Check VIP assignment

On MASTER:
```
ip addr
```
---
Comparison with Your Current Setup:

| Technology                            | Automatic Failover | Split Brain Protection | Complexity | Cost   |
| ------------------------------------- | ------------------ | ---------------------- | ---------- | ------ |
| Your Current Dual Master + Keepalived | Partial            | Limited                | Low        | Low    |
| InnoDB Cluster                        | Yes                | Strong                 | Medium     | Medium |
| Galera Cluster                        | Yes                | Strong                 | High       | High   |
| ProxySQL + Orchestrator               | Yes                | Medium                 | Medium     | Medium |
| Cloud Managed HA                      | Yes                | Strong                 | Low        | High   |
| Distributed DB                        | Yes                | Strong                 | High       | High   |
