---
layout: post
title: "Installing SSL on Dovecot and Postfix"
description: ""
category:
tags: ['ssl', 'dovecot', 'postfix']
---

SSL-enabling your IMAP and POP3 servers adds more security when receiving mail.

Doing on your Mail Transport Agent (MTA) allows for secure sending of mail through your server anywhere on the Internet.

The following post shows how to enable this on Dovecot and Postfix.

## Dovecot

The SSL-specific bits in `/etc/dovecot/dovecot.conf` are:

{% highlight bash %}
ssl = yes
ssl_cert = </etc/ssl/certs/ssl-certificate.pem
ssl_key = </etc/ssl/private/ssl.key.nopass

{% endhighlight %}

## Postfix
The SSL-specific bits in `/etc/postfix/main.cf` are:

{% highlight bash %}
smtpd_tls_cert_file = /etc/postfix/smtpd.cert
smtpd_tls_key_file = /etc/postfix/smtpd.key
smtpd_use_tls = yes
smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache
smtpd_tls_security_level = may
smtp_tls_security_level = may
smtpd_tls_mandatory_protocols = !SSLv2, !SSLv3
smtpd_tls_protocols = !SSLv2,!SSLv3
smtp_tls_protocols = !SSLv2,!SSLv3

{% endhighlight %}
