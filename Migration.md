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
-rw-rw-rw- 1 zubahin zubahin 40960 Nov 25 18:46 nginx_conf.tar-252831-0242cd
```
Копируем содержимое в файл /etc/nginx/nginx.conf:
```
zubahin@compute-vm-angie01:~$ cp -i /home/zubahin/nginx_conf.tar-252831-0242cd /etc/nginx/nginx.conf
cp: unwritable '/etc/nginx/nginx.conf' (mode 0644, rw-r--r--); try anyway? 
zubahin@compute-vm-angie01:~$ 
zubahin@compute-vm-angie01:~$ 
zubahin@compute-vm-angie01:~$ sudo cp -i /home/zubahin/nginx_conf.tar-252831-0242cd /etc/nginx/nginx.conf
cp: overwrite '/etc/nginx/nginx.conf'? y
```
Проверяем содержимое файла конфигурации nginx:

<details>
```
zubahin@compute-vm-angie01:~$ cat /etc/nginx/nginx.conf
nginx/0000755000000000000000000000000014671024202010675 5ustar  rootrootnginx/mime.types0000644000000000000000000000756514435431402012731 0ustar  rootroot
types {
    text/html                             html htm shtml;
    text/css                              css;
    text/xml                              xml;
    image/gif                             gif;
    image/jpeg                            jpeg jpg;
    application/javascript                js;
    application/atom+xml                  atom;
    application/rss+xml                   rss;

    text/mathml                           mml;
    text/plain                            txt;
    text/vnd.sun.j2me.app-descriptor      jad;
    text/vnd.wap.wml                      wml;
    text/x-component                      htc;

    image/png                             png;
    image/tiff                            tif tiff;
    image/vnd.wap.wbmp                    wbmp;
    image/x-icon                          ico;
    image/x-jng                           jng;
    image/x-ms-bmp                        bmp;
    image/svg+xml                         svg svgz;
    image/webp                            webp;

    application/font-woff                 woff;
    application/java-archive              jar war ear;
    application/json                      json;
    application/mac-binhex40              hqx;
    application/msword                    doc;
    application/pdf                       pdf;
    application/postscript                ps eps ai;
    application/rtf                       rtf;
    application/vnd.apple.mpegurl         m3u8;
    application/vnd.ms-excel              xls;
    application/vnd.ms-fontobject         eot;
    application/vnd.ms-powerpoint         ppt;
    application/vnd.wap.wmlc              wmlc;
    application/vnd.google-earth.kml+xml  kml;
    application/vnd.google-earth.kmz      kmz;
    application/x-7z-compressed           7z;
    application/x-cocoa                   cco;
    application/x-java-archive-diff       jardiff;
    application/x-java-jnlp-file          jnlp;
    application/x-makeself                run;
    application/x-perl                    pl pm;
    application/x-pilot                   prc pdb;
    application/x-rar-compressed          rar;
    application/x-redhat-package-manager  rpm;
    application/x-sea                     sea;
    application/x-shockwave-flash         swf;
    application/x-stuffit                 sit;
    application/x-tcl                     tcl tk;
    application/x-x509-ca-cert            der pem crt;
    application/x-xpinstall               xpi;
    application/xhtml+xml                 xhtml;
    application/xspf+xml                  xspf;
    application/zip                       zip;

    application/octet-stream              bin exe dll;
    application/octet-stream              deb;
    application/octet-stream              dmg;
    application/octet-stream              iso img;
    application/octet-stream              msi msp msm;

    application/vnd.openxmlformats-officedocument.wordprocessingml.document    docx;
    application/vnd.openxmlformats-officedocument.spreadsheetml.sheet          xlsx;
    application/vnd.openxmlformats-officedocument.presentationml.presentation  pptx;

    audio/midi                            mid midi kar;
    audio/mpeg                            mp3;
    audio/ogg                             ogg;
    audio/x-m4a                           m4a;
    audio/x-realaudio                     ra;

    video/3gpp                            3gpp 3gp;
    video/mp2t                            ts;
    video/mp4                             mp4;
    video/mpeg                            mpeg mpg;
    video/quicktime                       mov;
    video/webm                            webm;
    video/x-flv                           flv;
    video/x-m4v                           m4v;
    video/x-mng                           mng;
    video/x-ms-asf                        asx asf;
    video/x-ms-wmv                        wmv;
    video/x-msvideo                       avi;
}
nginx/sites-enabled/0000755000000000000000000000000014671023503013417 5ustar  rootrootnginx/sites-enabled/default0000777000000000000000000000000014502570050023351 2/etc/nginx/sites-available/defaultustar  rootrootnginx/fastcgi_params0000644000000000000000000000203714435431402013607 0ustar  rootroot
fastcgi_param  QUERY_STRING       $query_string;
fastcgi_param  REQUEST_METHOD     $request_method;
fastcgi_param  CONTENT_TYPE       $content_type;
fastcgi_param  CONTENT_LENGTH     $content_length;

fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
fastcgi_param  REQUEST_URI        $request_uri;
fastcgi_param  DOCUMENT_URI       $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  REQUEST_SCHEME     $scheme;
fastcgi_param  HTTPS              $https if_not_empty;

fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;

fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  REMOTE_USER        $remote_user;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;

# PHP only, required if PHP was built with --enable-force-cgi-redirect
fastcgi_param  REDIRECT_STATUS    200;
nginx/scgi_params0000644000000000000000000000117414435431402013115 0ustar  rootroot
scgi_param  REQUEST_METHOD     $request_method;
scgi_param  REQUEST_URI        $request_uri;
scgi_param  QUERY_STRING       $query_string;
scgi_param  CONTENT_TYPE       $content_type;

scgi_param  DOCUMENT_URI       $document_uri;
scgi_param  DOCUMENT_ROOT      $document_root;
scgi_param  SCGI               1;
scgi_param  SERVER_PROTOCOL    $server_protocol;
scgi_param  REQUEST_SCHEME     $scheme;
scgi_param  HTTPS              $https if_not_empty;

scgi_param  REMOTE_ADDR        $remote_addr;
scgi_param  REMOTE_PORT        $remote_port;
scgi_param  SERVER_PORT        $server_port;
scgi_param  SERVER_NAME        $server_name;
nginx/koi-win0000644000000000000000000000425714633077114012215 0ustar  rootroot
charset_map  koi8-r  windows-1251 {

    80  88 ; # euro

    95  95 ; # bullet

    9A  A0 ; # &nbsp;

    9E  B7 ; # &middot;

    A3  B8 ; # small yo
    A4  BA ; # small Ukrainian ye

    A6  B3 ; # small Ukrainian i
    A7  BF ; # small Ukrainian yi

    AD  B4 ; # small Ukrainian soft g
    AE  A2 ; # small Byelorussian short u

    B0  B0 ; # &deg;

    B3  A8 ; # capital YO
    B4  AA ; # capital Ukrainian YE

    B6  B2 ; # capital Ukrainian I
    B7  AF ; # capital Ukrainian YI

    B9  B9 ; # numero sign

    BD  A5 ; # capital Ukrainian soft G
    BE  A1 ; # capital Byelorussian short U

    BF  A9 ; # (C)

    C0  FE ; # small yu
    C1  E0 ; # small a
    C2  E1 ; # small b
    C3  F6 ; # small ts
    C4  E4 ; # small d
    C5  E5 ; # small ye
    C6  F4 ; # small f
    C7  E3 ; # small g
    C8  F5 ; # small kh
    C9  E8 ; # small i
    CA  E9 ; # small j
    CB  EA ; # small k
    CC  EB ; # small l
    CD  EC ; # small m
    CE  ED ; # small n
    CF  EE ; # small o

    D0  EF ; # small p
    D1  FF ; # small ya
    D2  F0 ; # small r
    D3  F1 ; # small s
    D4  F2 ; # small t
    D5  F3 ; # small u
    D6  E6 ; # small zh
    D7  E2 ; # small v
    D8  FC ; # small soft sign
    D9  FB ; # small y
    DA  E7 ; # small z
    DB  F8 ; # small sh
    DC  FD ; # small e
    DD  F9 ; # small shch
    DE  F7 ; # small ch
    DF  FA ; # small hard sign

    E0  DE ; # capital YU
    E1  C0 ; # capital A
    E2  C1 ; # capital B
    E3  D6 ; # capital TS
    E4  C4 ; # capital D
    E5  C5 ; # capital YE
    E6  D4 ; # capital F
    E7  C3 ; # capital G
    E8  D5 ; # capital KH
    E9  C8 ; # capital I
    EA  C9 ; # capital J
    EB  CA ; # capital K
    EC  CB ; # capital L
    ED  CC ; # capital M
    EE  CD ; # capital N
    EF  CE ; # capital O

    F0  CF ; # capital P
    F1  DF ; # capital YA
    F2  D0 ; # capital R
    F3  D1 ; # capital S
    F4  D2 ; # capital T
    F5  D3 ; # capital U
    F6  C6 ; # capital ZH
    F7  C2 ; # capital V
    F8  DC ; # capital soft sign
    F9  DB ; # capital Y
    FA  C7 ; # capital Z
    FB  D8 ; # capital SH
    FC  DD ; # capital E
    FD  D9 ; # capital SHCH
    FE  D7 ; # capital CH
    FF  DA ; # capital hard sign
}
nginx/snippets/0000755000000000000000000000000014506577032012555 5ustar  rootrootnginx/snippets/snakeoil.conf0000644000000000000000000000033114435431402015215 0ustar  rootroot# Self signed certificates generated by the ssl-cert package
# Don't use them in a production server!

