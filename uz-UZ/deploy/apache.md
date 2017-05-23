---
name: Deploy with Apache
sort: 4
---

# Apache configuration

The concept of apache is same as the nginx, serving as a reverse proxy :TODO and sending requests to backend. Here is the configuration example:

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
