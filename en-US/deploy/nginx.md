---
name: Deploy with nginx
sort: 3
---

# Deploy with Nginx

Go is a stand alone http server, but we want to nginx to do more for us such as logging, cc attack and static file serve, because nginx is doing good as a web server. Go can just focus on functionalities and logic. We can use nginx proxy to deploy multiple application at the same time. Here is a example for two applications share the 80 port but different domain, and forward to different application by nginx.

```
server {
    listen       80;
    server_name  .a.com;

    charset utf-8;
    access_log  /home/a.com.access.log  main;

    location /(css|js|fonts|img)/ {
        access_log off;
        expires 1d;

        root "/path/to/app_a/static"
        try_files $uri @backend
    }

    location / {
        try_files /_not_exists_ @backend;
    }

    location @backend {
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header Host            $http_host;

        proxy_pass http://127.0.0.1:8080;
    }
}

server {
    listen       80;
    server_name  .b.com;

    charset utf-8;
    access_log  /home/b.com.access.log  main;

    location /(css|js|fonts|img)/ {
        access_log off;
        expires 1d;

        root "/path/to/app_b/static"
        try_files $uri @backend
    }

    location / {
        try_files /_not_exists_ @backend;
    }

    location @backend {
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header Host            $http_host;

        proxy_pass http://127.0.0.1:8081;
    }
}
```
