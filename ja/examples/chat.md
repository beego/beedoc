---
chat.md
name: チャットルーム
sort: 1
---

# チャットルーム

このサンプルでは、Web IMアプリケーションを実装する2つの方法を示しています:

ロングポーリングを使用しています。
WebSocketを使用しています。

どちらもデフォルトではメモリにデータを保存していますので、毎回やり直してスタートしますが、データベースを使用するために`conf/app.conf`内で設定を変更することができます。

プロジェクト構造は、次のとおりです:

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

[GitHub上でコードを見る](https://github.com/beego/samples/tree/master/WebIM)
