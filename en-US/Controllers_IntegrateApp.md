# Integrate third-party applications

Beego supports to integrate third-party application, you can customize `http.Handler` as follows:

	beego.RouterHandler("/chat/:info(.*)", sockjshandler)

sockjshandler implemented interface `http.Handler`.

Beego has an example for supporting chat of sockjs, here is the code:

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
	m.TplNames = "index.html"
}

func main() {
	conf := sockjs.NewConfig()
	sockjshandler := sockjs.NewHandler("/chat", chatHandler, conf)
	beego.Router("/", &MainController{})
	beego.RouterHandler("/chat/:info(.*)", sockjshandler)
	beego.Run()
}
```

The above example implemented a simple chat room for sockjs, and you can use `http.Handler` for more extensions.
