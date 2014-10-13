# Integra aplicaciones de terceros

Beego soporta integración con aplicaciones de terceros, y puedes personalizar `http.Handler` de la siguiente manera:

	beego.RouterHandler("/chat/:info(.*)", sockjshandler)

sockjshandler implementa una interface `http.Handler`.

Beego tiene un ejemplo para soportar chat de sockjs, aquí el código:

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

El ejemplo de arriba implementa una sala simple de chat para sockjs, y puedes usar `http.Handler` para más extensiones.
