# Secure CentOS box

## SELinux

1. Set `SELINUX=enforcing` in `/etc/sysconfig/selinux`
1. Reboot

## Firewall

1. `systemctl start firewalld.service`
1. `systemctl enable firewalld.service`
1. `sudo firewall-cmd --permanent --zone=public --add-service=http`
1. `sudo firewall-cmd --permanent --zone=public --add-service=https`
1. `sudo firewall-cmd --permanent --zone=public --add-service=ssh`
1. `sudo firewall-cmd --reload`

## Secure ssh

1. Upload your public key to the server if it isn't present - [Specific instructions](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server)
1. Edit `/etc/ssh/sshd_config` and make/verify the following:
```
AuthorizedKeysFile      .ssh/authorized_keys
PasswordAuthentication	no
```
1. `systemclt restart sshd`

# Web Sites (including SSL)

## Install nginx, config nginx

Make sure your HTML files are here: /usr/share/nginx/html/new_domain_name

1. `yum install -y nginx`
1. You need to make DNS changes in the for @ and * for the IP address of the web server. This needs to be done in at your DNS registrar
1. Modify the `pre-ssl_nginx_config` template in this directory and replace `new_domain_name.com` with your new domain name
1. Put the new nginx conf file here: `/etc/nginx/conf.d/new_domain_name.conf`
1. `nginx -t` > make sure this passes
1. `nginx -s reload`
1. Browse to your new site with www & non-www. It should load properly.

## Configure SSL for nginx

Lets get some SSL set up...it's free, there's no reason not to  
[Detailed instructions if needed](http://mangolassi.it/topic/7127/setting-up-letsencrypt-on-a-centos-7-nginx-proxy)

1. Update your nginx conf file for new_domain_name.com to allow SSL, do this by using the `ssl_nginx_config` template in this directory
1. Replace your new nginx ssl config file with the old one at `/etc/nginx/conf.d/new_domain_name.conf`
1. `nginx -t` > this could fail because the SSL keys aren't present yet

### First time configuring SSL on a server: 

1. `yum -y install git python-tools python-pip letsencrypt`
1. `systemctl stop nginx`
1. `letsencrypt certonly --standalone --email your_email_address@email.com --agree-tos -d new_domain_name.com -d www.new_domain_name.com`
1. `systemctl start nginx`
1. Browse to your new site via www & non-www. It should redirect you to https://www.new_domain_name.com

### Anytime after the first time on a server: 

1. `systemctl stop nginx`
1. `letsencrypt certonly --standalone -d new_domain_name.com -d www.new_domain_name.com`
1. `systemctl start nginx`
1. Browse to your new site via www & non-www. It should redirect you to https://www.new_domain_name.com
