# Postfix setup Guide for GitLab

## Setup main configuration file main.cf

```shell

# backup main.cf
cp main.cf main.cf_$(date +%Y%m%d).bak

# myhostname

MY_HOSTNAME="lifestealer.cnsuning.com"
MY_DOMAIN="cnsuning.com"

sed -i \
-e "s#\(myhostname =\) .*#\1 ${MY_HOSTNAME}#g" \
-e "s#\(mydomain =\) .*#\1 ${MY_DOMAIN}#g" \
-e 's/#myorigin = $mydomain/myorigin = $mydomain/' \
-e 's/#myorigin = $mydomain/myorigin = $mydomain/' \
-e 's/#myorigin = $mydomain/myorigin = $mydomain/' \
main.c
```

## A simple demenstration of the configuration

```
[root@lifestealer postfix]# egrep -v '^#|^$' main.cf
queue_directory = /var/spool/postfix
command_directory = /usr/sbin
daemon_directory = /usr/libexec/postfix
data_directory = /var/lib/postfix
mail_owner = postfix
myhostname = lifestealer.cnsuning.com
mydomain = cnsuning.com
myorigin = $mydomain
inet_interfaces = all
inet_protocols = all
mydestination = $myhostname, localhost.$mydomain, localhost
unknown_local_recipient_reject_code = 550
mynetworks_style = host
mynetworks = 10.19.37.0/24, 192.168.56.0/24
relay_domains = $mydestination
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases


debug_peer_level = 2
debugger_command =
         PATH=/bin:/usr/bin:/usr/local/bin:/usr/X11R6/bin
         ddd $daemon_directory/$process_name $process_id & sleep 5
sendmail_path = /usr/sbin/sendmail.postfix
newaliases_path = /usr/bin/newaliases.postfix
mailq_path = /usr/bin/mailq.postfix
setgid_group = postdrop
html_directory = no
manpage_directory = /usr/share/man
sample_directory = /usr/share/doc/postfix-2.6.6/samples
readme_directory = /usr/share/doc/postfix-2.6.6/README_FILES
```



# Set up Postfix for Reply by email

This document will take you through the steps of setting up a basic Postfix mail server with IMAP authentication on Ubuntu, to be used with Reply by email.

The instructions make the assumption that you will be using the email address incoming@gitlab.example.com, that is, username incoming on host gitlab.example.com. Don't forget to change it to your actual host when executing the example code snippets.

# Configure your server firewall
1. Open up port 25 on your server so that people can send email into the server over SMTP.
2. If the mail server is different from the server running GitLab, open up port 143 on your server so that GitLab can read email from the server over IMAP.


# Install packages
1. Install the postfix package if it is not installed already:
```
sudo apt-get install postfix
```
When asked about the environment, select 'Internet Site'. When asked to confirm the hostname, make sure it matches gitlab.example.com.

2. Install the mailutils package.
```
sudo apt-get install mailutils
```

# Create user

1. Create a user for incoming email.
```
sudo useradd -m -s /bin/bash incoming
```

2. Set a password for this user.
```
sudo passwd incoming
M1t!s-Born
```
>Be sure not to forget this, you'll need it later.
