# Giriş

> Not: Bu kitap baskısı, [No Starch Press][nsp] tarafından basılı ve e-kitap formatında yayınlanan [The Rust Programming Language][nsprust] kitabıyla aynıdır.

[nsprust]: https://nostarch.com/rust-programming-language-3rd-edition
[nsp]: https://nostarch.com/

_Rust Programlama Dili_'ne hoş geldiniz, Rust hakkında giriş niteliğinde bir kitap. Rust programlama dili size daha hızlı ve daha güvenilir yazılımlar yazmanıza yardımcı olur. Yüksek seviyeli ergonomi ve düşük seviyeli kontrol genellikle programlama dili tasarımında çelişkilidir; Rust bu çatışmaya meydan okur. Güçlü teknik kapasite ve harika bir geliştirici deneyimi arasındaki dengeyi sağlayarak, Rust size geleneksel olarak bu tür kontrolle ilişkilendirilen tüm sıkıntılar olmadan düşük seviyeli detayları (örneğin bellek kullanımı) kontrol etme seçeneği sunar.

## Rust Kimler İçin

Rust çeşitli nedenlerle birçok kişi için idealdir. En önemli gruplardan birkaçına bakalım.

### Geliştirici Ekipleri

Rust, farklı seviyelerde sistem programlama bilgisine sahip büyük geliştirici ekipleri arasında verimli bir işbirliği aracı olduğunu kanıtlamaktadır. Düşük seviyeli kod çeşitli ince hatalara yatkındır ve bu hataların çoğu diğer dillerde sadece kapsamlı testler ve deneyimli geliştiriciler tarafından dikkatli kod incelemesi yoluyla yakalanabilir. Rust'ta derleyici, eşzamanlılık hataları da dahil olmak üzere bu zor bulunabilir hataları olan kodu derlemeyi reddederek bir kapı bekçisi rolü oynar. Derleyici ile birlikte çalışarak ekip, hataları takip etmek yerine programın mantığına odaklanarak zamanını harcayabilir.

Rust ayrıca sistem programlama dünyasına çağdaş geliştirici araçları getirir:

- Cargo, dahil edilen bağımlılık yöneticisi ve derleme aracı, bağımlılıkları eklemeyi, derlemeyi ve yöneteyi Rust ekosistemi boyunca zahmetsiz ve tutarlı hale getirir.
- `rustfmt` biçimlendirme aracı, geliştiriciler arasında tutarlı bir kodlama stili sağlar.
- Rust Language Server, kod tamamlama ve satır içi hata mesajları için tümleşik geliştirme ortamı (IDE) entegrasyonunu destekler.

Rust ekosistemindeki bu ve diğer araçları kullanarak geliştiriciler, sistem seviyesinde kod yazarken verimli olabilirler.

### Öğrenciler

Rust, sistem kavramlarını öğrenmekle ilgilenen öğrenciler ve kişiler içindir. Rust'ı kullanarak birçok kişi işletim sistemi geliştirme gibi konular hakkında bilgi edinmiştir. Topluluk çok misafirperver ve öğrencilerin sorularını cevaplamaktan mutluluk duyar. Bu kitap gibi çabalarla Rust ekipleri, sistem kavramlarını daha fazla kişiye, özellikle programlamaya yeni başlayanlara daha erişilebilir hale getirmek istemektedir.

### Şirketler

Yüzlerce şirket, büyük ve küçük, Rust'ı üretim ortamında çeşitli görevler için kullanmaktadır; bunlar arasında komut satırı araçları, web servisleri, DevOps araçları, gömülü cihazlar, ses ve video analizi ve transcoding, kripto para birimleri, biyoinformatik, arama motorları, Nesnelerin İnterneti uygulamaları, makine öğrenimi ve hatta Firefox web tarayıcısının önemli kısımları yer alır.

### Açık Kaynak Geliştiricileri

Rust, Rust programlama dilini, topluluğu, geliştirici araçlarını ve kütüphaneleri oluşturmak isteyen kişiler içindir. Rust diline katkıda bulunmanızı isteriz.

### Hız ve Kararlılığı Önemseyenler

Rust, bir dilde hız ve kararlılık isteyen kişiler içindir. Hızdan kastımız hem Rust kodunun ne kadar hızlı çalışabildiği hem de Rust'ın programları ne kadar hızlı yazmanıza izin verdiğidir. Rust derleyicisinin kontrolleri, özellik eklemeleri ve yeniden düzenleme yoluyla kararlılığı sağlar. Bu, bu kontrolleri olmayan dillerdeki kırılgan miras kodun aksidir; bu kodlarda geliştiriciler genellikle değişiklik yapmaktan korkarlar. Sıfır maliyetli soyutlamalar için çabalayarak—el ile yazılan kod kadar hızlı çalışan düşük seviyeli koda derlenen üst seviyeli özellikler—Rust güvenli kodun aynı zamanda hızlı kod olmasına çalışır.

