---
name: Deployment with nginx
sort: 3
---

# Deployment with Nginx

Go already has a standalone http server. But we still want to have nginx to do more for us such as logging, CC attack and act as a static file server because nginx performs well as a web server. 
So Go can just focus on functionality and logic. We can also use the nginx proxy to deploy multiple applications at the same time. Here is an example of two applications that share port 80 but have different domains, and requests are forwarding to different applications by nginx.

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
