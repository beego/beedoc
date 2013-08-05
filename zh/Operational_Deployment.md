# 发行部署

Go 语言的应用最后编译之后是一个二进制文件，你只需要 copy 这个应用到服务器上，运行起来就行。Beego 由于带有几个静态文件、配置文件、模板文件三个目录，所以用户部署的时候需要同时 copy 这三个目录到相应的部署应用之下，下面以我实际的应用部署为例：

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

这样我们就已经把我们需要的应用搬到服务器了，那么接下来就可以开始部署了，我现在服务器端用两种方式来 run，

- Supervisord 

	安装和配置见 [Supervisord](Supervisord.md)。

- nohup方式

	nohup ./beepkg &

个人比较推荐第一种方式，可以很好的管理起来应用。