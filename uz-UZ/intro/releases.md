---
name: Release Notes
sort: 2
---
# beego 1.4.3
New Features:

1. ORM support default settting
2. improve logs/file line count
3. sesesion ledis support select db
4. session redis support select db
5. cache redis support select db
6. `UrlFor` support all type of the parameters
7. controller `GetInt/GetString` function support default value, like: `GetInt("a",12)`
8. add `CompareNot/NotNil` template function
9. support Controller defeine error，[controller Error](http://beego.me/docs/mvc/controller/errors.md#controller%E5%AE%9A%E4%B9%89error)
10. `ParseForm` support slices of ints and strings
11. improve ORM interface

bugfix:
1. context get wrong subdomain
2. `beego.AppConfig.Strings` when the strings is empty, always return `[]string{}`
3. utils/pagination can't modify the attributes
4. when the request url is empty, route tree crashes
5. can't click the link to run the task in adminui
6. FASTCGI restart didn't delete the unix Socket file

# beego 1.4.2
New Features:

1. Added SQL Constructor inspired by ZEND ORM.
2. Added `GetInt()`, `GetInt8()`, `GetInt16()`, `GetInt32()`, `GetInt64()` for Controller.
3. Improved the logging. Added `FilterHandler` for filter logging output.
4. Static folder supports `index.html`. Automatically adding `/` for static folders.
5. `flash` supports `success` and `set` methods.
6. Config for ignoring case for routers: `RouterCaseSensitive`. Case sensitive by default.
7. Configs load based on environment: `beego.AppConfig.String("myvar")` return 456 on dev mode and return 123 on the other modes.

    > runmode = dev
    > myvar = 123
    > [dev]
    > myvar = 456

8. Added `include` for `ini` config files:

    > appname = btest
    > include b.conf

9. Added `paginator` utils.
10. Added `BEEGO_RUNMODE` environment variable. You can change the application mode by changing this environment variable.
11. Added Json function for fetching `statistics` in `toolbox`.
12. Attachements support for mail utils.
13. Turn on fastcgi by standard IO.
14. Using `SETEX` command to support the old version redis in redis Session engine.
15. RenderForm supports html id and class by using id and class tag.
16. ini config files support BOM head.
17. Added new Session engine `ledis`.
18. Improved file uploading in `httplib`. Supporting extremely large files by using `io.Pipe`.
19. Binding to TCP4 address by default. It will bind to ipv6 in GO. Added config variable `ListenTCP4`.
20. off/on/yes/no/1/0 will parse to `bool` in form rendering. Support time format.
21. Simplify the generating of SeesionID. Using golang buildin `rand` function other than `hmac_sha1`.

bugfix:

1. XSRF verification failure while `PUT` and `DELETE` cased by lowercased `_method`
2. No error message returned while initialize the cache by `StartAndGC`
3. Can't set `User-Agent` in `httplib`
4. Improved `DelStaticPath`
5. Only finding files in the first static folder when using multiple static folders
6. `Filter` functions can't execute after `AfterExec` and `FinishRouter`
7. Fixed uninitialized mime
9. Wrong file name and line number in the log
10. Can't send the request while only uploading one file in `httplib`
11. Improved the `Abort` output message. It couldn't out undefined error message before.
12. Fixed the issue that can't add inner Filter while no out Filter set in the nested `namespaces`
13. Router mapping error while router has multiple level parameters. #824
14. The information lossing while having many `namespaces` for the commented router. #770
15. `urlfor` function calling useless {{placeholder}} #759

# beego 1.4.1
New features:

1. `context.Input.Url` get path info without domain scheme.
2. Added plugin `apiauth` to simulate the `AWS` encrypted requests.
3. Simplified the debug output for router info.
4. Supportting pointer type in ORM.
5. Added `BasicAuth`, cache for multiple requests

bugfix:
1. Router *.* can't be parsed

# beego 1.3.0
Hi guys! After the hard working for one month, we are so excited to release beego 1.3.0. We brought many useful features. [Upgrade notes](http://beego.me/docs/intro/upgrade.md)

#### The brand new router system
We rewrote the router system to tree router. It improved the performance significantly and supported more formats.

For the routers below:

    /user/astaxie
    /user/:username

If the request is `/user/astaxie`, it will match fixed router which is the first one; If the request is `/user/slene`, it will match the second one. The register order doesn't matter.

#### namespace is more elegant
`namespace` is designed for modular applications. It was using chain style similar to jQuery in previous version but `gofmt` can't format it very well. Now we are using multi parameters style: (The chain style still works)

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
For more information please check [namespace](http://beego.me/docs/mvc/controller/router.md#namespace)

#### Annotation Router
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
[Annotation Router](http://beego.me/docs/mvc/controller/router.md#annotations)

#### Automated API Document
Automated document is a very cool feature that I wish to have. Now it became real in Beego. As I said beego will not only boost the development of API but also make the API easy to use for the user.

The API document can be generated by annotations automatically and can be tested online.

![](../images/docs.png)


![](../images/doc_test.png)

For more information please check [Automated Document](http://beego.me/docs/advantage/docs.md)

#### config supports different Runmode

You can set configurations for different Runmode under their own sections. beego will take the configurations of current Runmode by default. For example:

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

The configurations above set up httpport for dev, prod and test environment. beego will take httpport = 8080 for current runmode "dev".

#### Support Two Way Authentication for SSL

```
config := tls.Config{
    ClientAuth: tls.RequireAndVerifyClientCert,
    Certificates: []tls.Certificate{cert},
    ClientCAs: pool,
}
config.Rand = rand.Reader

beego.BeeApp.Server.TLSConfig = &config
```

#### beego.Run supports parameter

`beego.Run()` Run on `HttpPort` by default

`beego.Run(":8089")`

`beego.Run("127.0.0.1:8089")`

#### Increased XSRFKEY token from 15 characters to 32 characters.

#### Removed hot reload

#### Template function supports Config. Get Config value from Template easily.

    {{config returnType key defaultValue}}

    {{config "int" "httpport" 8080}}

#### httplib supports cookiejar. Thanks to curvesft

#### orm suports time format. If empty return nil other than 0000.00.00 Thanks to JessonChan

#### config module supports parsing a json array. Thanks to chrisport

### bug fix

- Fixed static folder infinite loop
- Fixed typo



# beego 1.2.0

Hi guys! After one month of hard work, we released the new awesome version 1.2.0. beego is the fastest Go framework in the latest [Web Framework Benchmarks](http://www.techempower.com/benchmarks/#section=data-r9&hw=i7&test=json) already though our goal is to make beego the best and easiest framework to use. In this new release, we improved even more in both usability and performance which is closer to native Go.

### New Features:

#### 1. `namespace` Support

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

The code above supports the URL requests below:

```
GET       /v1/notallowed
GET       /v1/version
GET       /v1/changepassword
POST      /v1/changepassword
GET       /v1/shop/123
```

`namespace` also supports pre-filters, conditions checking and unlimited nested `namespace`

#### 2. Supporting more flexible router modes

Custom functions from RESTful router

```
beego.Get(router, beego.FilterFunc)
beego.Post(router, beego.FilterFunc)
beego.Put(router, beego.FilterFunc)
beego.Head(router, beego.FilterFunc)
beego.Options(router, beego.FilterFunc)
beego.Delete(router, beego.FilterFunc)

beego.Get("/user", func(ctx *context.Context) {
    ctx.Output.Body([]byte("Get userlist"))
})
```

More flexible Handler

`beego.Handler(router, http.Handler)`

Integrating other services easily

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

#### 3. Binding request parameters to object directly


For example: this request parameters

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

#### 4. Optimized the form parsing flow and improved the performance

#### 5. Added more testcases

#### 6. Added links for admin monitoring module

#### 7. supporting saving struct into session

#### 8.httplib supports file upload interface

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
`httplib` also supports custom protocol version

#### 9. ORM supports all the unexport fields of struct

#### 10. Enable XSRF in controller level. XSRF can only be controlled in the whole project level. However, you may want to have more control for XSRF, so we let you control it in Prepare function in controller level. Default is true which means using the global setting.

```
func (a *AdminController) Prepare(){
       a.EnableXSRF = false
}
```

#### 11. controller supports ServeFormatted function which supports calling ServeJson or ServeXML based on the request's Accept

#### 12. session supports memcache engine

#### 13. The Download function of Context supports custom download file name

## Bug Fixes

1. Fixed the bug that session's Cookie engine can't set expiring time
2. Fixed the bug of saving and parsing flash data
3. Fixed all the problems of `go vet`
4. Fixed the bug of ParseFormOrMulitForm
5. Fixed the bug that only POST can parse raw body. Now all the requests except GET and HEAD support raw body.
6. Fixed the bug that config module can't parse `xml` and `yaml`



# beego 1.1.4

This is an emergency release for solving a serious security problem. Please update to the latest version! By the way released all changes together.

1. fixed a security problem. I will show the details in beego/security.md later.

2. `statifile` move to new file.

3. move dependence of the third libs,if you use this module in your application: session/cache/config, please import the submodule of the third libs:

	```
	import (
	     "github.com/astaxie/beego"
	   _ "github.com/astaxie/beego/session/mysql"
	)
	```
4. modify some functions to private.

5. improve the FormParse.

released date: 2014-04-08

# beego 1.1.3
this is a hot fixed:

1. console engine for logs.It will not run if there's no config.

2. beego 1.1.2 support `go run main.go`, but if `main.go` bot abute the beego's project rule,use own AppConfigPath or not exist app.conf will panic.

3. beego 1.1.2 supports `go test` parse config,but actually when call TestBeegoInit still can't parseconfig

released date: 2014-04-04

# beego 1.1.2
The improvements:

1. Added ExceptMethodAppend fuction which supports filter out some functions while run autorouter
2. Supporting user-defined FlashName, FlashSeperator
3. ORM supports user-defined types such as type MyInt int
4. Fixed validation module return user-defined validating messages
5. Improved logs module, added Init processing errors. Changed some unnecessory public function to private
6. Added PostgreSQL engine for session module
7. logs module supports output caller filename and line number. Added EnableFuncCallDepth function, closed by default.
8. Fixed bugs of Cookie engine in session module
9. Improved the error message for templates parsing error
10. Allowing modifing Context by Filter to skip beego's routering rules and using uder-defined routering rules. Added parameters RunController and RunMethod
11. Supporting to run beego APP by using `go run main.go`
12. Supporting to run test cases by using `go test`. Added TestBeegoInit function.

released date: 2014-04-03

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

released date: 2014-03-12

# beego 1.1.0
Added some new features and fixed some bugs in this release.

New features

1. Supporting AddAPPStartHook function
2. Supporting plugin mode; Supporting AddGroupRouter for configuring plugin routes.
3. Response supporting HiJacker interface
4. AddFilter supports batch matching
5. Refactored session module, supporting Cookie engine
6. Performance benchmark for ORM
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

released date: 2014-02-10

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

released date: 2013-12-19
