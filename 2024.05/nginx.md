
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

        location ^~ /v1/ {
            proxy_pass http://host.docker.internal:8000/;
            proxy_http_version 1.1;
            proxy_cache off;  # 关闭缓存
            proxy_buffering off;  # 关闭代理缓冲
            chunked_transfer_encoding on;  # 开启分块传输编码
            tcp_nopush on;  # 开启TCP NOPUSH选项，禁止Nagle算法
            tcp_nodelay on;  # 开启TCP NODELAY选项，禁止延迟ACK算法
            keepalive_timeout 300;  # 设定keep-alive超时时间为65秒

            #proxy_set_header Host $host;
            #proxy_set_header X-Real-IP $remote_addr;
            #proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        location /v2/ {
            proxy_pass http://host.docker.internal:8001/;
            proxy_http_version 1.1;
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

## location配置

1. 只改变端口，路径完全保持不变
例如有一个服务 locolhost:8000/v1/chat/xxx
于是希望访问localhost:80/v1/<任何>路径转到/localhost:8000/v1/<任何> 路径

```
location ^~ /v1/ {
    proxy_pass http://host.docker.internal:8000;
```

注意，下面的情况不行，会少一个v1
```
location ^~ /v1/ {
    proxy_pass http://host.docker.internal:8000/;
```


## 很坑的情况
1. docker 挂载进容器的文件修改后没有改变
解决办法1: `chmod 666 /root/test.txt` 给这个文件更多权限
其他解决办法：不挂载文件，而是挂载文件夹；使用 cat 重定向来修改文件，而不是使用 vim 或者 vi

3. 撒
