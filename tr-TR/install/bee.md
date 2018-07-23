---
name: bee tool usage
sort: 2
---

# bee aracına giriş

bee aracı hızlı Beego geliştirmesi yapmak içindir. bee aracı ile kolayca

* Beego uygulamaları oluşturabilirsiniz
* Otomatik derleme ve yenileme (auto compile & reload) yapabilirsiniz
* Geliştirme, test ve deploy yapabilirsiniz

## bee aracının kurulumu

Aşağıdaki komutu çalıştırarak bee aracını yükleyiniz :

    go get github.com/beego/bee

`bee`, sabit olarak `GOPATH/bin` dizininde yüklenecektir.

Not : `bee` komutunun çağrıldığında çalışması için `GOPATH/bin` dizininin sistem PATH'inizde ekli olması gerekmektedir.

# bee aracı komutları

Komut satırına `bee` yazdığınızda aşağıdaki mesaj görüntülenecektir :

```
bee is a tool for managing Beego framework.

Usage:

	bee command [arguments]

The commands are:

	new         Create a Beego application
	run         run the app and start a Web server for development
	pack        Compress a Beego project into a single file
	api         create an API Beego application
	bale        packs non-Go files to Go source files
	version     show the bee, Beego and Go version
	generate    source code generator
	migrate     run database migrations
```

### Komut : `new`

`new` komutu yeni bir web projesi oluşturur. `bee new <proje adı>` komutunu `$GOPATH/src` dizini altında çalıştırarak yeni bir Beego projesi oluşturabilirsiniz. Bu komut sabit olarak aşağıdaki proje klasörlerini ve dosyalarını oluşturur :

```
bee new myproject
[INFO] Creating application...
/gopath/src/myproject/
/gopath/src/myproject/conf/
/gopath/src/myproject/controllers/
/gopath/src/myproject/models/
/gopath/src/myproject/static/
/gopath/src/myproject/static/js/
/gopath/src/myproject/static/css/
/gopath/src/myproject/static/img/
/gopath/src/myproject/views/
/gopath/src/myproject/conf/app.conf
/gopath/src/myproject/controllers/default.go
/gopath/src/myproject/views/index.tpl
/gopath/src/myproject/main.go
13-11-25 09:50:39 [SUCC] New application successfully created!
```

```
myproject
├── conf
│   └── app.conf
├── controllers
│   └── default.go
├── main.go
├── models
├── routers
│   └── router.go
├── static
│   ├── css
│   ├── img
│   └── js
├── tests
│   └── default_test.go
└── views
    └── index.tpl

8 directories, 4 files
```

### Komut : `api`

`new` komutu yeni bir web uygulaması oluşturmak içindi. `api` komutu ise yeni bir API uyguluması oluşturmak içindir.

`bee api <proje adı>` komutu çalıştığında sonuç aşağıdaki gibi olur :

```
bee api apiproject
create app folder: /gopath/src/apiproject
create conf: /gopath/src/apiproject/conf
create controllers: /gopath/src/apiproject/controllers
create models: /gopath/src/apiproject/models
create tests: /gopath/src/apiproject/tests
create conf app.conf: /gopath/src/apiproject/conf/app.conf
create controllers default.go: /gopath/src/apiproject/controllers/default.go
create tests default.go: /gopath/src/apiproject/tests/default_test.go
create models object.go: /gopath/src/apiproject/models/object.go
create main.go: /gopath/src/apiproject/main.go
```

Oluşturduğunuz yeni API uygulamasının proje yapısı aşağıdaki gibi olacaktır :


```
apiproject
├── conf
│   └── app.conf
├── controllers
│   └── object.go
│   └── user.go
├── docs
│   └── doc.go
├── main.go
├── models
│   └── object.go
│   └── user.go
├── routers
│   └── router.go
└── tests
    └── default_test.go
```

Bu proje yapısıyla `bee new <proje adı>` komutuyla oluşturduğunuz projenin farkına bakınız.

API uygulamasının `static` ve `views` klasörlerine sahip olmadığını göreceksiniz.

Ayrıca mevcutta bir veritabanınız varsa Beego veritabanı bağlantınız üzerinden uyumlu model ve controller'ları oluşturabilir.

