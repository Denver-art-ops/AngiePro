### Миграция с NGINX на Angie

#### Шаг 1:
Создали ВМ в облаке.

#### Шаг 2: Копируем файлы из ДЗ по SFTP в директорию:

/usr/share/angie/html/site/static_site/

#### Шаг 3: Правим конфигурацию:

Вносим изменения в конфигурацию http.d\default.conf: 
```
server {
    listen       80;
    server_name  localhost;

    #access_log  /var/log/angie/host.access.log  main;

    location / {
        root   /usr/share/angie/html;
        index  index.html index.htm;
    }

    location /status/ {
        api     /status/;
        allow   127.0.0.1;
        deny    all;
    }

     location /site/ {
        alias /usr/share/angie/html/site/static_site/;
        index  index.html index.htm;

   }
      location ~*\.(gif|jpg|jpeg|jiff)$ {
    root /usr/share/angie/html/site/static_site/images/;
    }
#error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/angie/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}
# deny access to .htaccess files, if Apache's document root
    # concurs with angie's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```