ssl_certificate /etc/ssl/certs/ssl-cert-snakeoil.pem;
ssl_certificate_key /etc/ssl/private/ssl-cert-snakeoil.key;
nginx/snippets/fastcgi-php.conf0000644000000000000000000000064714435431402015627 0ustar  rootroot# regex to split $uri to $fastcgi_script_name and $fastcgi_path
fastcgi_split_path_info ^(.+?\.php)(/.*)$;

# Check that the PHP script exists before passing it
try_files $fastcgi_script_name =404;

# Bypass the fact that try_files resets $fastcgi_path_info
# see: http://trac.nginx.org/nginx/ticket/321
set $path_info $fastcgi_path_info;
fastcgi_param PATH_INFO $path_info;

fastcgi_index index.php;
include fastcgi.conf;
nginx/nginx.conf0000644000000000000000000000453114671024167012704 0ustar  rootrootuser www-data;
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
nginx/static.conf0000644000000000000000000000011614671024117013036 0ustar  rootrootadd_header Cache-Control "max-age=31536000, public, no-transform, immutable";
nginx/html/0000755000000000000000000000000014507252436011653 5ustar  rootrootnginx/html/50x.html0000644000000000000000000000076114507252434013157 0ustar  rootroot<!DOCTYPE html>
<html>
<head>
<title>Error</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>An error occurred.</h1>
<p>Sorry, the page you are looking for is currently unavailable.<br/>
Please try again later.</p>
<p>If you are the system administrator of this resource then you should check
the error log for details.</p>
<p><em>Faithfully yours, nginx.</em></p>
</body>
</html>
nginx/html/index.html0000644000000000000000000000114714507252434013651 0ustar  rootroot<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
nginx/proxy_params0000644000000000000000000000026414435431402013350 0ustar  rootrootproxy_set_header Host $http_host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
nginx/static-avif.conf0000644000000000000000000000017614671024050013763 0ustar  rootrootadd_header Vary $vary_header;
add_header Cache-Control $cache_control;
try_files $uri$avif_suffix $uri$webp_suffix $uri =404;
nginx/win-utf0000644000000000000000000000703214633077114012223 0ustar  rootroot
# This map is not a full windows-1251 <> utf8 map: it does not
# contain Serbian and Macedonian letters.  If you need a full map,
# use contrib/unicode2nginx/win-utf map instead.

