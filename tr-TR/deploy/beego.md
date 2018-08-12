---
name: Stand alone Deployment
sort: 1
---

# Kendi başına çalışan uygulama deploymentı

Uygulamanın arka planda deamon olarak çalıştırılması demektir. (Görev yöneticisinde yeni bir görev yaratmak gibi)

## linux

Linux'ta `nohup` komutunu kullanarak uygulamanın arka planda çalışması şu şekilde sağlanır :

    nohup ./beepkg &

Uygulamanız artık Linux'ta çalışan bir process olarak kalacaktır.

## Windows

Windows'ta başlangıçta otomatik olarak çalıştırmak için iki yol vardır:

1. Bat dosyası olışturmak ve içerisine "Run" eklemek
2. Servis oluşturmak
