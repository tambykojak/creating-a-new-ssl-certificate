# Creating and configuring a new SSL certificate for free.

These instructions, as of August 7th, will get you an **A+** SSL rating with [SSL Labs](https://www.ssllabs.com/ssltest/index.html) for free!

Special thanks to:
- [Let's Encrypt](https://letsencrypt.org/), a free, automated, and open Certificate Authority
- [Certbot](https://certbot.eff.org/) for building the tools to help automate this process.

Notes:
- This walkthrough assumes you are using nginx on an Amazon Linux instance.
- Let's Encrypt does not yet support wildcard domains, but they have [announced](https://letsencrypt.org/2017/07/06/wildcard-certificates-coming-jan-2018.html) that these will be available at the beginning of 2018. 

# Wakthrough

## Step 1: Download certbot-auto
In this step, you will download certbot-auto and make it executable. As an optional step, you may also move it to your bin folder.
```
curl -O https://dl.eff.org/certbot-auto
chmod +x certbot-auto 
sudo mv certbot-auto /usr/local/bin/certbot-auto
```
## Step 2: Generate your certificate.

Run the below command, but make sure to replace the domain piece at the end.
```
certbot-auto certonly --standalone -d subdomain.your-domain-here.com
```
You can repeat this step for as many domains and subdomains as you'd like. Remember, there is no support for a wildcard domain just yet so you'll have to run this multiple times if you are managing a couple different subdomains.

Your newly generated private key can be found at `/etc/letsencrypt/live/subdomain.your-domain-here.com/fullchain.pem`

## Step 3: Setup Diffie Hellman params
Run the below command to generate your Diffie Hellman params.
```
openssl dhparam -dsaparam -out /etc/letsencrypt/live/subdomain.your-domain-here.com/dhparam.pem 4096
```

## Step 4: Configure your nginx virtual server
Add the below to the server block that is listening for the respective domain name.
```
  ssl_certificate "/etc/letsencrypt/live/subdomain.your-domain-here.com/fullchain.pem";
  ssl_certificate_key "/etc/letsencrypt/live/subdomain.your-domain-here.com/privkey.pem";
  ssl_dhparam "/etc/letsencrypt/live/subdomain.your-domain-here.com/dhparams.pem";  
```

## Step 5: Setup forward secrecy
You can **and should** read more about what forward secrecy is [here](https://github.com/ssllabs/research/wiki/Forward-Secrecy), but setting it up is easy. Simply add the following lines to your nginx virtual server block.
```
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_prefer_server_ciphers on;
  ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:ECDHE-RSA-AES128-GCM-SHA256:AES256+EECDH:DHE-RSA-AES128-GCM-SHA256:AES256+EDH:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES256-GCM-SHA384:AES128-GCM-SHA256:AES256-SHA256:AES128-SHA256:AES256-SHA:AES128-SHA:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4";
  ```
  
## Step 6: Opt in to HSTS
HSTS basically means that your server will only allows https traffic. You can read more about it [here](https://github.com/ssllabs/research/wiki/SSL-and-TLS-Deployment-Best-Practices#46-deploy-http-strict-transport-security). To enable it, simply add the below line to your virtual server block.
```
  add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```
  
# Overview

Your server block should look something like below at this point.
```
server {
  listen        *:443 ssl;
  server_name   subdomain.your-domain-here.com;

  ssl_certificate "/etc/letsencrypt/live/subdomain.your-domain-here.com/fullchain.pem";
  ssl_certificate_key "/etc/letsencrypt/live/subdomain.your-domain-here.com/privkey.pem";
  ssl_dhparam "/etc/letsencrypt/live/subdomain.your-domain-here.com/dhparams.pem";
  
  add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
  
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_prefer_server_ciphers on;
  ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:ECDHE-RSA-AES128-GCM-SHA256:AES256+EECDH:DHE-RSA-AES128-GCM-SHA256:AES256+EDH:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES256-GCM-SHA384:AES128-GCM-SHA256:AES256-SHA256:AES128-SHA256:AES256-SHA:AES128-SHA:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4";

  # Other nginx stuff here.
}
```
