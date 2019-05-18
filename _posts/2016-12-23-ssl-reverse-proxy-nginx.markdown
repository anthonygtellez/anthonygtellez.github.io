---
title:  "SSL Proxy: Splunk & NGINX"
tags: Blogs Splunk NGINX Proxy SSL Security Cryptography LetsEncrypt
---

### Who is this guide for?
It is a best practice to install Splunk as a non-root user or service account as part of a defense in depth strategy. This installation choice comes with the consequences of preventing the Splunk user from using privileged ports (Anything below 1024). Some of the solutions to this problem, found on Splunk Answers require iptables rules or other methods. In my experience, the iptables method is not that reliable, and many newer distributions of Linux are abandoning iptables in favor of firewalld as the default host firewall. In this guide, I will show you how to use Nginx, and Let’s Encrypt to secure your Splunk Search Head, while allowing ssl traffic on port 443.
![Hero](http://tellez.sfo2.digitaloceanspaces.com/hero.png)

### Prerequisites
* OS which supports the latest version of Nginx 
* Linux OS which supports Let's Encrypt (If you choose to use that as you CA) 
* Root access to the search head

### Configuration
The easiest way to get both products installed is to use yum or apt depending on your flavor of Linux.

#### Install Let's Encrypt, Configure Splunk Web SSL
In a previous blog post, I provided a guide to generate SSL certs and configure Splunkweb to make use of them. You should follow that guide to generate your certs or your own organizational process for generating certificates before proceeding with the next steps.

#### Install Nginx
`sudo apt install nginx`

#### Configure Nginx to use SSL
Create a configuration for your site, it is best to use the hostname/domainname of the Splunk server. This file should be created in `/etc/nginx/sites-enabled`

`touch /etc/nginx/sites-enabled/splunk-es.anthonytellez.com`

To configure Nginx, you only need three pieces of information: 
* location of the certificate 
* location of the private key used for the certificate 
* ssl port to redirect

Example configuration of splunk-es.anthonytellez.com:

```
server {
    listen 443 ssl;
    ssl on;
    ssl_certificate /opt/splunk/etc/auth/anthonytellez/fullchain.pem;
    ssl_certificate_key /opt/splunk/etc/auth/anthonytellez/privkey.pem;
    location / {
        proxy_pass https://127.0.0.1:8000;
    }
}
```

Reload Nginx:
`$ nginx -s reload`

#### Optional: Redirect all http requests
To prevent users from seeing the default webpage served by Nginx, you should also redirect traffic over port 80 to port 443 to prevent leaking information about the version of Nginx running on your server.

```
server {
    listen 80;
    server_name splunk-es.anthonytellez.com;
    return 301 https://$host$request_uri;
}
```

#### Optional: Enable HSTS
HSTS is web security policy mechanism which helps to protect websites against [protocol downgrade attacks and cookie hijacking](https://tools.ietf.org/html/rfc6797). Enabling this in Nginx can help to protect you if you are ever accessing your Splunk instance from an unprotected network. The included example is set with a max-age of 300 seconds, you can increase this to a larger time once you have validated the configuration is working.

```
server {
    listen 443 ssl;
    add_header Strict-Transport-Security "max-age=300; includeSubDomains" always;
    ssl on;
    ssl_certificate /opt/splunk/etc/auth/anthonytellez/fullchain.pem;
    ssl_certificate_key /opt/splunk/etc/auth/anthonytellez/privkey.pem;
    location / {
        proxy_pass https://127.0.0.1:8000;
    }
}
```

HSTS will force all browsers to query the https version of the site once they have processed this header. If you have issues validating if HSTS is working in your browser of choice, check out this resource on stack exchange: [How can I see which sites have set the HSTS flag in my browser?](http://security.stackexchange.com/questions/92954/how-can-i-see-which-sites-have-set-the-hsts-flag-in-my-browser)