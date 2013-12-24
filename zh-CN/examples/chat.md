---
name: 在线聊天室
sort: 1
---

# 在线聊天室

本示例通过两种形式展示了如何实现一个在线聊天室应用：

- 使用长轮询模式。
- 使用 WebSocket 模式。

以上两种模式均默认将数据存储在内存中，因此每次启动都会被重置。但您也可以通过修改 `conf/app.conf` 中的设置来启用数据库。

以下为项目组织大纲：

```
WebIM/
	WebIM.go			# main 包的文件
	conf
		app.conf		# 配置文件
	controllers
		app.go			# 供用户选择技术和用户名的欢迎页面
		chatroom.go		# 数据管理相关的函数
		longpolling.go	# 长轮询模式的控制器和方法
		websocket.go	# WebSocket 模式的控制器和方法
	models
		archive.go		# 操作数据相关的函数
	views
		...				# 模板文件
	static
		...				# JavaScript 和 CSS 文件
```

[到 GitHub 上浏览代码](https://github.com/beego/samples/tree/master/WebIM)
