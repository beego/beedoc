---
name: 升级指南
sort: 3
---
## beego 1.6.0 升级指南
获取最新版本的 bee 工具 `go get -u github.com/beego/bee`
更新 beego 框架 `go get -u github.com/astaxie/beego`

然后进入项目，执行: `bee fix`

## beego 1.4.2 升级指南

GetInt函数需要改成GetInt64

## beego 1.3 升级指南

1. 在 beego 1.3 里面删除了之前已经在代码里面公示了三四个版本的 AddFilter 函数，如果出现找不到该函数，请使用如下方式进行改进：

		beego.AddFilter("/user/*", "BeforeRouter", cpt.Handler)

 	修改为

		beego.InsertFilter("/user/*", beego.BeforeRouter, cpt.Handler)

1. 之前的 beego.AfterStatic 已经删除，和 beego.BeforeRouter 处于同一个级别的过滤，所以删除了该定义变量

## beego 1.0 版本
beego 1.0 之后 API 已经稳定,每个版本升级基本都可以做到兼容,如果你是 1.0 之前的版本,那么升级之后可能调整了某些参数和函数,请你根据新版本的 API 进行调整.
