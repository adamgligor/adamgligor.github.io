---
layout: post
title: TLS with self signed certificates
description: 
date: '2021-11-01'
tags: programming
---


Setting up TSL for self hosted hobby websites.


## Scenario 

I have a photo gallery web app that is for my personal use only. It doesn't need to run 24/7 and I'd like to be able to access it from the internet ocasionally. I'm fine with hosting it on my pc provided that it's a bit isolated from my os and my pc is behind a router.


This is what I came up with: the gallery app will run in a docker container with nginx as a reverse proxy in front of it; nginx and also handles the https encryption. A dynamic dns service will assign a permanent domain name to the public ip address that is likely dynamic.


And it works as follows: the request comes in from the internet, hits the router, router redirects it to my pc where it's handled by docker. There it goes through nginx first which forwards it to the photo gallery app where it's finally handled.

## The app 

Let's assume that the app is already setup to run in a docker container.

## The certificates

There are two distinct paths when it comes to setting up TLS depending whether the domain is public or private.

For public domains one popular method is to use `letsencrypt` and `certbot`; for private domains it's self signed certificates.

To generate a self signed certificate i use `openssl` (ubuntu)

```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ~/nginx-selfsigned.key -out ~/nginx-selfsigned.crt
```

## Nginx

The next piece is to set up nginx. Here's the Dockerfile for it.

```
FROM nginx:1.20-alpine
COPY nginx.conf /etc/nginx/nginx.conf
COPY nginx-selfsigned.crt /etc/ssl/certs/nginx-selfsigned.crt
COPY nginx-selfsigned.key /etc/ssl/private/nginx-selfsigned.key
EXPOSE 8011
```

The previously generated keys and the nginx config are copied to their designated places, the rest is stock.


Next, the nginx config which looks like this:


```
worker_processes  1;

events {
    worker_connections  1024; 
}
    
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

    server {
        listen 80 ssl;
        listen [::]:80 ssl;
        ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
        ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;

        server_name  myphotogalery.local;

        location / {            
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;

            proxy_pass http://myphotogalery:2342/;

            proxy_buffering off;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        } # end location
    } # end server
} # end http
```

Nginx is set up to run on port 80 with ssl, then the keys are referenced, then the forwarding to the gallery app at `http://myphotogalery:2342/`. Note that the `server_name` can be set to localhost, otherwise if it's a different value it has to be present in the hosts config.


## Docker compose

The docker-compose to run the two (nginx and the gallery app) looks lke this:

```
version: '3.5'
services:

  myphotogalery:
  image: dockerrepo/myphotogalery:latest
  container_name: myphotogalery
  ports:
    - 2342:2342 # [local port]:[container port]

  nginx:
    build: nginx
    links:
      - myphotogalery
    extra_hosts:
      - "myphotogalery.local:127.0.0.1"
    ports:
      - "8011:80"
```

The `myphotogalery` is set to run on port 2342, nginx on port 80 but exposed as 8001 on the host os; additionally docker-compose will add `myphotogalery.local` to the hosts config in the nginx container.

## Post forwarding and dynamic dns

My pc is behind a router so a port forwarding is required to instruct the router to redirect any https calls on port 8011 from the internet to my computer in the local network. With this the web app is already accessible from the outside but only by ip address. Since that is likely to change a final improvement is assigning a permanent domain, a service called called dynamic dns service can be used for that like [dyndns](`http://dyndns.org/`). Some newer routers allow setting this up straight in the router configuration.


## Conclusion

All that's left is to `docker-compose up` to start it up then open `https://my-dyndns-domain.com:8011` in the browser.


## Links

- nginx reverse proxy [here](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)
- nginx with certbot [here](https://www.nginx.com/blog/using-free-ssltls-certificates-from-lets-encrypt-with-nginx/)
- nginx https setup [here](https://nginx.org/en/docs/http/configuring_https_servers.html)
- letsencrypt [here](https://letsencrypt.org/)
- certbot [here](https://certbot.eff.org/)
