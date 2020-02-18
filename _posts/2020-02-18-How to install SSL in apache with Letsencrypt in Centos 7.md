---
layout: post
title: "How to install SSL in apache with Letsencrypt in Centos 7"
date: 2020-02-18 16:35:10 -0000
categories: Centos
---

## Prepare enviroment
### Install Apache:
#### Prepare Apache
* Update repo:

    `sudo yum update`
* Install Apache:

    `sudo yum install httpd openssl`
* Active Apache:

    `sudo systemctl start httpd`
* Set Apache as a service in Centos:

    `sudo systemctl enable httpd`
* Verify Apache service:

    `sudo systemctl status httpd`
* Configure firewalld to Allow Apache Traffic
    * Modify firewall to allow connections on these ports using the following commands:

        `sudo firewall-cmd --permanent --add-port=80/tcp`

        `sudo firewall-cmd --permanent --add-port=443/tcp`
    * Reload firewall to enable config:

        `sudo firewall-cmd --reload`
* Install `mod_ssl` module:

    `sudo yum install mod_ssl`
#### Create Apache's virtual host:
* We need to add to `/etc/httpd/conf/httpd.conf`, this line:

    `IncludeOptional sites-enabled/*.conf`
* If you want to change default root document, you can change this line:

    `DocumentRoot "/var/www/html"`
    
    to

    `DocumentRoot "/var/www/yourdomain.com/html"`
* Create `sites-available` folder if not exist:

    `sudo mkdir -p /etc/httpd/sites-available`
* Create `sites-enable` folder if not exist:

    `sudo mkdir -p /etc/httpd/sites-enable`
* Create virtual host for `http`, note: you must enable HTTP protocol in port 80 to help acme.sh works.
    * In `sites-availabel`, create a file with name `yourdomain.com.conf`, with this content:
    ~~~
        <VirtualHost *:80>
            ServerName yourdomain1.com
            ServerAlias www.yourdomain1.com
            ServerAlias yourdomain2.com
            ServerAlias www.yourdomain2.vn
            DocumentRoot /var/www/yourdomain.com/html
            ErrorLog /var/www/yourdomain.com/log/error.log
            CustomLog /var/www/yourdomain.com/log/requests.log combined
            #<Directory />
            #Options FollowSymLinks
            #AllowOverride All
            #</Directory>
        </VirtualHost>
    ~~~
    * Link `yourdomain.com.conf` to `sites-enable` folder:

        `sudo ln -s yourdomain.com.conf /etc/httpd/sites-enable/yourdomain.com.conf`
    * Restart `httpd`:

        `sudo systemctl restart httpd`
### Install Acme:
* Install git:

    `sudo yum install git`
* Get acme.sh software:

    `git clone https://github.com/Neilpang/acme.sh.git`
* Install acme.sh:

    `cd ./acme.sh && ./acme.sh --install`
* Test acme.sh install:

    `acme.sh -h`
* Create a new `/.well-known/acme-challenge` directory in `yourdomain.com/html` directory

    `sudo mkdir -p /var/www/yourdomain.com/html/.well-known/acme-challenge/`

    `sudo chown apache:apache -R /var/www/yourdomain.com/html/.well-known/acme-challenge`

    `sudo chmod -R 0555 /var/www/yourdomain.com/html/.well-known/acme-challenge`
## Install SSL
* Generate `dhparams`:

    `sudo openssl dhparam -out /etc/httpd/ssl/yourdomain/dhparams.pem -dsaparam 4096`
* Obtain an SSL certificate for your domain:

    `acme.sh --issue -w /var/www/yourdomain.com/html -d yourdomain1.com -k 2048`

    `acme.sh --issue -w /var/www/yourdomain.com/html -d yourdomain2.com -k 2048`
* Install certificates:

    `acme.sh --installcert -d yourdomain1.com --keypath /etc/httpd/ssl/yourdomain/yourdomain1.key --fullchainpath /etc/httpd/ssl/yourdomain/yourdomain1.cer --reload "systemctl reload httpd"`

    `acme.sh --installcert -d yourdomain2.com --keypath /etc/httpd/ssl/yourdomain/yourdomain2.key --fullchainpath /etc/httpd/ssl/yourdomain/yourdomain2.cer --reload "systemctl reload httpd"`
