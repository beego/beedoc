---
root: true
name: FAQ
sort: 99
---

# S.S.S. (Sıkça Sorulan Sorular)

1. Template dosyaları veya konfigürasyon dosyaları bulunamadı (nil pointer) hatası alıyorum ? Nasıl çözülür?

   Uygulamayı `go run main.go` komutu ile çalıştırdığınızdan kaynaklanabilir. `go run` main dosyasını compile eder ve çıktısını tmp klasörüne koyarak uygulamayı çalıştırır. Fakat Beego'nun statik dosyalara, templatelere ve config dosyasına da ihtiyacı vardır. Bu nedenle `go build` komutunu çalıştırıp `./app` dizini içerisinden uygulamayı çalıştırmanız gerekmektedir. Ya da `bee run app` komutuyla da uygulamanızı çalıştırabilirsiniz.

2. Beego production ortamı için kullanılabilir mi?

   Evet. Beego production ortamında kullanılmıştır. Örneğin SNDA'in CDN sisteminde, 360 arama API'nda, Bmob mobile cloud API'nda, weico backend API'nda kullanılmıştır. Hepsi yüksek dağıtılabilir ve yüksek performanslı uygulamalardır.

3. Gelecek güncellemeler şu an kullandığım API'yı etkileyecek mi?

   Beego 0.1 versiyonundan beri kararlı API durumunda kalmaktadır. Birçok uygulama son Beego sürümüne kolayca yükseltilmiştir. Bizler gelecekte de API'ı kararlı bir durumda tutmaya çalışacağız.

4. Beego geliştirilmeye devam edecek mi?

   Birçok insan açık kaynak projelerin gelişiminin durması hakkında endişe yaşıyor. Bizler koda katkıda bulunan dört insanız ve Beego'nun çok daha iyi olmasını sağlayacağız.