Rust dili birçok başka kullanıcıyı da desteklemeyi umar; burada bahsedilenler sadece en büyük paydaşlardan birkaçıdır. Genel olarak, Rust'ın en büyük hedefi, güvenlik VE verimlilik, hız VE ergonomi sağlayarak programcıların onlarca yıldır kabul ettikleri ödünleri ortadan kaldırmaktır. Rust'ı deneyin ve seçeneklerin sizin için çalışıp çalışmadığını görün.

## Bu Kitap Kimin İçin

Bu kitap, başka bir programlama dilinde kod yazdığınızı varsayar, ancak hangi dil olduğu hakkında hiçbir varsayım yapmaz. Materyali çok çeşitli programlama geçmişlerinden gelen kişiler için geniş ölçüde erişilebilir hale getirmeye çalıştık. Programlamanın ne OLDUĞU veya nasıl düşünüldüğü hakkında çok fazla zaman harcamıyoruz. Programlamaya tamamen yeniyseniz, size özel olarak programlamaya giriş sunan bir kitap okumak daha faydalı olacaktır.

## Bu Kitabı Nasıl Kullanmalısınız

Genel olarak bu kitap onu sırayla, baştan sona okuduğunuzu varsayar. Sonraki bölümler önceki bölümlerdeki kavramlara dayanır ve önceki bölümler belirli bir konuda detaya girmeyebilir ancak konuyu daha sonraki bir bölümde tekrar ele alacaktır.

Bu kitapta iki tür bölüm bulacaksınız: kavram bölümleri ve proje bölümleri. Kavram bölümlerinde, Rust'ın bir yönünü öğreneceksiniz. Proje bölümlerinde, öğrendiklerinizi uygulayarak küçük programlar birlikte oluşturacağız. Bölüm 2, Bölüm 12 ve Bölüm 21 proje bölümleridir; geri kalanları kavram bölümleridir.

**Bölüm 1** Rust'ı nasıl kuracağınızı, bir "Hello, world!" programını nasıl yazacağınızı ve Rust'ın paket yöneticisi ve derleme aracı olan Cargo'yu nasıl kullanacağınızı açıklar. **Bölüm 2**, Rust'ta bir program yazmaya uygulamalı bir giriş sunar, sizden bir sayı tahmin oyunu oluşturmanızı ister. Burada kavramları üst düzeyde ele alıyoruz ve sonraki bölümler ek detaylar sağlayacak. Hemen elini kirletmek istiyorsanız, Bölüm 2 bunun için uygun yerdir. Özellikle titiz bir öğreniciyseniz ve ayrılmadan önce her detayı öğrenmeyi tercih ederseniz, Bölüm 2'yi atlayıp doğrudan diğer programlama dillerinin özelliklerine benzer Rust özelliklerini kapsayan **Bölüm 3**'e gidebilirsiniz; ardından, öğrendiğiniz detayları uygulayan bir proje üzerinde çalışmak istediğinizde Bölüm 2'ye dönebilirsiniz.

**Bölüm 4**'te Rust'ın sahiplik sistemini öğreneceksiniz. **Bölüm 5** yapıları (struct) ve yöntemleri (method) tartışır. **Bölüm 6** enumları, `match` ifadelerini ve `if let` ve `let...else` kontrol akışı yapılarını kapsar. Yapıları ve enumları özel tipler oluşturmak için kullanacaksınız.

**Bölüm 7**'te, kodunuzu ve genel uygulama programlama arayüzünü (API) düzenlemek için Rust'ın modül sistemini ve gizlilik kurallarını öğreneceksiniz. **Bölüm 8** standart kütüphanenin sağladığı bazı ortak koleksiyon veri yapılarını tartışır: vektörler, dizeler ve hash haritaları. **Bölüm 9** Rust'ın hata işleme felsefesini ve tekniklerini inceler.

**Bölüm 10**, birden çok tipe uygulanabilen kod tanımlama gücü veren generic'leri, trait'leri ve yaşam sürelerini (lifetime) derinlemesine inceler. **Bölüm 11**, Rust'ın güvenlik garantilerine rağmen programınızın mantığının doğru olduğunu sağlamak için gerekli olan testler hakkındadır. **Bölüm 12**'de, dosyalar içinde metin arayan `grep` komut satırı aracının işlevselliğinin bir alt kümesinin kendi uygulamasını oluşturacağız. Bunun için önceki bölümlerde tartıştığımız birçok kavramı kullanacağız.

