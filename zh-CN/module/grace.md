---
name: grace 模块
sort: 1
---

# 热升级是什么？

热升级是什么呢？了解nginx的同学都知道，nginx是支持热升级的，可以用老进程服务先前链接的链接，使用新进程服务新的链接，即在不停止服务的情况下完成系统的升级与运行参数修改。那么热升级和热编译是不同的概念，热编译是通过监控文件的变化重新编译，然后重启进程，例如`bee run`就是这样的工具

# 热升级有必要吗？

很多人认为HTTP的应用有必要支持热升级吗？那么我可以很负责的说非常有必要，不中断服务始终是我们所追求的目标，虽然很多人说可能服务器会坏掉等等，这个是属于高可用的设计范畴，不要搞混了，这个是可预知的问题，所以我们需要避免这样的升级带来的用户不可用。你还在为以前升级搞到凌晨升级而烦恼嘛？那么现在就赶紧拥抱热升级吧。

# grace模块

grace模块是beego新增的一个独立支持热重启的模块。主要的思路来源于： http://grisha.org/blog/2014/06/03/graceful-restart-in-golang/



# 如何使用热升级

```
 import(
   "log"
	"net/http"
	"os"
    "strconv"

   "github.com/astaxie/beego/grace"
 )

  func handler(w http.ResponseWriter, r *http.Request) {
	  w.Write([]byte("WORLD!"))
      w.Write([]byte("ospid:" + strconv.Itoa(os.Getpid())))
  }

  func main() {
      mux := http.NewServeMux()
      mux.HandleFunc("/hello", handler)

	    err := grace.ListenAndServe("localhost:8080", mux1)
      if err != nil {
		   log.Println(err)
	    }
      log.Println("Server on 8080 stopped")
	     os.Exit(0)
    }
```


打开两个终端

一个终端输入：`ps -ef|grep 应用名`

一个终端输入请求：`curl "http://127.0.0.1:8080/"`

热升级

kill -HUP 进程ID

打开一个终端输入请求：`curl "http://127.0.0.1:8080/?sleep=0"`
