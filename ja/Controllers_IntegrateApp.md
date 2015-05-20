# サードパーティ製のアプリケーションを統合

beegoは、サードパーティ製のアプリケーションの統合をサポートし、次のように`http.Handler`カスタマイズすることができます:

	beego.RouterHandler("/chat/:info(.*)", sockjshandler)

`sockjshandler`は`http.Handler`インターフェイスを実装しています。

beegoはSockJSのチャットをサポートするための例があり、コードは次のとおりです:

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

上記の例では、 SockJSのためのシンプルなチャットルームを実装し、より多くの拡張のために`http.Handler`を使用することができます