Örnek :

`bee api [appname] [-tables=""] [-driver=mysql] [-conn=root:@tcp(127.0.0.1:3306)/test]`

### Komut : `run`

`bee run` komutu [inotify](http://en.wikipedia.org/wiki/Inotify) kullanarak Beego projesinin dosya sistemini denetler ve çalışmasını sağlar. Beego proje klasörleri içerisinde yapılan değişiklikler otomatik olarak derlenir (autocompile) ve görüntülenir.

`bee run` komutu çalıştırıldığında komut satırında aşağıdaki gibi bir sonuç görünecektir :

```
13-11-25 09:53:04 [INFO] Uses 'myproject' as 'appname'
13-11-25 09:53:04 [INFO] Initializing watcher...
13-11-25 09:53:04 [TRAC] Directory(/gopath/src/myproject/controllers)
13-11-25 09:53:04 [TRAC] Directory(/gopath/src/myproject/models)
13-11-25 09:53:04 [TRAC] Directory(/gopath/src/myproject)
13-11-25 09:53:04 [INFO] Start building...
13-11-25 09:53:16 [SUCC] Build was successful
13-11-25 09:53:16 [INFO] Restarting myproject ...
13-11-25 09:53:16 [INFO] ./myproject is running...
```

Sonrasında uygulamanız çalışırken `http://localhost:8080/` adresini ziyaret ettiğinizde sonuç aşağıdaki gibi olacaktır :

![](../images/beerun.png)

Eğer uygulama çalışırken `controllers` klasörü içerisindeki `default.go` dosyasını değiştirirseniz komut satırı çıktısı şu şekilde olacaktır :

```
13-11-25 10:11:20 [EVEN] "/gopath/src/myproject/controllers/default.go": DELETE|MODIFY
13-11-25 10:11:20 [INFO] Start building...
13-11-25 10:11:20 [SKIP] "/gopath/src/myproject/controllers/default.go": CREATE
13-11-25 10:11:23 [SKIP] "/gopath/src/myproject/controllers/default.go": MODIFY
13-11-25 10:11:23 [SUCC] Build was successful
13-11-25 10:11:23 [INFO] Restarting myproject ...
13-11-25 10:11:23 [INFO] ./myproject is running...
```

Sonrasında yaptığınız değişikleri görmek için tarayıcınızı yenileyebilirsiniz.

### Komut : `pack`

`pack` komutu proje dosyalarınızı sıkıştırarak tek dosya haline getirir. Sıkıştırılmış zip dosyasını sunucunuza yükleyip çıkardıktan sonra deploy işlemini yapabilirsiniz.

```
bee pack
app path: /gopath/src/apiproject
GOOS darwin GOARCH amd64
build apiproject
build success
exclude prefix:
exclude suffix: .go:.DS_Store:.tmp
file write to `/gopath/src/apiproject/apiproject.tar.gz`
```

Sıkıştırılmış dosya, siz komutu çalıştırdıktan sonra proje klasörü içinde olacaktır :

```
rwxr-xr-x  1 astaxie  staff  8995376 11 25 22:46 apiproject
-rw-r--r--  1 astaxie  staff  2240288 11 25 22:58 apiproject.tar.gz
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 conf
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 controllers
-rw-r--r--  1 astaxie  staff      509 11 25 22:31 main.go
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 models
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 tests
```

### Komut : `bale`

Bu komut şu an için sadece geliştirici takım için erişilebilir durumdadır. Projedeki tüm statik dosyaları tek binary dosya haline getirir. Deployment yaparken js,css,resim ve view tipindeki dosyaları ayrıca sunucuya taşımanız gerekmez. Bu dosyalar uygulama başladığında üzerlerine yazılamayacak şekilde kendi kendilerini çıkarırlar (self-extracting).

### Komut : `version`

Bu komut `bee`, `beego`, ve `go` versiyonlarını görüntüler :

```
$ bee version
bee   :1.2.2
Beego :1.4.2
Go    :go version go1.3.3 darwin/amd64
```

### Komut : `generate`

Bu komut controller dosyaları içerisindeki fonksiyonlara bakarak router'lar oluşturur.

```
bee generate scaffold [scaffoldname] [-fields=""] [-driver=mysql] [-conn="root:@tcp(127.0.0.1:3306)/test"]
    The generate scaffold command will do a number of things for you.
    -fields: a list of table fields. Format: field:type, ...
    -driver: [mysql | postgres | sqlite], the default is mysql
    -conn:   the connection string used by the driver, the default is root:@tcp(127.0.0.1:3306)/test
    example: bee generate scaffold post -fields="title:string,body:text"

bee generate model [modelname] [-fields=""]
    generate RESTful model based on fields
    -fields: a list of table fields. Format: field:type, ...

bee generate controller [controllerfile]
    generate RESTful controllers

bee generate view [viewpath]
    generate CRUD view in viewpath

bee generate migration [migrationfile] [-fields=""]
    generate migration file for making database schema update
    -fields: a list of table fields. Format: field:type, ...

bee generate docs
    generate swagger doc file

bee generate test [routerfile]
    generate testcase

bee generate appcode [-tables=""] [-driver=mysql] [-conn="root:@tcp(127.0.0.1:3306)/test"] [-level=3]
    generate appcode based on an existing database
    -tables: a list of table names separated by ',', default is empty, indicating all tables
    -driver: [mysql | postgres | sqlite], the default is mysql
    -conn:   the connection string used by the driver.
             default for mysql:    root:@tcp(127.0.0.1:3306)/test
             default for postgres: postgres://postgres:postgres@127.0.0.1:5432/postgres
    -level:  [1 | 2 | 3], 1 = models; 2 = models,controllers; 3 = models,controllers,router
```

### Komut : `migrate`

Bu komut database migration scriptlerini çalıştırır.

```
bee migrate [-driver=mysql] [-conn="root:@tcp(127.0.0.1:3306)/test"]
    run all outstanding migrations
    -driver: [mysql | postgresql | sqlite], the default is mysql
    -conn:   the connection string used by the driver, the default is root:@tcp(127.0.0.1:3306)/test

bee migrate rollback [-driver=mysql] [-conn="root:@tcp(127.0.0.1:3306)/test"]
    rollback the last migration operation
    -driver: [mysql | postgresql | sqlite], the default is mysql
    -conn:   the connection string used by the driver, the default is root:@tcp(127.0.0.1:3306)/test

bee migrate reset [-driver=mysql] [-conn="root:@tcp(127.0.0.1:3306)/test"]
    rollback all migrations
    -driver: [mysql | postgresql | sqlite], the default is mysql
    -conn:   the connection string used by the driver, the default is root:@tcp(127.0.0.1:3306)/test

bee migrate refresh [-driver=mysql] [-conn="root:@tcp(127.0.0.1:3306)/test"]
    rollback all migrations and run them all again
    -driver: [mysql | postgresql | sqlite], the default is mysql
    -conn:   the connection string used by the driver, the default is root:@tcp(127.0.0.1:3306)/test
```

## bee aracı konfigürasyonu

`bee.json`, bee aracı kaynak kodu klasörünün altında olan Beego konfigürasyon dosyasıdır. Bu dosya hala geliştirilmektedir fakat bazı seçenekler kullanıma hazırdır :

* `"version": 0`:
    Dosyanın versiyonu,uyumlu olmayan format versiyon kontrolü.
* `"go_install": false`:
    Eğer tüm pathleri projeye dahil ettiyseniz (örn : `github.com/user/repo/subpkg`) `go install` komutunu çalıştırarak derlenme sürecinizi hızlandırabilirsiniz.
* `"watch_ext": []`:
    Diğer dosya uzantılarını watch (değişiklik izleme) moduna ekler. (default olarak sadece `.go` uzantılı dosyalar watch modundadır). Örnek parametreler : `.ini`, `.conf` vb'dir.
* `"dir_structure":{}`:
    Eğer klasör isimleriniz MVC klasik adlandırmasıyla aynı değilse bu seçenek ile değişiklik yapabilirsiniz.
* `"cmd_args": []`: 
    Her çalışma durumu için yeni parametreler girmenizi sağlar.
* `"envs": []`: 
    Her çalışma durumu için sistem değişkenlere (enviroment variables) değer atamanızı sağlar.
