# 框架概念

Beego 要求您的应用程序使用固定的模型-视图-控制器（MVC）模式来简化 Web 应用程序开发过程中的相关步骤。同时，您也可以进行一些配置并实现应用程序的快速开发。

### MVC

关于 MVC 模式的简短描述：

- 模型（Models）是基本数据对象，用于描述您的应用程序域。模型还包含特定于域的逻辑查询和更新数据。
- 视图（Views）描述了数据的展示与操作。对于我们而言，Go 的模板则负责用于向用户展示和控制数据。
- 控制器（Controllers）处理请求过程。它针对用户期望的情况来展示视图，并将必要的数据传递给视图层进行渲染。 

### 服务器

Beego 基于使用 goroutine（轻量级线程）来处理请求的 Go HTTP 服务器构建。这使得您的代码可以进行阻塞操作，同时还能并发处理其余的请求过程。

### 过滤器

**过滤器** 提供了许多由 Beego 提供过滤功能，并将经过一系列过滤后的请求交由路由器（router）处理。

### 控制器（controller）与动作（action）

每个 HTTP 请求都包含一个用于处理和响应的 **动作**。相应的 **动作** 都属于 **控制器** 的方法。类型 **[Controller](http://gowalker.org/github.com/astaxie/beego#Controller)** 包含了有关字段和默认方法。

作为 HTTP 请求的处理的一部分，Beego 要求在您的 **控制器** 中通过嵌入字段的形式实例化一个 **Controller** 类型的 `beego.Controller`。Beego 不会共享各个请求之间的 **控制器** 实例。

一个 **控制器** 可以是嵌入了 `beego.Controller` 的任意类型（直接或间接均可）。

	type AppController struct {
	  beego.Controller
	}

一个 **行为** 可以是某个 **控制器** 的任意的导出方法。

例如：

	func (this *AppController) Get() {
		..
	}

在这个例子中，`Get` 方法将会在  `AppController` 接受 HTTP GET 请求后自动被调用，`Post`、`Delete` 和 `Put` 也同理。当然，您可以使用其它名字，详情参阅 [智能路由](/docs/Controllers_Routing)。