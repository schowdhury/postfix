# postfix


## Postfix Setup to relay email through AWS SES

I have had to deal with email.  A lot.  We are not talking Gmail or Hotmail, we’re talking about sending out massive amounts of email from your cloud servers.  Unlike managed servers that may have IPs in good standing, cloud IPs are disposable and untrustable.  With cloud servers, all the stars have to align, starting with proper DNS for TXT, PTR, SPF records to start with.  You can sidestep some of this by relaying your email through AWS SES.  Following is a basic postfix configuration to accomplish this.

 

###set system hostname to `my-website-hostname`
> hostname `my-website-hostname`

###create a cert
```
mkdir /etc/postfix/ssl
cd /etc/postfix/ssl
openssl req -new -x509 -nodes -out smtpd.pem -keyout smtpd.pem -days 3650
postconf -e 'smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt'
```

### Add your AWS creds for relaying email:
```
cat /etc/postfix/sasl_passwd

email-smtp.us-east-1.amazonaws.com:25 <creds>
ses-smtp-prod-335357831.us-east-1.elb.amazonaws.com:25 <creds>
```

#### you want to hash the file and get rid of the plain text creds above
postmap hash:/etc/postfix/sasl_passwd

###make sure mydestination doesn’t include `my-website-hostname`
#### append to main.cf

```
mydestination = localhost
mydomain = <my-website-hostname>
myorigin = $mydomain
myhostname = <my-website-hostname>
relayhost = email-smtp.us-east-1.amazonaws.com:25
smtp_sasl_auth_enable = yes
smtp_sasl_security_options = noanonymous
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_use_tls = yes
smtp_tls_security_level = encrypt
smtp_tls_note_starttls_offer = yes
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
smtpd_sasl_local_domain = <my-website-hostname>
```

###---- TEST ----
apt-get install mailutils
```
echo "This is a test" | mail -s "YO" anyuser@gmail.com -aFrom:user@<my-website-hostname>
```
