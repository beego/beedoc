---
name: Stand alone Deployment
sort: 1
---

# Stand alone Deployment

This will run application in the backend as a daemon.

## linux

In Linux we can use `nohup` command to run the application in the backend:

	nohup ./beepkg &
	
Your application is running in the keep process of Linux.

## Windows

In Windows, set to auto run in the backend on start. Two ways to do that:

1. Create a bat file and put it into "Run"
2. Create a service
