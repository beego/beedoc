---
name: 发布版本
sort: 2
---
# beego 1.3.0
经过了一个多月的开发，我们很高兴的宣布，beego1.3.0来了，这个版本我们做了非常多好玩并且有用的功能，升级请看[升级指南](http://beego.me/docs/intro/upgrade.md)

## 路由重写
这一次路由进行了全部改造，从之前的三个路由模式，改成了tree路由，第一性能得到了提升，第二路由支持的格式更加丰富，第三路由更加符合我们的思考方式，

例如现在注册如下路由规则：

	/user/astaxie
	/user/:username
	
如果你的访问地址是`/user/astaxie`，那么优先匹配固定的路由，也就是第一条，如果访问是`/user/slene`，那么就匹配第二个，和你注册的路由的先后顺序无关

## namespace更优雅
设计namespace主要是为了大家模块化设计的，之前是采用了类似jQuery的链式方式，当然新版本也是支持的，但是由于gofmt的格式无法很直观的看出来整个路由的目录结构，所以我采用了多参数注册方式，现在看上去就更加的优雅：

```
ns := 
beego.NewNamespace("/v1",
    beego.NSNamespace("/shop",
        beego.NSGet("/:id", func(ctx *context.Context) {
            ctx.Output.Body([]byte("shopinfo"))
        }),
    ),
    beego.NSNamespace("/order",
        beego.NSGet("/:id", func(ctx *context.Context) {
            ctx.Output.Body([]byte("orderinfo"))
        }),
    ),
    beego.NSNamespace("/crm",
        beego.NSGet("/:id", func(ctx *context.Context) {
            ctx.Output.Body([]byte("crminfo"))
        }),
    ),
)
```
更多详细信息请参考文档：[namespace](http://beego.me/docs/mvc/controller/router.md#namespace)

## 注解路由
```
// CMS API
type CMSController struct {
    beego.Controller
}

func (c *CMSController) URLMapping() {
    c.Mapping("StaticBlock", c.StaticBlock)
    c.Mapping("AllBlock", c.AllBlock)
}

// @router /staticblock/:key [get]
func (this *CMSController) StaticBlock() {

}

// @router /all/:key [get]
func (this *CMSController) AllBlock() {
}
```
更多请参考文档：[注解路由](http://beego.me/docs/mvc/controller/router.md#%E6%B3%A8%E8%A7%A3%E8%B7%AF%E7%94%B1)

## 自动化文档
自动化文档一直是我梦想中的一个功能，这次借着公司的项目终于实现了出来，我说过beego不仅仅要让开发API快，而且让使用API的用户也能快速的使用我们开发的API，这个就是我开发这个项目的初衷。

可以通过注释自动化的生成文档，并且在线测试，详细的请看下面的截图

![](../images/docs.png)

而且可以通过文档进行API的测试：

![](../images/doc_test.png)

更多请参考文档：[自动化文档](http://beego.me/docs/advantage/docs.md)
## config支持不同模式的配置
在配置文件里面支持section，可以有不同的Runmode的配置，默认优先读取runmode下的配置信息，例如下面的配置文件：

	appname = beepkg
	httpaddr = "127.0.0.1"
	httpport = 9090
	runmode ="dev"
	autorender = false
	autorecover = false
	viewspath = "myview"
	
	[dev]
	httpport = 8080
	[prod]
	httpport = 8088
	[test]
	httpport = 8888
	
上面的配置文件就是在不同的runmode下解析不同的配置，例如在dev模式下，httpport是8080，在prod模式下是8088，在test模式下是8888.其他配置文件同理。解析的时候优先解析runmode下地配置，然后解析默认的配置。

## 支持双向的SSL认证
```
config := tls.Config{
    ClientAuth: tls.RequireAndVerifyClientCert,
    Certificates: []tls.Certificate{cert},
    ClientCAs: pool,
}
config.Rand = rand.Reader

beego.BeeApp.Server.TLSConfig = &config
```
## beego.Run支持带参数

beego.Run() 默认执行HttpPort

beego.Run(":8089")

beego.Run("127.0.0.1:8089")

## XSRFKEY的token从15个字符增加到32各字符，增强安全性

## 删除热更新

## 模板函数增加Config，可以方便的在模板中获取配置信息

	{{config returnType key defaultValue}}
	
	{{config "int" "httpport" 8080}}

## httplib支持cookiejar功能，感谢curvesft		

## orm时间格式，如果为空就设置为nil，感谢JessonChan

## config模块支持json解析就一个array格式，感谢chrisport

## bug fix
- 静态文件目录循环跳转
- fix typo


# beego 1.2.0
大家好,经过我们一个多月的努力,今天我们发布一个很帅的版本,之前性能测试框架出来beego已经跃居Go框架第一了,虽然这不是我们的目标,我们的目标是做最好用,最易用的框架.http://www.techempower.com/benchmarks/#section=data-r9&hw=i7&test=json 但是这个版本我们还是在性能和易用性上面做了很多改进.应该说性能更加的接近Go原生应用.

### 新特性:

#### 1.namespace支持

```
	beego.NewNamespace("/v1").
		Filter("before", auth).
		Get("/notallowed", func(ctx *context.Context) {
		ctx.Output.Body([]byte("notAllowed"))
	}).
		Router("/version", &AdminController{}, "get:ShowAPIVersion").
		Router("/changepassword", &UserController{}).
		Namespace(
		beego.NewNamespace("/shop").
			Filter("before", sentry).
			Get("/:id", func(ctx *context.Context) {
			ctx.Output.Body([]byte("notAllowed"))
		}))
```

上面这个代码支持了如下这样的请求URL
- GET       /v1/notallowed
- GET       /v1/version
- GET       /v1/changepassword
- POST     /v1/changepassword
- GET       /v1/shop/123

而且还支持前置过滤,条件判断,无限嵌套namespace

#### 2.beego支持更加自由化的路由方式

RESTFul的自定义函数

- beego.Get(router, beego.FilterFunc)
- beego.Post(router, beego.FilterFunc)
- beego.Put(router, beego.FilterFunc)
- beego.Head(router, beego.FilterFunc)
- beego.Options(router, beego.FilterFunc)
- beego.Delete(router, beego.FilterFunc)

```
beego.Get("/user", func(ctx *context.Context) {
	ctx.Output.Body([]byte("Get userlist"))
})
```
更加自由度的Handler

- beego.Handler(router, http.Handler)

可以很容易的集成其他服务

```
import (
    "http"
    "github.com/gorilla/rpc"
    "github.com/gorilla/rpc/json"
)

func init() {
    s := rpc.NewServer()
    s.RegisterCodec(json.NewCodec(), "application/json")
    s.RegisterService(new(HelloService), "")
    beego.Handler("/rpc", s)
}
```

#### 3.支持从用户请求中直接数据bind到指定的对象

例如请求地址如下

	?id=123&isok=true&ft=1.2&ol[0]=1&ol[1]=2&ul[]=str&ul[]=array&user.Name=astaxie

```
var id int  
ctx.Input.Bind(&id, "id")  //id ==123

var isok bool  
ctx.Input.Bind(&isok, "isok")  //isok ==true

var ft float64  
ctx.Input.Bind(&ft, "ft")  //ft ==1.2

ol := make([]int, 0, 2)  
ctx.Input.Bind(&ol, "ol")  //ol ==[1 2]

ul := make([]string, 0, 2)  
ctx.Input.Bind(&ul, "ul")  //ul ==[str array]

user struct{Name}  
ctx.Input.Bind(&user, "user")  //user =={Name:"astaxie"}
```

#### 4.优化解析form的流程,让性能更加提升

#### 5.增加更多地testcase进行自动化测试

#### 6.admin管理模块所有的增加可点击的链接,方便直接查询

#### 7.session的除了memory之外的引擎支持struct存储

#### 8.httplib支持文件直接上传接口

```
b:=httplib.Post("http://beego.me/")
b.Param("username","astaxie")
b.Param("password","123456")
b.PostFile("uploadfile1", "httplib.pdf")
b.PostFile("uploadfile2", "httplib.txt")
str, err := b.String()
if err != nil {
    t.Fatal(err)
}
```
httplib支持自定义协议版本

#### 9.ORM支持struct中有unexport的字段

#### 10.XSRF支持controller级别控制是否启用, 之前XSRF是全局设置只要开启了就会影响所有的 POST PUT DELET请求,但是项目中可能API和页面共存的情况,页面可能不需要类似的XSRF,因此支持在Prepare函数中设置值来控制controller是否启用XSRF. 默认是true,也就是根据全局的来执行.用户可以在prepare中设置是否关闭.

```
func (a *AdminController) Prepare(){
       a.EnableXSRF = false
}
```

#### 11.controller支持ServeFormatted函数,支持根据请求Accept来判断是调用ServeJson还是ServeXML

#### 12.session提供memcache引擎

#### 13.Context中的Download函数支持自定义文件名提供下载 

## bug修复

1. session的Cookie引擎修复无法设置过期的bug
2. 修复flash数据的存储和解析问题
3. 修复所有go vet出现的问题
4. 修复ParseFormOrMulitForm问题
5. 修复只有POST才能解析raw body,现在支持除了GET和HEAD之外的其他请求
6. config模块修复xml和yaml无法解析的问题

# beego 1.1.4

发布一个紧急的版本, beego存在一个严重的安全漏洞,请大家更新到最新版本. 顺便把最近做的一起发布了

1. 紧急修复一个安全漏洞,稍后会在beego/SECURITY.md公布详细的情况

2. 静态文件处理独立到文件

3. 第三方依赖库移除,目前如果你使用session/cache/config,使用的是依赖第三方库的,那么现在都移到了子目录,如果你想使用这些就需要在使用的地方采用mysql类似的方式引入

	```
	import (
	     "github.com/astaxie/beego"
	   _ "github.com/astaxie/beego/session/mysql"
	)
	```
4. 修改部分导出的函数为private,因为外部不需要调用

5. 优化formparse的过程,根据不同的content-type进行解析

发布时间: 2014-04-08

# beego 1.1.3
这是一个hotfix的版本,主要是修复了以下bug

1. console日志输出,如果不设置配置文件,不能正常输出

2. 支持了go run main.go,但是main.go没有遵循beego的目录结构,自定义了配置文件或者不存在配置文件,就会panic找不到app.conf.

3. 支持了在go test中解析配置,但是实际上调用TestBeegoInit无法解析配置文件

发布时间: 2014-04-04

# beego 1.1.2
beego1.1.2版本发布,这个版本主要是一些改进:

1. 增加ExceptMethodAppend函数,支持autorouter的时候过滤一些函数

2. 支持自定义FlashName,FlashSeperator

3. ORM支持自定义的类型，例如type MyInt int 这种

4. 修复验证模块返回自定义验证信息

5. 改进logs模块, 增加Init处理error,设置一些不必要的publice函数为pravite

6. 增加PostgreSQL的session引擎

7. logs模块,支持输出调用的文件名和行号,增加设置函数EnableFuncCallDepth,默认关闭

8. 修改session模块中Cookie引擎的一个隐藏bug

9. 模板解析错误的时候,提示语进行了改进

10. 允许用户通过Filter修改Context,跳过beego的路由查找方法,直接使用自己的路由规则.增加参数RunController 和RunMethod

11. 支持go run main.go执行beego的应用

12. 支持go test执行测试用例,无法读取配置文件,模板的问题,增加TestBeegoInit函数调用.

发布时间: 2014-04-03

# beego 1.1.1
这个版本主要是一些bug的修复和增加新功能

1. session模块file引擎无法删除文件,不断刷新引起的文件读取失败问题

2. 文件缓存无法读取struct,改进gob自动化注册

3. session模块增加新引擎couchbase

4. httplib 支持设置 transport 和 proxy

5. 改进context中Cookie函数,默认支持httponly,以及其他一些默认参数行为

6. 验证模块的改进,支持更多地手机号码

7. getstrings函数行为也改成getstring一样,不需要自己parseform

8. session模块redis引擎,在连接失败的情况下返回错误

9. 修复无法添加GroupRouters的bug问题

10. 改进多个静态文件的一些潜在bug,路径匹配问题,以及自动跳转静态目录显示.

11. ORM增加 GetDB 获取已连接的 *sql.DB

12. ORM增加 ResetModelCache 重置已注册缓存的模型struct，方便写测试

13. ORM支持 between

14. ORM支持 sql.Null* 类型

15. 修改 auto_now_add，用户有自定义值时跳过自动设置时间

发布时间: 2014-03-12

# beego 1.1.0
这个版本增加了一些新特性，修复了一些bug

新特性

1. 支持AddAPPStartHook函数
2. 支持插件模式，支持AddGroupRouter，用于插件路由设置
3. response支持HiJacker接口
4. AddFilter支持批量匹配
5. session重构，支持Cookie引擎
6. 主流 ORM 性能测试
7. config增加strings接口，允许设置
8. 支持controller级别的模板渲染控制
9. 增加插件basicauth，可以方便的使用该插件实现认证
10. #436 一次插入多个对象
11. #384 query map to struct

bugfix

1. 修复FileCache的bug
2. websocket的例子修正引用库
3. 当发生程序内部错误时。http的status默认修改为500而不是200
4. gmfim map in memzipfile.go file should use some synchronization mechanism (for example sync.RWMutex) otherwise it errors sometimes.
5.  #440 on_delete 不自动删除的问题
6. #441 时区问题

发布时间: 2014-02-10

# beego 1.0.0
经过了四个多月的重构开发，beego 终于发布了第一个正式稳定版本。这个版本我们进行了重构，同时针对很多细节进行了改进。下面列一些主要的改进功能：

1. 模块化的设计，现在基本上 beego 做成一个轻量的组装框架，重模块的设计，目前实现了cache、config、logs、sessions、httplibs、toolbox、orm、context 等八个模块，以后可能会更多。用户可以直接引用这些模块应用于自己的应用中，不仅仅局限于Web应用，beego用户中有应用logs、config、cache这些模块到页游、手游中。

2. 工程化的设计，部署项目之后，经常需要进行对线上程序进行各种信息的统计和分析，统计包括QPS，分析包括GC、内存使用量、CPU，如果出现问题的时候我们还希望通过profile来调试，那么beego都为你考虑到了这些，集成了监控模块，默认是关闭的，用户可以开启，并在另一个端口监听，通过http://127.0.0.1:8088/访问。

3. 详细的文档，这个版本的文档全部是新写的，在之前文档用户的各方面反馈之后，进行了很多细节上面的改进，目前文档中英文版本都已经完成，中英文文档的评论分离，针对不同的用户群交流。

4. 丰富的示例，这一次更新我们开发组写了三个例子，聊天室、短域名、todo任务三个比较有典型意义的例子。让用户在熟悉beego之前有一个更深入的了解。

5. 全新设计的官方网站，这一次我们通过社区获得了很多人的帮助，logo 设计，网站UI的改进。

6. 越来越多的用户，官方网站列举了一些典型的用户，都是一些比较大的公司，他们内部都在使用beego开发对外的API应用，说明 beego 是得到了线上项目验证的框架。

7. 越来越活跃的社区，在 github上面目前已经差不多有390个的 issue，贡献者超过36个，commit 超过了700个，Google groups 目前还在稳步发展中。

8. 周边产品越来越多，基于beego的开源产品也越来越多，例如cms系统，https://github.com/insionng/toropress  例如管理后台系统，https://github.com/beego/admin 

9. beego 的辅助工具越来越强大，bee 工具是专门辅助用户开发 beego 应用的，可以快速的创建应用，动态编译，打包部署等

发布时间: 2013-12-19