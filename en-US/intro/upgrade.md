---
name: Upgrade Beego
sort: 3
---

## Upgrade beego 2.0.0

Install the latest bee tool `go get -u github.com/beego/bee/v2`
Upgrading Beego `go get -u github.com/beego/beego/v2`

Running: `bee fix -t 2`

## Upgrade to beego 1.6.0

Install the latest bee tool `go get -u github.com/beego/bee`
Upgrade Beego `go get -u github.com/astaxie/beego@v1.6.0`

Running : `bee fix`

## Upgrade to beego 1.4.2 

Change GetInt to GetInt64

## Upgrade to beego 1.3

1. `AddFilter` method was removed, and you could update your code from：

		beego.AddFilter("/user/*", "BeforeRouter", cpt.Handler)

 	To

		beego.InsertFilter("/user/*", beego.BeforeRouter, cpt.Handler)

1. beego.AfterStatic was removed，please using beego.BeforeRouter
