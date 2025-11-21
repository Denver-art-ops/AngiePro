Проверка установки Angie (СПО версия) и дополнительных модулей к нему:

zubahin@compute-vm-angie01:~$ sudo apt-get install -y angie
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  angie
0 upgraded, 1 newly installed, 0 to remove and 13 not upgraded.
Need to get 1151 kB of archives.
After this operation, 4125 kB of additional disk space will be used.
Get:1 https://download.angie.software/angie/ubuntu/24.04 noble/main amd64 angie amd64 1.10.3-1~noble [1151 kB]
Fetched 1151 kB in 0s (5022 kB/s)
Selecting previously unselected package angie.
(Reading database ... 106318 files and directories currently installed.)
Preparing to unpack .../angie_1.10.3-1~noble_amd64.deb ...
Unpacking angie (1.10.3-1~noble) ...
Setting up angie (1.10.3-1~noble) ...
Created symlink /etc/systemd/system/multi-user.target.wants/angie.service → /usr/lib/systemd/system/angie.service.
----------------------------------------------------------------------

Thanks for using Angie!

Please find the official documentation for Angie here:
* https://en.angie.software/angie/docs/

----------------------------------------------------------------------
Processing triggers for man-db (2.12.0-4build2) ...
Scanning processes...                                                                                                                                                 
Scanning linux images...                                                                                                                                              

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
zubahin@compute-vm-angie01:~$ angie -v
Angie version: Angie/1.10.3
zubahin@compute-vm-angie01:~$ angie -V
Angie version: Angie/1.10.3
nginx version: nginx/1.27.5
built on Thu, 13 Nov 2025 10:52:28 GMT
built with OpenSSL 3.0.13 30 Jan 2024
TLS SNI support enabled
configure arguments: --prefix=/etc/angie --conf-path=/etc/angie/angie.conf --error-log-path=/var/log/angie/error.log --http-log-path=/var/log/angie/access.log --lock-path=/run/angie.lock --modules-path=/usr/lib/angie/modules --pid-path=/run/angie.pid --sbin-path=/usr/sbin/angie --http-acme-client-path=/var/lib/angie/acme --http-client-body-temp-path=/var/cache/angie/client_temp --http-fastcgi-temp-path=/var/cache/angie/fastcgi_temp --http-proxy-temp-path=/var/cache/angie/proxy_temp --http-scgi-temp-path=/var/cache/angie/scgi_temp --http-uwsgi-temp-path=/var/cache/angie/uwsgi_temp --user=angie --group=angie --with-file-aio --with-http_acme_module --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-http_v3_module --with-mail --with-mail_ssl_module --with-stream --with-stream_acme_module --with-stream_mqtt_preread_module --with-stream_rdp_preread_module --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-threads --feature-cache=../angie-feature-cache --with-ld-opt='-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro -Wl,-z,now'
zubahin@compute-vm-angie01:~$ 
