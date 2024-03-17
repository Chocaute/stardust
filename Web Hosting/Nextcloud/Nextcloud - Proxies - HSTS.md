---
created: 2024-02-29T15:20
updated: 2024-02-29T15:32
tags:
  - Nextcloud
  - Caddy
  - WebHosting
related link: https://docs.nextcloud.com/server/latest/admin_manual/installation/harden_server.html#use-https
---
> While redirecting all traffic to HTTPS is good, it may not completely prevent man-in-the-middle attacks. Thus administrators are encouraged to set the HTTP Strict Transport Security header, which instructs browsers to not allow any connection to the Nextcloud instance using HTTP, and it attempts to prevent site visitors from bypassing invalid certificate warnings.

This can be achieved by setting the following settings within the Apache VirtualHost file:
```apacheconf
<VirtualHost *:443>
  ServerName cloud.nextcloud.com
    <IfModule mod_headers.c>
      Header always set Strict-Transport-Security "max-age=15552000; includeSubDomains"
    </IfModule>
 </VirtualHost>
```

- This example configuration will make all subdomains only accessible via HTTPS. If you have subdomains not accessible via HTTPS, remove `includeSubDomains`.

- This requires the `mod_headers` extension in Apache.

> [!NOTE] WARNING
> We recommend the additional setting `; preload` to be added to that header. Then the domain will be added to a hardcoded list that is shipped with all major browsers and enforce HTTPS upon those domains. See the [HSTS preload website for more information](https://hstspreload.org/). Due to the policy of this list you need to add it to the above example for yourself once you are sure that this is what you want. [Removing the domain from this list](https://hstspreload.org/#removal) could take some months until it reaches all installed browsers.
> 
___
#### Caddyfile:
```ini
cloud.mustang.infra {
        redir /.well-known/carddav /remote.php/dav 301
        redir /.well-known/caldav /remote.php/dav 301
        reverse_proxy 10.10.10.70:80
        tls internal
        header {
        Strict-Transport-Security max-age=15552000;
        }
}
```