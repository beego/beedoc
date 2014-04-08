---
name: Release Notes
sort: 2
---
# beego 1.1.4

Release an emergency version for beego has a serious security problem, please update to the latest version. By the way released all changes together

1. fixed a security problem. I will show the detail in beego/security.md later.

2. statifile move to new file.

3. move dependence of the third libs,if you use this module in your application: session/cache/config, please import the submodule of the third libs:

	```
	import (
	     "github.com/astaxie/beego"
	   _ "github.com/astaxie/beego/session/mysql"
	)
	```
4. modify some functions to private.

5. improve the FormParse.

released data: 2014-04-08

# beego 1.1.3
this is a hot fixed:

1. console engine for logs.It will not run if there's no config.

2. beego 1.1.2 support `go run main.go`, but if `main.go` bot abute the beego's project rule,use own AppConfigPath or not exist app.conf will panic.

3. beego 1.1.2 supports `go test` parse config,but actually when call TestBeegoInit still can't parseconfig

released data: 2014-04-04

# beego 1.1.2
The improvements:

1. Added ExceptMethodAppend fuction which supports filter out some functions while run autorouter
2. Supporting user-defined FlashName, FlashSeperator
3. ORM supports user-defined types such as type MyInt int
4. Fixed validation module return user-defined validating messages
5. Improved logs module, added Init processing errors. Changed some unnecessory public fucntion to private
6. Added PostgreSQL engine for session module
7. logs module supports output caller filename and line number. Added EnableFuncCallDepth function, closed by default.
8. Fixed bugs of Cookie engine in session module
9. Improved the error message for templates parsing error
10. Allowing modifing Context by Filter to skip beego's routering rules and using uder-defined routering rules. Added parameters RunController and RunMethod
11. Supporting to run beego APP by using `go run main.go` 
12. Supporting to run test cases by using `go test`. Added TestBeegoInit function.

released data: 2014-04-03

# beego 1.1.1
Added some new features and fixed some bugs in this release.

1. File engine can't delete file in session module which will raise reading failure.
2. File cache can't read struct. Improved god automating register
3. New couchbase engine for session module
4. httplib supports transport and proxy
5. Improved the Cookie function in context which support httponly by default as well as some other default parameters.
6. Improved validation module to support different cellphone No. 
7. Made getstrings function to as same as getstring which doesn't need parseform
8. Redis engine in session module will return error while connection failure
9. Fixed the bug of unable to add GroupRouters
10. Fixed the bugs for multiple static files, routes matching bug and display the static folder automatically
11. Added GetDB to get connected *sql.DB in ORM
12. Added ResetModelCache for ORM to reset the struct which has already registered the cache in order to write tests easily
13. Supporting between in ORM
14. Supporting sql.Null* type in ORM
15. Modified auto_now_add which will skip time setting if there is default value.

released data: 2014-03-12

# beego 1.1.0
Added some new features and fixed some bugs in this release.

New features

1. Supporting AddAPPStartHook function
2. Supporting plugin mode; Supporting AddGroupRouter for configuring plugin routes.
3. Response supporting HiJacker interface
4. AddFilter supports batch matching
5. Refactored session module, supporting Cookie engine
6. Perfermance benchmark for ORM
7. Added strings interface for config which allows configuration
8. Supporting template render control in controller level
9. Added basicauth plugin which can implement authentication easily
10. #436 insert multiple objects
11. #384 query map to struct

bugfix

1. Fixed the bug of FileCache
2. Fixed the import lib of websocket
3. Changed http status from 200 to 500 when there are internal error.
4. gmfim map in memzipfile.go file should use some synchronization mechanism (for example sync.RWMutex) otherwise it errors sometimes.
5. Fixed #440 on_delete bug that not getting delted automatically 
6. Fixed #441 timezone bug

released data: 2014-02-10

# beego 1.0 release
After four months code refactoring, we released the first stable version of Beego. We did a lot of refactoring and improved a lot in detail. Here is the list of the main improvements:

1. Modular design. Right now Beego is a light weight assembling framework with eight powerful stand alone modules including cache, config, logs, sessions, httplibs, toolbox, orm and context. It might have more in the future. You can use all of these stand alone modules in your other applications directly no matter it’s web applications or any other applications such as web games and mobile games.

2. Supervisor module. In the real world engineering, after the deployment of the application, we need to do many kinds of statistics and analytics for the application such as QPS statistics, GC analytics, memory and CPU monitoring and so on. When the live issue happends we also want to debug and profile our application on live. All of these real world engineering features are included in Beego. You can enable the supervisor module in Beego and visit it from default port 8088.

3. Detailed document. We rewritten all the document. We improved the document based on many advices from the users. To make it communicate easier for different language speakers, now the comments of the document in each language are separated.

4. Demos. We provided three examples, chat room, url shortener and todo list. You can understand and use Beego easier and faster by learning the demos.

5. Redesigned Beego website. Nice people from Beego community helped Beego for logo design and website design.

6. More and more users. We listed our typical users in our homepage. They are all big companies and they are using Beego for their products already. Beego already tested by those live applications.

7. Growing active communities. There are more than 390 issues on github, more than 36 contributors and more than 700 commits. Google groups is also growing.

8. More and more applications in Beego. There are some open source applications as well. E.g.: CMS system: https://github.com/insionng/toropress and admin system: https://github.com/beego/admin

9. Powerful assistance tools. bee is used to assist the development of Beego applications. It can create, compile, package the Beego application easily.

released data: 2013-12-19