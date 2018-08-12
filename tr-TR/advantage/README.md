---
root: true
name: Advanced Beego
sort: 6
---

# Gelişmiş Beego

Beego'nun basit kullanım şekillerini incelemiştik. Şimdi de daha gelişmiş kullanım şekilleri üzerinde konuşacağız.

- [Canlı Monitör](./monitor.md)

  Beego default olarak 2 tane port sunar. İlk olan 8080 portu uygulamayı kullanıcılara sunmak içindir. Diğer port olan 8088 portu üzerinde ise monitör uygulaması çalışır. Burada işlem durumları ve görevleri çalıştırma gibi özellikler vardır.

	
- [Filtreler](../mvc/controller/filter.md)

  Filtreler uygulama mantığınızı genişletmek için çok işe yarar özelliklere sahiptir. Yetkilendirme (authentication), ziyaret loglama, uygunluk (compatibility) değiştirme bunlardan bazılarıdır.

	
- [Yeniden Yükleme (Reload)](./reload.md)

  Yeniden yükleme, kullanıcı isteklerini kesmeden uygulamayı deploy etmenizi (sunucuya yüklemenizi) sağlar.


>>> Bu özellik henüz tam olarak tamamlanmamıştır. Sadece Mac ve Linux'te test edilmiştir. Production (canlı) ortamda denenmemiştir ve hala üzerinde testler yapılmaktadır. Kullanım durumunda oluşacak riskler kullanıcıya aittir. Önerilen durum nginx (upsteam) üzerinden kullanmaktır.
