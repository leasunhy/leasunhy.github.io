---
title: 'Gitlab: Custom Nginx Setup'
date: 2015-07-20 22:49:05
tags:
  - Gitlab
  - Nginx
  - Https
  - Centos
---

Gitlab is a code hosting platform similar to Github. The community edition is feature-full (e.g. unlimited number of private repos!) and free to use.

Let's try to deploy Gitlab on our CentOS 7.1 server.

<!-- more -->

My Server Configuration
--------------------
* OS: Centos 7.1
    * Kernel: 3.10.0-229.7.2.el7.x86_64
* Memory: 1 GiB physical + 2 GiB swap
    * *Gitlab [require](https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/install/requirements.md) at least 2GiB addressable memory to function*
* Storage: 20 GiB SSD
* Provider: [Vultr](http://www.vultr.com/?ref=6821868)
* Website: [My Gitlab](https://git.leasunhy.com)

Gitlab runs smoothly in my first tests.

Install
-------
Simply use the [Omnibus package](https://about.gitlab.com/downloads/), but don't `reconfigure` yet.

```bash
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
sudo yum install gitlab-ce
```

Configure
---------
Since we want to host multiple websites using the same server (and the same ip address), we need to use our own Nginx, rather than the bundled one.

Furthermore, I would like to enable HTTPS to obtain enhanced security and privacy, so I allow HTTPS as well.

At last, for sending emails, I would like to use the service provided by [Mailgun](https://mailgun.com), rather than `postfix` on my server.

### Firewall ###
HTTP and HTTPS inbound traffic should be already enabled if you have other websites running on this server as well. I just list the following commands for reference.

```bash
sudo firewall-cmd --zone=public --permanent --add-service=https
sudo firewall-cmd --zone=public --permanent --add-service=http
sudo firewall-cmd --reload
```

### Use our own Nginx ###
To use our own Nginx, the configuration of the Gitlab must be changed, and an Nginx vhost conf file should be added.

#### Gitlab Configuration ####
File `/etc/gitlab/gitlab.rb` (only modifications to original file is listed):

```ruby
# set the url of your gitlab website
external_url 'https://git.YOURDOMAIN.com'

# since the original 8080 may conflict with your other websites on this server
unicorn['port'] = SOME_PORT

# allow Nginx user to access gitlab files
web_server['external_users'] = ['nginx']

# disable the bundled version of Nginx
nginx['enable'] = false
```

#### Nginx vhost file ####
Add a vhost configuration file to your fimiliar location (e.g. `/usr/local/etc/nginx/sites.avail`), with the following content:

```nginx
upstream gitlab {
    server unix:/var/opt/gitlab/gitlab-rails/sockets/gitlab.socket fail_timeout=0;
}

server {
    listen 80;
    server_name git.YOURDOMAIN.com;
    
    location / {
        rewrite ^ https://$server_name$request_uri permanent;
    }
}

server {
    listen 443 ssl;
    server_name git.YOURDOMAIN.com;

    server_tokens off;
    root /opt/gitlab/embedded/service/gitlab-rails/public;

    client_max_body_size 250m;

    access_log /var/log/gitlab/nginx/gitlab_access.log;
    error_log /var/log/gitlab/nginx/gitlab_error.log;

    ssl_certificate /PATH/TO/YOUR/DOMAIN/CERTIFICATE;
    ssl_certificate_key /PATH/TO/YOUR/PRIVATE/KEY;

    ssl_prefer_server_ciphers on;
    ssl_dhparam /PATH/TO/DHPARAM/PEM;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers "EECDH+ECDSA+AESGCM EECDH+aRSA+AESGCM EECDH+ECDSA+SHA384 EECDH+ECDSA+SHA256 EECDH+aRSA+SHA384 EECDH+aRSA+SHA256 EECDH+aRSA+RC4 EECDH EDH+aRSA !aNULL !eNULL !LOW !3DES !MD5 !EXP !PSK !SRP !DSS !RC4";
    keepalive_timeout 70;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    location / {
        try_files $uri $uri/index.html $uri.html @gitlab;
    }

    location /uploads/ {
        gzip off;
        proxy_read_timeout 300;
        proxy_connect_timeout 300;
        proxy_redirect off;

        proxy_set_header HOST $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwareded-Ssl on;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Frame-Options SAMEORIGIN;

        proxy_set_header X-NginX-Proxy true;

        proxy_pass http://gitlab;
    }

    location @gitlab {
        gzip off;

        proxy_read_timeout 300;
        proxy_connect_timeout 300;
        proxy_redirect off;

        proxy_set_header HOST $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwareded-Ssl on;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Frame-Options SAMEORIGIN;

        proxy_set_header X-NginX-Proxy true;

        proxy_pass http://gitlab;
    }

    location ~ ^/(assets)/ {
        root /opt/gitlab/embedded/service/gitlab-rails/public;
        gzip_static on;
        expires max;
        add_header Cache-Control public;
    }

    error_page 502 /502.html;
}
```

#### Selinux ####
According to [https://gitlab.com/gitlab-org/gitlab-recipes/tree/master/web-server/apache#selinux-modifications](https://gitlab.com/gitlab-org/gitlab-recipes/tree/master/web-server/apache#selinux-modifications), you need either disable Selinux or do the following:

```bash
setsebool -P httpd_can_network_connect on
setsebool -P httpd_can_network_relay on
setsebool -P httpd_read_user_content on
semanage -i - <<EOF
fcontext -a -t user_home_dir_t '/home/git(/.*)?'
fcontext -a -t ssh_home_t '/home/git/.ssh(/.*)?'
fcontext -a -t httpd_sys_content_t '/home/git/gitlab/public(/.*)?'
fcontext -a -t httpd_sys_content_t '/home/git/repositories(/.*)?'
EOF
restorecon -R /home/git
```

#### Test ####
The Nginx part of configuration is now over! Go test it!

```bash
sudo systemctl enable nginx
sudo systemctl restart nginx
sudo gitlab-ctl reconfigure
sudo gitlab-ctl restart
```

Use your favourite browser and navigate to `https://git.YOURDOMAIN.com`. Voila! Your brand new Gitlab website is now before your eyes!

If you are unlucky enough (like me...) to encounter a 502 error, don't upset! It's likely a Selinux problem, so just do the following:

```bash
sudo grep nginx /var/log/audit.log | audit2allow -M nginx
sudo semodule -i nginx.pp
```

Now refresh your page. Hooray!

If still, some error occurs... Go check the log files to see what's going on:

```bash
sudo vim /var/log/gitlab/gitlab-rails/application.log
sudo vim /var/log/gitlab/gitlab-rails/production.log
sudo vim /var/log/gitlab/nginx/access.log
sudo vim /var/log/gitlab/nginx/error.log
```

### Use Mailgun to send emails ###
Just follow the [official instructions](https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/smtp.md).

Make sure to `sudo gitlab-ctl reconfigure`.

Conclusion
----------
Git is an excellent version control tool, and a personal code hosting platform running at the remote will be very convenient when using git. For me, I use my Gitlab to store some codes that I'm shy to show to others, and some personal stuff like notes, diaries, etc.

Feel attracted? Why not give it a shot ;-)

