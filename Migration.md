### Миграция с NGINX на Angie

#### Шаг 1:
Создали ВМ в облаке.

#### Шаг 2: Подключились к консоли и выполнили все рекомендации по установке nginx:

Debian, Ubuntu 
Установите вспомогательные пакеты для подключения репозитория Angie: 
```
sudo apt-get update
```

```
sudo apt-get install -y ca-certificates curl
```

Устанавливаем nginx:
```
sudo apt install nginx
```
Ставим nginx в автозагрузку:
```
sudo systemctl enable nginx
```
Проверяем установку:
```
zubahin@compute-vm-angie01:~$ sudo service nginx status
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Tue 2025-11-25 17:57:41 UTC; 30s ago
       Docs: man:nginx(8)
   Main PID: 3405 (nginx)
      Tasks: 3 (limit: 2313)
     Memory: 2.2M (peak: 4.9M)
        CPU: 19ms
     CGroup: /system.slice/nginx.service
             ├─3405 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             ├─3407 "nginx: worker process"
             └─3408 "nginx: worker process"

Nov 25 17:57:41 compute-vm-angie01 systemd[1]: Starting nginx.service - A high performance web server and a reverse proxy server...
Nov 25 17:57:41 compute-vm-angie01 systemd[1]: Started nginx.service - A high performance web server and a reverse proxy server.
```

Копируем конфигурацию ngnix.conf с лабы (по SFTP) и применяем ее для нашего сервера:
```
zubahin@compute-vm-angie01:~$ ls -la /home/zubahin
total 72
drwxr-x--- 4 zubahin zubahin  4096 Nov 25 18:46 .
drwxr-xr-x 3 root    root     4096 Nov 25 14:57 ..
-rw------- 1 zubahin zubahin    20 Nov 25 18:44 .bash_history
-rw-r--r-- 1 zubahin zubahin   220 Mar 31  2024 .bash_logout
-rw-r--r-- 1 zubahin zubahin  3771 Mar 31  2024 .bashrc
drwx------ 2 zubahin zubahin  4096 Nov 25 15:05 .cache
-rw-r--r-- 1 zubahin zubahin   807 Mar 31  2024 .profile
drwx------ 2 zubahin zubahin  4096 Nov 25 14:57 .ssh
-rw-rw-rw- 1 zubahin zubahin 40960 Nov 25 18:46 nginx_conf.tar-252831-0242cd/gz
```

Распаковываем архив:
```
zubahin@compute-vm-angie01:~$ tar xzf nginx_conf.tar-252831-0242cd.gz
zubahin@compute-vm-angie01:~$ ls -la
total 48
drwxr-x--- 5 zubahin zubahin 4096 Nov 25 19:46 .
drwxr-xr-x 3 root    root    4096 Nov 25 14:57 ..
-rw------- 1 zubahin zubahin  583 Nov 25 19:23 .bash_history
-rw-r--r-- 1 zubahin zubahin  220 Mar 31  2024 .bash_logout
-rw-r--r-- 1 zubahin zubahin 3771 Mar 31  2024 .bashrc
drwx------ 2 zubahin zubahin 4096 Nov 25 15:05 .cache
-rw------- 1 zubahin zubahin   20 Nov 25 19:26 .lesshst
-rw-r--r-- 1 zubahin zubahin  807 Mar 31  2024 .profile
drwx------ 2 zubahin zubahin 4096 Nov 25 14:57 .ssh
drwxr-xr-x 6 zubahin zubahin 4096 Sep 13  2024 nginx
-rw-rw-rw- 1 zubahin zubahin 7172 Nov 25 19:40 nginx_conf.tar-252831-0242cd.gz
zubahin@compute-vm-angie01:~$ ls -la /home/zubahin/nginx
total 72
drwxr-xr-x 6 zubahin zubahin 4096 Sep 13  2024 .
drwxr-x--- 5 zubahin zubahin 4096 Nov 25 19:46 ..
-rw-r--r-- 1 zubahin zubahin 1125 May 30  2023 fastcgi.conf
-rw-r--r-- 1 zubahin zubahin 1055 May 30  2023 fastcgi_params
drwxr-xr-x 2 zubahin zubahin 4096 Oct  4  2023 html
-rw-r--r-- 1 zubahin zubahin 2837 Jun 14  2024 koi-utf
-rw-r--r-- 1 zubahin zubahin 2223 Jun 14  2024 koi-win
-rw-r--r-- 1 zubahin zubahin 3957 May 30  2023 mime.types
-rw-r--r-- 1 zubahin zubahin 2393 Sep 13  2024 nginx.conf
-rw-r--r-- 1 zubahin zubahin  180 May 30  2023 proxy_params
```

