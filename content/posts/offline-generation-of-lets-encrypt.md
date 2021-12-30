---
title: 'Offline generation of Let’s Encrypt certificates '
date: 2021-04-12T17:23:00.005-04:00
draft: false
#url: /2021/04/offline-generation-of-lets-encrypt.html
tags: 
- ssl
- letsencrypt
- certbot
- tls
---

Sometimes we need to get a Let's Encrypt SSL certificate for a system that might not be connected in the internet or where the certbot client is not able to be installed. There is an easy way to generate a SSL chain that we can use in our internal applications.

# Install certbot

On a linux system (even a temporary one) install certbot. The example below is performed on a Ubuntu 18.04 box.

```bash
$ sudo apt install certbot  
$ certbot --version  
certbot 0.27.0
```

We are going to generate a certificate for our host with a host name _hostname.domain.com_. Let's encrypt will allow an offline update through a DNS challenge so that means that during the certificate generation you should have an open screen of you DNS registrars/manager.


Initiate the request by the following command  

```bash
$ sudo certbot certonly --manual --preferred-challenges dns -d hostname.domain.com  
Saving debug log to /var/log/letsencrypt/letsencrypt.log  
Plugins selected: Authenticator manual, Installer None  
Obtaining a new certificate  
Performing the following challenges:  
dns-01 challenge for hostname.domain.com  
  
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  
NOTE: The IP of this machine will be publicly logged as having requested this  
certificate. If you're running certbot in manual mode on a machine that is not  
your server, please ensure you're okay with that.  
  
Are you OK with your IP being logged?  
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  
(Y)es/(N)o: Y  
  
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  
Please deploy a DNS TXT record under the name  
_acme-challenge.hostname.domain.com with the following value:  
  
VUiRL_FOsDDlOFGYVhZCsIHVtfJ03usFLxkPfVvmOos  
```

Before continuing, verify the record is deployed

```bash  
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  
  
Press Enter to Continue  
  
Waiting for verification...  
```

Now it is time to add the TXT record on your DNS server. As soon the record is there and you click enter the following will continue on your terminal  
  
```bash
Cleaning up challenges  
  
IMPORTANT NOTES:
- Congratulations! Your certificate and chain have been saved at:  
/etc/letsencrypt/live/hostname.domain.com/fullchain.pem  
Your key file has been saved at:  
/etc/letsencrypt/live/hostname.domain.com/privkey.pem  
Your cert will expire on 2021-07-11. To obtain a new or tweaked  
version of this certificate in the future, simply run certbot  
again. To non-interactively renew \*all\* of your certificates, run  
"certbot renew"  
- If you like Certbot, please consider supporting our work by:  
  
Donating to ISRG / Let's Encrypt: https://letsencrypt.org/donate  
Donating to EFF: https://eff.org/donate-le
```
  
And there you go! The new private key and the certificate chain are in the default letsencrypt location at:

```
/etc/letsencrypt/live/hostname.domain.com/fullchain.pem
/etc/letsencrypt/live/hostname.domain.com/privkey.pem
```

In case you already have a CSR file from a device or server then just add the --csr to the above command with the csr file as argument:
  
```bash
$ sudo certbot certonly --manual --preferred-challenges dns -d hostname.domain.com --csr <csr\_file.csr>
```