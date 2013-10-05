# 基本说明

该手册详细描述了 beego 应用框架的各个方面。

### 新手指导

如果您是首次了解 beego，我们建议您先阅读 [框架概念](/docs/Overview_Concepts)。

### 其它资源

如果您无法找到相关问题的满意答案，可以在 [邮件列表 (beego-framework@googlegroups.com)](https://groups.google.com/forum/#!forum/beego-framework) 中搜索。

### 寻求帮助

如果您遇到无法解决的问题，可以发送邮件到 [beego-framework@googlegroups.com](mailto:beego-framework@googlegroups.com) 提问。

### 问题提交

如果您在使用过程中发现 beego 有潜在的问题，请通过 [Github](https://github.com/astaxie/beego/issues) 上的问题列表提交。

### 完善文档

您可以通过派生 [文档项目](https://github.com/beego/beedoc)、编辑，然后提交合并请求。

### 简单的 API 文档

请移步 [Go Walker](http://gowalker.org/github.com/astaxie/beego)。

### 网站源码

请移步 [Beego Web](https://github.com/beego/beeweb)。

### 第三方依赖
其实目前beego的运行基本都不需要依赖其他第三方的库，但是由于一些模块为了支持各种功能，所以引入了第三方库，以下一一介绍一下各种第三方库的作用：

- code.google.com/p/vitess/go/memcache cache库支持memcahe支持

- github.com/garyburd/redigo/redis session、cache支持redis支持

- github.com/clbanning/x2j config解析xml支持

- github.com/wendal/goyaml2 config解析yaml支持

- github.com/go-sql-driver/mysql mysql数据库引擎，session支持mysql存储
