---
name: Deployment with nginx
sort: 3
---

# nginx ile deployment

Go halihazırda kendi kendine çalışabilen bir http sunucuya sahiptir. Fakat sunucumuzda loglama, CC ataklarına karşı savunma ve statik dosya sunucusu gibi bazı ekstra özellikler isteyebiliriz. nginx bir web server olarak bunları iyi yapmaktadır.

Bu yaklaşımda Go sadece fonksiyonelliğe ve uygulamanın mantığına odaklanabilir. Biz de nginx proxy ile birden fazla uygulamayı aynı anda deploy edebiliriz. Aşağıda iki uygulamanın 80 portunu paylaştığı ama değişik domainlere sahip olduğu bir senaryo göreceksiniz. Bu konfigürasyonda nginx kendisine gelen istekleri başka uygulamalara yönlendiriyor :

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