Копируем содержимое в папку /etc/nginx/:
```
zubahin@compute-vm-angie01:~$ sudo cp -r -i /home/zubahin/nginx/  /etc/nginx/
```
Проверяем содержимое файла конфигурации nginx:

<details>

```
zubahin@compute-vm-angie01:~$ cat /etc/nginx/nginx.conf
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        proxy_cache_valid 1m;
        proxy_cache_key $scheme$host$request_uri;
        proxy_cache_path /var/www/cache levels=1:2 keys_zone=one:10m inactive=48h max_size=800m;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##

        gzip on;

        gzip_vary on;
        gzip_proxied any;
        gzip_comp_level 6;
        gzip_buffers 16 8k;
        gzip_http_version 1.1;
        gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    brotli_static               on;
    brotli                              on;
    brotli_comp_level   5;
    brotli_types                text/plain text/css text/xml application/javascript application/json image/x-icon image/svg+xml;

    #zstd                       on;
    #zstd_min_length    256;
    #zstd_comp_level    5;
    #zstd_static                on;
    #zstd_types                 text/plain text/css text/xml application/javascript application/json image/x-icon image/svg+xml;

    map $http_accept $webp_suffix {
        "~*webp"  ".webp";
    }

    map $http_accept $avif_suffix {
        "~*avif"  ".avif";
        "~*webp"  ".webp";
    }

    map $msie $cache_control {
      default "max-age=31536000, public, no-transform, immutable";
        "1"     "max-age=31536000, private, no-transform, immutable";
    }

    map $msie $vary_header {
        default "Accept";
        "1"     "";
    }


        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}


#mail {
#       # See sample authentication script at:
#       # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#       # auth_http localhost/auth.php;
#       # pop3_capabilities "TOP" "USER";
#       # imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#       server {
#               listen     localhost:110;
#               protocol   pop3;
#               proxy      on;
#       }
#
#       server {
#               listen     localhost:143;
#               protocol   imap;
#               proxy      on;
#       }
#}
```
</details>

Проверяем работоспособность ngnix:
```
zubahin@compute-vm-angie01:~$ sudo  nginx -t
2025/11/25 19:58:27 [emerg] 4459#4459: unknown directive "brotli_static" in /etc/nginx/nginx.conf:59
nginx: configuration file /etc/nginx/nginx.conf test failed
```
Находим строчки в конфиге, на которые ругается тест:
```
    brotli_static               on;
    brotli                              on;
    brotli_comp_level   5;
    brotli_types                text/plain text/css text/xml application/javascript application/json image/x-icon image/svg+xml;
```
Нам не хватает модуля brotli.  Его нужно поставить.
<details>
     
