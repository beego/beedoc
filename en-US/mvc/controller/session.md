---
name: Session control
sort: 5
---

# Session control

Beego has a built-in session module that supports memory, file, mysql, redis, couchbase, memcache and postgres as the save provider. Other providers can be implemented according to the interface.

To use session in Beego switch it on in the main function:

	web.BConfig.WebConfig.Session.SessionOn = true

Or it can be activated in the configuration file:

	SessionOn = true

After being switched on, session can used be used like this:

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

There are several useful methods to handle session:

- SetSession(name string, value interface{})
- GetSession(name string) interface{}
- DelSession(name string)
- SessionRegenerateID()
- DestroySession()

The most commonly used methods are `SetSession`, `GetSession`, and `DelSession`.

Custom logic can also be used:

	sess := this.StartSession()
	defer sess.SessionRelease()

sess object has following methods:

* Set
* Get
* Delete
* SessionID
* SessionRelease
* Flush

SetSession, GetSession and DelSession methods are recommended for session operation as it will release resource automatically.

Here are some parameters used in the Session module:

- SessionOn

  Enables Session. Default value is `false`. Parameter name in configuration file: `SessionOn`

- SessionProvider
  Sets Session provider.  Set to `memory` by default. `File`, `mysql` and `redis` are also supported. Parameter name in configuration file: `sessionprovider`.

- SessionName
  Sets the cookie name. Session is stored in browser's cookies by default. The default name is beegosessionID. Parameter name in configuration file: `sessionname`.

- SessionGCMaxLifetime
  Sets the Session expire time. Default value is `3600s`. Parameter name in configuration file: `sessiongcmaxlifetime`.

- SessionProviderConfig
  Sets the save path or connection string for file, mysql or redis.  Default value is empty. Parameter name in configuration file: `sessionproviderconfig`.

- SessionHashFunc
  Sets the function used to generate sessionid.  The default value is `sha1`.

- SessionHashKey
  Sets the session hash key.  The default key is beegoserversessionkey. This value should be changed.

- SessionCookieLifeTime
  Sets the cookie expire time. The cookie is used to store data in client.

## Package Installation

If you are not using Go modules, manual installation may be required.

*Note: Beego >= 1.1.3 removed all dependencies*

```bash
# Couchbase
go get -u github.com/beego/beego/v2/server/web/session/couchbase

# Ledis
go get -u github.com/beego/beego/v2/server/web/session/ledis

# Memcache
go get -u github.com/beego/beego/v2/server/web/session/memcache

# MySQL
go get -u github.com/beego/beego/v2/server/web/session/mysql

# Postgres
go get -u github.com/beego/beego/v2/server/web/session/postgres

# Redis
go get -u github.com/beego/beego/v2/server/web/session/redis

# Redis (cluster mode)
go get -u github.com/beego/beego/v2/server/web/session/redis_cluster

# Redis (sentinel)
go get -u github.com/beego/beego/v2/server/web/session/redis_sentinel

# SSDB
go get -u github.com/beego/beego/v2/server/web/session/ssdb
```

## Example Usage

### Couchbase

SessionProviderConfig is connection address using [couchbase](https://github.com/couchbaselabs/go-couchbase).

```go
// main.go
package main

import (
  "github.com/beego/beego/v2/server/web"
  _ "github.com/beego/beego/v2/server/web/session/couchbase"
)

func init() {
  web.BConfig.WebConfig.Session.SessionOn = true
  web.BConfig.WebConfig.Session.SessionProvider = "couchbase"
  web.BConfig.WebConfig.Session.SessionProviderConfig = "http://bucketname:bucketpass@myserver:8091/"
}
```

### File
```go
// main.go
package main

import (
  "github.com/beego/beego/v2/server/web"
)

func init() {
  web.BConfig.WebConfig.Session.SessionOn = true
  web.BConfig.WebConfig.Session.SessionProvider = "file"
  web.BConfig.WebConfig.Session.SessionProviderConfig = "/tmp"
}
```

### Memcache

SessionProviderConfig is the connection address using [memcache](https://github.com/beego/memcache).

```go
// main.go
package main

import (
  "github.com/beego/beego/v2/server/web"
  _ "github.com/beego/beego/v2/server/web/session/memcache"
)

func init() {
  web.BConfig.WebConfig.Session.SessionOn = true
  web.BConfig.WebConfig.Session.SessionProvider = "memcache"
  web.BConfig.WebConfig.Session.SessionProviderConfig = "127.0.0.1:7080"
}
```

### MySQL

SessionProviderConfig is the connection address using [go-sql-driver](https://github.com/go-sql-driver/mysql).

```go
// main.go
package main

import (
  "github.com/beego/beego/v2/server/web"
  _ "github.com/beego/beego/v2/server/web/session/mysql"
)

func init() {
  web.BConfig.WebConfig.Session.SessionOn = true
  web.BConfig.WebConfig.Session.SessionProvider = "mysql"
  web.BConfig.WebConfig.Session.SessionProviderConfig = "username:password@protocol(address)/dbname?param=value"
}
```

### Postgres

SessionProviderConfig is the connection address using [postgres](https://github.com/lib/pq).

```go
// main.go
package main

import (
  "github.com/beego/beego/v2/server/web"
  _ "github.com/beego/beego/v2/server/web/session/postgres"
)

func init() {
  web.BConfig.WebConfig.Session.SessionOn = true
  web.BConfig.WebConfig.Session.SessionProvider = "postgresql"
  web.BConfig.WebConfig.Session.SessionProviderConfig = "postgres://pqgotest:password@localhost/pqgotest?sslmode=verify-full"
}
```

### Redis

SessionProviderConfig is the connection address using [redigo](https://github.com/garyburd/redigo).

```go
// main.go
package main

import (
  "github.com/beego/beego/v2/server/web"
  _ "github.com/beego/beego/v2/server/web/session/redis"
)

func init() {
  web.BConfig.WebConfig.Session.SessionOn = true
  web.BConfig.WebConfig.Session.SessionProvider = "redis"
  web.BConfig.WebConfig.Session.SessionProviderConfig = "127.0.0.1:6379"
}
```
	
## Note:
Session uses `gob` to register objects. When using a session engine other than `memory`, objects must be registered in session before they can be used. Use `gob.Register()` to register them in `init()` function. 