charset_map  windows-1251  utf-8 {

    82  E2809A ; # single low-9 quotation mark

    84  E2809E ; # double low-9 quotation mark
    85  E280A6 ; # ellipsis
    86  E280A0 ; # dagger
    87  E280A1 ; # double dagger
    88  E282AC ; # euro
    89  E280B0 ; # per mille

    91  E28098 ; # left single quotation mark
    92  E28099 ; # right single quotation mark
    93  E2809C ; # left double quotation mark
    94  E2809D ; # right double quotation mark
    95  E280A2 ; # bullet
    96  E28093 ; # en dash
    97  E28094 ; # em dash

    99  E284A2 ; # trade mark sign

    A0  C2A0 ;   # &nbsp;
    A1  D18E ;   # capital Byelorussian short U
    A2  D19E ;   # small Byelorussian short u

    A4  C2A4 ;   # currency sign
    A5  D290 ;   # capital Ukrainian soft G
    A6  C2A6 ;   # borken bar
    A7  C2A7 ;   # section sign
    A8  D081 ;   # capital YO
    A9  C2A9 ;   # (C)
    AA  D084 ;   # capital Ukrainian YE
    AB  C2AB ;   # left-pointing double angle quotation mark
    AC  C2AC ;   # not sign
    AD  C2AD ;   # soft hypen
    AE  C2AE ;   # (R)
    AF  D087 ;   # capital Ukrainian YI

    B0  C2B0 ;   # &deg;
    B1  C2B1 ;   # plus-minus sign
    B2  D086 ;   # capital Ukrainian I
    B3  D196 ;   # small Ukrainian i
    B4  D291 ;   # small Ukrainian soft g
    B5  C2B5 ;   # micro sign
    B6  C2B6 ;   # pilcrow sign
    B7  C2B7 ;   # &middot;
    B8  D191 ;   # small yo
    B9  E28496 ; # numero sign
    BA  D194 ;   # small Ukrainian ye
    BB  C2BB ;   # right-pointing double angle quotation mark

    BF  D197 ;   # small Ukrainian yi

    C0  D090 ;   # capital A
    C1  D091 ;   # capital B
    C2  D092 ;   # capital V
    C3  D093 ;   # capital G
    C4  D094 ;   # capital D
    C5  D095 ;   # capital YE
    C6  D096 ;   # capital ZH
    C7  D097 ;   # capital Z
    C8  D098 ;   # capital I
    C9  D099 ;   # capital J
    CA  D09A ;   # capital K
    CB  D09B ;   # capital L
    CC  D09C ;   # capital M
    CD  D09D ;   # capital N
    CE  D09E ;   # capital O
    CF  D09F ;   # capital P

    D0  D0A0 ;   # capital R
    D1  D0A1 ;   # capital S
    D2  D0A2 ;   # capital T
    D3  D0A3 ;   # capital U
    D4  D0A4 ;   # capital F
    D5  D0A5 ;   # capital KH
    D6  D0A6 ;   # capital TS
    D7  D0A7 ;   # capital CH
    D8  D0A8 ;   # capital SH
    D9  D0A9 ;   # capital SHCH
    DA  D0AA ;   # capital hard sign
    DB  D0AB ;   # capital Y
    DC  D0AC ;   # capital soft sign
    DD  D0AD ;   # capital E
    DE  D0AE ;   # capital YU
    DF  D0AF ;   # capital YA

    E0  D0B0 ;   # small a
    E1  D0B1 ;   # small b
    E2  D0B2 ;   # small v
    E3  D0B3 ;   # small g
    E4  D0B4 ;   # small d
    E5  D0B5 ;   # small ye
    E6  D0B6 ;   # small zh
    E7  D0B7 ;   # small z
    E8  D0B8 ;   # small i
    E9  D0B9 ;   # small j
    EA  D0BA ;   # small k
    EB  D0BB ;   # small l
    EC  D0BC ;   # small m
    ED  D0BD ;   # small n
    EE  D0BE ;   # small o
    EF  D0BF ;   # small p

    F0  D180 ;   # small r
    F1  D181 ;   # small s
    F2  D182 ;   # small t
    F3  D183 ;   # small u
    F4  D184 ;   # small f
    F5  D185 ;   # small kh
    F6  D186 ;   # small ts
    F7  D187 ;   # small ch
    F8  D188 ;   # small sh
    F9  D189 ;   # small shch
    FA  D18A ;   # small hard sign
    FB  D18B ;   # small y
    FC  D18C ;   # small soft sign
    FD  D18D ;   # small e
    FE  D18E ;   # small yu
    FF  D18F ;   # small ya
}
nginx/sites-available/0000755000000000000000000000000014671023451013747 5ustar  rootrootnginx/sites-available/default0000644000000000000000000000260414671023503015316 0ustar  rootrootupstream backend {
    #hash $request_uri;
    server 127.0.0.1:9000 weight=2;
    server 127.0.0.1:9001 weight=5;
    server 127.0.0.1:9002 weight=8;
    server 127.0.0.1:9003 weight=4 fail_timeout=5s;
}

