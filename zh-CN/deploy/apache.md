---
name: apache 部署
sort: 4
---

# Apache 配置

apache 和 nginx 的实现原理一样，都是做一个反向代理，把请求向后端传递，配置如下所示：

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
