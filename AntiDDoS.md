### В качестве основы возьмем конфигурацию Angie из задания с TLS где уже настроен HTTPS на тестовую страницу, но добавим в дополнение к Angie развернутому на хосте
### контейнеры с wordpress и базой данных mysql с помощью docker-compose

#### Создаем необходимые для docker-compose файлы в директории проекта:

Создаем папку проекта:
```
zubahin@compute-vm-3:~$ mkdir project
zubahin@compute-vm-3:~$ ls -la
total 60
drwxr-x--- 6 zubahin zubahin  4096 Feb  5 11:49 .
drwxr-xr-x 3 root    root     4096 Jan 23 15:09 ..
-rw------- 1 zubahin zubahin 11058 Feb  4 20:47 .bash_history
-rw-r--r-- 1 zubahin zubahin   220 Mar 31  2024 .bash_logout
-rw-r--r-- 1 zubahin zubahin  3771 Mar 31  2024 .bashrc
drwx------ 2 zubahin zubahin  4096 Jan 23 15:28 .cache
-rw-r--r-- 1 zubahin zubahin   807 Mar 31  2024 .profile
drwx------ 2 zubahin zubahin  4096 Feb  5 06:30 .ssh
-rw------- 1 zubahin zubahin  9607 Feb  4 20:47 .viminfo
drwxrwxr-x 2 zubahin zubahin  4096 Feb  5 11:49 project
drwx------ 3 zubahin zubahin  4096 Jan 23 17:36 snap
zubahin@compute-vm-3:~$ cd project/
zubahin@compute-vm-3:~/project$ 
```

В папке проекта создаем YAML -файл docker-compose.yml
```
sudo vim docker-compose.yml
```

```
version: '3.8'

services:
  wordpress-db:
    image: mysql:8.0
    container_name: wordpress-db
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: ${DB_PASSWORD:-Gh56Tyfg091df}
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD:-Gh56Tyfg091df_root}
    volumes:
      - wordpress_db_data:/var/lib/mysql
    networks:
      - wordpress_network
    command: 
      - --default-authentication-plugin=mysql_native_password
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci

  wordpress-app:
    image: wordpress:latest
    container_name: wordpress-app
    restart: unless-stopped
    ports:
      - "127.0.0.1:8080:80"
    environment:
      WORDPRESS_DB_HOST: wordpress-db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: ${DB_PASSWORD:-Gh56Tyfg091df}
      WORDPRESS_DB_NAME: wordpress
      # УДАЛИТЕ WORDPRESS_CONFIG_EXTRA отсюда
    volumes:
      - wordpress_data:/var/www/html
      - ./uploads.ini:/usr/local/etc/php/conf.d/uploads.ini
    networks:
      - wordpress_network
    depends_on:
      - wordpress-db

networks:
  wordpress_network:
    driver: bridge

volumes:
  wordpress_db_data:
    name: wordpress_db_data
  wordpress_data:
    name: wordpress_data

```

Создаем файл uploads.ini для увеличения лимитов загрузки файлов:

```
sudo vim uploads.ini

file_uploads = On
memory_limit = 256M
upload_max_filesize = 64M
post_max_size = 64M
max_execution_time = 300
```

### Шаг 2.  Меняем конфигурацию Angie

Меняем содержимое конфигурации в /etc/angie/http.d/wordpress.conf

```
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    http2 on;
    
    server_name denis-otus.mtdlb.ru www.denis-otus.mtdlb.ru;
    
    # SSL конфигурация
    ssl_certificate /etc/letsencrypt/live/denis-otus.mtdlb.ru/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/denis-otus.mtdlb.ru/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256';
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;
    ssl_dhparam /etc/ssl/certs/dhparam.pem;
    
    # Security headers
    add_header Strict-Transport-Security "max-age=63072000" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Permissions-Policy "geolocation=(), microphone=(), camera=(), payment=()" always;
    
    # Проксирование на WordPress
    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $host;
        
        # Критически важные настройки для WordPress
        proxy_redirect http://127.0.0.1:8080/ https://$host/;
        proxy_redirect http://$host:8080/ https://$host/;
        proxy_redirect http://$host/ https://$host/;
        
        # Оптимизации
        proxy_buffering on;
        proxy_buffer_size 128k;
        proxy_buffers 256 16k;
        proxy_busy_buffers_size 256k;
        proxy_temp_file_write_size 256k;
        proxy_connect_timeout 90;
        proxy_send_timeout 90;
        proxy_read_timeout 90;
    }
    
    # Кэширование статических файлов
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot|webp)$ {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        expires 1y;
        add_header Cache-Control "public, immutable";
        add_header X-Content-Type-Options "nosniff";
    }
    
    # Let's Encrypt
    location ^~ /.well-known/acme-challenge/ {
        root /var/www/denis-otus.mtdlb.ru/html;
        allow all;
        try_files $uri =404;
        access_log off;
    }
    
    # Блокировка нежелательных запросов
    location ~* /(wp-config\.php|xmlrpc\.php|readme\.html|license\.txt) {
        deny all;
        return 404;
    }
    
    # Логирование
    access_log /var/log/angie/wordpress.access.log;
    error_log /var/log/angie/wordpress.error.log warn;
}

# Перенаправление HTTP -> HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name denis-otus.mtdlb.ru www.denis-otus.mtdlb.ru;
    
    location ^~ /.well-known/acme-challenge/ {
        root /var/www/denis-otus.mtdlb.ru/html;
        allow all;
        try_files $uri =404;
    }
    
    location / {
        return 301 https://$server_name$request_uri;
    }
}
```

