---
name: session 模块
sort: 1
---

# 特别注意

*这个文档是session独立模块，即你单独拿这个模块应用于其他应用中，如果你想在beego中使用session，请查看文档[session 控制](../mvc/controller/session.md)*

# session 介绍

session 模块是用来存储客户端用户，session 模块目前只支持 cookie 方式的请求，如果客户端不支持 cookie，那么就无法使用该模块。

session 模块参考了 `database/sql` 的引擎写法，采用了一个接口，多个实现的方式。目前实现了 memory、 file、Redis 和 MySQL 四种存储引擎。

通过下面的方式安装 session：

	go get github.com/astaxie/beego/session

## session 使用

首先你必须导入包：

	import (
		"github.com/astaxie/beego/session"
	)

然后你初始化一个全局的变量用来存储 session 控制器：

	var globalSessions *session.Manager
	
接着在你的入口函数中初始化数据：

	func init() {
		globalSessions, _ = session.NewManager("memory", `{"cookieName":"gosessionid", "enableSetCookie,omitempty": true, "gclifetime":3600, "maxLifetime": 3600, "secure": false, "sessionIDHashFunc": "sha1", "sessionIDHashKey": "", "cookieLifeTime": 3600, "providerConfig": ""}`)
		go globalSessions.GC()
	}
			
NewManager 函数的参数的函数如下所示

1. 引擎名字，可以是 memory、file、mysql 或 redis。
2. 一个JSON字符串,传入Manager的配置信息
	1. cookieName: 客户端存储 cookie 的名字。
	2. enableSetCookie,omitempty: 是否开启SetCookie,omitempty这个设置
	3. gclifetime: 触发 GC 的时间。
	4. maxLifetime: 服务器端存储的数据的过期时间
	5. secure: 是否开启 HTTPS，在 cookie 设置的时候有 cookie.Secure 设置。
	6. sessionIDHashFunc: sessionID 生产的函数，默认是 sha1 算法。
	7. sessionIDHashKey: hash 算法中的 key。
	8. cookieLifeTime: 客户端存储的 cookie 的时间，默认值是 0，即浏览器生命周期。
	9. providerConfig: 配置信息，根据不同的引擎设置不同的配置信息，详细的配置请看下面的引擎设置

最后我们的业务逻辑处理函数中可以这样调用：

	func login(w http.ResponseWriter, r *http.Request) {
		sess, _ := globalSessions.SessionStart(w, r)
		defer sess.SessionRelease(w)
		username := sess.Get("username")
		if r.Method == "GET" {
			t, _ := template.ParseFiles("login.gtpl")
			t.Execute(w, nil)
		} else {
			sess.Set("username", r.Form["username"])
		}
	}

globalSessions 有多个函数如下所示：

- SessionStart 根据当前请求返回 session 对象
- SessionDestroy 销毁当前 session 对象
- SessionRegenerateId 重新生成 sessionID
- GetActiveSession 获取当前活跃的 session 用户
- SetHashFunc 设置 sessionID 生成的函数
- SetSecure 设置是否开启 cookie 的 Secure 设置

返回的 session 对象是一个 Interface，包含下面的方法

* Set(key, value interface{}) error 
* Get(key interface{}) interface{}  
* Delete(key interface{}) error     
* SessionID() string                
* SessionRelease()                  
* Flush() error

## 引擎设置

上面已经展示了 memory 的设置，接下来我们看一下其他三种引擎的设置方式：

- mysql

	其他参数一样，只是第四个参数配置设置如下所示，详细的配置请参考 [mysql](https://github.com/go-sql-driver/mysql#dsn-data-source-name)：
	
		username:password@protocol(address)/dbname?param=value
		
- redis

	配置文件信息如下所示，表示链接的地址，连接池，访问密码，没有保持为空：
	
		127.0.0.1:6379,100,astaxie
		
- file

	配置文件如下所示，表示需要保存的目录，默认是两级目录新建文件，例如 sessionID 是 `xsnkjklkjjkh27hjh78908`，那么目录文件应该是 `./tmp/x/s/xsnkjklkjjkh27hjh78908`：
	
		./tmp

## 如何创建自己的引擎

在开发应用中，你可能需要实现自己的 session 引擎，beego 的这个 session 模块设计的时候就是采用了 interface，所以你可以根据接口实现任意的引擎，例如 memcache 的引擎。

	type SessionStore interface {
		Set(key, value interface{}) error //set session value
		Get(key interface{}) interface{}  //get session value
		Delete(key interface{}) error     //delete session value
		SessionID() string                //back current sessionID
		SessionRelease()                  // release the resource & save data to provider
		Flush() error                     //delete all data
	}
	
	type Provider interface {
		SessionInit(maxlifetime int64, savePath string) error
		SessionRead(sid string) (SessionStore, error)
		SessionExist(sid string) bool
		SessionRegenerate(oldsid, sid string) (SessionStore, error)
		SessionDestroy(sid string) error
		SessionAll() int //get all active session
		SessionGC()
	}	

最后需要注册自己写的引擎：

	func init() {
		Register("own", ownadaper)
	}
						
