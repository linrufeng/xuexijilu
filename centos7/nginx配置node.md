
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}


## 在 conf.d 中的配置文件
nginx 配置版本 1.8.0
server {
        listen       80;
        server_name  m.pingshuo.xyz;    #要访问的域名，我这里用的测试域名，如果有多个，用逗号分开
 
        charset utf8;
 
        location / {
            proxy_pass       http://127.0.0.1:3001;               #映射到代理服务器，可以是ip加端口,   或url 
            # proxy_set_header Host      $host;
            # proxy_set_header X-Real-IP $remote_addr;
            # proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
 
 }
}

server {
    listen       80;
    server_name  jiqing.snake.com;
    access_log  /home/wwwlogs/access.log;
    location / {
                if ($request_uri ~ ^/api.php/) {
                        proxy_pass http://xxx.snake.com;
                }
                if ($request_uri !~ ^/api.php/) {
                        proxy_pass http://localhost:7002;
                }

        }
}