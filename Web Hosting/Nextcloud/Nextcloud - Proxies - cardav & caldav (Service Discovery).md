---
created: 2024-02-29T14:30
updated: 2024-03-12T15:23
tags:
  - Nextcloud
  - Caddy
  - WebHosting
  - Linux
related link: https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/reverse_proxy_configuration.html#service-discovery
---
> [!NOTE]
> The problem usually come from the proxy in front of the server. 

The redirects for CalDAV or CardDAV does not work if Nextcloud is running behind a reverse proxy. The recommended solution is that your reverse proxy does the redirects.
___
Apache2
```apacheconf
RewriteEngine On
RewriteRule ^/\.well-known/carddav https://%{SERVER_NAME}/remote.php/dav/ [R=301,L]
RewriteRule ^/\.well-known/caldav https://%{SERVER_NAME}/remote.php/dav/ [R=301,L]
```
___
Traefik 1
Using Docker labels:
```yml
traefik.frontend.redirect.permanent: 'true'
traefik.frontend.redirect.regex: 'https://(.*)/.well-known/(?:card|cal)dav'
traefik.frontend.redirect.replacement: 'https://$$1/remote.php/dav'
```

Using traefik.toml:
```toml
[frontends.frontend1.redirect]
  regex = "https://(.*)/.well-known/(?:card|cal)dav"
  replacement = "https://$1/remote.php/dav
  permanent = true
```
___
Traefik 2
Using Docker labels:
```yml
traefik.http.routers.nextcloud.middlewares: 'nextcloud_redirectregex'
traefik.http.middlewares.nextcloud_redirectregex.redirectregex.permanent: true
traefik.http.middlewares.nextcloud_redirectregex.redirectregex.regex: 'https://(.*)/.well-known/(?:card|cal)dav'
traefik.http.middlewares.nextcloud_redirectregex.redirectregex.replacement: 'https://$${1}/remote.php/dav'
```

Using a TOML file:
```toml
[http.middlewares]
  [http.middlewares.nextcloud-redirectregex.redirectRegex]
    permanent = true
    regex = "https://(.*)/.well-known/(?:card|cal)dav"
    replacement = "https://${1}/remote.php/dav"
```
___
HAProxy
```cnf
acl url_discovery path /.well-known/caldav /.well-known/carddav
http-request redirect location /remote.php/dav/ code 301 if url_discovery
```
___
NGINX
```nginx
location /.well-known/carddav {
    return 301 $scheme://$host/remote.php/dav;
}

location /.well-known/caldav {
    return 301 $scheme://$host/remote.php/dav;
}
```

or

```nginx
rewrite ^/\.well-known/carddav https://$server_name/remote.php/dav/ redirect;
rewrite ^/\.well-known/caldav https://$server_name/remote.php/dav/ redirect;
```
___
Caddy
```caddyfile
subdomain.example.com {
    redir /.well-known/carddav /remote.php/dav 301
    redir /.well-known/caldav /remote.php/dav 301

    reverse_proxy {$NEXTCLOUD_HOST:localhost}
}
```
