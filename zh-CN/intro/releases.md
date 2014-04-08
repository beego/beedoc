---
name: 发布版本
sort: 2
---
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