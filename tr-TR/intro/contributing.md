---
name: Contributing
sort: 1
---

# Beego'ya katkıda bulunmak

## Giriş

Beego ücretsiz ve açık kaynak bir yazılım olmakla birlikte herhangi bir kişi Apache 2.0 Lisansı (http://www.apache.org/licenses/LICENSE-2.0.html) altında yazılımın geliştirilmesine katkıda bulunabilir. Beego'nun kaynak kodları github üzerinde yayınlanmaktadır (https://github.com/astaxie/beego).

### Nasıl Beego destekçisi olabilirim ?

Fork yapabilir, değiştirebilir ve bize Pull Request gönderebilirsiniz.
Bizler kodunuzu inceleyeceğiz ve mümkün olan en kısa zamanda değişiklikleriniz hakkında geri bildirim vereceğiz.

## Pull Request'ler

Yeni geliştirmeler ve hata düzeltmeleri için pull request süreçleri aynı değildir.

### Hata düzeltmeleri (Bug fixes)

Hata düzeltmeleri için yeni bir issue oluşturmanız gerekli değildir. Eğer hatayı çözmek için çözümünüz varsa lütfen pull request gönderirken çözümünüzün açıklamasını belirtiniz.

### Dökümantasyon geliştirmeleri

[beedoc](https://github.com/beego/beedoc) projesine pull request göndererek dökümantasyonun geliştirilmesine yardımcı olabilirsiniz.

### Yeni özellik önerileri

Yeni bir özellik için pull request göndermeden önce `[Proposal]` başlığı altında yeni bir issue oluşturmalısınız. Burada yeni özelliğin tanımını ve yaklaşımını da belirtmeniz gerekmektedir.

Öneriler çekirdek geliştirici takımındaki kişiler tarafından incelenecek ve tartışılacaktır. Bunun sonucunda kabul edilecek veya reddedilecektir.

Öneriniz kabul edildiğinde yeni özelliği eklemek için implementasyonlarınızı oluşturun ve pull request olarak gönderin. Eğer rehberdeki adımlar takip edilmezse pull request'iniz kabul edilmeyecektir.

Beego [Git Flow](http://nvie.com/posts/a-successful-git-branching-model/) branch modelleme


Before you submit a pull request for a new feature, you should first create an
issue with `[Proposal]` in the title, describing the new feature, as well as the
implementation approach.

Proposals will be reviewed and discussed by the core contributors, and can be
adopted or potentially rejected.

Once a proposal is accepted, create an implementation of the new features and
submit it as a pull request. If the guidelines are not followed the pull
request will be rejected immediately.

Since Beego follows the [Git Flow](http://nvie.com/posts/a-successful-git-branching-model/)
branching model, ongoing development happens in the `develop` branch. Therefore,
please base your pull requests on the HEAD of the `develop` branch.

### The git branches of Beego

The master branch is relatively stable and the dev branch is for developers. Here is a
sample figure to show you how our branches work:

![](../images/git-branch-1.png)

For more information about the branching model: http://nvie.com/posts/a-successful-git-branching-model/
