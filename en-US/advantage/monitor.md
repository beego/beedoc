---
name: Live Monitor
sort: 1
---

# Live Monitor

We talked about toolbox module before. It will listen `127.0.0.1:8088` by default when application is running. It can't be accessed from internet but you can visit it by other ways such as nginx proxy.

>>> For security reason it is recommend to block 8088 in firewall.

Monitor is disabled by default. You can enabled it by:

	beego.EnableAdmin = true
	
Also you can change the port it listened:

	beego.AdminHttpAddr = "localhost"
	beego.AdminHttpPort = 8888
	
Open browser and visit `http://localhost:8088/` you will see `Welcome to Admin Dashboard`.

It's the first version now. We will keep on developing it.
	
## Requests statistics

Visit `http://localhost:8088/qps` you will see it:

	| requestUrl                                        | method     | times            | used             | max used         | min used         | avg used         |
	| /                                                 | GET        |  2               | 2.35ms           | 1.30ms           | 1.04ms           | 1.17ms           |
	| /favicon.ico                                      | GET        |  1               | 79.30us          | 79.30us          | 79.30us          | 79.30us          |
	| /src/xx                                           | GET        |  1               | 923.09us         | 923.09us         | 923.09us         | 923.09us         |
	| /src                                              | GET        |  1               | 792.93us         | 792.93us         | 792.93us         | 792.93us         |
	| /123                                              | GET        |  1               | 906.04us         | 906.04us         | 906.04us         | 906.04us         |

## Performance profiling

There are several params for profiling. Visit `http://localhost:8088/prof` and with params below and you can get different information.

	request url like '/prof?command=lookup goroutine'
	the command have below types:
	1. lookup goroutine
	2. lookup heap
	3. lookup threadcreate
	4. lookup block
	5. start cpuprof
	6. stop cpuprof
	7. get memprof
	8. gc summary


## Healthcheck

You need to manually register the healthcheck logic to see the status of the healthcheck from `http://localhost:8088/healthcheck`

## Tasks

You can add task in your applicaion and check the task status or trigger it manually.

- Check task status: `http://localhost:8088/task`
- Run task manually: `http://localhost:8088/runtask?taskname=task_name`

## Config Status

After the development of the application, we may also want to know the config when the application is running. Beego's Monitor also provided this feature.

- Show all configurations: `http://localhost:8088/listconf?command=conf`
- Show all routers: `http://localhost:8088/listconf?command=router`
- Show all filters: `http://localhost:8088/listconf?command=filter`
