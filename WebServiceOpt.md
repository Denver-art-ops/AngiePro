
### Подготовительные шаги:

#### Шаг 1:
Используем Docker-compose и набор файлов (.env dockerignore docker-compose.yaml) в директорию project:


```
zubahin@compute-vm-angie01:~$ cd project
zubahin@compute-vm-angie01:~/project$ ls -la
total 24
drwxrwxr-x 2 zubahin zubahin 4096 Jan  5 11:18 .
drwxr-x--- 6 zubahin zubahin 4096 Jan  5 11:32 ..
-rw-rw-rw- 1 zubahin zubahin    5 Dec 26 07:38 .dockerignore
-rw-rw-rw- 1 zubahin zubahin   83 Dec 26 07:38 .env
-rw-rw-rw- 1 zubahin zubahin 1193 Dec 26 08:47 angie.conf
-rw-rw-rw- 1 zubahin zubahin 1106 Jan  5 11:18 docker-compose.yml
```

Файл приложены к материалам.
Ниже продублировано содержание docker-compose.yaml:

<details>
    
```
version: '3'

services:
  db:
    image: mysql:8.0
    container_name: db
    restart: unless-stopped
    env_file: .env
    environment:
      - MYSQL_DATABASE=wordpress
    volumes:
      - dbdata:/var/lib/mysql
    command: '--default-authentication-plugin=mysql_native_password'
    networks:
      - app-network

  wordpress:
    depends_on:
      - db
    image: wordpress:6.0.1-php8.0-fpm-alpine
    container_name: wordpress
    restart: unless-stopped
    env_file: .env
    environment:
      - WORDPRESS_DB_HOST=db:3306
      - WORDPRESS_DB_USER=$MYSQL_USER
      - WORDPRESS_DB_PASSWORD=$MYSQL_PASSWORD
      - WORDPRESS_DB_NAME=wordpress
    volumes:
      - wordpress:/var/www/html
    networks:
      - app-network

  angie:
   depends_on:
      - wordpress
    image: docker.angie.software/angie:1.10.3-ubuntu
    container_name: angie
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - wordpress:/var/www/html
      - /home/zubahin/angie:/etc/angie:ro
    networks:
      - app-network
volumes:
  wordpress:
  dbdata:

networks:
  app-network:
    driver: bridge


```

</details>

ключевыми настройками здесь является проброс портов 80:80  мапирование директории для конфигураций контейнера angie в файлах хостовой системы (home/zubahin/angie) и создание сети ( bridge) 
с названием app-network  чтобы контейнеры видели друг-друга:

```
    ports:
      - "80:80"
    volumes:
      - wordpress:/var/www/html
      - /home/zubahin/angie:/etc/angie:ro
    networks:
      - app-network
```

Запускаем docker-compose:

```
docker-compose up -d
```

Проверяем состояние контейнеров и сети:

```
zubahin@compute-vm-angie01:~/project$ docker ps -a
CONTAINER ID   IMAGE                                       COMMAND                  CREATED        STATUS         PORTS                                 NAMES
b8516624afcb   docker.angie.software/angie:1.10.3-ubuntu   "angie -g 'daemon of…"   30 hours ago   Up 9 minutes   0.0.0.0:80->80/tcp, [::]:80->80/tcp   angie
1037e688e707   wordpress:6.0.1-php8.0-fpm-alpine           "docker-entrypoint.s…"   30 hours ago   Up 9 minutes   9000/tcp                              wordpress
f524c6d07282   mysql:8.0                                   "docker-entrypoint.s…"   30 hours ago   Up 9 minutes   3306/tcp, 33060/tcp                   db

zubahin@compute-vm-angie01:~/project$ docker network ls
NETWORK ID     NAME                  DRIVER    SCOPE
6b3b65f6e386   bridge                bridge    local
bacb1e0938f0   host                  host      local
21964b12fec6   none                  null      local
e850726d3268   project_app-network   bridge    local

```


#### Шаг 2: Вносим изменения в конфигурационные файлы.

#####  корневой файл angie.conf:
```
zubahin@compute-vm-angie01:~/angie$ vim angie.conf
```

<details>
    
