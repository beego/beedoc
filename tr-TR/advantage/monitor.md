---
name: Live Monitor
sort: 1
---

# Canlı Monitör

Daha önce toolbox (araç) modülüne değinmiştik. Bu modül, uygulama çalıştığı zaman `127.0.0.1:8088` adresini dinler. İnternet üzerinden ulaşılamaz ama nginx proxy'si üzerinden görüntülenebilir.

>>> Güvenlik sebebiyle firewall (güvenlik duvarı) üzerinden 8088 portunu engellemeniz önerilir.

Monitör özelliği siz uygulama ayarlarından aktif hale getirmediğiniz sürece kapalıdır. Aktif etmek için aşağıdaki satırı conf/app.conf içerisine eklemeniz gereklidir : 

	EnableAdmin = true

Ayrıca monitör uygulamasının dinleyeceği portu da aynı dosya içerisine aşağıdaki kodları ekleyerek değiştirebilirsiniz : 

	AdminAddr = "localhost"
	AdminPort = 8088

Şimdi `http://localhost:8088/` adresini ziyaret ettiğinizde `Welcome to Admin Dashboard` (Admin Paneline Hoşgeldiniz) yazısını göreceksiniz.


## Request istatistikleri

`http://localhost:8088/qps` adresini ziyaret ettiğinizde aşağıdaki gibi bir sayfa göreceksiniz :

![](../images/monitoring.png)

## Performans profili görüntüleme (Performance profiling)

`goroutine`, `heap`, `threadcreate`, `block`, `cpuprof`, `memoryprof`, `gc summary` tanımlamaları için bilgi edinebilir ve performans görüntülemesi yapabilirsiniz.


## Sağlık Durumu (Healthcheck)

Sağlık durumunu görebilmek için manuel olarak ilgili mantıksal tanımlamaları `http://localhost:8088/healthcheck` adresi üzerinden yapmanız gereklidir.

## Görevler (Tasks)

Uygulama içerisinde yeni bir görev tanımlayabilirsiniz. Sonrasında manuel olarak çalıştırabilir veya yine manuel olarak durumunu kontrol edebilirsiniz.

- Görevin (task) durumunu kontrol etme : `http://localhost:8088/task`
- Görevi manuel olarak çalıştırma : `http://localhost:8088/runtask?taskname=task_name`

## Ayar Durumu (Config Status)

Uygulamayı geliştirdikten sonra hangi uygulama ayarlarını tanımladığımızı bilmek isteyebiliriz. Beego Monitör ayrıca bu ayarları da görüntülemenizi sağlar.

- Tüm ayarları görüntüleme: `http://localhost:8088/listconf?command=conf`
- Tüm yönlendirmeleri (routers) görüntüleme: `http://localhost:8088/listconf?command=router`
- Tüm filtreleri (filters) görüntüleme : `http://localhost:8088/listconf?command=filter`
