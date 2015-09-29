# Интеграция сторонних приложений

BeeGo поддерживает интеграцию сторонних приложений, вы можете настроить `http.Handler`:

	beego.RouterHandler("/chat/:info(.*)", sockjshandler)

sockjshandler реализует интервейс `http.Handler`.

Пример BeeGo чата с sockjs, ниже код:

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

Пример выше реализует простой чат с sockjs, и как вы можете использовать `http.Handler` для расширения.
