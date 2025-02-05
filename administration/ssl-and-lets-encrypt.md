---
title: SSL & Let's Encrypt
description: Securing your AzuraCast installation with SSL / HTTPS
published: true
date: 2021-04-08T10:05:40.029Z
tags: administration, docker
editor: markdown
dateCreated: 2021-02-05T19:28:14.682Z
---

# Enabling HTTPS with LetsEncrypt

> Automatic LetsEncrypt setup and renewal are only available in the Docker installation. For other installation types, you can directly use [Certbot](https://certbot.eff.org/).
{.is-info}

LetsEncrypt is a free and simple way to allow safe and secure connections to your AzuraCast installation. With a valid SSL certificate, you can:

- Secure your connection to AzuraCast when administering your stations,

- Enforce security for all AzuraCast administrators via HTTP Strict Transport Security (HSTS), and

- Provide a secure listening endpoint to listeners, avoiding "Mixed Content" warnings when your radio signal is played from a secure web page.

# Important Considerations

Before setting up LetsEncrypt, you should make sure the following conditions are met:

- **AzuraCast must be on its own domain or subdomain.** You can't set up LetsEncrypt using only an IP address; you must have a domain (i.e. mysite.com) or a subdomain (radio.mysite.com) set up to point to your AzuraCast installation.

- **AzuraCast's web server must be served on the default ports, 80 for HTTP and 443 for HTTPS.** By default, AzuraCast is already set up this way, but if you've modified the ports to serve the site on a secondary port, you must switch the ports back to the defaults when setting up LetsEncrypt and when performing renewals.

# Enabling LetsEncrypt

Connect to your host server via a terminal (SSH) connection and execute the following commands:

```
cd /var/azuracast
./docker.sh update-self
./docker.sh letsencrypt-create
```

Answer the prompts as shown to complete the setup process.

<br>

## Renewing a Let's Encrypt Certificate

The web service will automatically renew your LetsEncrypt certificates. If you provide an e-mail in the initial setup process, that e-mail will be used to send you reminders of upcoming expiration in the event that automatic renewal fails.

# What to do when Let's Encrypt is not working

The first thing that you should do when you have set up Let's Encrypt as described above and you still see AzuraCast serving a self-signed certificate is to restart AzuraCast via the following commands:

```
docker-compose down
docker-compose up -d
```

After starting AzuraCast wait a few minutes just to be sure that everything has started up correctly then check if AzuraCast is still serving the self-signed certificate.

If it is still serving the self-signed certificate take a look into the logs of the `nginx_proxy_letsencrypt` container to see if there are any errors related to retrieving or renewing the certificate:

```
docker-compose logs nginx_proxy_letsencrypt
```

If you have been tinkering with the Let's Encrypt integration of AzuraCast for a bit you might be running into the rate limiting of the Let's Encrypt API which would look like this in the logs:

```
nginx_proxy_letsencrypt_1 | "detail": "Error creating new order :: too many certificates already issued for exact set of domains: example.com: see https://letsencrypt.org/docs/rate-limits/",
```

More about the rate limits of the Let's Encrypt API can be found here: https://letsencrypt.org/docs/rate-limits/

When you reach out to us about issues with Let's Encrypt in AzuraCast make sure to always check the logs of the `nginx_proxy_letsencrypt` first and include them in your message to us. Without these logs we can't help you and the first thing we will do is ask for those logs.

# Using a Custom Certificate

If you have a custom SSL certificate on your host, you should create a `docker-compose.override.yml` file in your `/var/azuracast` directory on the host server with the contents below, modified to reflect your domain name and the path to your SSL certificate and key:

```
version: '2.2'

services:
  nginx_proxy:
    volumes:
      - /path/on/host/to/ssl.crt:/etc/nginx/certs/yourdomain.com.crt:ro
      - /path/on/host/to/ssl.key:/etc/nginx/certs/yourdomain.com.key:ro

  stations:
    volumes:
      - /path/on/host/to/ssl.crt:/etc/nginx/certs/yourdomain.com.crt:ro
      - /path/on/host/to/ssl.key:/etc/nginx/certs/yourdomain.com.key:ro
```
      
The naming convention for the mapping (the second part of each of the `volumes` section above) is the following:

- **Domain name:** `example.azuracast.com`
- **Certificate path:** `/etc/nginx/certs/example.azuracast.com.crt`
- **Key path:** `/etc/nginx/certs/example.azuracast.com.key`