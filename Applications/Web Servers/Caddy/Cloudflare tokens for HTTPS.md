---
created: 2024-02-22T14:43
updated: 2024-02-22T14:47
---
https://github.com/caddy-dns/cloudflare
https://caddyserver.com/docs/running

1. Get your Cloudflare api token.
2. 
```shell
sudo systemctl edit caddy
```
3. 
```shell
[Service]
Environment="CF_API_TOKEN=super-secret-cloudflare-tokenvalue"
```
4. 
```shell
sudo systemctl restart caddy
```
5. In the caddyfile:
```bash
tls {
	dns cloudflare {env.CF_API_TOKEN}
}
```

