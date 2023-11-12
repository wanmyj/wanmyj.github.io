---
layout: post
data: "2023-11-04"
tags: [config]
categories: [PROXY]
---
Either of the following ways, you have to make change on `/usr/lib/lua/luci/view/netdata/netdata.htm` accordingly.

If you want to use like this `https://192.168.1.1:9880/netstat`, do the following:   
Add `/etc/nginx/conf.d/netdata.conf`
```config
upstream netdata {
    server 127.0.0.1:19999;
    keepalive 64;
}

server {
    listen 9880 ssl;
    # uncomment the line if you want nginx to listen on IPv6 address
    #listen [::]:9880;

    # the virtual host name of this subfolder should be exposed
    #server_name netdata.example.com;

    ssl_certificate /etc/nginx/conf.d/_lan.crt;
    ssl_certificate_key /etc/nginx/conf.d/_lan.key;
    ssl_session_cache shared:SSL:32k;
    ssl_session_timeout 64m;

    location = /netdata {
        return 301 /netdata/;
    }

    location ~ /netdata/(?<ndpath>.*) {
        proxy_redirect off;
        proxy_set_header Host $host;

        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;
        proxy_pass_request_headers on;
        proxy_set_header Connection "keep-alive";
        proxy_store off;
        proxy_pass http://netdata/$ndpath$is_args$args;

        gzip on;
        gzip_proxied any;
        gzip_types *;
    }
}
```
If you want to use like this `https://192.168.1.1/netstat`, do the following:
Add `/etc/nginx/conf.d/netdata.locations`
```config
location = /netdata {
    return 301 /netdata/;
}

location ~ /netdata/(?<ndpath>.*) {
    proxy_redirect off;
    proxy_set_header Host $host;

    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Server $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_http_version 1.1;
    proxy_pass_request_headers on;
    proxy_set_header Connection "keep-alive";
    proxy_store off;
    proxy_pass http://127.0.0.1:19999/$ndpath$is_args$args;

    gzip on;
    gzip_proxied any;
    gzip_types *;
}
```