```
zubahin@compute-vm-angie01:~$ apt search nginx | grep brotli

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

libnginx-mod-http-brotli-filter/noble 1.0.0~rc-5build1 amd64
libnginx-mod-http-brotli-static/noble 1.0.0~rc-5build1 amd64
zubahin@compute-vm-angie01:~$ 
zubahin@compute-vm-angie01:~$ 
zubahin@compute-vm-angie01:~$ 
zubahin@compute-vm-angie01:~$ sudo apt install libnginx-mod-http-brotli-static/noble
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Selected version '1.0.0~rc-5build1' (Ubuntu:24.04/noble [amd64]) for 'libnginx-mod-http-brotli-static'
The following NEW packages will be installed:
  libnginx-mod-http-brotli-static
0 upgraded, 1 newly installed, 0 to remove and 20 not upgraded.
Need to get 7322 B of archives.
After this operation, 34.8 kB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble/universe amd64 libnginx-mod-http-brotli-static amd64 1.0.0~rc-5build1 [7322 B]
Fetched 7322 B in 0s (40.7 kB/s)                          
Selecting previously unselected package libnginx-mod-http-brotli-static.
(Reading database ... 106402 files and directories currently installed.)
Preparing to unpack .../libnginx-mod-http-brotli-static_1.0.0~rc-5build1_amd64.deb ...
Unpacking libnginx-mod-http-brotli-static (1.0.0~rc-5build1) ...
Setting up libnginx-mod-http-brotli-static (1.0.0~rc-5build1) ...
Processing triggers for nginx (1.24.0-2ubuntu7.5) ...
Scanning processes...                                                                                                                                                 
Scanning linux images...                                                                                                                                              

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
zubahin@compute-vm-angie01:~$ apt search nginx | grep brotli

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

libnginx-mod-http-brotli-filter/noble 1.0.0~rc-5build1 amd64
libnginx-mod-http-brotli-static/noble,now 1.0.0~rc-5build1 amd64 [installed]
zubahin@compute-vm-angie01:~$ sudo apt install libnginx-mod-http-brotli-filter/noble
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Selected version '1.0.0~rc-5build1' (Ubuntu:24.04/noble [amd64]) for 'libnginx-mod-http-brotli-filter'
The following NEW packages will be installed:
  libnginx-mod-http-brotli-filter
0 upgraded, 1 newly installed, 0 to remove and 20 not upgraded.
Need to get 9346 B of archives.
After this operation, 43.0 kB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble/universe amd64 libnginx-mod-http-brotli-filter amd64 1.0.0~rc-5build1 [9346 B]
Fetched 9346 B in 0s (187 kB/s)                           
Selecting previously unselected package libnginx-mod-http-brotli-filter.
(Reading database ... 106410 files and directories currently installed.)
Preparing to unpack .../libnginx-mod-http-brotli-filter_1.0.0~rc-5build1_amd64.deb ...
Unpacking libnginx-mod-http-brotli-filter (1.0.0~rc-5build1) ...
Setting up libnginx-mod-http-brotli-filter (1.0.0~rc-5build1) ...
Processing triggers for nginx (1.24.0-2ubuntu7.5) ...
Scanning processes...                                                                                                                                                 
Scanning linux images...                                                                                                                                              

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
```
</details>

Теперь все в порядке:
```
zubahin@compute-vm-angie01:~$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
zubahin@compute-vm-angie01:~$ 
```

Проверяем, что ngnix запущен (видим, что еще не запущен и перезапускаем):
```
zubahin@compute-vm-angie01:~$ !ps
ps aux | grep nginx 
zubahin     4785  0.0  0.1   7076  2176 pts/3    S+   20:15   0:00 grep --color=auto nginx
zubahin@compute-vm-angie01:~$ sudo systemctl start ngnix
Failed to start ngnix.service: Unit ngnix.service not found.
zubahin@compute-vm-angie01:~$ sudo systemctl start nginx
zubahin@compute-vm-angie01:~$ !ps
ps aux | grep nginx 
root        4807  0.0  0.0  23688  1604 ?        Ss   20:15   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
www-data    4808  0.0  0.1  23688  2628 ?        S    20:15   0:00 nginx: worker process
www-data    4809  0.0  0.1  23688  2628 ?        S    20:15   0:00 nginx: worker process
www-data    4810  0.0  0.1  23688  2372 ?        S    20:15   0:00 nginx: cache manager process
www-data    4812  0.0  0.1  23688  2372 ?        S    20:15   0:00 nginx: cache loader process
zubahin     4823  0.0  0.1   7076  2176 pts/3    S+   20:16   0:00 grep --color=auto nginx
```

Ставим nginx в автозагрузку:
```
sudo systemctl enable nginx

zubahin@compute-vm-angie01:~$ sudo systemctl enable nginx
Synchronizing state of nginx.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable nginx
```

#### Шаг 3: Подключились к консоли и выполнили все рекомендации по установке angie:

Скачайте открытый ключ репозитория Angie для проверки подлинности пакетов: 
```
sudo curl -o /etc/apt/trusted.gpg.d/angie-signing.gpg \
            https://angie.software/keys/angie-signing.gpg
```

Подключите репозиторий Angie:
```
echo "deb https://download.angie.software/angie/$(. /etc/os-release && echo "$ID/$VERSION\_ID $VERSION\_CODENAME") main" \\ | sudo tee /etc/apt/sources.list.d/angie.list > /dev/null
```

Обновите индексы репозиториев: 
```
sudo apt-get update
```

Установите пакет Angie: 
```
sudo apt-get install -y angie
```
Проверяем установку:

```
zubahin@compute-vm-angie01:~$ angie -V
Angie version: Angie/1.10.3
nginx version: nginx/1.27.5
built on Thu, 13 Nov 2025 10:52:28 GMT
built with OpenSSL 3.0.13 30 Jan 2024
TLS SNI support enabled configure arguments: --prefix=/etc/angie --conf-path=/etc/angie/angie.conf --error-log-path=/var/log/angie/error.log --http-log-
path=/var/log/angie/access.log --lock-path=/run/angie.lock --modules-path=/usr/lib/angie/modules --pid-path=/run/angie.pid --sbin-path=/usr/sbin/angie
--http-acme-client-path=/var/lib/angie/acme --http-client-body-temp-path=/var/cache/angie/client\_temp --http-fastcgi-temp-path=/var/cache/angie/fastcgi\_temp
--http-proxy-temp-path=/var/cache/angie/proxy\_temp --http-scgi-temp-path=/var/cache/angie/scgi\_temp --http-uwsgi-temp-path=/var/cache/angie/uwsgi\_temp
--user=angie --group=angie --with-file-aio --with-http\_acme\_module --with-http\_addition\_module --with-http\_auth\_request\_module --with-http\_dav\_module
 --with-http\_flv\_module --with-http\_gunzip\_module --with-http\_gzip\_static\_module --with-http\_mp4\_module --with-http\_random\_index\_module
--with-http\_realip\_module --with-http\_secure\_link\_module --with-http\_slice\_module --with-http\_ssl\_module --with-http\_stub\_status\_module
--with-http\_sub\_module --with-http\_v2\_module --with-http\_v3\_module --with-mail --with-mail\_ssl\_module --with-stream --with-stream\_acme\_module
--with-stream\_mqtt\_preread\_module --with-stream\_rdp\_preread\_module --with-stream\_realip\_module --with-stream\_ssl\_module --with-
stream\_ssl\_preread\_module --with-threads --feature-cache=../angie-feature-cache --with-ld-opt='-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,
-z,relro -Wl,-z,now' zubahin@compute-vm-angie01:~$
```

### Проверяем, какие пакеты установлены для NGNIX:



### Смотрим содержимое папок в nginx:
```
zubahin@compute-vm-angie01:~$ ls -la /etc/nginx
total 80
drwxr-xr-x   9 root root 4096 Nov 25 19:49 .
drwxr-xr-x 108 root root 4096 Nov 26 07:59 ..
drwxr-xr-x   2 root root 4096 Aug 22 12:45 conf.d
-rw-r--r--   1 root root 1125 Nov 30  2023 fastcgi.conf
-rw-r--r--   1 root root 1055 Nov 30  2023 fastcgi_params
-rw-r--r--   1 root root 2837 Nov 30  2023 koi-utf
-rw-r--r--   1 root root 2223 Nov 30  2023 koi-win
-rw-r--r--   1 root root 5465 Nov 30  2023 mime.types
drwxr-xr-x   2 root root 4096 Aug 22 12:45 modules-available
drwxr-xr-x   2 root root 4096 Nov 25 20:11 modules-enabled
drwxr-xr-x   6 root root 4096 Nov 25 19:49 nginx
-rw-r--r--   1 root root 2393 Nov 25 19:52 nginx.conf
-rw-r--r--   1 root root  180 Nov 30  2023 proxy_params
-rw-r--r--   1 root root  636 Nov 30  2023 scgi_params
drwxr-xr-x   2 root root 4096 Nov 25 17:57 sites-available
drwxr-xr-x   2 root root 4096 Nov 25 17:57 sites-enabled
drwxr-xr-x   2 root root 4096 Nov 25 17:57 snippets
-rw-r--r--   1 root root  664 Nov 30  2023 uwsgi_params
-rw-r--r--   1 root root 3071 Nov 30  2023 win-utf
zubahin@compute-vm-angie01:~$ cat /etc/nginx/conf.d
cat: /etc/nginx/conf.d: Is a directory
zubahin@compute-vm-angie01:~$ ls -la /etc/nginx/conf.d
total 8
drwxr-xr-x 2 root root 4096 Aug 22 12:45 .
drwxr-xr-x 9 root root 4096 Nov 25 19:49 ..
zubahin@compute-vm-angie01:~$ ls -la /etc/nginx/snippets
total 16
drwxr-xr-x 2 root root 4096 Nov 25 17:57 .
drwxr-xr-x 9 root root 4096 Nov 25 19:49 ..
-rw-r--r-- 1 root root  423 Nov 30  2023 fastcgi-php.conf
-rw-r--r-- 1 root root  217 Nov 30  2023 snakeoil.conf
zubahin@compute-vm-angie01:~$ ls -la /etc/nginx/sites-enabled
total 8
drwxr-xr-x 2 root root 4096 Nov 25 17:57 .
drwxr-xr-x 9 root root 4096 Nov 25 19:49 ..
lrwxrwxrwx 1 root root   34 Nov 25 17:57 default -> /etc/nginx/sites-available/default
zubahin@compute-vm-angie01:~$ ls -la /etc/nginx/sites-available
total 12
drwxr-xr-x 2 root root 4096 Nov 25 17:57 .
drwxr-xr-x 9 root root 4096 Nov 25 19:49 ..
-rw-r--r-- 1 root root 2412 Nov 30  2023 default
zubahin@compute-vm-angie01:~$ ls -la /etc/nginx/modules-enabled
total 16
drwxr-xr-x 2 root root 4096 Nov 25 20:11 .
drwxr-xr-x 9 root root 4096 Nov 25 19:49 ..
lrwxrwxrwx 1 root root   62 Nov 25 20:11 50-mod-http-brotli-filter.conf -> /usr/share/nginx/modules-available/mod-http-brotli-filter.conf
lrwxrwxrwx 1 root root   62 Nov 25 20:11 50-mod-http-brotli-static.conf -> /usr/share/nginx/modules-available/mod-http-brotli-static.conf
zubahin@compute-vm-angie01:~$ ls -la /etc/nginx/modules-available
total 8
drwxr-xr-x 2 root root 4096 Aug 22 12:45 .
drwxr-xr-x 9 root root 4096 Nov 25 19:49 ..
```

### Находим значимые include в конфигурации:
```
zubahin@compute-vm-angie01:~$ cat /etc/nginx/nginx.conf | grep include
        include /etc/nginx/modules-enabled/*.conf;
        include /etc/nginx/mime.types;
        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
```

### Учитывая содержание папок (что проверялось на прошлом шаге), нас интересует только конфигурация одного файла:
```
ubahin@compute-vm-angie01:~$ cat /etc/nginx/sites-available/default
##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# https://www.nginx.com/resources/wiki/start/
# https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
# https://wiki.debian.org/Nginx/DirectoryStructure
#
# In most cases, administrators will remove this file from sites-enabled/ and
# leave it as reference inside of sites-available where it will continue to be
# updated by the nginx packaging team.
#
# This file will automatically load configuration files provided by other
# applications, such as Drupal or Wordpress. These applications will be made
# available underneath a path with that package name, such as /drupal8.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##

# Default server configuration
#
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        # SSL configuration
        #
        # listen 443 ssl default_server;
        # listen [::]:443 ssl default_server;
        #
        # Note: You should disable gzip for SSL traffic.
        # See: https://bugs.debian.org/773332
        #
        # Read up on ssl_ciphers to ensure a secure configuration.
        # See: https://bugs.debian.org/765782
        #
        # Self signed certs generated by the ssl-cert package
        # Don't use them in a production server!
        #
        # include snippets/snakeoil.conf;

        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
        }

        # pass PHP scripts to FastCGI server
        #
        #location ~ \.php$ {
        #       include snippets/fastcgi-php.conf;
        #
        #       # With php-fpm (or other unix sockets):
        #       fastcgi_pass unix:/run/php/php7.4-fpm.sock;
        #       # With php-cgi (or other tcp sockets):
        #       fastcgi_pass 127.0.0.1:9000;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #       deny all;
        #}
}


# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
#server {
#       listen 80;
#       listen [::]:80;
#
#       server_name example.com;
#
#       root /var/www/example.com;
#       index index.html;
#
#       location / {
#               try_files $uri $uri/ =404;
#       }
#}
```
Как видим большая часть строк закоментирована.  Перенесем их в конфигурацию angie.