Смотрим две конфигурации:
```
zubahin@compute-vm-3:/etc/angie/http.d$ ls -la
total 32
drwxr-xr-x 4 root root 4096 Feb  5 12:05 .
drwxr-xr-x 4 root root 4096 Jan 28 11:17 ..
drwxr-xr-x 2 root root 4096 Jan 28 11:30 arc
-rw-r--r-- 1 root root 5023 Jan 29 13:11 denis-otus.mtdlb.ru-ssl2.conf
drwxr-xr-x 2 root root 4096 Jan 27 19:21 sites-enabled
-rw-r--r-- 1 root root 5093 Feb  5 12:05 wordpress.conf
```
Старую конфигурацию переносим в архив (arc):
```
sudo mv /etc/angie/http.d/denis-otus.mtdlb.ru-ssl2.conf /etc/angie/http.d/arc
```

### Шаг 3.  Создаем .env файл в папке проекта, устанавливаем docker-compose  и запускаем docker-compose:

```
zubahin@compute-vm-3:~/project$ sudo vim .env

DB_PASSWORD=Gh56Tyfg091df
DB_ROOT_PASSWORD=Gh56Tyfg091df_root

```

Обновляем индексы репозиториев,устанавливаем Docker и docker-compose:
```
sudo apt-get update

sudo apt install docker.io

sudo apt update

sudo apt install docker-compose

```

Запускаем docker-compose:

```
sudo docker-compose up -d
```

### Шаг 4.  Проверяем результаты:

```
zubahin@compute-vm-3:~/project$ sudo docker ps -a
CONTAINER ID   IMAGE              COMMAND                  CREATED          STATUS          PORTS                    NAMES
238a66cc4c55   wordpress:latest   "docker-entrypoint.s…"   13 minutes ago   Up 13 minutes   127.0.0.1:8080->80/tcp   wordpress-app
5f462c30f52a   mysql:8.0          "docker-entrypoint.s…"   13 minutes ago   Up 13 minutes   3306/tcp, 33060/tcp      wordpress-db
```
И идем донастраивать Wordpress:



### Таким образом настроены:

1 Рабочий WordPress в Docker
2 MySQL в отдельном контейнере
3 Angie на хосте как HTTPS прокси с SSL
4 Автоматические редиректы HTTP→HTTPS
5 Безопасные заголовки
6 Кэширование статики


### Шаг 5.  Определяем потенциально уязвимые location (API, админка, в меньшей степени статические файлы) и добавляем ограничения на подключения (авторизация, limit conn и т.п):

<details>
  
