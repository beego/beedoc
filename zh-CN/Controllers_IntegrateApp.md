# 集成第三方应用

beego 支持第三方应用的集成，用户可以自定义 `http.Handler`，用户可以通过如下方式进行注册路由：

	beego.RouterHandler("/chat/:info(.*)", sockjshandler)

sockjshandler 实现了接口 `http.Handler`。

目前在 beego 的 example 中有支持 sockjs 的 chat 例子，示例代码如下：

```go
package main

import (
	"fmt"
	"github.com/astaxie/beego"
	"github.com/fzzy/sockjs-go/sockjs"
	"strings"
)

var users *sockjs.SessionPool = sockjs.NewSessionPool()

func chatHandler(s sockjs.Session) {
	users.Add(s)
	defer users.Remove(s)
	
	for {
		m := s.Receive()
		if m == nil {
			break
		}
		fullAddr := s.Info().RemoteAddr
		addr := fullAddr[:strings.LastIndex(fullAddr, ":")]
		m = []byte(fmt.Sprintf("%s: %s", addr, m))
		users.Broadcast(m)
	}
}

type MainController struct {
	beego.Controller
}

func (m *MainController) Get() {
	m.TplName = "index.html"
}

func main() {
	conf := sockjs.NewConfig()
	sockjshandler := sockjs.NewHandler("/chat", chatHandler, conf)
	beego.Router("/", &MainController{})
	beego.RouterHandler("/chat/:info(.*)", sockjshandler)
	beego.Run()
}
```

通过上面的代码很简单的实现了一个多人的聊天室。上面这个只是一个 sockjs 的例子，我想通过大家自定义 `http.Handler`，可以有很多种方式来进行扩展 beego 应用。
