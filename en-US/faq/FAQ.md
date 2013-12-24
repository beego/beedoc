---
root: true
name: FAQ
sort: 99
---

# FAQ

1. Can't find template files or configuration files or nil pointer error

  It may because you used `go run main.go` to run your application. `go run` will compile the file and put it to tmp folder to run it. But Beego need the static files, templates and config files. So you need to use `go build` and run the application by `./app`. Or you can use `bee run app` to run your application.

2. Can Beego used for productions?

  Yes. Beego has been using in many productions. E.g.: SNDA CND system, 360 search API, Bmob mobile cloud API, weico backend API etc. They are all high concurrence and high performance applications. 
	
3. Will the future upgrades affect the API I am using right now?

  Beego is keeping the stable API since version 0.1. Many application upgraded to latest Beego easily. We will try to keek the API stable in the future.
	
4. Will Beego keep been developing?

  Many people worried about the open source projects stop developing. We have four people are contributing the code. We can keep making Beego better and better.

