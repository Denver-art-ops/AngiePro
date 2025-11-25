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

Проверяем, что ngnix запущен:
```
zubahin@compute-vm-angie01:~$ !ps
ps aux | grep ngnix
zubahin     4093  0.0  0.1   7076  2176 pts/3    S+   19:14   0:00 grep --color=auto ngnix
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




(Необязательно) Установите пакеты необходимых вам дополнений:
```
sudo apt-get install -y <ИМЯ ПАКЕТА>/br
```

### Результат

```
zubahin@compute-vm-angie01:~$ sudo apt-get install -y angie-module-image-filter\*\*
Reading package lists...
Done Building dependency tree...
Done Reading state information...
Done The following NEW packages will be installed: angie-module-image-filter 0 upgraded, 1 newly installed, 0 to remove and 13 not upgraded.
Need to get 16.5 kB of archives. After this operation, 72.7 kB of additional disk space will be used.
Get:1 https://download.angie.software/angie/ubuntu/24.04 noble/main amd64 angie-module-image-filter amd64 1.10.3-1~noble \[16.5 kB\]
Fetched 16.5 kB in 0s (116 kB/s) Selecting previously unselected package angie-module-image-filter.
(Reading database ... 106354 files and directories currently installed.)
Preparing to unpack .../angie-module-image-filter\_1.10.3-1~noble\_amd64.deb ...
Unpacking angie-module-image-filter (1.10.3-1~noble) ...
Setting up angie-module-image-filter (1.10.3-1~noble) ...
---------------------------------------------------------------------- The image-filter dynamic module for Angie has been installed.
To enable this module, add the following to /etc/angie/angie.conf and reload angie:
load\_module modules/ngx\_http\_image\_filter\_module.so;
Please refer to the modules documentation for further details: https://en.angie.software/angie/docs/configuration/modules/http/http\_image\_filter/
---------------------------------------------------------------------- Scanning processes...
Scanning linux images...
Running kernel seems to be up-to-date.
No services need to be restarted. No containers need to be restarted. No user sessions are running outdated binaries.
No VM guests are running outdated hypervisor (qemu) binaries on this host.
```