**Bölüm 13** kapanışları (closure) ve yineleyicileri (iterator) inceler: bu özellikler Rust'ta fonksiyonel programlama dillerinden gelir. **Bölüm 14**'te Cargo'yu daha derinlemesine inceleyeceğiz ve kütüphanelerinizi başkalarıyla paylaşma için en iyi uygulamaları konuşacağız. **Bölüm 15** standart kütüphanenin sağladığı akıllı işaretçileri (smart pointer) ve işlevselliğini sağlayan trait'leri tartışır.

**Bölüm 16**'da, eşzamanlı programlamanın farklı modellerini inceleyeceğiz ve Rust'ın size korkmadan çoklu iş parçacığı (thread) ile programlamanıza nasıl yardımcı olduğunu konuşacağız. **Bölüm 17**'de bunu, Rust'ın async ve await sözdizimini, görevleri (task), futures ve stream'leri ve sağladıkları hafif eşzamanlılık modelini inceleyerek üzerine inşa edeceğiz.

**Bölüm 18**, Rust deyimlerinin (idiom) aşina olabileceğiniz nesne yönelimli programlama ilkeleriyle nasıl karşılaştığına bakar. **Bölüm 19**, desenleri (pattern) ve desen eşleşmesine (pattern matching) bir referanstır; bu, Rust programları boyunca fikirleri ifade etmenin güçlü yollarıdır. **Bölüm 20**, güvenli olmayan Rust, makrolar ve yaşam süreleri, trait'ler, tipler, fonksiyonlar ve kapanışlar hakkında daha fazla şey de dahil olmak üzere ilginç ileri düzey konuların bir koleksiyonunu içerir.

**Bölüm 21**'de, düşük seviyeli çoklu iş parçacıklı bir web sunucusu uygulayacağımız bir projeyi tamamlayacağız!

Son olarak, bazı ekler, dili hakkında daha fazla referans benzeri formatta yararlı bilgiler içerir. **Ek A** Rust'ın anahtar kelimelerini, **Ek B** Rust'ın operatörlerini ve sembollerini, **Ek C** standart kütüphane tarafından sağlanan türetilebilir trait'leri, **Ek D** bazı faydalı geliştirme araçlarını ve **Ek E** Rust sürümlerini açıklar. **Ek F**'de, kitabın çevirilerini bulabilirsiniz ve **Ek G**'de Rust'ın nasıl yapıldığını ve gece sürümü (nightly) Rust'ın ne olduğunu kapsayacağız.

Bu kitabı okumanın yanlış bir yolu yoktur: İleriye atlamak istiyorsanız, yapın! Karşılaştığınız herhangi bir kafa karışıklığında önceki bölümlere geri dönmeniz gerekebilir. Ancak sizin için ne işe yarıyorsa onu yapın.

<span id="ferris"></span>

Rust öğrenme sürecinin önemli bir kısmı, derleyicinin görüntülediği hata mesajlarını nasıl okuyacağınızı öğrenmektir: Bu mesajlar sizi çalışan koda yönlendirir. Bu nedenle, derleyicinin her durumda size göstereceği hata mesajı ile birlikte derlenmeyen birçok örnek sağlayacağız. Rastgele bir örnek girip çalıştırırsanız, derlenmeyebilir! Çalıştırmaya çalıştığınız örneğin hata vermesi gerekip gerekmediğini görmek için çevreleyen metni okuduğunuzdan emin olun. Çoğu durumda, derlenmeyen herhangi bir kodun doğru sürümüne sizi yönlendireceğiz. Ferris de çalışması amaçlanmayan kodu ayırt etmenize yardımcı olacaktır:

| Ferris                                                                                                           | Anlamı                                           |
| ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| <img src="img/ferris/does_not_compile.svg" class="ferris-explain" alt="Ferris with a question mark"/>            | Bu kod derlenmez!                                |
| <img src="img/ferris/panics.svg" class="ferris-explain" alt="Ferris throwing up their hands"/>                   | Bu kod panik atar!                               |
| <img src="img/ferris/not_desired_behavior.svg" class="ferris-explain" alt="Ferris with one claw up, shrugging"/> | Bu kod istenen davranışı üretmez.                |

Çoğu durumda, derlenmeyen herhangi bir kodun doğru sürümüne sizi yönlendireceğiz.

## Kaynak Kodu

Bu kitabın oluşturulduğu kaynak dosyaları [GitHub][book] üzerinde bulunabilir.

[book]: https://github.com/rust-lang/book/tree/main/src