server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;

        index index.html index.htm index.nginx-debian.html;

        server_name _;

        access_log /var/log/nginx/default.log;

        location / {
                #proxy_cache one;
            proxy_cache_valid 200 1h;
            proxy_cache_lock on;
            proxy_cache_use_stale updating error timeout invalid_header http_500 http_502 http_504;
            proxy_cache_background_update on;
                #proxy_pass     http://127.0.0.1:8008;
                proxy_pass      http://backend;
                #try_files $uri $uri/ =404;
        }

        location /stat {
                include         uwsgi_params;
                uwsgi_pass      127.0.0.1:81;
        }

  # Static files location
  location ~* \.(jpg|jpeg|gif|png)$ {
        include /etc/nginx/static-avif.conf;
  }

  # Static files location
  location ~* \.(ttf|eot|svg|woff|woff2|css|js|json|ico|zip|tgz|gz|rar|bz2|doc|docx|xlsx|pptx|xls|exe|pdf|ppt|txt|tar|mid|midi|wav|bmp|rtf|avi|swf|flv|mp3|mp4|fla)$ {
                expires max;
        include /etc/nginx/static.conf;
  }


        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                # With php-fpm (or other unix sockets):
                fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        #       # With php-cgi (or other tcp sockets):
        #       fastcgi_pass 127.0.0.1:9000;
        }
}
nginx/koi-utf0000644000000000000000000000542514633077114012214 0ustar  rootroot
# This map is not a full koi8-r <> utf8 map: it does not contain
# box-drawing and some other characters.  Besides this map contains
# several koi8-u and Byelorussian letters which are not in koi8-r.
# If you need a full and standard map, use contrib/unicode2nginx/koi-utf
# map instead.

