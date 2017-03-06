# Integrate third-party applications

Beego supports to integrate third-party application, you can customize `http.Handler` as follows:

	beego.Handler("/chat/:info(.*)", sockjs.NewHandler("/chat", opt, YouHandlerFunc))

sockjshandler implemented interface `http.Handler`.

Beego has an example for supporting echo app of sockjs, here is the code:

```go
package main

import (
	"log"

	"time"

	"gopkg.in/igm/sockjs-go.v2/sockjs"
)

func LiveUpdate(session sockjs.Session) {
	var closedSession = make(chan struct{})
	reader := make(chan string)
	go func() {
		for {
			reader <- "echo"
			time.Sleep(time.Second)
		}
	}()
	go func() {
		for {
			select {
			case <-closedSession:
				return
			case msg := <-reader:
				if err := session.Send(msg); err != nil {
					return
				}
			}
		}
	}()
	for {
		if _, err := session.Recv(); err == nil {
			continue
		}
		break
	}
	close(closedSession)
	log.Println("sockjs session closed")
}

type MainController struct {
	beego.Controller
}

func (m *MainController) Get() {
	m.TplNames = "index.html"
}

func main() {
	beego.Router("/", &MainController{})
	beego.Handler("/chat/:info(.*)", sockjs.NewHandler("/chat", sockjs.DefaultOptions, YouHandlerFunc))
	beego.Run()
}
```
JS code:
```
if (!window.location.origin) { // Some browsers (mainly IE) do not have this property, so we need to build it manually...
    window.location.origin = window.location.protocol + '//' + window.location.hostname + (window.location.port ? (':' + window.location.port) : '');
}
var recInterval = null;
var socket = null;


var new_conn = function() {
    socket = new SockJS('/live_update', null, {
        'protocols_whitelist': ['websocket', 'xdr-streaming', 'xhr-streaming',
            'iframe-eventsource', 'iframe-htmlfile',
            'xdr-polling', 'xhr-polling', 'iframe-xhr-polling',
            'jsonp-polling'
        ]
    });
    clearInterval(recInterval);

    socket.onclose = function() {
        socket = null;
        recInterval = setInterval(function() {
            new_conn();
        }, 2000);
    };
    socket.onmessage = function(e) {
        document.getElementById("output").value += e.data +"\n";
    };
};

(function () {
    new_conn();
})();

```
The above example implemented a simple echo app for sockjs, and you can use `http.Handler` for more extensions.
