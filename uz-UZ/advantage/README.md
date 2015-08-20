---
root: true
name: Advanced beego
sort: 6
---

# Advanced beego

We have demonstrated the basic use of beego. Now we will talk about more advanced topics.

- [In process monitor](./monitor.md)

  beego will serve at two ports. One is 8080 for application to serve users. Another is 8088, to monitor the process status, to execute tasks and so on.
	
- [Filters](../mvc/controller/filter.md)

  Filters is a very convenient feature for you to extend your logic. You can easily implement user authentication, log visiting, compatibility switching and so on.
	
- [Reload](./reload.md)

  Reload is always mentioned in web development that allows deploying application without interrupt user requests.
	
>>> This feature is not well done yet. It only tested on Mac and Linux. It haven't been tested on production environment yet. It's still under testing, so take your own risk to use it. It's recommended to use upstream of nginx.