```
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    http2 on;

    server_name denis-otus.mtdlb.ru www.denis-otus.mtdlb.ru;

    # 1. SSL НАСТРОЙКИ

    # Пути к сертификатам
    ssl_certificate /etc/letsencrypt/live/denis-otus.mtdlb.ru/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/denis-otus.mtdlb.ru/privkey.pem;

    # Современные протоколы и шифры
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256';
    ssl_prefer_server_ciphers off;

    # Оптимизация SSL сессий
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;

    # DH параметры
    ssl_dhparam /etc/ssl/certs/dhparam.pem;

     # 2. БАЗОВЫЕ ЛИМИТЫ

    # Ограничение размера запросов
    client_max_body_size 10M;
    client_body_buffer_size 128k;
    client_header_buffer_size 1k;
    large_client_header_buffers 4 8k;
# 3. ЗАЩИТА ОТ МЕДЛЕННЫХ СОЕДИНЕНИЙ

    # Ограничение времени чтения тела запроса
    client_body_timeout 5s;  # Максимальное время для передачи тела запроса от клиента. Если клиент не успевает за 5 секунд - соединение разрывается.

    # Ограничение времени чтения заголовков
    client_header_timeout 5s;   # Максимальное время для получения заголовков от клиента. Защита от Slowloris-атак.

    # Ограничение времени передачи ответа клиенту
    send_timeout 5s; #  Максимальное время для отправки ответа клиенту. Если клиент не читает - соединение закрывается.

    # Keepalive настройки для предотвращения удержания соединений
    keepalive_timeout 15s; # Время удержания keepalive соединения. Короткий таймаут освобождает соединения быстрее.
    keepalive_requests 100;  # Задает максимальное число запросов, которые можно сделать по одному keep-alive соединению. После того, как сделано максимальное число запросов, соединение закрывается. ((по умолчанию 1000)

    # Максимальное количество соединений с одного IP
    limit_conn conn_limit_per_ip 100; # Максимально 100 одновременных соединений с одного IP.

    # Защита от Slowloris атак
    limit_conn slow_conn 1000;  # Глобальное ограничение на медленные соединения.

     # 4. RATE LIMITING

    # Общий rate limiting
    limit_req zone=req_limit_per_ip burst=50 nodelay;
    limit_req_status 429;

    # 5. SECURITY HEADERS

    # HSTS - принудительное использование HTTPS
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;

    # Защита от MIME sniffing
    add_header X-Content-Type-Options "nosniff" always;
# Защита от clickjacking
    add_header X-Frame-Options "SAMEORIGIN" always;

    # XSS защита (устарело,для старых браузеров)
    add_header X-XSS-Protection "1; mode=block" always;

    # Referrer policy
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    # Permissions policy
    add_header Permissions-Policy "geolocation=(), microphone=(), camera=(), payment=()" always;

    # CSP - политика безопасности контента
    add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; font-src 'self'; connect-src 'self';" always;

    # 6. НАСТРОЙКИ КЭШИРОВАНИЯ

    # Ключи кэширования
    proxy_cache_key "$scheme$request_method$host$request_uri";
    proxy_cache_valid 200 302 10m;
    proxy_cache_valid 404 1m;

    # Байпас кэша для определенных условий
    proxy_cache_bypass $cookie_nocache $arg_nocache;
    proxy_no_cache $cookie_nocache $arg_nocache;

    # 7. ПРОКСИРОВАНИЕ

    # Основной location
    location / {
        proxy_pass http://127.0.0.1:8080;

        # Заголовки для правильной работы приложений
proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Port $server_port;

        proxy_redirect http://127.0.0.1:8080/ https://$host/;
        proxy_redirect http://$host/ https://$host/;

        # Оптимизации
        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
        proxy_busy_buffers_size 8k;

        # Таймауты
        proxy_connect_timeout 5s;
        proxy_send_timeout 10s;
        proxy_read_timeout 10s;

        # Включение кэширования
#        proxy_cache proxy_cache;
#        proxy_cache_lock on;
#        proxy_cache_lock_timeout 5s;
#        proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;

        # Заголовки кэша
#        add_header X-Cache-Status $upstream_cache_status;

        # Rate limiting для динамического контента
        limit_req zone=req_limit_per_ip burst=30 delay=20;
    }
# 8. СТАТИЧЕСКИЕ ФАЙЛЫ

    location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot|webp|avif)$ {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Агрессивное кэширование статики
        expires 1y;
        add_header Cache-Control "public, immutable";
        add_header X-Content-Type-Options "nosniff";

        # Кэширование на стороне Angie
        proxy_cache static_cache;
        proxy_cache_valid 200 302 365d;
        proxy_cache_valid 404 1d;
        proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;

        # Разрешаем больше параллельных запросов к статике
        limit_req zone=static_limit burst=200 nodelay;

        # Более быстрые таймауты для статики
        proxy_connect_timeout 3s;
        proxy_read_timeout 5s;
    }

    # 9. ЗАЩИЩЕННЫЕ LOCATION

    # Защита входа в систему
    location = /wp-login.php {   # на нее перенаправляется админка со стороны wordpress
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
    # Строгий rate limiting для логина
        limit_req zone=login_limit burst=3 nodelay;
        limit_req_status 429;

        # HTTP базовая авторизация
        auth_basic "Restricted Area";
        auth_basic_user_file /etc/angie/htpasswd;

        # Ограничение по IP   # для тестов отключено,можно включать
        #allow 127.0.0.1;
        #allow 158.160.82.102; # Ваш IP
        #deny all;

        # Логирование попыток доступа
        access_log /var/log/angie/auth.log;

        # Отключаем кэш для защищенных зон
        proxy_no_cache 1;
        proxy_cache_bypass 1;
    }

    #Админка:
    location ~ ^/wp-admin/ {
    proxy_pass http://127.0.0.1:8080;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;

    # Снижаем ограничения для админки
    limit_req zone=req_limit_per_ip burst=50 nodelay;

    # Отключаем кэширование
    proxy_no_cache 1;
    proxy_cache_bypass 1;
# Увеличиваем таймауты
    proxy_connect_timeout 30s;
    proxy_read_timeout 60s;
}

    # API endpoint protection
    location ~ ^/api/ {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Rate limiting для API
        limit_req zone=api_limit burst=20 nodelay;

        # Заголовки для API
        add_header X-API-Version "1.0" always;
        add_header X-RateLimit-Limit "10" always;
        add_header X-RateLimit-Remaining "9" always;

        # Кэширование API ответов
        proxy_cache proxy_cache;
        proxy_cache_valid 200 10s;
        proxy_cache_methods GET HEAD;
        proxy_cache_key "$scheme$request_method$host$request_uri$is_args$args";
    }

    # 10. ЗАЩИТА ОТ БОТОВ И СКАНЕРОВ

    # Блокировка известных сканеров
    if ($http_user_agent ~* (nmap|nikto|sqlmap|w3af|acunetix|openvas|nessus|metasploit|dirbuster|wapiti|burpsuite|hydra)) {
        return 444;
    }
# Блокировка ботов
    if ($http_user_agent ~* (bot|crawl|spider|scraper|python|java|wget|libwww)) {
        return 444;
    }

    # Блокировка пустых User-Agent
    if ($http_user_agent = "") {
        return 444;
    }

    # Блокировка нестандартных методов
    if ($request_method !~ ^(GET|HEAD|POST|PUT|DELETE|PATCH|OPTIONS)$) {
        return 444;
    }

    # Блокировка чувствительных файлов
    location ~* \.(log|sql|conf|config|yml|yaml|env|ini|bak|backup|tar|gz|zip|swp)$ {
        deny all;
        return 404;
    }

    # Блокировка скрытых файлов
    location ~ /\. {
        deny all;
        return 404;
    }

    # 11. LET'S ENCRYPT

    location ^~ /.well-known/acme-challenge/ {
        root /var/www/denis-otus.mtdlb.ru/html;
        allow all;
        try_files $uri =404;
        access_log off;
}

    # 12. КАСТОМНЫЕ ОШИБКИ

    error_page 429 =429 /429.html;
    error_page 444 =444 /444.html;
    error_page 403 =403 /403.html;
    error_page 404 =404 /404.html;

    location = /429.html {
        internal;
        return 429 '{"error": "Too Many Requests", "message": "Rate limit exceeded. Please try again later."}';
    }

    location = /444.html {
        internal;
        return 444;
    }

    location = /403.html {
        return 403 '{"error": "Forbidden", "message": "Access denied"}';
    }

    # 13. ЛОГИРОВАНИЕ

    # Основной лог (использует формат security из основного конфига)
    access_log /var/log/angie/access.log security;
    error_log /var/log/angie/error.log warn;

    # Отдельные логи для мониторинга
    access_log /var/log/angie/security.log security if=$limit_req_status;
    access_log /var/log/angie/slow.log security if=$request_time>5;
}
14. HTTP REDIRECT

server {
    listen 80;
    listen [::]:80;
    server_name denis-otus.mtdlb.ru www.denis-otus.mtdlb.ru;

    # Let's Encrypt
    location ^~ /.well-known/acme-challenge/ {
        root /var/www/denis-otus.mtdlb.ru/html;
        allow all;
        try_files $uri =404;
        access_log off;
    }

    # Редирект на HTTPS
    location / {
        return 301 https://$server_name$request_uri;
    }
}

   
```
</details>

