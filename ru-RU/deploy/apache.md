---
name: Развертывание с Apache
sort: 4
---

# Конфигурирование Apache

Концепция Apache сильно похожа на nginx, работает как reverse proxy :TODO и посылаем запросы на бэкэнд. Пример конфигурации:

```
NameVirtualHost *:80
<VirtualHost *:80>
	ServerAdmin webmaster@dummy-host.example.com
	ServerName www.a.com
	ProxyRequests Off
	<Proxy *>
		Order deny,allow
		Allow from all
	</Proxy>
	ProxyPass / http://127.0.0.1:8080/
	ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>

<VirtualHost *:80>
	ServerAdmin webmaster@dummy-host.example.com
	ServerName www.b.com
	ProxyRequests Off
	<Proxy *>
		Order deny,allow
		Allow from all
	</Proxy>
	ProxyPass / http://127.0.0.1:8081/
	ProxyPassReverse / http://127.0.0.1:8081/
</VirtualHost>
```
