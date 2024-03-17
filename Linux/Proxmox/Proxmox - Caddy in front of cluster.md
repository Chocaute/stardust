---
created: 2023-12-21T17:36
updated: 2024-02-28T17:10
tags:
  - Proxmox
  - Caddy
  - Linux
  - ReverseProxy
---
https://forum.proxmox.com/threads/proxmox-ui-caddy-and-reverse-proxy.83198/
```yml
proxmox.domain.com {
    reverse_proxy * {
        to server01:8006
        to server02:8006
        to server03:8006

        lb_policy ip_hash     # Makes backend sticky based on client ip
        lb_try_duration 1s
        lb_try_interval 250ms

        health_uri /          # Backend health check path
        # health_port 80      # Default same as backend port
        health_interval 10s
        health_timeout 2s
        health_status 200

        transport http {
            tls_trusted_ca_certs /etc/pve/pve-root-ca.pem # Path to PVE root cert
        }
    }
}
```

```
        transport http {
            tls_insecure_skip_verify
        }
```