---
name: 升级指南
sort: 3
---
## beego 1.6.0升级指南
获取最新版本的bee工具 `go get -u github.com/beego/bee`
更新beego框架 `go get -u github.com/astaxie/beego`

然后进入项目，执行: `bee fix`

## beego1.4.2升级指南

GetInt函数需要改成GetInt64

## beego1.3升级指南

1. 在beego1.3里面删除了之前已经在代码里面公示了三四个版本的AddFilter函数，如果出现找不到该函数，请使用如下方式进行改进：

		beego.AddFilter("/user/*", "BeforeRouter", cpt.Handler)

 	修改为

		beego.InsertFilter("/user/*", beego.BeforeRouter, cpt.Handler)

1. 之前的beego.AfterStatic已经删除，和beego.BeforeRouter处于同一个级别的过滤，所以删除了该定义变量

## beego1.0版本
beego1.0之后API已经稳定,每个版本升级基本都可以做到兼容,如果你是1.0之前的版本,那么升级之后可能调整了某些参数和函数,请你根据新版本的API进行调整.
