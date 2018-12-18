---
Title: sending mail from ubuntu (trying to use gmail's server)
Date: 2015-10-24 13:32
Author: slikwhit
Slug: sending-mail-from-ubuntu-trying-to-use-gmails-server
Status: published
draft: true
---

Ultimately I am trying to figure out why my Ubuntu server routinely craps out and is unable to reach the outside world.  I find that if I restart the system, networking comes back up ok...at least for a while.  So I am in the process of writing a script that will reboot it if unable to connect, but I want to know when this happens so I can attempt to troubleshoot.  So I want this system to email me some information when this scenario occurs, right before rebooting.  I don't want to go overboard with configuring anything and I've found some pretty simple configuration recommendations using msmtp on the interwebs (see references below) and I wanted to capture what I've done here.

**Install msmtp and mutt**

    sudo apt-get update
    sudo apt-get -y install msmtp mutt

*(the msmtp installation is smart enough to try to configure some stuff for you but since I will be trying to follow other instructions I opted for "No configuration")*

This section did not work but I might revisit it so I will leave it for reference:  
~~I want to use Gmail so Google's smtp services and I want to attempt to use TLS encryption. So I need to find information about gmail's issuer:~~

    msmtp --serverinfo --host=smtp.gmail.com --tls=on --tls-certcheck=off | grep -A2 Issuer

~~As it turns out, it is "Google Internet Authority G2." A quick internet search reveals that there is more information at <https://pki.google.com/> That page also indicates that the Issuing CA certificate is available at <https://pki.google.com/GIAG2.crt> so let's get that cert and manipulate it so that msmtp can use it:~~

    wget -P /tmp https://pki.google.com/GIAG2.crt
    openssl x509 -inform DER -in /tmp/GIAG2.crt -outform PEM -out /tmp/gmail-smtp.crt
    msmtp --serverinfo --host=smtp.gmail.com --tls=on --tls-trust-file=/tmp/gmail-smtp.crt

This didn't work...I got an error about `msmtp: TLS certificate verification failed: the certificate hasn't got a known issuer` and instead of troubleshooting, I am going to attempt to use the certificate file as specified in reference \#2.  This works but I'm not really sure of its validity:

    msmtp --serverinfo --host=smtp.gmail.com --tls=on --tls-trust-file=/etc/ssl/certs/ca-certificates.crt

So now I just need to configure the msmtp with the information for the gmail account I'll be using:

    echo -e "defaults
    tls on
    tls_starttls on
    tls_trust_file /etc/ssl/certs/ca-certificates.crt
     
    account default
    host smtp.gmail.com
    port 587
    auth on
    user email@gmail.com
    password mypass
    from email@gmail.com
    logfile /var/log/msmtp.log" | tee /tmp/msmtprc
    sudo mv -v /tmp/msmtprc /etc/
    sudo chmod -v 0600 /etc/msmtprc

...this isn't working.  Google is smarter than me and is not allowing me to log in and use the mail server this way.  To be continued...

Side note:  this is my first attempt at a technical article on wordpress...let's see how it goes...

REFERENCES:

1.  http://devblog.virtage.com/2013/05/email-sending-from-ubuntu-server-via-google-apps-smtp-with-msmtp/
2.  http://www.absolutelytech.com/2010/07/17/howto-configure-msmtp-to-work-with-gmail-on-linux/
