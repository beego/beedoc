---
name: Deployment with Supervisord 
sort: 2
---

# Supervisord

Supervisord is a very useful process manager implemented in Python.  Supervisord can change your non-daemon application into a daemon application. The application needs to be a non-daemon app. 
So if you want to use Supervisord to manage nginx, you need to set daemon off to run nginx in non-daemon mode.


## Install Supervisord

1. install setuptools

		wget http://pypi.python.org/packages/2.7/s/setuptools/setuptools-0.6c11-py2.7.egg
		
		sh setuptools-0.6c11-py2.7.egg 
		
		easy_install supervisor
		
		echo_supervisord_conf >/etc/supervisord.conf
		
		mkdir /etc/supervisord.conf.d

2. config `/etc/supervisord.conf`

		[include]
		files = /etc/supervisord.conf.d/*.conf

3. Create new application to be managed

		cd /etc/supervisord.conf.d
		vim beepkg.conf
	
	Configurations：
	
		[program:beepkg]
		directory = /opt/app/beepkg
		command = /opt/app/beepkg/beepkg
		autostart = true
		startsecs = 5
		user = root
		redirect_stderr = true
		stdout_logfile = /var/log/supervisord/beepkg.log
		
## Supervisord Manage

Supervisord provides two commands, supervisord and supervisorctl:

* supervisord: Initialize Supervisord, run configed processes
* supervisorctl stop programxxx: Stop process programxxx. programxxx is configed name in [program:beepkg]. Here is beepkg.
* supervisorctl start programxxx: Run the process.
* supervisorctl restart programxxx: Restart the process.
* supervisorctl stop groupworker:  Restart all processes in group groupworker
* supervisorctl stop all: Stop all processes. Notes: start, restart and stop won't reload the latest configs.
* supervisorctl reload: Reload the latest configs.
* supervisorctl update: Reload all the processes who's config has changed.


>>>Notes: The processes stopped by `stop` manually won't restart after reload or update.
