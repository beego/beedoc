# Integrate third-party applications

Beego supports to integrate third-party application, you can customize `http.Handler` as follows:

	beego.Handler("/chat/:info(.*)", sockjs.NewHandler("/chat", opt, YouHandlerFunc))

sockjshandler implemented interface `http.Handler`.

Beego has an example for supporting chat of sockjs, here is the code:

```go
package main

import (
	"fmt"
	"github.com/astaxie/beego"
	"gopkg.in/igm/sockjs-go.v2/sockjs"
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
	beego.Router("/", &MainController{})
	beego.Handler("/chat/:info(.*)", sockjs.NewHandler("/chat", opt, YouHandlerFunc))
	beego.Run()
}
```

The above example implemented a simple chat room for sockjs, and you can use `http.Handler` for more extensions.
