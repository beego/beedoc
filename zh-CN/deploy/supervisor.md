---
name: supervisor 部署
sort: 2
---

# Supervisord

Supervisord 是用 Python 实现的一款非常实用的进程管理工具，supervisord 还要求管理的程序是非 daemon 程序，supervisord 会帮你把它转成 daemon 程序，因此如果用 supervisord 来管理 nginx 的话，必须在 nginx 的配置文件里添加一行设置 daemon off 让 nginx 以非 daemon 方式启动。


## supervisord 安装

1. 安装 setuptools

		wget http://pypi.python.org/packages/2.7/s/setuptools/setuptools-0.6c11-py2.7.egg

		sh setuptools-0.6c11-py2.7.egg

		easy_install supervisor

		echo_supervisord_conf >/etc/supervisord.conf

		mkdir /etc/supervisord.conf.d

2. 修改配置 `/etc/supervisord.conf`

		[include]
		files = /etc/supervisord.conf.d/*.conf

3. 新建管理的应用

		cd /etc/supervisord.conf.d
		vim beepkg.conf

	配置文件：

		[program:beepkg]
		directory = /opt/app/beepkg
		command = /opt/app/beepkg/beepkg
		autostart = true
		startsecs = 5
		user = root
		redirect_stderr = true
		stdout_logfile = /var/log/supervisord/beepkg.log

## supervisord 管理

Supervisord 安装完成后有两个可用的命令行 supervisord 和 supervisorctl，命令使用解释如下：

* supervisord，初始启动 Supervisord，启动、管理配置中设置的进程。
* supervisorctl stop programxxx，停止某一个进程(programxxx)，programxxx 为 [program:beepkg] 里配置的值，这个示例就是 beepkg。
* supervisorctl start programxxx，启动某个进程
* supervisorctl restart programxxx，重启某个进程
* supervisorctl stop groupworker: ，重启所有属于名为 groupworker 这个分组的进程(start,restart 同理)
* supervisorctl stop all，停止全部进程，注：start、restart、stop 都不会载入最新的配置文件。
* supervisorctl reload，载入最新的配置文件，停止原有进程并按新的配置启动、管理所有进程。
* supervisorctl update，根据最新的配置文件，启动新配置或有改动的进程，配置没有改动的进程不会受影响而重启。


>>>注意：显示用 stop 停止掉的进程，用 reload 或者 update 都不会自动重启。
