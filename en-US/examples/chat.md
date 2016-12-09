---
name: Chat Room
sort: 1
---

# Chat Room

This demo shows two ways of implementing a Web Instant Messaging application:

Using long polling.
Using WebSocket.

Both of them save data in memory by default so everything will be lost every time the application restarts, but you can change this setting in `conf/app.conf` to enable a database adapter for data persistence.

Here is the project structure:

```bash
WebIM/
    WebIM.go            # File of main package
    conf
        app.conf        # Configuration file
    controllers
        app.go          # The welcome screen that allows the user to pick a technology and username
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