### А также конфигурация модулей:
Поскольку мы увидели строчку: include /etc/nginx/modules-enabled/*.conf;
но как мы знаем:
```
zubahin@compute-vm-angie01:~$ ls -la /etc/nginx/modules-enabled
total 16
drwxr-xr-x 2 root root 4096 Nov 25 20:11 .
drwxr-xr-x 9 root root 4096 Nov 25 19:49 ..
lrwxrwxrwx 1 root root   62 Nov 25 20:11 50-mod-http-brotli-filter.conf -> /usr/share/nginx/modules-available/mod-http-brotli-filter.conf
lrwxrwxrwx 1 root root   62 Nov 25 20:11 50-mod-http-brotli-static.conf -> /usr/share/nginx/modules-available/mod-http-brotli-static.conf
```
Смотрим содержимое файлов:
```
ubahin@compute-vm-angie01:~$ cat /usr/share/nginx/modules-available/mod-http-brotli-filter.conf
load_module modules/ngx_http_brotli_filter_module.so;
zubahin@compute-vm-angie01:~$ cat /usr/share/nginx/modules-available/mod-http-brotli-static.conf
load_module modules/ngx_http_brotli_static_module.so;
```
### Устанавливаем модуль brotli на Angie:

Установите пакеты необходимых вам дополнений:
```
sudo apt-get install -y <ИМЯ ПАКЕТА>/br
```
В нашем случае:
```
zubahin@compute-vm-angie01:~$ sudo apt-get install -y angie-module-brotli
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  angie-module-brotli
0 upgraded, 1 newly installed, 0 to remove and 9 not upgraded.
Need to get 13.7 kB of archives.
After this operation, 95.2 kB of additional disk space will be used.
Get:1 https://download.angie.software/angie/ubuntu/24.04 noble/main amd64 angie-module-brotli amd64 1.10.3-1~noble [13.7 kB]
Fetched 13.7 kB in 0s (105 kB/s)                
Selecting previously unselected package angie-module-brotli.
(Reading database ... 106411 files and directories currently installed.)
Preparing to unpack .../angie-module-brotli_1.10.3-1~noble_amd64.deb ...
Unpacking angie-module-brotli (1.10.3-1~noble) ...
Setting up angie-module-brotli (1.10.3-1~noble) ...
----------------------------------------------------------------------

The Brotli dynamic module for Angie has been installed.
To enable this module, add the following to /etc/angie/angie.conf
and reload angie:

    load_module modules/ngx_http_brotli_filter_module.so;
or:
    load_module modules/ngx_http_brotli_static_module.so;

Please refer to the module documentation for further details:
https://github.com/google/ngx_brotli

----------------------------------------------------------------------
Scanning processes...                                                                                                                                                 
Scanning candidates...                                                                                                                                                
Scanning linux images...                                                                                                                                              

Pending kernel upgrade!
Running kernel version:
  6.8.0-87-generic
Diagnostics:
  The currently running kernel version is not the expected kernel version 6.8.0-88-generic.

Restarting the system to load the new kernel will not be handled automatically, so you should consider rebooting.

Restarting services...

Service restarts being deferred:
 systemctl restart unattended-upgrades.service

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
```

### На всякий случай проверим, какие порты слушает nginx (вдруг есть что-то экзотическое):
```

```
### Начинаем перенос конфигурации в файл angie.conf


