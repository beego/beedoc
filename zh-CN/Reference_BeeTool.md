# Bee 命令行工具

Bee是为了快速开发beego应用的辅助工具.

获取Bee  `go get github.com/beego/bee` 注意Bee已经从github.com/astaxie/bee 移至 github.com/beego/bee

它有如下几个命令

    new         创建一个新的Web应用
    run         实现热编译的启动应用，能够监控当前目录下的Go文件变更
    pack        把当前项目进行打包，如果有静态文件、视图就可以打包直接cp到服务器部署
    api         创建一个API应用
    router      目前这个还在开发中，能够自动生成路由器、Model等

我们一般都会根据需求创建应用，就和快速入门这样的一样，首先`bee new 项目名`,然后你执行`bee run 项目名`,开发完毕之后进行打包操作`bee pack 项目名`
