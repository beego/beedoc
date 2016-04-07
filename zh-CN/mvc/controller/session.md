---
name: session 控制
sort: 5
---

# session 控制
 
beego 内置了 session 模块，目前 session 模块支持的后端引擎包括 memory、cookie、file、mysql、redis、couchbase、memcache、postgres，用户也可以根据相应的 interface 实现自己的引擎。

beego 中使用 session 相当方便，只要在 main 入口函数中设置如下：

	beego.SessionOn = true

或者通过配置文件配置如下：

	sessionon = true

通过这种方式就可以开启 session，如何使用 session，请看下面的例子：

```go
func (this *MainController) Get() {
	v := this.GetSession("asta")
	if v == nil {
		this.SetSession("asta", int(1))
		this.Data["num"] = 0
	} else {
		this.SetSession("asta", v.(int)+1)
		this.Data["num"] = v.(int)
	}
	this.TplName = "index.tpl"
}
```

session 有几个方便的方法：

- SetSession(name string, value interface{})
- GetSession(name string) interface{}
- DelSession(name string)
- SessionRegenerateID()
- DestroySession()

session 操作主要有设置 session、获取 session、删除 session。

当然你可以通过下面的方式自己控制这些逻辑：

	sess:=this.StartSession()
	defer sess.SessionRelease()

sess 对象具有如下方法：

* sess.Set()
* sess.Get()
* sess.Delete()
* sess.SessionID()
* sess.Flush()

但是我还是建议大家采用 SetSession、GetSession、DelSession 三个方法来操作，避免自己在操作的过程中资源没释放的问题。

关于 Session 模块使用中的一些参数设置：

- SessionOn

	设置是否开启 Session，默认是 false，配置文件对应的参数名：sessionon。

- SessionProvider

	设置 Session 的引擎，默认是 memory，目前支持还有 file、mysql、redis 等，配置文件对应的参数名：sessionprovider。

- SessionName

	设置 cookies 的名字，Session 默认是保存在用户的浏览器 cookies 里面的，默认名是 beegosessionID，配置文件对应的参数名是：sessionname。

- SessionGCMaxLifetime

	设置 Session 过期的时间，默认值是 3600 秒，配置文件对应的参数：sessiongcmaxlifetime。

- SessionSavePath

	设置对应 file、mysql、redis 引擎的保存路径或者链接地址，默认值是空，配置文件对应的参数：sessionsavepath。
	
- SessionHashFunc

	默认值为sha1，采用sha1加密算法生产sessionid
	
- SessionHashKey

	默认的key是beegoserversessionkey，建议用户使用的时候修改该参数
	
- SessionCookieLifeTime

	设置cookie的过期时间，cookie是用来存储保存在客户端的数据。

从beego1.1.3版本开始移除了第三方依赖库,也就是如果你想使用mysql、redis、couchbase、memcache、postgres这些引擎,那么你首先需要安装

	go get -u github.com/astaxie/beego/session/mysql 
	
然后在你的 main 函数中引入该库, 和数据库的驱动引入是一样的:

	import _ "github.com/astaxie/beego/session/mysql"	

当 SessionProvider 为 file 时，SessionSavePath 是只保存文件的目录，如下所示：

	beego.SessionProvider = "file"
	beego.SessionSavePath = "./tmp"

当 SessionProvider 为 mysql 时，SessionSavePath 是链接地址，采用 [go-sql-driver](https://github.com/go-sql-driver/mysql)，如下所示：

	beego.SessionProvider = "mysql"
	beego.SessionSavePath = "username:password@protocol(address)/dbname?param=value"

当 SessionProvider 为 redis 时，SessionSavePath 是 redis 的链接地址，采用了 [redigo](https://github.com/garyburd/redigo)，如下所示：

	beego.SessionProvider = "redis"
	beego.SessionSavePath = "127.0.0.1:6379"
	
当 SessionProvider 为 memcache 时，SessionSavePath 是 memcache 的链接地址，采用了 [memcache](https://github.com/beego/memcache)，如下所示：

	beego.SessionProvider = "memcache"
	beego.SessionSavePath = "127.0.0.1:7080"
	
当 SessionProvider 为 postgres 时，SessionSavePath 是 postgres 的链接地址，采用了 [postgres](https://github.com/lib/pq)，如下所示：

	beego.SessionProvider = "postgresql"
	beego.SessionSavePath = "postgres://pqgotest:password@localhost/pqgotest?sslmode=verify-full"
	
当 SessionProvider 为 couchbase 时，SessionSavePath 是 couchbase 的链接地址，采用了 [couchbase](https://github.com/couchbaselabs/go-couchbase)，如下所示：

	beego.SessionProvider = "couchbase"
	beego.SessionSavePath = "http://bucketname:bucketpass@myserver:8091/"		
    
    
## 特别注意点
因为session内部采用了gob来注册存储的对象，例如struct，所以如果你采用了非memory的引擎，请自己在main.go的init里面注册需要保存的这些结构体，不然会引起应用重启之后出现无法解析的错误    	
