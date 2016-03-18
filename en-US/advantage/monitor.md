---
name: Live Monitor
sort: 1
---

# Live Monitor

We discussed the toolbox module before. It will listen on `127.0.0.1:8088` by default when the application is running. It can't be accessed from the internet but you can browse to it by other means such as by nginx proxy.

>>> For security reason it is recommend that you block port 8088 in firewall.

Monitor is disabled by default. You can enable it by adding the following line in `conf/app.conf` file:

	EnableAdmin = true

Also you can change the port it listens on:

	AdminHttpAddr = "localhost"
	AdminHttpPort = 8088

Open browser and visit `http://localhost:8088/` you will see `Welcome to Admin Dashboard`.

## Requests statistics

Browse to `http://localhost:8088/qps` and you will see the following:

![](../images/monitoring.png)

## Performance profiling

You can also see the information for `goroutine`, `heap`, `threadcreate`, `block`, `cpuprof`, `memoryprof`, `gc summary` and do profiling.

## Healthcheck

You need to manually register the healthcheck logic to see the status of the healthcheck from `http://localhost:8088/healthcheck`

## Tasks

You can add task in your application and check the task status or trigger it manually.

- Check task status: `http://localhost:8088/task`
- Run task manually: `http://localhost:8088/runtask?taskname=task_name`

## Config Status

After the development of the application, we may also want to know the config when the application is running. Beego's Monitor also provided this feature.

- Show all configurations: `http://localhost:8088/listconf?command=conf`
- Show all routers: `http://localhost:8088/listconf?command=router`
- Show all filters: `http://localhost:8088/listconf?command=filter`
