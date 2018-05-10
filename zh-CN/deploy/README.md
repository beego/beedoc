---
root: true
name: 应用部署
sort: 7
---

# 发行部署

### 开发模式

通过 bee 创建的项目，beego 默认情况下是开发模式。

我们可以通过如下的方式改变我们的模式：

	beego.RunMode = "prod"

或者我们在 `conf/app.conf` 下面设置如下：

	runmode = prod

以上两种效果一样。

开发模式中

- 开发模式下，如果你的目录不存在 views 目录，那么会出现类似下面的错误提示：

		2013/04/13 19:36:17 [W] [stat views: no such file or directory]

- 模板每次使用都会重新加载，不进行缓存。

- 如果服务端出错，那么就会在浏览器端显示如下类似的截图：

![](./../images/dev.png)

### 发行部署

Go 语言的应用最后编译之后是一个二进制文件，你只需要 copy 这个应用到服务器上，运行起来就行。beego 由于带有几个静态文件、配置文件、模板文件三个目录，所以用户部署的时候需要同时 copy 这三个目录到相应的部署应用之下，下面以我实际的应用部署为例：

	$ mkdir /opt/app/beepkg
	$ cp beepkg /opt/app/beepkg
	$ cp -fr views /opt/app/beepkg
	$ cp -fr static /opt/app/beepkg
	$ cp -fr conf /opt/app/beepkg

这样在 `/opt/app/beepkg` 目录下面就会显示如下的目录结构：

	.
	├── conf
	│   ├── app.conf
	├── static
	│   ├── css
	│   ├── img
	│   └── js
	└── views
	    └── index.tpl
	├── beepkg

这样我们就已经把我们需要的应用搬到服务器了，那么接下来就可以开始部署了。


这里部署首先你需要把应用跑起来，这分为两种方式：

- [独立部署](./beego.md)
- [Supervisord 部署](./supervisor.md)

上面只是把应用程序完全暴露在外部，我们大多数的应用会在前端部署一个 nginx 或者 apache 利用这些成熟的 HTTP 服务器做负载均衡或者其他认证之类的。

- [Nginx 部署](./nginx.md)
- [Apache 部署](./apache.md)
