Домашнее задание к занятию 2 «Кластеризация и балансировка нагрузки» Тихун Вадим 
=
---------
Задание 1
=
Запустите два simple python сервера на своей виртуальной машине на разных портах
Установите и настройте HAProxy, воспользуйтесь материалами к лекции по ссылке
Настройте балансировку Round-robin на 4 уровне.
На проверку направьте конфигурационный файл haproxy, скриншоты, 
где видно перенаправление запросов на разные серверы при обращении к HAProxy.

----------
Решение 1
=

1 Создал 2 вм как на видео и 2 дерриктории 
mkdir http1
mkdir http2

2 Установил haproxy

```
sudo apt-get install haproxy
```
создал 2 сервера python 

![8888](https://github.com/sailent9/-/assets/130309754/0b211487-fa5f-467b-b5d0-c606e0155a22)
![9999](https://github.com/sailent9/-/assets/130309754/3b919358-0fd2-4b07-9176-9561f458177d)

Из материалов к лекции выполнил настройку конфигурационного файла для балансировки на L4 уровне по модели OSI
```
global
	log /dev/log	local0
	log /dev/log	local1 notice
	chroot /var/lib/haproxy
	stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
	stats timeout 30s
	user haproxy
	group haproxy
	daemon

	# Default SSL material locations
	ca-base /etc/ssl/certs
	crt-base /etc/ssl/private

	# See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
	log	global
	mode	http
	option	httplog
	option	dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
	errorfile 400 /etc/haproxy/errors/400.http
	errorfile 403 /etc/haproxy/errors/403.http
	errorfile 408 /etc/haproxy/errors/408.http
	errorfile 500 /etc/haproxy/errors/500.http
	errorfile 502 /etc/haproxy/errors/502.http
	errorfile 503 /etc/haproxy/errors/503.http
	errorfile 504 /etc/haproxy/errors/504.http

listen stats  # веб-страница со статистикой
        bind                    :888
        mode                    http
        stats                   enable
        stats uri               /stats
        stats refresh           5s
        stats realm             Haproxy\ Statistics

frontend example  # секция фронтенд
        mode http
        bind :8088
        #default_backend web_servers
	acl ACL_example.com hdr(host) -i example.com
	use_backend web_servers if ACL_example.com

backend web_servers    # секция бэкенд
        mode http
        balance roundrobin
        option httpchk
        http-check send meth GET uri /index.html
        server s1 127.0.0.1:8888 check
        server s2 127.0.0.1:9999 check


listen web_tcp

	bind :1325

	server s1 127.0.0.1:8888 check inter 3s
	server s2 127.0.0.1:9999 check inter 3s
```
проверил в браузере что все работает 
![stats](https://github.com/sailent9/-/assets/130309754/8b3984e2-7b65-4a49-b57e-c07f4aaf1e93)








































Задание 2 
=
Запустите три simple python сервера на своей виртуальной машине на разных портах
Настройте балансировку Weighted Round Robin на 7 уровне, чтобы первый сервер имел вес 2, второй - 3, а третий - 4
HAproxy должен балансировать только тот http-трафик, который адресован домену example.local
На проверку направьте конфигурационный файл haproxy, скриншоты, 
где видно перенаправление запросов на разные серверы при обращении к HAProxy c использованием домена example.local и без него.

Решение 2
=
создал 3-ю диркторию http3, в ней создал файл index.html в ней Python-server
```
sudo nano /etc/haproxy/haproxy.cfg
```
![2-1](https://github.com/sailent9/-/assets/130309754/6bfe018c-fc90-4283-8c48-4ae96995aa87)

---
Перезагружаю сервис haproxy

---
```
sudo systemctl reload haproxy
```

отправляю запрос домену example.local

```
curl -H 'Host:example.local' http://127.0.0.1:8088
```
![2-2](https://github.com/sailent9/-/assets/130309754/4cc2b6e6-1cf3-4c42-9e8a-85c2a3be658f)





