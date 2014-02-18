---
name: Chat Room
sort: 1
---

# Chat Room

This sample demonstrates two ways to implment Web IM app:

Using long polling.
Using WebSocket.

Both of them save data in memory as default so everything will start over every time, but you can change setting in conf/app.conf to enable database use.

Here is the project organization:

```bash
WebIM/
    WebIM.go            # File of main package
    conf
        app.conf        # Configuration file
    controllers
        app.go          # The welcome screen that allows user to pick a technology and username
        chatroom.go     # Functions for data management
        longpolling.go  # Controller and methods for long polling chat demo
        websocket.go    # Controller and methods for WebSocket chat demo
    models
        archive.go      # Functions of chat data operations for both demos.
    views
        ...             # Template files
    static
        ...             # JavaScript and CSS files
```

[Browse the code on GitHub](https://github.com/beego/samples/tree/master/WebIM)
