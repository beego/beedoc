---
name: Развертывание с Supervisord 
sort: 2
---

# Supervisord

Supervisord - очень полезный процесс менеджер написанный на Python. Supervisord может преобразовать non-daemon приложение в daemon приложение.

## Установка Supervisord

1. Установка setuptools

		wget http://pypi.python.org/packages/2.7/s/setuptools/setuptools-0.6c11-py2.7.egg
		
		sh setuptools-0.6c11-py2.7.egg 
		
		easy_install supervisor
		
		echo_supervisord_conf >/etc/supervisord.conf
		
		mkdir /etc/supervisord.conf.d

2. Настройка `/etc/supervisord.conf`

		[include]
		files = /etc/supervisord.conf.d/*.conf

3. Создайте новое приложение для управления

		cd /etc/supervisord.conf.d
		vim beepkg.conf
	
	Настройка：
	
		[program:beepkg]
		directory = /opt/app/beepkg
		command = /opt/app/beepkg/beepkg
		autostart = true
		startsecs = 5
		user = root
		redirect_stderr = true
		stdout_logfile = /var/log/supervisord/beepkg.log
		
## Управление с Supervisord

Supervisord предоставляет две команды, supervisord и supervisorctl:

* supervisord: Инициализирует Supervisord, запустит конфигурируемый процесс
* supervisorctl stop programxxx: Остановит programxxx. programxxx is configed name in [program:beepkg]. Here is beepkg.
* supervisorctl start programxxx: Запустить процесс programxxx.
* supervisorctl restart programxxx: Перезапустит процесс programxxx.
* supervisorctl stop groupworker:  Перезапустит все процессы в группе groupworker
* supervisorctl stop all: Остановит все процессы. Заметка: start, restart и stop не перезапустят последний конфиг.
* supervisorctl reload: Перезапустит последнюю конфигурацию.
* supervisorctl update: Перезапустит все процессы, в которых был иземенен конфиг.


>>>Заметка: Процессы остановленные вручную с помощью `stop` не будут перезапущены после reload или update.