* Config Apache to use SSL/TLS:
    * Access `sites-available` folder.
    * Create `yourdomain.com.ssl.conf` file with this content:

    ~~~
    #
    # When we also provide SSL we have to listen to the 
    # standard HTTPS port in addition.
    #
    Listen 443 https

    ##
    ##  SSL Global Context
    ##
    ##  All SSL configuration in this context applies both to
    ##  the main server and all SSL-enabled virtual hosts.
    ##

    #   Pass Phrase Dialog:
    #   Configure the pass phrase gathering process.
    #   The filtering dialog program (`builtin' is a internal
    #   terminal dialog) has to provide the pass phrase on stdout.
    SSLPassPhraseDialog exec:/usr/libexec/httpd-ssl-pass-dialog

    #   Inter-Process Session Cache:
    #   Configure the SSL Session Cache: First the mechanism 
    #   to use and second the expiring timeout (in seconds).
    SSLSessionCache         shmcb:/run/httpd/sslcache(512000)
    SSLSessionCacheTimeout  300

    #
    # Use "SSLCryptoDevice" to enable any supported hardware
    # accelerators. Use "openssl engine -v" to list supported
    # engine names.  NOTE: If you enable an accelerator and the
    # server does not start, consult the error logs and ensure
    # your accelerator is functioning properly. 
    #
    SSLCryptoDevice builtin
    #SSLCryptoDevice ubsec
    ## Turn on HTTP2 support
    Protocols h2 http/1.1
    #RewriteEngine On
    #RewriteCond %{HTTPS} off
    #RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI} [R=302,L,QSA]
    ##
    ## SSL Virtual Host Context
    ##

    <VirtualHost *:443>

    ServerName yourdomain1.com

    ErrorLog logs/ssl_error_log
    TransferLog logs/ssl_access_log
    LogLevel debug

    SSLEngine on

    SSLHonorCipherOrder on

    SSLCipherSuite PROFILE=SYSTEM
    SSLProxyCipherSuite PROFILE=SYSTEM

    SSLCertificateFile /etc/httpd/ssl/yourdomain/cert.cer

    SSLCertificateKeyFile /etc/httpd/ssl/yourdomain/key.key
    SSLOpenSSLConfCmd DHParameters "/etc/httpd/ssl/yourdomain/dhparams.pem"

    <FilesMatch "\.(cgi|shtml|phtml|php)$">
        SSLOptions +StdEnvVars
    </FilesMatch>
    <Directory "/var/www/cgi-bin">
        SSLOptions +StdEnvVars
    </Directory>

    BrowserMatch "MSIE [2-5]" \
            nokeepalive ssl-unclean-shutdown \
            downgrade-1.0 force-response-1.0

    CustomLog logs/ssl_request_log \
            "%t %h %{SSL_PROTOCOL}x %{SSL_CIPHER}x \"%r\" %b"


    </VirtualHost>
    <VirtualHost *:443>

    ServerName yourdomain2.com
    ErrorLog logs/ssl_error_log
    TransferLog logs/ssl_access_log
    LogLevel warn

    SSLEngine on

    SSLHonorCipherOrder on

    SSLCipherSuite PROFILE=SYSTEM
    SSLProxyCipherSuite PROFILE=SYSTEM

    SSLCertificateFile /etc/httpd/ssl/yourdomain/yourdomain2.cer

    SSLCertificateKeyFile /etc/httpd/ssl/yourdomain/yourdomain2.key
    SSLOpenSSLConfCmd DHParameters "/etc/httpd/ssl/yourdomain/yourdomain2.pem"

    <FilesMatch "\.(cgi|shtml|phtml|php)$">
        SSLOptions +StdEnvVars
    </FilesMatch>
    <Directory "/var/www/cgi-bin">
        SSLOptions +StdEnvVars
    </Directory>

    BrowserMatch "MSIE [2-5]" \
            nokeepalive ssl-unclean-shutdown \
            downgrade-1.0 force-response-1.0

    CustomLog logs/ssl_request_log \
            "%t %h %{SSL_PROTOCOL}x %{SSL_CIPHER}x \"%r\" %b"


    </VirtualHost>

    SSLUseStapling          on
    SSLStaplingResponderTimeout 5
    SSLStaplingReturnResponderErrors off
    SSLStaplingCache        shmcb:/var/run/ocsp(128000)
    ~~~
## Reference link:
[How To Set Up Multiple SSL Certificates On a CentOS VPS With Apache Using One IP Address](https://www.rosehosting.com/blog/how-to-set-up-multiple-ssl-certificates-on-a-centos-vps-with-apache-using-one-ip-address/)
[Apache with Let’s Encrypt Certificates on CentOS 8](https://www.cyberciti.biz/faq/apache-with-lets-encrypt-certificates-on-centos-8/)


