# Create Free SSL certificate using Let's Encrypt

Documentation: https://letsencrypt.readthedocs.org/en/latest/

Source on Github: https://github.com/letsencrypt/letsencrypt

Quick start guide: https://community.letsencrypt.org/t/quick-start-guide/1631

### Install Let's Encrypt client

SSH into your production server:

```sh
$ sudo apt-get install git python
```

```sh
$ sudo git clone https://github.com/letsencrypt/letsencrypt /opt/letsencrypt
$ cd /opt/letsencrypt
$ sudo ./letsencrypt-auto --help all
```

### Create certificate

```sh
$ sudo ./letsencrypt-auto --apache --test-cert --email me@helloworld.com --agree-tos -d hi.helloworld.com
```

If you have an existing apache config with wildcard server alias:

```sh
$ sudo ./letsencrypt-auto certonly --email me@helloworld.com --agree-tos -d hi.helloworld.com --server https://acme-v01.api.letsencrypt.org/directory
```

For --server:
- acme-v01.api.letsencrypt.org (Production)
- acme-staging.api.letsencrypt.org (Staging)

Need to use --server https://acme-v01.api.letsencrypt.org/directory.
Not using --server, or using --server https://acme-staging.api.letsencrypt.org/directory, the Certificate Issue will be CN=happy hacker fake CA.

##### Certificates location

```sh
$ sudo ls -al /etc/letsencrypt/live/hi.helloworld.com
```

* cert.pem
* chain.pem
* fullchain.pem
* privkey.pem

## Update apache config

```sh
SSLCertificateFile      /etc/letsencrypt/live/hi.helloworld.com/cert.pem
SSLCertificateKeyFile   /etc/letsencrypt/live/hi.helloworld.com/privkey.pem
SSLCertificateChainFile /etc/letsencrypt/live/hi.helloworld.com/fullchain.pem
```

If you have an apache server running, reload the config.

```sh
$ sudo service apache2 reload
```

## CA bundle

Find **DST Root CA X3** from Internet


## Notes (2016-03-01)

Ref: https://community.letsencrypt.org/t/frequently-asked-questions-faq/26#topic-title

The Let's Encrypt Client BETA SOFTWARE. 
It contains plenty of bugs and rough edges, and should be tested thoroughly in staging environments before use on production systems.

**Service Status (letsencrypt.status.io)**
https://letsencrypt.status.io/

**Are certificates from Let’s Encrypt trusted by my browser?**
The short answer is “yes”.
The long answer is that our issuing intermediates are cross-signed by a widely trusted IdenTrust root918. This allows our certificates to be trusted while we work on propagating our own root. Most platforms that trust that root should trust Let's Encrypt certs. One notable exception is Windows XP, which currently doesn't accept our intermediate

**Can I use certificates from Let’s Encrypt for code signing or email encryption?**
No. Email encryption and code signing require a different type of certificate than Let’s Encrypt will be issuing.

**Will Let’s Encrypt issue wildcard certificates?**
We currently have no plans to do so, but it is a possibility in the future. Hopefully wildcards aren’t necessary for the vast majority of our potential subscribers because it should be easy to get and manage certificates for all subdomains.

**Will the Let’s Encrypt client support my operating system?**
We intend to launch with Let’s Encrypt client support for major Linux and BSD variant operating systems. We are hoping to have a Windows port (PowerShell) ready at launch as well.

**Renewals and Lifetimes**
Certificates from Let's Encrypt are valid for 90 days. We recommend renewing them every 60 days to provide a nice margin of error. As a beta participant, you should be prepared to manually renew your certificates at that time. As we get closer to General Availability, we hope to have automatic renewal tested and working on more platforms, but for now, please play it safe and keep track.

Example of renewal script: https://letsencrypt.org/howitworks/

**Rate Limiting**
During the beta phase, Let’s Encrypt enforces strict rate limits on the number of certificates issued for one domain. It is recommended to initially use the test server via --test-cert until you get the desired certificates. ([REF|https://letsencrypt.readthedocs.org/en/latest/using.html#installation])

During this beta test we have very tight rate-limiting in place. We plan to loosen these limits as the beta proceeds.
There are two rate limits in play: Registrations/IP address, and Certificates/Domain.

Registrations/IP address limits the number of registrations you can make in a given time period; currently 10 per 3 hours. This means you should avoid deleting the /etc/letsencrypt/accounts folder, or you may not be able to re-register.

Certificates/Domain you could run into through repeated re-issuance. This limit measures certificates issued for a given combination of Top Level Domain + Domain (a "registered domain"). This means if you issue certificates for the following domains, at the end you would have what we consider 4 certificates for the domain example.com.

www.example.com
example.com www.example.com
webmail.example.com ldap.example.com
example.com www.example.com

The limit on Certificates/Domain is 5 certificates for a registered domain in a sliding window of 7 days. 

Ref: https://community.letsencrypt.org/t/beta-program-announcements/1631