### Шаг 6. Создаем файлы для авторизации и кеша и заодно установим apache2-utils:

```
# Создаем директории для кэша (уже были созданы ранее)
sudo mkdir -p /var/cache/angie
sudo chown -R angie:angie /var/cache/angie

# Создаем файл с паролями для HTTP авторизации
sudo apt-get update
sudo apt-get install apache2-utils

sudo htpasswd -c /etc/angie/htpasswd admin
# Вводим пароль при запросе  (например Otus2026!)

# Можно добавить дополнительных пользователей
sudo htpasswd /etc/angie/htpasswd user1
sudo htpasswd /etc/angie/htpasswd user2

# Настраиваем права на файл с паролями
sudo chown angie:angie /etc/angie/htpasswd
sudo chmod 640 /etc/angie/htpasswd

# Директории для статики Let's Encrypt (уже созданы были ранее):
sudo mkdir -p /var/www/denis-otus.mtdlb.ru/html/.well-known/acme-challenge
sudo chown -R angie:angie /var/www/denis-otus.mtdlb.ru/html

```

###  Шаг 7 Проверяем ограничения на rate-limit и пароль на вход в админку

Сначала заходим на саму страничку и видим, что все работает:


Пробуем зайти в админку: https://denis-otus.mtdlb.ru/wp-admin/


Видим, как срабатывает ограничение (вход по логину/паролю)


Пробуем несколько раз подключиться -попадаем на rate-limit:


Проверяем ограничение по IP (в конфиге раскомментируем строчку):

Включим  подключения только с российских IP. Используем модуль geoip (расскомментируем модуль в конфигурации)


