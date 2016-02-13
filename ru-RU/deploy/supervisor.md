---
name: Развертывание с Supervisord 
sort: 2
---

# Supervisord

Supervisord - очень полезный процесс менеджер написанный на Python. Supervisord может преобразовать non-daemon приложение в daemon приложение. The application need to be non-daemon app. So if you want to use Supervisord to manage nginx, you need to set daemon off to run nginx in non-daemon mode.


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

Supervisored предоставляет две команды, supervisord и supervisorctl:

* supervisord: Инициализирует Supervisord, запустит конфигурируемый процесс
* supervisorctl stop programxxx: Остановит programxxx. programxxx is configed name in [program:beepkg]. Here is beepkg.
* supervisorctl start programxxx: Запустить процесс programxxx.
* supervisorctl restart programxxx: Перезапустит процесс programxxx.
* supervisorctl stop groupworker:  Перезапустит все процессы в группе groupworker
* supervisorctl stop all: Остановит все процессы. Заметка: start, restart и stop не перезапустят последний конфиг.
* supervisorctl reload: Перезапустить последнюю конфигурацию.
* supervisorctl update: Перезапустить все процессы где конфиг был изменен.


>>>Заметка: Процессы остановленный через `stop` вручную не будут перезапущенны после перезапуска или обновления.
