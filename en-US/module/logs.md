---
name: Logs Module
sort: 3
---

# Logging

The logging module is inspired by `database/sql`. It supports file, console, net and smtp as destination providers by default. It can be installed like this:

	go get github.com/astaxie/beego/logs

## Basic Usage

Import package:

	import (
		"github.com/astaxie/beego/logs"
	)

Initialize log variable (10000 is the cache size):

	log := NewLogger(10000)

Then add the output provider (it supports outputting to multiple providers at the same time). The first parameter is the provider name (`console`, `file`, `conn` or `smtp`). The second parameter is a provider-specific configuration string (see below for details).

	log.SetLogger("console", "")

Then we can use it in our code:

	log.Trace("trace %s %s","param1","param2")
	log.Debug("debug")
	log.Info("info")
	log.Warn("warning")
	log.Error("error")
	log.Critical("critical")

## output file and line number

Log output not call file by default, if you're expecting output calls file name and line number, set as follows

	log.EnableFuncCallDepth(true)

true for turn on and false for turn off. default is false.

If your application encapsulates the call log packages, so you need to set up SetLogFuncCallDepth, the default is 2, namely direct call hierarchy, if you encapsulate the multilayer, so need to adjust according to their own needs.

	log.SetLogFuncCallDepth(3)

## Provider configuration

Each provider supports a set of configuration options.

- console

	Can set output level or use default. Uses `os.Stdout` by default.

		log := NewLogger(10000)
		log.SetLogger("console", `{"level":1}`)

- file

	E.g.:

		log := NewLogger(10000)
		log.SetLogger("file", `{"filename":"test.log"}`)

	Parameters:
	- filename: Save to filename.
	- maxlines: Maximum lines each log file, 1000000 by default.
	- maxsize: Maximum size of each log file, 1 << 28 or 256M by default.
	- daily: If log rotate by day, true by default.
	- maxdays: Maximum number of days log files will be kept, 7 by default.
	- rotate: Enable logrotate or not, true by default.
	- level: Log level, Trace by default.

- conn

	Net output:

		log := NewLogger(1000)
		log.SetLogger("conn", `{"net":"tcp","addr":":7020"}`)

	Parameters:
	- reconnectOnMsg: If true: reopen and close connection every time a message is sent. False by default.
	- reconnect: If true: auto connect. False by default.
	- net: connection type: tcp, unix or udp.
	- addr: net connection address.
	- level: Log level, Trace by default.

- smtp

	Log by email:

		log := NewLogger(10000)
		log.SetLogger("smtp", `{"username":"beegotest@gmail.com","password":"xxxxxxxx","host":"smtp.gmail.com:587","sendTos":["xiemengjun@gmail.com"]}`)

	Parameters:
	- username: smtp username.
	- password: smtp password.
	- host: SMTP server host.
	- sendTos: emails addresses to which the logs will be sent.
	- subject: email subject, `Diagnostic message from server` by default.
	- level: Log level, Trace by default.
