
## 最简单的配置文件

```
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    ## 新服务(服务处理)
    server {
        listen       80;
        server_name  localhost;

        location / {
            proxy_pass http://host.docker.internal:8000;
            proxy_buffering off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        location /api {
            proxy_pass http://host.docker.internal:8001;
        }

        location /service2 {
            proxy_pass http://host.docker.internal:8002;
        }
    }
}
```

``` bash
docker run --name nginx -p 80:80 -v /home/lighthouse/nginx/nginx.conf:/etc/nginx/nginx.conf:ro --add-host=host.docker.internal:host-gateway -d nginx
```
