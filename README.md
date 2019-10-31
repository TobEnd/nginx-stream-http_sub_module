# Nginx Stream Dockerfile
Nginx compiled with --with-stream and --ngx_http_sub_module to be able to create proxies or loadbalancers for non http protocols and modify responses by replacing specified strings.

This repository contains **Dockerfile** of Nginx Stream and Nginx HTTP Sub Module

### Base Docker Image

* [debian](https://hub.docker.com/_/debian/)


### Installation

1. Install [Docker](https://www.docker.com/).

2. Download: `docker pull tobend/nginx-stream-http_sub_module`

(alternatively, you can build an image from Dockerfile: 
```bash
$ docker build -t="tobend/nginx-stream-http_sub_module" github.com/tobend/nginx-stream-http_sub_module
```

### Usage

Start deamon with configs
```bash
$ docker run -d -p 80:80 -p 11122:11122 -v `pwd`\http.conf.d:/opt/nginx/http.conf.d  -v `pwd`\stream.conf.d:/opt/nginx/stream.conf.d --name nginx tobend/nginx-stream-http_sub_module
```

### Configure
Create configuration files in folders example:
```bash
.
|
|-- stream.conf.d
|   `-- tunnel.conf
|
|-- http.conf.d
    `-- myhttpservice.conf
```

tunnel.conf example config:
```conf
server {
    listen 11122;
    proxy_pass 192.168.2.4:22;
}
```

myhttpservice.conf example config:
```conf
upstream myhttpservice {
    server srv1.example.com;
    server srv2.example.com;
    server srv3.example.com;
}

server {
    listen 80;

    location / {
        proxy_pass myhttpservice;
        sub_filter '<a href="http://127.0.0.1:8080/'  '<a href="https://$host/';
        sub_filter_once on;
  }
}
```
For little more help on stream config and the http replacement config:
https://nginx.org/en/docs/stream/ngx_stream_core_module.html

https://nginx.org/en/docs/http/ngx_http_sub_module.html



### Zero downtime reloading of changed configs
If you change settings and need to relaod them on a running container w/o downtime
```bash
$ docker exec -ti nginx bash -c 'zero_downtime_reload.sh'
```
