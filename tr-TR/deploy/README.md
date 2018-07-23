---
root: true
name: Deployment
sort: 7
---

# Releasing ve deploy

### Development modu

Uygulamayı `bee` komutu ile oluşturduğunuzda default olarak development modu aktif olur.

Bunu değiştirmek için aşağıdaki parametreyi ekleyebilirsiniz

    beego.RunMode = "prod"

ya da app/app.config dizininde bulunan uygulama ayarlarınızı açabilir ve `runmode` değerini aşağıdaki gibi değiştirebilirsiniz :

    runmode = prod

Development modundayken :

* Eğer `views` klasörünüz yoksa aşağıdaki gibi bir hata alırsınız :

      		2013/04/13 19:36:17 [W] [stat views: no such file or directory]

* Templateler her zaman cache (önbellek) kullanılmadan yüklenecektir.

* Eğer sunucu hata verirse yanıt aşağıdaki gibi olacaktır :

![](./../images/dev.png)

### Releasing ve Deploying

Go uygulaması derlendikten sonra bytecode dosyası haline gelir. Siz de sadece bu dosyayı sunucunuza kopyalayıp uygulamanızı çalıştırabilirsiniz. Fakat unutmayın Beego statik dosyalar,konfigurasyon dosyaları ve templateler gibi 3 klasörü içerebilir. Bu nedenle deploy yaparken bu klasörleri de sunucuya kopyalamalısınız :

    $ mkdir /opt/app/beepkg
    $ cp beepkg /opt/app/beepkg
    $ cp -fr views /opt/app/beepkg
    $ cp -fr static /opt/app/beepkg
    $ cp -fr conf /opt/app/beepkg

`/opt/app/beepkg` dizini içerisindeki dosya yapısı aşağıdaki gibi olur:

    .
    ├── conf
    │   ├── app.conf
    ├── static
    │   ├── css
    │   ├── img
    │   └── js
    └── views
        └── index.tpl
    ├── beepkg

Tüm uygulamamızı sunucuya kopyaladık. Sıradaki adım ise yayınlamak olacak.

Bunun için iki yol vardır:

* [Kendi başına çalışan uygulama deploymentı](./beego.md)
* [Supervisord ile deploy](./supervisor.md)

Uygulamalarımızda genellikle sayfaları yayınlamak için ve yük dengeleme (load balancing) yapabilmek için nginx veya apache kullanırız. Bunun için bilgiler :

* [Nginx ile deploy](./nginx.md)
* [Apache ile deploy](./apache.md)