```
 package: angie-module-auth-jwt
#load_module modules/ngx_http_auth_jwt_module.so;

# package: angie-module-auth-ldap
#load_module modules/ngx_http_auth_ldap_module.so;

# package: angie-module-auth-pam
#load_module modules/ngx_http_auth_pam_module.so;

# package: angie-module-auth-spnego
#load_module modules/ngx_http_auth_spnego_module.so;

# package: angie-module-auth-totp
#load_module modules/ngx_http_auth_totp_module.so;

# package: angie-module-brotli
#load_module modules/ngx_http_brotli_filter_module.so;
#load_module modules/ngx_http_brotli_static_module.so;

# package: angie-module-cache-purge
#load_module modules/ngx_http_cache_purge_module.so;

# package: angie-module-cgi
#load_module modules/ngx_http_cgi_module.so;

# package: angie-module-combined-upstreams
#load_module modules/ngx_http_combined_upstreams_module.so;

# package: angie-module-dav-ext
#load_module modules/ngx_http_dav_ext_module.so;

# package: angie-module-dynamic-limit-req
#load_module modules/ngx_http_dynamic_limit_req_module.so;

# package: angie-module-echo
#load_module modules/ngx_http_echo_module.so;

# package: angie-module-enhanced-memcached
#load_module modules/ngx_http_enhanced_memcached_module.so;

# package: angie-module-eval
#load_module modules/ngx_http_eval_module.so;

# package: angie-module-geoip2
#load_module modules/ngx_http_geoip2_module.so;
#load_module modules/ngx_stream_geoip2_module.so;

# package: angie-module-headers-more
#load_module modules/ngx_http_headers_more_filter_module.so;

# package: angie-module-http-auth-radius
# uncommenting line below make sure you have configured 'radius_server'
#load_module modules/ngx_http_auth_radius_module.so;

# package: angie-module-image-filter
#load_module modules/ngx_http_image_filter_module.so;

# package: angie-module-keyval
#load_module modules/ngx_http_keyval_module.so;
#load_module modules/ngx_stream_keyval_module.so;

# package: angie-module-ndk
#load_module modules/ndk_http_module.so;

# package: angie-module-lua
#load_module modules/ngx_http_lua_module.so;
#load_module modules/ngx_stream_lua_module.so;

#package: angie-module-modsecurity
#load_module modules/ngx_http_modsecurity_module.so;

# package: angie-module-njs
#load_module modules/ngx_http_js_module.so;
#load_module modules/ngx_stream_js_module.so;

# package: angie-module-opentracing
#load_module modules/ngx_http_opentracing_module.so;

# package: angie-module-otel
#load_module modules/ngx_otel_module.so;

# package: angie-module-perl
#load_module modules/ngx_http_perl_module.so;

# package: angie-module-postgres
#load_module modules/ngx_postgres_module.so;

# package: angie-module-redis2
#load_module modules/ngx_http_redis2_module.so;

# package: angie-module-rtmp
#load_module modules/ngx_rtmp_module.so;

# package: angie-module-set-misc
#load_module modules/ngx_http_set_misc_module.so;

# package: angie-module-subs
#load_module modules/ngx_http_subs_filter_module.so;

# package: angie-module-testcookie
#load_module modules/ngx_http_testcookie_access_module.so;

# package: angie-module-unbrotli
# load_module modules/ngx_http_unbrotli_filter_module.so;

# package: angie-module-upload
#load_module modules/ngx_http_upload_module.so;

# package: angie-module-vod
#load_module modules/ngx_http_vod_module.so;

# package: angie-module-vts
#load_module modules/ngx_http_stream_server_traffic_status_module.so;
#load_module modules/ngx_http_vhost_traffic_status_module.so;
#load_module modules/ngx_stream_server_traffic_status_module.so;

# package: angie-module-wasm
#load_module modules/ngx_wasm_module.so;
#load_module modules/ngx_wasm_core_module.so;
#load_module modules/ngx_http_wasm_host_module.so;
#load_module modules/ngx_wasmtime_module.so;

# package: angie-module-xslt
#load_module modules/ngx_http_xslt_filter_module.so;

# package: angie-module-zip
#load_module modules/ngx_http_zip_module.so;

# package: angie-module-zstd
load_module modules/ngx_http_zstd_filter_module.so;
load_module modules/ngx_http_zstd_static_module.so;

user  angie;
worker_processes  auto;
worker_rlimit_nofile 65536;

error_log  /var/log/angie/error.log notice;
pid        /run/angie/angie.pid;

events {
    worker_connections  65536;
}


http {
    include       /etc/angie/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    log_format extended '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" rt="$request_time" '
                        '"$http_user_agent" "$http_x_forwarded_for" '
                        'h="$host" sn="$server_name" ru="$request_uri" u="$uri" '
                        'ucs="$upstream_cache_status" ua="$upstream_addr" us="$upstream_status" '
                        'uct="$upstream_connect_time" urt="$upstream_response_time"';

    access_log  /var/log/angie/access.log  main;

    sendfile        on;
    tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

# Включаем zstd
    zstd on;
    zstd_min_length 256;
    zstd_comp_level 5;
    #zstd_static on;
    zstd_types text/plain text/css text/xml application/javascript
    application/json image/x-icon image/svg+xml;

    # Proxy cache
    proxy_cache_valid 1m;
    proxy_cache_key $scheme$host$request_uri;
    proxy_cache_path /cache levels=1:2 keys_zone=one:10m inactive=48h max_size=800m;


    include /etc/angie/http.d/*.conf;

# MAP для картинок

map $msie $cache_control {
 default "max-age=31536000, public, no-transform, immutable";
 "1" "max-age=31536000, private, no-transform, immutable";
 }
 map $msie $vary_header {
default "Accept";
"1" "";
 }
 map $http_accept $webp_suffix {
 "~*webp" ".webp";
 }
 map $http_accept $avif_suffix {
 "~*avif" ".avif";
 "~*webp" ".webp";
 }

}

#stream {
#    include /etc/angie/stream.d/*.conf;
#}



```

