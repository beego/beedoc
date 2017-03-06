---
name: Release Notes
sort: 2
---
# beego 1.6.0

New features:

1. `log` supports rotating files like `xx.2013-01-01.2.log` [#1265](https://github.com/astaxie/beego/pull/1265)
2. `context.response` supports Flush, Hijack, CloseNotify
3. ORM supports Distinct [#1276](https://github.com/astaxie/beego/pull/1276)
4. `map_get` template method [#1305](https://github.com/astaxie/beego/pull/1305)
5. ORM supports [tidb](https://github.com/pingcap/tidb) engine [#1366](https://github.com/astaxie/beego/pull/1366)
6. httplib request supports []string [#1308](https://github.com/astaxie/beego/pull/1308)
7. ORM `querySeter` added `GroupBy`  method [#1345](https://github.com/astaxie/beego/pull/1345)
8. Session's MySQL engine supports custom table name [#1348](https://github.com/astaxie/beego/pull/1348)
9. Performance of log's file engine improved 30%; Supports set log file's permission [#1560](https://github.com/astaxie/beego/pull/1560)
10. Get session by query [#1507](https://github.com/astaxie/beego/pull/1507)
11. Cache module supports multiple Cache objects.
12. validation supports custom validation functions

bugfix:

1. `bind` method in `context` caused crash when parameter is empty. [#1245](https://github.com/astaxie/beego/issues/1245)
2. manytomany in ORM reverse error [#671](https://github.com/astaxie/beego/issues/671)
3. http: multiple response.WriteHeader calls [#1329](https://github.com/astaxie/beego/pull/1329)
4. ParseForm uses local timezone while parsing date [#1343](https://github.com/astaxie/beego/pull/1343)
5. Emails sent by log's SMTP engine can't be authorised
6. Fixed some issues in router: `/topic/:id/?:auth`, `/topic/:id/?:auth:int` [#1349](https://github.com/astaxie/beego/pull/1349)
7. Fixed the crash caused by nil while parsing comment documentation. [#1367](https://github.com/astaxie/beego/pull/1367)
8. Can't read `index.html` in static folder
9. `dbBase.Update` doesn't return err if failed [#1384](https://github.com/astaxie/beego/pull/1384)
10. `Required` in `validation` only works for int but not for int64
11. orm: Fix handling of rel(fk) to model with string pk [#1379](https://github.com/astaxie/beego/pull/1379)
12. graceful error while both http and https enabled [#1414](https://github.com/astaxie/beego/pull/1414)
13. If ListenTCP4 enabled and httpaddr is empty, it still listens TCP6
14. migration doesn't support postgres [#1434](https://github.com/astaxie/beego/pull/1434)
15. Default values of ORM text, bool will cause error while creating tables.
16. graceful panic: negative WaitGroup counter

Improvement:

1. Moved example to [samples](https://github.com/beego/samples)
2. Passed golint
3. Rewrote router, improved performance by 3 times.
4. Used `sync.Pool` for `context` to improve performance
5. Improved template compiling speed. [#1298](https://github.com/astaxie/beego/pull/1298)
6. Improved config
7. Refactored whole codebase for readability and maintainability
8. Moved all init code into `AddAPPStartHook`
9. Removed `middleware`. Will only use `plugins`
10. Refactored `Error` handling.

# Beego 1.5.0
New Features:

1. Graceful shutdown
2. Added `JsonBody` method to `httplib` which supporting sending raw body as JSON format
3. Added `AcceptsHtml` `AcceptsXml` `AcceptsJson` methods to `context input`
4. Get config files from Runmode first
5. `httplib` supports `gzip`
6. `log` module stop using asynchronous mode by default
7. `validation` supports recursion
8. Added `apk mime`
9. `ORM` supports `eq` an `ne`

Bugfixes:

1. Wrong parameters for ledis driver.
2. When user refresh the page after the captcha code expired from the cache, it returns 404. Generating new captcha code for reloading.
3. Controller defines Error exception
4. cookie doesn't work in window IE
5. GetIn returns nil error while getting non-exist variable
6. More cellphone validation code
7. Wrong router matching
8. The `panic` returns http 200
9. The database setting erros caused by redis session
10. The issue that https and http don't share session
11. Memcache session driver returns error if it's empty

# Beego 1.4.3
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

# BeeGo 1.4.2
Новые возможности:

1. Added SQL Constructor inspired by ZEND ORM.
2. Added `GetInt()`, `GetInt8()`, `GetInt16()`, `GetInt32()`, `GetInt64()` for Controller
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
11. Added Json function for fetching `statistics` in `toolbox`
12. Attachements support for mail utils.
13. Turn on fastcgi by standard IO
14. Using `SETEX` command to support the old version redis in redis Session engine.
15. RenderForm supports html id and class by using id and class tag.
16. ini config files support BOM head
17. Added new Session engine `ledis`
18. Improved file uploading in `httplib`. Supporting extremely large files by using `io.Pipe`
19. Binding to TCP4 address by default. It will bind to ipv6 in GO.  Added config variable `ListenTCP4`
20. off/on/yes/no/1/0 will parse to `bool` in form rendering. Support time format.
21. Simplify the generating of SeesionID. Using golang buildin `rand` function other than `hmac_sha1`

Испраленны:

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

# BeeGo 1.4.1
Новые возможности:

1. `context.Input.Url` get path info without domain scheme.
2. Added plugin `apiauth` to simulate the `AWS` encrypted requests.
3. Simplified the debug output for router info.
4. Supportting pointer type in ORM.
5. Added `BasicAuth`, cache for multiple requests

Испраленны:
1. Router *.* can't be parsed

# BeeGo 1.3.0
Hi guys! After the hard working for one month, we are so excited to release Beego 1.3.0. We brought many useful features. [Upgrade notes](http://beego.me/docs/intro/upgrade.md)

## The brand new router system
We rewrote the router system to tree router. It improved the performance significantly and supported more formats.

For the routers below:

    /user/astaxie
    /user/:username

If the request is `/user/astaxie`, it will match fixed router which is the first one; If the request is `/user/slene`, it will match the second one. The register order doesn't matter.

## namespace is more elegant
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

## Annotation Router
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

## Automated API Document
Automated document is a very cool feature that I wish to have. Now it became real in BeeGo. As I said BeeGo will not only boost the development of API but also make the API easy to use for the user.

The API document can be generated by annotations automatically and can be tested online.

![](../images/docs.png)


![](../images/doc_test.png)

For more information please check [Automated Document](http://beego.me/docs/advantage/docs.md)

## config supports different Runmode

You can set configurations for different Runmode under their own sections. BeeGo will take the configurations of current Runmode by default. For example:

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

The configurations above set up httpport for dev, prod and test environment. BeeGo will take httpport = 8080 for current runmode "dev".

## Support Two Way Authentication for SSL
```
config := tls.Config{
    ClientAuth: tls.RequireAndVerifyClientCert,
    Certificates: []tls.Certificate{cert},
    ClientCAs: pool,
}
config.Rand = rand.Reader

beego.BeeApp.Server.TLSConfig = &config
```
## beego.Run supports parameter

beego.Run() Run on HttpPort by default

beego.Run(":8089")

beego.Run("127.0.0.1:8089")

## Increased XSRFKEY token from 15 characters to 32 characters.

## Removed hot reload

## Template function supports Config. Get Config value from Template easily.

    {{config returnType key defaultValue}}

    {{config "int" "httpport" 8080}}

## httplib supports cookiejar. Thanks for curvesft

## orm suports time format. If empty return nil other than 0000.00.00 Thanks for JessonChan

## config module supports parsing a json array. Thanks for chrisport

## bug fix
- Fixed static folder infinite loop
- Fixed typo



# BeeGo 1.2.0

Hi guys! After the hard working for one month, we released the new awesome version 1.2.0. BeeGo is the fastest Go framework in the latest [Web Framework Benchmarks](http://www.techempower.com/benchmarks/#section=data-r9&hw=i7&test=json) already though our goal is to make BeeGo the best and easiest framework to use. In this new release, we improved even more in both usability and performance which is closer to native Go.

### Новые возможности:

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
- GET       /v1/notallowed
- GET       /v1/version
- GET       /v1/changepassword
- POST     /v1/changepassword
- GET       /v1/shop/123

`namespace` also supports pre-filters, conditions checking and unlimited nested `namespace`

#### 2. Supporting more flexible router modes

Custom functions from RESTful router

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

More flexible Handler

- beego.Handler(router, http.Handler)

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
httplib also supports custom protocol version

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



# BeeGo 1.1.4

Release an emergency version for BeeGo has a serious security problem, please update to the latest version. By the way released all changes together

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

# BeeGo 1.1.3
this is a hot fixed:

1. console engine for logs.It will not run if there's no config.

2. BeeGo 1.1.2 support `go run main.go`, but if `main.go` bot abute the BeeGo's project rule,use own AppConfigPath or not exist app.conf will panic.

3. BeeGo 1.1.2 supports `go test` parse config,but actually when call TestBeegoInit still can't parseconfig

Дата выхода: 2014-04-04

# BeeGo 1.1.2
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
10. Allowing modifing Context by Filter to skip BeeGo's routering rules and using uder-defined routering rules. Added parameters RunController and RunMethod
11. Supporting to run BeeGo APP by using `go run main.go`
12. Supporting to run test cases by using `go test`. Added TestBeeGoInit function.

Дата выхода: 2014-04-03

# BeeGo 1.1.1
Добавлены некоторые новые возможности и исправлены некоторые проблемы в этом выпуске.

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

Дата выхода: 2014-03-12

# BeeGo 1.1.0
Добавлены некоторые новые возможности и исправлены некоторые проблемы в этом выпуске.

Новые возможности:

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

Исправления:

1. Исправлен баг в FileCache
2. Fixed the import lib of websocket
3. Изменен http статус c 200 на 500 где произошла внутренняя ошибка.
4. gmfim map в memzipfile.go файл должен использовать некоторый механизм синхронизации (например sync.RWMutex) в противном случае выдавать ошибку.
5. Исправлен #440 on_delete bug that not getting deleted automatically
6. Исправлен #441 баг с часовым поясом

Дата выхода: 2014-02-10

# BeeGo 1.0 release
После четырех месяцев рефакторинга, мы выпустили первую стабильную версию BeeGo. Мы много отрефакторили и улучшили в деталях. Список основных улучшений:

1. Модульный дизайн. Сейчас BeeGo легковесный фреймворк включающий 8 назависимых модулейcache, config, logs, sessions, httplibs, toolbox, orm и context. В будущем может быть больше. Вы можете использовать все эти автономные модули в ваших других приложениях напрямую независимо от того, что это веб-приложения или любые другие приложения, такие как веб-игры или игры для мобильных телефонов.

2. Модуль Supervisor. В реальном мире, после развертывания приложения, мы должны иметь много видов статистики и аналитики для приложений, таких как QPS, GC аналитика, мониторинг памяти и процессора и так далее. Когда проблемы случаются в реальном времени мы также хотим отлаживать и профилировать наше приложение в реальном времени. Все эти функций, включаются в BeeGo. Вы можете включить модуль supervisor в BeeGo и посетить его с портом по умолчанию 8088.

3. Подробная документация. Мы переписали все документация. Мы улучшили документация на основании многих советы от пользователей. Чтобы сделать общение проще на разных языках, комментарии документа на каждом языке разделяются.

4. Демо. Мы предоставили три примера, чат, URL Shortener и todo список. Вы можете понять и использовать BeeGo проще и быстрее, изучая демо.

5. Переработанный BeeGo сайт. Хорошие люди из BeeGo сообщества помогли BeeGo в дизайне логотипа и веб-дизайна.

6. Все больше и больше пользователей. Мы перечислили наши типичные пользователи в нашей домашней странице. Они все крупные компании, и они используют BeeGo для своих продуктов уже. BeeGo уже проходят проверку на этих живых приложениях.

7. Рост активного сообщества. Есть более чем 390 issue, на GitHub, более 36 contributors и более 700 commits. Группа Google также растет.

8. Все больше и больше приложений в BeeGo. Есть некоторые приложения с открытым исходным кодом, а также. Система CMS:: Например, https://github.com/insionng/toropress и админка: https://github.com/BeeGo/admin

9. Мощные инструменты помощи. bee используется для оказания помощи в разработке приложений BeeGo. Он может легко создать, собрать, упаковать приложение BeeGo.

Дата выхода: 2013-12-19
