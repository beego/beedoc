---
name: Развертывание с nginx
sort: 3
---

# Конфигурация для Nginx

Go имеет свой встроенный http сервер, но вы можете использовать nginx, чтобы иметь больше возможностей при логгировании, cc атаках и обработки статических файлов, как делает это обычный веб-сервер. Go, в этом случае, может просто сфокусироваться на обработке логики и функциональности приложения. Мы можем использовать nginx proxy для разворачивания распределенного приложения в тоже время. Здесь приведен привер для двух приложений, использующих 80 порт для разных доменов, который перенаправляет запросы к нужному Go приложению.

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
