---
name: Deployment with Supervisord 
sort: 2
---

# Supervisord

Supervisord Python ile oluşturulmuş çok kullanışlı bir process (süreç) yöneticisidir. Supervisord deamon olmayan uygulamalarınızı deamon uygulamaya çevirir.

Eğer Supervisord kullanarak nginx'i yönetmek isterseniz deamon özelliğini kapatıp nginx'i deamon olmayan modda çalıştırmanız gerekmektedir.

## Supervisord Kurulumu

1. Kurulum dosyalarını yükleme

       wget http://pypi.python.org/packages/2.7/s/setuptools/setuptools-0.6c11-py2.7.egg

       sh setuptools-0.6c11-py2.7.egg

       easy_install supervisor

       echo_supervisord_conf >/etc/supervisord.conf

       mkdir /etc/supervisord.conf.d

2. config `/etc/supervisord.conf`

       [include]
       files = /etc/supervisord.conf.d/*.conf

3. Yönetmek için yeni bir uygulama oluşturulur

       cd /etc/supervisord.conf.d
       vim beepkg.conf

   Konfigürasyonlar :

       [program:beepkg]
       directory = /opt/app/beepkg
       command = /opt/app/beepkg/beepkg
       autostart = true
       startsecs = 5
       user = root
       redirect_stderr = true
       stdout_logfile = /var/log/supervisord/beepkg.log

## Supervisord Yönetimi

Supervisord iki komut ile çalışır, supervisord ve supervisorctl :

* supervisord: Supervisord'u hazır hale getirir ve config edilmiş processleri çalıştırır.
* supervisorctl stop programxxx: programxxx processlerini durdurur. programxxx [program:beepkg] adıyla config edilir. (beepkg)
* supervisorctl start programxxx: Process çalıştırır.
* supervisorctl restart programxxx: Processleri tekrar çalıştırır.
* supervisorctl stop groupworker: Groupworker grubundaki tüm processleri tekrar çalıştırır.
* supervisorctl stop all: Tüm processleri durdurur. Not : start, stop, restart ve stop son configleri yenilemez (reload yapmaz)
* supervisorctl reload: Tüm configleri yeniler
* supervisorctl update: Tüm processleri değişen configleriyle beraber yeniler.

> > > Not : `stop` komutuyla manuel olarak durdurulan processler, reload veya update işlemlerinden sonra yeniden başlamayacaklardır.
