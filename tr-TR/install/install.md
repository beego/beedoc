---
root: true
name: Install / Upgrade
sort: 1
---

# Beego kurulumu

Klasik Go yöntemiyle Beego'yu yükleyebilirsiniz :

    go get github.com/astaxie/beego

Sıkça karşılaşılan durumlar ve çözümleri :

* git is not installed :

  git sisteminizde kurulu değil. Lütfen git yükleyiniz.

* "git https is not accessible. Please config local git and close https validation hatası" çözümü:

  git config --global http.sslVerify false

* Beego'yu internet bağlantısı olmadan nasıl yüklerim?

  Şu an bunun için iyi bir çözümümüz yoktur. İndirip yükleme yapmanız için gelecek sürümlerde paketler oluşturacağız.

# Beego'yu güncellemek

Beego'yu Go komutu ile güncelleyebilirsiniz. Ya da Beego kaynak kodlarını indirip güncelleme işlemini yapabilirsiniz.

* Go komutu ile güncelleme (Önerilen):

  go get -u github.com/astaxie/beego

* `https://github.com/astaxie/beego` adresini ziyaret ederek kaynak kodu indirebilirsiniz. Daha sonra bu dosyaları kopyalayıp `$GOPATH/src/github.com/astaxie/beego` dizini içerisindeki dosyaların üzerine yazın. Son adım olarak `go install` komutu ile güncelleme işlemini tamamlayın :

	go install github.com/astaxie/beego

**1.0 Sürümü Öncesine Güncelleme:**

Beego API 1.0 sürümünden sonra gelen güncellemelerle kararlı hale gelmiştir. Fakat 1.0'dan daha düşük bir versiyon kullanıyorsanız son API sürümünü baz alarak uygulama parametrelerinizi ayarlamanız gerekebilir.