</details>


#####  файл с настройками модуля server в http.d:

```
zubahin@compute-vm-angie01:~/angie/http.d$ vim angie.conf
```

<details>
    
```
server {
        listen 80 default_server reuseport;
        reset_timedout_connection on;

       # Keepalive (для клиентов):
       keepalive_timeout 300;
       keepalive_requests 10000;
       #Таймауты:
       send_timeout 10;
       client_body_timeout 10;
       client_header_timeout 10;


       #Сокращение задержек (fail fast)
       proxy_connect_timeout 5;
       proxy_send_timeout 10;
       proxy_read_timeout 10;
       #Буфер для чтения ответа от бэкенда
       proxy_temp_file_write_size 64k;
       proxy_buffer_size 4k;
       proxy_buffers 64 4k;
       proxy_busy_buffers_size 32k;


        server_name example.com www.example.com;

        index index.php index.html index.htm;

        root /var/www/html;

        location ~ /.well-known/acme-challenge {
                allow all;
                root /var/www/html;
        }
       location / {
                try_files $uri $uri/ /index.php$is_args$args;

        # Серверное кеширование
        proxy_cache one;
        proxy_cache_valid 200 1h;
        proxy_cache_lock on;
        proxy_cache_min_uses 2;
        proxy_ignore_headers "Cache-Control" "Expires";
        proxy_cache_use_stale updating error timeout invalid_header http_500 http_502 http_504;
        proxy_cache_background_update on;

       }

        location ~ \.php$ {
                try_files $uri =404;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass wordpress:9000;
                fastcgi_index index.php;
                include fastcgi_params;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                fastcgi_param PATH_INFO $fastcgi_path_info;
        }

        location ~ /\.ht {
                deny all;
        }

        location = /favicon.ico {
                log_not_found off; access_log off;
        }
        location = /robots.txt {
                log_not_found off; access_log off; allow all;
        }
        location ~* \.(css|gif|ico|jpeg|jpg|js|png)$ {
                expires max;
                log_not_found off;
        }

# оптимизация картинок
location /img {
    add_header Vary $vary_header;
    add_header Cache-Control $cache_control;
    try_files $uri$avif_suffix $uri$webp_suffix $uri =404;
    expires max;
    log_not_found off;
}

# Для других изображений:
location ~* \.(gif|ico|jpeg|jpg|png|svg|webp|avif)$ {
    expires max;
    log_not_found off;
    add_header Vary $vary_header;
    add_header Cache-Control $cache_control;
    try_files $uri$avif_suffix $uri$webp_suffix $uri =404;
}

# Для CSS/JS:
location ~* \.(css|js)$ {
    expires max;
    log_not_found off;
}

}

