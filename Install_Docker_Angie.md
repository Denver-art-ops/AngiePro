### Установка docker:

#### Шаг 1: Установка Docker
Устанавливаем докер:
```
sudo apt install docker.io
```
Проверяем, что процесс докер запущен:
```
zubahin@compute-vm-angie01:~$ systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: enabled)
     Active: active (running) since Fri 2025-11-21 20:32:30 UTC; 6min ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 6979 (dockerd)
      Tasks: 9
     Memory: 21.1M (peak: 22.2M)
        CPU: 322ms
     CGroup: /system.slice/docker.service
             └─6979 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

#### Шаг 2: Ставим образ Angie

```
docker run --name angie -v /var/www/html:/usr/share/angie/html:ro \
 -p 8800:80 -d docker.angie.software/angie:latest
```
```
zubahin@compute-vm-angie01:~$ sudo docker run --name angie -v /var/www/html:/usr/share/angie/html:ro \
 -p 8800:80 -d docker.angie.software/angie:latest
Unable to find image 'docker.angie.software/angie:latest' locally
latest: Pulling from angie
f637881d1138: Pull complete 
a4c1f716105c: Pull complete 
2155a2bb836c: Pull complete 
d8084345baa5: Pull complete 
974a23b277e5: Pull complete 
Digest: sha256:f79d88b4971d1357a57b50b49702a8162fbb9164e5ce1e591374b188fa520a30
Status: Downloaded newer image for docker.angie.software/angie:latest
eaa279290873250c3b7202c4fbc73da2c6f73959144e051edd97e832e7d92e56
```
