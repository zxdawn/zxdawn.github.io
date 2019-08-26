---
title: How to set up Hexo
date: 2017-10-02
tags: ["hexo"]
categories: ["sci-tech"]
draft: false
toc: true
img: "/images/sci-tech/2017-10/hexo.jpg"
---

## Install **Node.js and npm** on my local Linux computer:

<!--more-->

   ```
   $ sudo apt-get update && sudo apt-get upgrade
   $ curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
   $ sudo apt-get install nodejs
   ```

   To verify that the installation is successful you can check the version of npm:

   ```
   $ npm -v
   ```

## Install **hexo**

   ```
   $ npm install hexo-cli -g
   $ hexo init blog #install to current_directory/blog/
   $ cd blog
   $ npm install
   $ hexo server
   ```
## Configure **Nginx** Virtual Host

   PS: [installation of Nginx, PHP and MySQL](https://tecadmin.net/install-php-7-nginx-mysql-on-ubuntu/)

   ```
   $ cd /etc/nginx/conf.d
   $ cp default.conf hexo.conf
   $ vi hexo.conf
   ```

   ```
   server {
       listen       80;
       root   /root/blog; #blog location
       index index.php index.html index.htm;
       server_name  www.dreambooker.site; #URL
       location / {
                    try_files $uri $uri/ =404;
       }
   }
   ```

   ```
   $ service nginx restart
   ```

## Set the permission of nginx

1. Check the username of nginx

   `grep user /etc/nginx/nginx.conf`
   My username of nginx is 'nginx'

2. Set the permission of blog directory

   ```
   chown -R nginx.nginx /root/
   chown -R nginx.nginx /root/blog/
   ```

## Other configurations

Because I installed h5ai, here's the configuration of h5ai:

   ```
   server {
    listen       80 default_server;
    root   /usr/share/nginx/html;
    index index.php index.html index.htm /_h5ai/public/index.php;
    server_name  _;

    location / {
              try_files $uri $uri/ /index.html;
    }

    error_page  404              /404.html;
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    location ~ \.php$ {
        fastcgi_pass  127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
   }
   ```

## Install **Rsync**

   VPS: 
   ```
   $ apt-get install rsync
   ```

   Local: 
   ```
   $ npm install hexo-deployer-rsync --save
   ```


## Local setting

   ```
   $ vi _config.yml
   ```

   ```
   deploy:
     type: rsync
     host: your_host
     user: root # your username
     root: /root/blog/ # directory of blog on vps
     port: 22
   ```

## Generate and deploy

   Open terminal in blog/

   ```
   $ hexo generate && hexo deploy
   ```