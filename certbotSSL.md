# Certbot LetsEncrypt SSL Certificates Installation

**Source:** [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-20-04)

Let’s Encrypt provides an accessible way to obtain TLS/SSL certificates, enabling encrypted HTTPS 
on web servers. 

Certbot ensures the certificate is obtained and installed on Nginx, and also handles automatically the 
certificate renewal process.

Install the Certbot software on your server `sudo apt install certbot python3-certbot-nginx`

Ensure the `server_name` in the server block points to your domain-name. Example:
```
...
server_name kendomain kendoc.com;
...
```
reload nginx `sudo systemctl reload nginx`

Open Https traffic using command: `ufw allow 443`


**Obtaining an SSL certifcate:** This will obtain the certifcate. the `-d` is used to specify the domain names we’d like
the certificate to be valid for. If it's your first time you will be prompted to enter an email address and agree to 
the terms of service. follow the steps to complete the configuration. 
`sudo certbot --nginx -d example.com -d www.example.com`

If that's successful you should see a congratulatory message like this:
**Output:**
```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/example.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/example.com/privkey.pem
   Your cert will expire on 2020-08-18. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

Certbot by default is programmatically set to run twice a day and automatically renew any certificate that’s 
within thirty days of expiration. 

Do a dry-run with `certbot` to simulate the renewal process. `sudo certbot renew --dry-run` 

if you see no errors, then you are all set to go!.

**Output:**
```
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/kendoc.vip.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Cert not due for renewal, but simulating renewal for dry run
Plugins selected: Authenticator nginx, Installer nginx
Renewing an existing certificate
Performing the following challenges:
http-01 challenge for www.kendoc.vip
http-01 challenge for kendoc.vip
Using default addresses 80 and [::]:80 ipv6only=on for authentication.
Waiting for verification...
Cleaning up challenges

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
new certificate deployed with reload of nginx server; fullchain is
/etc/letsencrypt/live/kendoc.vip/fullchain.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates below have not been saved.)

Congratulations, all renewals succeeded. The following certs have been renewed:
/etc/letsencrypt/live/kendoc.vip/fullchain.pem (success)
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates above have not been saved.)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