charset_map  koi8-r  utf-8 {

    80  E282AC ; # euro

    95  E280A2 ; # bullet

    9A  C2A0 ;   # &nbsp;

    9E  C2B7 ;   # &middot;

    A3  D191 ;   # small yo
    A4  D194 ;   # small Ukrainian ye

    A6  D196 ;   # small Ukrainian i
    A7  D197 ;   # small Ukrainian yi

    AD  D291 ;   # small Ukrainian soft g
    AE  D19E ;   # small Byelorussian short u

    B0  C2B0 ;   # &deg;

    B3  D081 ;   # capital YO
    B4  D084 ;   # capital Ukrainian YE

    B6  D086 ;   # capital Ukrainian I
    B7  D087 ;   # capital Ukrainian YI

    B9  E28496 ; # numero sign

    BD  D290 ;   # capital Ukrainian soft G
    BE  D18E ;   # capital Byelorussian short U

    BF  C2A9 ;   # (C)

    C0  D18E ;   # small yu
    C1  D0B0 ;   # small a
    C2  D0B1 ;   # small b
    C3  D186 ;   # small ts
    C4  D0B4 ;   # small d
    C5  D0B5 ;   # small ye
    C6  D184 ;   # small f
    C7  D0B3 ;   # small g
    C8  D185 ;   # small kh
    C9  D0B8 ;   # small i
    CA  D0B9 ;   # small j
    CB  D0BA ;   # small k
    CC  D0BB ;   # small l
    CD  D0BC ;   # small m
    CE  D0BD ;   # small n
    CF  D0BE ;   # small o

    D0  D0BF ;   # small p
    D1  D18F ;   # small ya
    D2  D180 ;   # small r
    D3  D181 ;   # small s
    D4  D182 ;   # small t
    D5  D183 ;   # small u
    D6  D0B6 ;   # small zh
    D7  D0B2 ;   # small v
    D8  D18C ;   # small soft sign
    D9  D18B ;   # small y
    DA  D0B7 ;   # small z
    DB  D188 ;   # small sh
    DC  D18D ;   # small e
    DD  D189 ;   # small shch
    DE  D187 ;   # small ch
    DF  D18A ;   # small hard sign

    E0  D0AE ;   # capital YU
    E1  D090 ;   # capital A
    E2  D091 ;   # capital B
    E3  D0A6 ;   # capital TS
    E4  D094 ;   # capital D
    E5  D095 ;   # capital YE
    E6  D0A4 ;   # capital F
    E7  D093 ;   # capital G
    E8  D0A5 ;   # capital KH
    E9  D098 ;   # capital I
    EA  D099 ;   # capital J
    EB  D09A ;   # capital K
    EC  D09B ;   # capital L
    ED  D09C ;   # capital M
    EE  D09D ;   # capital N
    EF  D09E ;   # capital O

    F0  D09F ;   # capital P
    F1  D0AF ;   # capital YA
    F2  D0A0 ;   # capital R
    F3  D0A1 ;   # capital S
    F4  D0A2 ;   # capital T
    F5  D0A3 ;   # capital U
    F6  D096 ;   # capital ZH
    F7  D092 ;   # capital V
    F8  D0AC ;   # capital soft sign
    F9  D0AB ;   # capital Y
    FA  D097 ;   # capital Z
    FB  D0A8 ;   # capital SH
    FC  D0AD ;   # capital E
    FD  D0A9 ;   # capital SHCH
    FE  D0A7 ;   # capital CH
    FF  D0AA ;   # capital hard sign
}
nginx/fastcgi.conf0000644000000000000000000000214514435431402013170 0ustar  rootroot
fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
fastcgi_param  QUERY_STRING       $query_string;
fastcgi_param  REQUEST_METHOD     $request_method;
fastcgi_param  CONTENT_TYPE       $content_type;
fastcgi_param  CONTENT_LENGTH     $content_length;

fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
fastcgi_param  REQUEST_URI        $request_uri;
fastcgi_param  DOCUMENT_URI       $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  REQUEST_SCHEME     $scheme;
fastcgi_param  HTTPS              $https if_not_empty;

fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;

fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  REMOTE_USER        $remote_user;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;

# PHP only, required if PHP was built with --enable-force-cgi-redirect
fastcgi_param  REDIRECT_STATUS    200;
nginx/uwsgi_params0000644000000000000000000000123014435431402013317 0ustar  rootroot
uwsgi_param  QUERY_STRING       $query_string;
uwsgi_param  REQUEST_METHOD     $request_method;
uwsgi_param  CONTENT_TYPE       $content_type;
uwsgi_param  CONTENT_LENGTH     $content_length;

uwsgi_param  REQUEST_URI        $request_uri;
uwsgi_param  PATH_INFO          $document_uri;
uwsgi_param  DOCUMENT_ROOT      $document_root;
uwsgi_param  SERVER_PROTOCOL    $server_protocol;
uwsgi_param  REQUEST_SCHEME     $scheme;
uwsgi_param  HTTPS              $https if_not_empty;

uwsgi_param  REMOTE_ADDR        $remote_addr;
uwsgi_param  REMOTE_PORT        $remote_port;
uwsgi_param  SERVER_PORT        $server_port;
uwsgi_param  SERVER_NAME        $server_name;
zubahin@compute-vm-angie01:~$ 
```
</details>

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

