自己用于远程调试nginx的步骤记录。

安装gdb等软件包。

apt install build-essential gdb gdbserver cmake

设置ptrace

echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope

创建调试目录

mkdir NginxDebug && cd NginxDebug

拉取稳定版源代码。

wget http://nginx.org/download/nginx-1.22.0.tar.gz

解压

tar -zxvf nginx-1.22.0.tar.gz

安装编译依赖，这里面只有pcre和zlib是必须的，其他是可选的。

apt install libpcre2-dev libgeoip-dev zlib1g-dev libssl-dev

生成makefile

./configure --prefix=/root/NginxDebug/install --with-debug --with-cc-opt='-O0'

~~cd nginx-1.22.0~~

~~./configure --prefix=/usr/local/nginx~~



make install

cp /usr/local/nginx/conf/nginx.conf /home/nginx.conf

vim /usr/local/nginx/conf/nginx.conf

```
user  www;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /usr/local/nginx/conf/vhost/*.conf;
}

```

mkdir /usr/local/nginx/conf/vhost

vim /usr/local/nginx/conf/vhost/test.conf

```
server {
       listen 80 ;
       server_name localhost;
       root /home/wwwroot/seiun.me;
       index index.php index.html index.htm;
       access_log /var/log/nginx/seiun.me.access.log;
       error_log /var/log/nginx/seiun.me.error.log;

       location / {
           if (!-e $request_filename) {
              rewrite ^/(.*) /index.php?s=/$1 last;
             }
          }

       location ~ \.php {
           fastcgi_pass unix:/run/php/php8.1-fpm.sock;
           fastcgi_index index.php;
           include fastcgi_params;
           fastcgi_split_path_info ^((?U).+\.php)(/?.+)$;
          fastcgi_param PATH_INFO  $fastcgi_path_info;
          fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            }

       location ~ /.ht {
             deny all;
        }

         location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$ {
             expires 15d;
        }
       location ~ .*\.(js|css)?$ {
             expires 1d;
         }
      }

```

## php安装

获取最新版php的deb包

```
curl -sSL https://packages.sury.org/php/README.txt | bash -x
```

安装php8.1

```
apt install -y php8.1 php8.1-cli php8.1-common php8.1-fpm php8.1-xml php8.1-curl php8.1-mysql php8.1-sqlite3 php8.1-mbstring php8.1-gd php8.1-fileinfo php8.1-exif php8.1-bcmath php8.1-imagick
```

打开fpm的php.ini，8.1版是在

```
vim /etc/php/8.1/fpm/php.ini
```

打开fpm的配置文件，8.1版是 `vim /etc/php/8.1/fpm/pool.d/www.conf`

## 安装gdb

`apt install gdb`

完整站点配置：

```
server {
   listen 80;
   server_name seiun.me;
   return 301 https://$server_name$request_uri;
}
server {
   listen 80;
   root /home/wwwroot/seiun.me;
   index index.html index.htm index.php;
   access_log /var/log/nginx/seiun.me.access.log main;
   
   listen 443 ssl http2 default_server;
   ssl on;
   ssl_certificate /usr/local/cert/seiun.me_bundle.crt;
   ssl_certificate_key /usr/local/cert/seiun.me.key;
   ssl_session_timeout 10m;
   ssl_protocols TLSv1 TLSv1.1 TLSv2;
   ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
   ssl_prefer_server_ciphers on;

   error_page 404 /404.html;
   error_page 500 502 503 504 /50x.html;
   location = /50x.html {
     root /etc/nginx/html;
   }
   location / {
       index index.html index.php;
       if (-f $request_filename/index.html){
         rewrite (.*) $1/index.html break;
       }
       if (-f $request_filename/index.php){
         rewrite (.*) $1/index.php;
       }
       if (!-f $request_filename){
         rewrite (.*) /index.php;
       }
   }
   
   location ~ \.php {
      fastcgi_pass unix:/var/run/php8.1/php8.1-fpm.sock;
      fastcgi_index index.php;
      include fastcgi_params;
      fastcgi_split_path_info ^((?U).+\.php)(/?.+)$;
      fastcgi_param PATH_INFO $fastcgi_path_info;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
   }
    location ~ ^.+\.php {
      fastcgi_split_path_info ^(.+\.php)(.*)$;
      fastcgi_param PATH_INFO $fastcgi_path_info;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
   }
   location ~ /.ht {
       deny all;
   }
   location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$ {
      expires 15d;
   }
   location ~ .*\.(js|css)?${
      expires 1d;
   }
}
```



```
listen 443 ssl http2 default_server;
server_name seiun.me;
root /home/wwwroot/seiun.me;
ssl on;
ssl_certificate /usr/local/cert/seiun.me_bundle.crt;
ssl_certificate_key /usr/local/cert/seiun.me.key;
ssl_session_timeout 10m;
ssl_protocols TLSv1 TLSv1.1 TLSv2;
ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
ssl_prefer_server_ciphers on;
```

最新的nginx配置：


server {
   listen 80;
   server_name seiun.me;
   return 301 https://$server_name$request_uri;
}
server {
   listen 80;
   root /home/wwwroot/seiun.me;
   index index.html index.htm index.php;
   access_log /var/log/nginx/seiun.me.access.log;
   error_log /var/log/nginx/seiun.me.error.log;

   listen 443 ssl http2 default_server;
   ssl_certificate /usr/local/cert/seiun.me_bundle.crt;
   ssl_certificate_key /usr/local/cert/seiun.me.key;
   ssl_session_timeout 10m;
   ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
   ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
   ssl_prefer_server_ciphers on;

   error_page 404 /404.html;
   error_page 500 502 503 504 /50x.html;
   location = /50x.html {
     root /etc/nginx/html;
   }
   location / {
       index index.html index.php;
       if (-f $request_filename/index.html){
         rewrite (.*) $1/index.html break;
       }
       if (-f $request_filename/index.php){
         rewrite (.*) $1/index.php;
       }
       if (!-f $request_filename){
         rewrite (.*) /index.php;
       }
   }

   location ~ \.php {
      fastcgi_pass unix:/run/php/php8.1-fpm.sock;
      fastcgi_index index.php;
      include fastcgi_params;
      fastcgi_split_path_info ^((?U).+\.php)(/?.+)$;
      fastcgi_param PATH_INFO $fastcgi_path_info;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_path_info;
   }
   location ~ ^.+\.php {
      fastcgi_split_path_info ^(.+\.php)(.*)$;
      fastcgi_param PATH_INFO $fastcgi_path_info;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
   }
   location ~ /.ht {
       deny all;
   }
   location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$ {
      expires 15d;
   }
   location ~ .*\.(js|css)?$ {
      expires 1d;
   }
}
