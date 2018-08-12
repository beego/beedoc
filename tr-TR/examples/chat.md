---
name: Chat Room
sort: 1
---

# Chat Odası

Bu demo Anlık Mesajlaşma Uygulaması (web) yapımını iki yol ile gösterecektir :

1. Long polling kullanımı ile yapma
2. WebSocket ile yapma

## WebSocket ile yapma

İki yolla da hafızada duran kayıtlı veriler uygulama baştan başladığında kaybolacaktır. Fakat bunu `conf/app.conf` dosyası içerisinden değiştirebilirsiniz. (database adapter for data persistence kuralını aktif ederek)

Projenin yapısı aşağıdaki gibidir :


```bash
WebIM/
    WebIM.go            # Main paketi dosyası
    conf
        app.conf        # Konfigürasyon dosyası
    controllers
        app.go          # Kullanıcının kullanıcı adı seçebileceği "hoşgeldiniz" ekranı
        chatroom.go     # Veri yönetimi için fonksiyonlar
        longpolling.go  # Long polling demosu için Controller ve metodları
        websocket.go    # WebSocket demosu için Controller and metodları
    models
        archive.go      # Chat veri operasyonları için fonksiyonlar (Her iki demo için)
    views
        ...             # Template dosyaları
    static
        ...             # JavaScript ve CSS dosyaları
```

[Kodu GitHub'ta Görüntüle](https://github.com/beego/samples/tree/master/WebIM)
