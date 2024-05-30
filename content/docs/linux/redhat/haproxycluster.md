---
weight: 999
title: "Setting Up HAProxy Clusters"
description: ""
icon: "article"
date: "2024-05-30T14:35:56-05:00"
lastmod: "2024-05-30T14:35:56-05:00"
draft: false
toc: true
---

## Install Required Packages

Install `haproxy` and `keepalived` packages.

```bash
dnf install -y keepalived haproxy
```

## Configure Keepalived 


Configure firewall rules for keepalived `VRRP` protocol.

```bash
firewall-cmd --add-rich-rule='rule protocol value="vrrp" accept' --permanent
firewall-cmd --reload
```

Configure keepalived.

```bash
vi /etc/keepalived/keepalived.conf
```

```bash
global_defs {

   notification_email {
       admin@example.com
   }
   notification_email_from noreply@example.com
   smtp_server 127.0.0.1
   smtp_connect_timeout 60
}

vrrp_script chk_haproxy {
  script "killall -0 haproxy"   # check the haproxy process
  interval 2                    # every 2 seconds
  weight 2                      # add 2 points if OK
}

vrrp_instance VI_1 {
  interface eth0                # interface to monitor
  state MASTER                  # MASTER on haproxy, BACKUP on haproxy2
  virtual_router_id 51
  priority 101                  # 101 on haproxy, 100 on haproxy2
  authentication {
    auth_type PASS
    auth_pass {{ auth_pass }}
  }
  virtual_ipaddress {
    192.168.0.100               # virtual ip address
  }
  track_script {
    chk_haproxy
  }
}
```

```bash
sysctl -w net.ipv4.ip_forward=1
systemctl enable --now keepalived
```

## Configure HAProxy

Configure `haproxy` for `SELinux` HTTP.

```bash
vi /etc/firewalld/services/haproxy-http.xml
```

```bash
<?xml version="1.0" encoding="utf-8"?>
<service>
<short>HAProxy-HTTP</short>
<description>HAProxy load-balancer</description>
<port protocol="tcp" port="80"/>
</service>
```

```bash
restorecon /etc/firewalld/services/haproxy-http.xml
chmod 640 /etc/firewalld/services/haproxy-http.xml
```

Configure the `haproxy` configuration file.

```bash
vi /etc/haproxy/haproxy.cfg
```

```bash
frontend main *:80
    mode http
    default_backend app

backend app
    balance roundrobin
    mode http
    server app1 10.0.0.71:80 check
    server app2 10.0.0.80:80 check
```

```bash
systemctl enable --now haproxy
systemctl restart haproxy
```