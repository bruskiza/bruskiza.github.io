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

## Combining certificates and intermediate certificates into one

My Certificate authority makes use of an intermediate certificate which needs to be added together.
The easiest way to do this is to have a single file with the certificate, followed by the intermediate certificate.
As follows:

{% highlight bash %}

cat > common.name.and.intermediate.pem.2016
<PASTE CONTENTS OF CERTIFICATE HERE>

cat >> common.name.and.intermediate.pem.2016
<PASTE CONTENTS OF INTERMEDIATE CERTIFICATE HERE>

{% endhighlight %}

This ensures that the certificate is parsed as valid when the SSL connections are made.

## Testing

Two test tools can be used to verify these in your MTA and POP3 servers:

### Secure SMTP

The service is here: [checktls.com](http://www.checktls.com/perl/TestSender.pl)

Below is an example of a correctly configured TLS SMTP server.
Notice that there are no errors in the `*** Error Note ***` field and it says: `Your message was successfully sent securely using TLS`.

{% highlight bash %}

Below are the details from your CheckTLS TestSender test
from <bruce.mcintyre@gmail.com> via [178.79.132.20]
run on 2016-01-10 12:50:35 EST.
Original email Subject: i8qnihsv3q4mk

Your email was successfully sent securely using TLS.

A transcript of the eMail SMTP session is below:
--> this would be a line from your email system to our test
<-- and this would be a line to your email system from our test

If TLS was negotiated, a line is added:
====tls negotiation successful (cypher: cyphername, client cert: certinfo)

Everything after that line is secure (encrypted), as indicated by:
~~> commands from your system then have wiggly lines
<~~ and responses from our system do too

Any errors that the test noticed are noted in the log by asterisk boxes:
***************************************
*** ********** Error Note ********* ***
***                                 ***
*** The error message would be here ***
***                                 ***
***************************************
***************************************

___TRANSCRIPT BEGINS ON THE NEXT LINE___
<-- 220 ts4.checktls.com CheckTLS TestSender Sun, 10 Jan 2016 12:50:33 -0500
--> EHLO linode.brandinc.co.za
<-- 250-ts4.checktls.com Hello  [178.79.132.20], pleased to meet you
<-- 250-ENHANCEDSTATUSCODES
<-- 250-8BITMIME
<-- 250-STARTTLS
<-- 250 HELP
--> STARTTLS
<-- 220 Ready to start TLS
====tls negotiation successful (cypher: DHE-RSA-AES256-GCM-SHA384, client cert: Subject Name: undefined;Issuer  Name: undefined;)
~~> EHLO linode.brandinc.co.za
<~~ 250-ts4.checktls.com Hello  [178.79.132.20], pleased to meet you
<~~ 250-ENHANCEDSTATUSCODES
<~~ 250-8BITMIME
<~~ 250 HELP
~~> MAIL FROM:<bruce.mcintyre@gmail.com> BODY=7BIT
<~~ 250 Ok - mail from bruce.mcintyre@gmail.com
~~> RCPT TO:<test@TestSender.CheckTLS.com>
<~~ 250 Ok - recipient test@TestSender.CheckTLS.com
~~> DATA
<~~ 354 Send data.  End with CRLF.CRLF
~~> Received: from localhost (linode.brandinc.co.za [127.0.0.1])
~~> 	by linode.brandinc.co.za (Postfix) with ESMTP id 7CCF9E29C
~~> 	for <test@TestSender.CheckTLS.com>; Sun, 10 Jan 2016 19:50:33 +0200 (SAST)
~~> X-Virus-Scanned: Debian amavisd-new at linode.brandinc.co.za
~~> Received: from linode.brandinc.co.za ([127.0.0.1])
~~> 	by localhost (linode.brandinc.co.za [127.0.0.1]) (amavisd-new, port 10024)
~~> 	with ESMTP id 1zZXXXUjsVCq for <test@TestSender.CheckTLS.com>;
~~> 	Sun, 10 Jan 2016 19:50:33 +0200 (SAST)
~~> Received: from [192.168.1.102] (196-210-154-112.dynamic.isadsl.co.za [196.210.154.112])
~~> 	(Authenticated sender: bruce@tradingp.com)
~~> 	by linode.brandinc.co.za (Postfix) with ESMTPSA id E1B90E29B
~~> 	for <test@TestSender.CheckTLS.com>; Sun, 10 Jan 2016 19:50:32 +0200 (SAST)
~~> Subject: i8qnihsv3q4mk
~~> Mime-Version: 1.0 (Mac OS X Mail 9.2 \(3112\))
~~> Content-Type: text/html; charset=us-ascii
~~> X-Apple-Base-Url: x-msg://11/
~~> X-Apple-Mail-Remote-Attachments: YES
~~> From: Bruce McIntyre <bruce.mcintyre@gmail.com>
~~> X-Apple-Auto-Saved: 1
~~> X-Apple-Windows-Friendly: 1
~~> Date: Sun, 10 Jan 2016 19:50:26 +0200
~~> X-Apple-Mail-Signature: SKIP_SIGNATURE
~~> Content-Transfer-Encoding: 7bit
~~> Message-Id: <3B6D87A9-95D9-4165-8E2E-BAF6A5B704C5@gmail.com>
~~> X-Uniform-Type-Identifier: com.apple.mail-draft
~~> To: test@TestSender.CheckTLS.com
~~> X-Mailer: Apple Mail (2.3112)
~~>
~~> <html><head></head><body dir="auto" style="word-wrap: break-word; -webkit-nbsp-mode: space; -webkit-line-break: after-white-space;">This is a test message.<br><br></body></html>
~~> .
<~~ 250 Ok
~~> QUIT
<~~ 221 ts4.checktls.com closing connection
SPF results: code="softfail", local="gmail.com ... _spf.google.com: Sender is not authorized by default to use 'bruce.mcintyre@gmail.com' in 'mfrom' identity, however domain is not currently prepared for false failures (mechanism '~all' matched)"
DKIM verify: "none"

{% endhighlight %}

### Secure POP3

The service is here: [wormly.com](https://www.wormly.com/test_pop3_mail_server)

Below is an example of a successful login using POP3S:

{% highlight bash %}
Resolving hostname...
Connecting...
S:+OK Dovecot (Ubuntu) ready.
C: CAPA
S:+OK
S:CAPA
S:TOP
S:UIDL
S:RESP-CODES
S:PIPELINING
S:AUTH-RESP-CODE
S:USER
S:SASL PLAIN LOGIN
S:.
C: USER bruce@*******
S:+OK
C: PASS ********
S:+OK Logged in.
C: QUIT
S:+OK Logging out.
POP3 test completed successfully.

{% endhighlight %}
