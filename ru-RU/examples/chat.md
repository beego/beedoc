---
name: Chat Room
sort: 1
---

# Чат

Это простая демонстрация двух путей, которыми можно реализовать Web IM приложение:

Используя long polling.
Используя WebSocket.

Оба способа, по умолчанию, сохраняют данные в памяти так, что каждый раз перезапуская приложение вы будете терять данные, но вы можете изменить настройки в файле `conf/app.conf`, чтобы включить использование базы данных.

Вот пример организации проекта:

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

[Посмотреть код на GitHub](https://github.com/beego/samples/tree/master/WebIM)
