---
name: Session Module
sort: 1
---

# Introduction to Session Module

Session module is used to store user data between different requests. It only support saving session id by cookie, so if the client doesn't support cookie, it won't work.

Session module is inspired by `database/sql`, one interface, multiple implementation. By default it support four saving providers: memory, file, redis and mysql.

Install session module:

	go get github.com/astaxie/beego/session

## Basic Usage:

Import package first:

	import (
		"github.com/astaxie/beego/session"
	)

Then initialize a gloal variable as session manager:

	var globalSessions *session.Manager
	
Then initialize data in your main function:

	func init() {
		globalSessions, _ = session.NewManager("memory", `{"cookieName":"gosessionid", "enableSetCookie,omitempty": true, "gclifetime":3600, "maxLifetime": 3600, "secure": false, "sessionIDHashFunc": "sha1", "sessionIDHashKey": "", "cookieLifeTime": 3600, "providerConfig": ""}`)
		go globalSessions.GC()
	}
			
Params of NewManager:

1. Saving provider name: memory、file、mysql、redis
2. A JSON string that contains the config infomation.
	1. cookieName: Cookie name of session id saved in client
	2. enableSetCookie,omitempty: Whether to enable SetCookie,omitempty
	3. gclifetime: The interval of GC.
	4. maxLifetime: Expiration time of data saved in server
	5. secure: Enable https or not. There is `cookie.Secure` while configure cookie.
	6. sessionIDHashFunc: SessionID generator function. sha1 by default.
	7. sessionIDHashKey: Hash key
	8. cookieLifeTime: Cookie expiration time in client. 0 by default, which means life time of browser
	9. providerConfig: Provider specified config. See more below.

Then we can use session in our code:

	func login(w http.ResponseWriter, r *http.Request) {
		sess := globalSessions.SessionStart(w, r)
		defer sess.SessionRelease()
		username := sess.Get("username")
		if r.Method == "GET" {
			t, _ := template.ParseFiles("login.gtpl")
			t.Execute(w, nil)
		} else {
			sess.Set("username", r.Form["username"])
		}
	}

Here is methods of globalSessions:

- `SessionStart` Return session object based on current request
- `SessionDestroy` Destroy current session object
- `SessionRegenerateId` Regenerate a new sessionID
- `GetActiveSession` Get active session user
- `SetHashFunc` Set sessionID generator function
- `SetSecure` Enable Secure of cookie or not

The returned session object is a Interface. Here are the methods:

* Set(key, value interface{}) error 
* Get(key interface{}) interface{}  
* Delete(key interface{}) error     
* SessionID() string                
* SessionRelease()                  
* Flush() error

## Saving Provider Config

We've already seen configuration of `memory` provider. Here is the config of the others:

- mysql

  All the params are same as memory's except the forth param. E.g.: 
	
		username:password@protocol(address)/dbname?param=value

  For details see [mysql](https://github.com/go-sql-driver/mysql#dsn-data-source-name)：

- redis

  Connection config: address,pool,password
	
		127.0.0.1:6379,100,astaxie
		
- file

  The session save path. Create new files in two levels by default.  E.g.: if sessionID is `xsnkjklkjjkh27hjh78908` the file will be saved as `./tmp/x/s/xsnkjklkjjkh27hjh78908`

		./tmp

## Create New Provider

Sometimes you need to create your own session provider. Session module uses interface, so you can implement this interface to create your own provider easily.

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

At last, Register your provider:

	func init() {
		Register("own", ownadaper)
	}
						
