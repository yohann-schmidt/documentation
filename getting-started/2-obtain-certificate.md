---
currentMenu: getting-started-obtain-certificate
---

# Obtain a certificate

Now that ACME PHP is available on your system ( the `php acmephp.phar --version` command should
display its version), you can use it to request certificates for your domains.

You must complete steps 1 through 4 to issue a certificate the first time. You only need to complete Step 5
to renew a certificate that has already been issued. Step 6 describes an optional command that lists the
status of certificates that are managed by ACME PHP:

1. [Register on the Let's Encrypt/ACME server](#1-register-on-the-lets-encryptacme-server)
2. [Prove you own the domain](#2-prove-you-own-the-domain)
3. [Ask the server to check your proof](#3-ask-the-server-to-check-your-proof)
4. [Request a certificate](#4-request-a-certificate)
5. [Renew a certificate](#5-renew-a-certificate)
6. [List the certificates managed by ACME PHP](#6-list-the-certificates-managed-by-acme-php)

## 1. Register on the Let's Encrypt/ACME server

The first thing you must do is generate a global private key (your "account key"). This key will be
used to prove your identity during future exchanges with Let's Encrypt.

To generate your global private key, run the following command:

``` console
php acmephp.phar register youremail@example.com
```

This command does three main things:
- Creates a default configuration file `~/.acmephp/acmephp.conf` that you may edit for advanced usages of ACME PHP
- Generates a key pair for your account and stores it in `~/.acmephp/master/private/_account`
- Registers your account key in the Let's Encrypt/ACME server, associating your key with your e-mail adress

## 2. Prove you own the domain

You can only request certificates for domains you own. Therefore, you must prove to the Let's Encrypt server that you own
the domain.

The principle is quite simple:
- You ask the server for a token and then you expose the token on a specific URL given by the server
- The server checks that URL and verifies that the token is properly exposed

**Note:** This technique is called HTTP checking as it uses a HTTP request to check your ownership.
You can also use [DNS checking](/documentation/guides/dns-challenge.html) which uses a DNS TXT record to expose the token.
DNS checking may be easier for you depending on your specific configuration.

Using either technique proves that you control the server behind the concerned domain.

**Note:** You only need to prove once that you own a domain (certificates renewals won't require it), as long as
you continue to use the same account key.

The first step is to get the token and expose it. Run the following:

``` console
# Get a token to expose
php acmephp.phar authorize yourdomain.org
```

The command will then display explanations to help you properly expose the token. Usually, a simple text file is
used to expose the token.

## 3. Ask the server to check your proof

After you have taken steps to expose the token (don't hesitate to check in your browser), run the following command
to ask the server to check your proof:

``` console
php acmephp.phar check yourdomain.org
```

The server will check the URL for the token. If the check is successful you will be able to request a certificate
for the domain.

If there is a problem, acmephp.phar usually displays a message describing the problem. If you are stuck, don't
hesitate to ask on [Github issues](https://github.com/acmephp/acmephp/issues).

## 4. Request a certificate

Finally! You have proven that you are the owner of your domain. You can now request and/or renew a certificate
for the domain. To do so, simply run the `request` command:

``` console
php acmephp.phar request yourdomain.org
```

The first time you run this command for a domain it will ask you for certain required information. After you enter
the required information, a request is sent to the server. If the request is successful the certificate files are
placed in the storage directory (`~/.acmephp`).

Six files will be created in the storage directory:
  
- **The full-chain certificate** at `~/.acmephp/master/certs/yourdomain.org/fullchain.pem`.
  This file is the certificate itself. You probably want to use this file in your webserver configuration as it
  includes all the issuers chain for better compatibility with older devices.

- **The certificate private key** at `~/.acmephp/master/private/yourdomain.org/private.pem`.
  This file is usually required by the webserver.
  
- **The chain alone** at `~/.acmephp/master/certs/yourdomain.org/chain.pem`.
  Some webservers require a separate chain and certificate. This is the chain part, if you need it.
  
- **The certificate alone** at `~/.acmephp/master/certs/yourdomain.org/cert.pem`.
  Some webservers require a separate chain and certificate. This is the certificate part, if you need it.
  
- **The combined certificate** at `~/.acmephp/master/certs/yourdomain.org/combined.pem`.
  Some webservers require a single file that combines the full-chain and the certificate private key.
  This is that single file, if you need it.

- **The certificate public key** at `~/.acmephp/master/private/yourdomain.org/public.pem`.
  This file is probably not needed. If it is, this is that file.

Now that you have the certificate you can [configure your webserver](/documentation/getting-started/3-configure-webserver.html)!

## 5. Renew a certificate

You can renew a certificate that will expire soon. To do so, simply run the `request` command:

``` console
php acmephp.phar request yourdomain.org
```

This time, the command will not ask you for any information because all the required information was stored
during the initial request. Instead, the command will request renewal of the certificate. If the renewal is successful
the appropriate files in the storage directory (~/.acmephp) are updated. 

Please note that there may be a limit of renewals per day or week depending on the configuration of the ACME
server. Let's Encrypt has this kind of limitation: renew only when required (for example, one week before
expiration of the certificate).

By default, the `request` command won't renew a certificate until one week before the expiration of the
certificate. You can automate the renewal process by creating a crontab entry that runs the 'renew' command every
day. This technique ensures that certificates are renewed only when required.

## 6. List the certificates managed by ACME PHP

The command `./acmephp.phar status` lets you see what certificates are managed by the ACME PHP client
and when they will expire. It's a useful tool to help avoid expired certificates:

``` console
php acmephp.phar status

+---------------+----------------------------+---------------------+---------------------+----------------+
| Domain        | Issuer                     | Valid from          | Valid to            | Needs renewal? |
+---------------+----------------------------+---------------------+---------------------+----------------+
| acmephp.com   | Let's Encrypt Authority X3 | 2016-06-17 13:08:00 | 2016-09-15 13:08:00 | No             |
+---------------+----------------------------+---------------------+---------------------+----------------+
```

---------------------------------------------------------------------

Next: [Configure your webserver](/documentation/getting-started/3-configure-webserver.html)
