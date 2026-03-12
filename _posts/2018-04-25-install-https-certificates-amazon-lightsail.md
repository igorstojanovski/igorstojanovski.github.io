---
layout: post
title: "Install HTTPS certificates in Amazon Lightsail"
excerpt: "The intention of this post is to explain how you can install HTTPS certificates
on your Amazon Lightsail WordPress site.  Amazon Lightsail uses Bitnami
containers."
date: 2018-04-25
tags: [Lightsail, operations]
feature_image: __GHOST_URL__/content/images/wordpress/2018/04/amazon_lightsail_logo-620x400.png
---

The intention of this post is to explain how you can install HTTPS certificates on your Amazon Lightsail WordPress site.  Amazon Lightsail uses Bitnami containers. All the information about how to setup things in Bitnami is the same for Amazon Lightsail as well. I did these steps for my blog that uses a Lightsail WordPress container. It is actually a pre-configured Ubuntu and Apache 2. I used GoDaddy to issue the HTTPS certificates but my guess is that steps won’t differ much if you use a different certificate authority.

[![Amazon Lightsail logo](https://i0.wp.com/igorski.co/wp-content/uploads/2018/04/amazon_lightsail_logo-620x400.png?resize=620%2C400)](https://i0.wp.com/igorski.co/wp-content/uploads/2018/04/amazon_lightsail_logo-620x400.png)

Amazon Lightsail logo

# Generate the HTTPS certificates

When you generate certificates on GoDaddy it asks for a CSR (Certificate Signing Request). To generate this, log into your Lightsail container and go in the `opt/bitnami/apache2/conf` folder. There might already be some dummy keys generated so just in case back them up:

```java
sudo mv /opt/bitnami/apache2/conf/server.crt /opt/bitnami/apache2/conf/server.crt.old
sudo mv /opt/bitnami/apache2/conf/server.key /opt/bitnami/apache2/conf/server.key.old
sudo mv /opt/bitnami/apache2/conf/server.csr /opt/bitnami/apache2/conf/server.csr.old
```

Run the following command:

`openssl req -new -newkey rsa:2048 -nodes -keyout server.key -out server.csr`

You will be asked a bunch of questions. Enter all the required info. Executing the command generates two files: **server.key** and **server.csr**. Copy the content of the **server.csr** to GoDaddy and generate the certificates. GoDaddy needs some time until it verifies everything and emails you when the certificates are ready. You will need to download two files. One has some random hashed string for name f68dc40404848.crt and the other is named gd\_bundle-g2-g1.crt. Copy both to your container to an arbitrary location like `/home/bitnami/keys` .

# Configure Apache to use the certificates

From inside your container run the following commands:

```java
sudo ln -s /home/bitnami/keys/f68dc40404848.crt /opt/bitnami/apache2/conf/server.crt
sudo ln -s /home/bitnami/keys/gd_bundle-g2-g1.crt /opt/bitnami/apache2/conf/server-ca.crt
sudo chown root:root /opt/bitnami/apache2/conf/server*
sudo chmod 600 /opt/bitnami/apache2/conf/server*
```

After this, you will have prepared the three files you need: **server.key**, **server.crt**, and **server-ca.crt**. Don’t miss to set the correct permissions for the files. For some silly reason the first time I did this I skipped those two lines and, of course, nothing worked.

In `/opt/bitnami/apache2/conf/bitnami/bitnami.conf` you should be able to find the following two lines:

```java
SSLCertificateFile "/opt/bitnami/apache2/conf/server.crt"
SSLCertificateKeyFile "/opt/bitnami/apache2/conf/server.key"
```

Add the line for the SSLCACertificateFile as well:

```java
SSLCACertificateFile "/opt/bitnami/apache2/conf/server-ca.crt"
```

## Force HTTPS

After I did all the previous steps the certificates were set up but the site was still using HTTP. In order to force Apache to use HTTPS you will have to do a couple of changes. Open `/opt/bitnami/apps/APPNAME/conf/httpd-prefix.conf` in an editor. Make sure you first substitute APPNAME with the name of the app you are using. In my case, that was ‘wordpress’. Add the following lines at the top of the file:

```java
RewriteEngine On
RewriteCond %{HTTPS} !=on
RewriteRule ^/(.*) https://%{SERVER_NAME}/$1 [R,L]
```

Unless you have some specific Apache configuration, it should be enough to add the following lines in the default Apache virtual host configuration file at `/opt/bitnami/apache2/conf/bitnami/bitnami.conf`, inside the default VirtualHost directive:

```java
<VirtualHost _default_:80>
  DocumentRoot "/opt/bitnami/apache2/htdocs"
  RewriteEngine On
  RewriteCond %{HTTPS} !=on
  RewriteRule ^/(.*) https://%{SERVER_NAME}/$1 [R,L]
  ...
</VirtualHost>
```

# Restart all the services

Before starting everything up, make sure that the 433 port is open on your Lightsail console.

[![Amazon Lightsail Firework](https://i2.wp.com/igorski.co/wp-content/uploads/2018/04/Screenshot_20180425_000739.png?resize=761%2C305)](https://i2.wp.com/igorski.co/wp-content/uploads/2018/04/Screenshot_20180425_000739.png)

You can find the Firewall settings in the Networking tab of your container control panel.

To start all the services again, run:

`sudo /opt/bitnami/ctlscript.sh start`

# Disclaimer

I am not even close to an expert on the subject. I needed to setup HTTPS for my blog and after a day of reading through documentation and forums, I compiled these steps. They worked perfectly fine for me but I give no guarantees they will work for you.

Bitnami is really well documented. I suggest these links for more details on the subject:

* [Generate And Install A Let’s Encrypt SSL Certificate For A Bitnami Application](https://docs.bitnami.com/aws/how-to/generate-install-lets-encrypt-ssl/)
* [How to force HTTPS for all applications with Apache?](https://docs.bitnami.com/aws/components/apache/#how-to-force-https-for-all-applications-with-apache)