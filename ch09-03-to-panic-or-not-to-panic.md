## `panic!` Yapmak veya Yapmamak (To `panic!` or Not to `panic!`)

O zaman, ne zaman `panic!` çağırmalı ve ne zaman `Result` dönmeli karar verirsiniz? Kod paniklediğinde, kurtarma yolunun yoktur. Herhangi bir hata durumunda kurtarma yolunun olsun veya olmasın `panic!` çağırmayı seçebilirsiniz ancak o zaman çağıran kod adına bir durumun kurtarılamaz olduğunu karar vermiş olursunuz. Bir `Result` değeri dönmeyi seçtiğinizde, çağıran koda seçenekler verirsiniz. Çağıran kod, kendi durumuna uygun bir şekilde kurtarmayı deneyebilir veya bu durumda bir `Err` değerinin kurtarılamaz olduğunu karar verip kurtarılabilir hatanızınızı kurtarılamaz hata haline getirip `panic!` çağırabilir. Bu yüzden, başarısız olabilecek bir fonksiyon tanımlarkenken `Result` dönmek iyi bir varsayılan seçenektir.

Örnekler, prototip kodu ve testler gibi durumlarda, bir `Result` dönmek yerine panikleyen kod yazmak daha uygundur. Neden bunu keşfedelim, sonra derleyici durumlarda hatanın imkansız olduğu mümkündür, ancak siz insan olarak görebileceksiniz. Bölüm, kütüphane kodda ne zaman panikleceye karar vermenizle ilgili bazı genel yönergelerle sona bulacak.

### Örnekler, Prototip Kodu ve Testler (Examples, Prototype Code, and Tests)

Baz kavramları somutlemek için bir örnek yazdığınızda, sağlam hata ele alma kodunu örnegi daha az anlaştırabilirsiniz. Örneklerde, panikleme ihtimali olan `unwrap` gibi bir yönteme çağırın, uygulamanızın hataları nasıl ele almak istediğinizin bir yer tutucusu olarak anlaşılmaktadır ki bu, kodunuzun geri kalanına göre değişebilir.

Benzer şekilde, henüz hataları nasıl ele alacağınıza karar vermediğinizde prototipleme yaptığınızda `unwrap` ve `expect` yöntemleri çok kullanışlıdır. Programınızı daha dayanıklı hale getirmeye hazır olduğunuzda kodunuzda net işaretler bıraklarlar.

Testte bir yöntem çağrısı başarısız olursa, o yöntem test altındaki işlevsellik olsa bile tüm testin başarısız olmasını istersiniz. `panic!` bir testin başarısız olarak işaretlendiği şekildir olduğundan, `unwrap` veya `expect` çağırmak tam olması gereken şeydir.

<!-- Old headings. Do not remove or links may break. -->

<a id="cases-in-which-you-have-more-information-than-the-compiler"></a>

### Derleyiciden Daha Bilginiz Olduğu Durumlarda (When You Have More Information Than the Compiler)

`Result`'un bir `Ok` değere sahip olacağını sağlayan başka bir mantığa sahipseniz ancak bu mantığın derleyicininin anlamadığı olmadığı durumlarda `expect` çağırmak da uygundur olacaktır. Hala bir `Result` değeri ele almanız gerekiyor: Çağırdığınız her operasyonun genel olarak başarısız olma ihtimali olsa da sizin özel durumunuzda mantıksal olarak imkansız. Kodunuzu manuel olarak inceleyerek hiçbir `Err` değişkeni olmayacağınıza sağlayabilirseniz, o zaman oradaki `Err` değişkeni olmayacağını düşünmeniz argüman metninde (argument text) belirtmek koşuluyla `expect` çağırmak tamamen kabul edilebilir. İşte bir örnek:

```rust
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-08-unwrap-that-cant-fail/src/main.rs:here}}
```

Sabit kodlanmış bir string'i ayrıştırarak bir `IpAddr` kopyası oluşturuyoruz. `127.0.0.1`'in geçerli bir IP adresi olduğunu görebilebiliyoruz, bu yüzden burada `expect` kullanmak kabul edilebilir. Ancak, sabit, geçerli bir stringe sahip olmak `parse` yönteminin dönüş tipini değiştirmez: Hala bir `Result` değeri alıyoruz ve derleyici `Err` değişkeni bir ihtimal olarak var olduğundan bizi `Result`'ü ele almaya zorluyor çünkü derleyici bu stringin her zaman geçerli bir IP adresi olduğunu görmek için yeterli değil. IP adresi stringi program içine sabitlenmiş olmak yerine bir kullanıcıdan gelmiş ve dolayısıyla _başarısız olma ihtimaline sahip olsa_, biz kesinlikle `Result`'ü daha dayanıklı bir şekilde ele almak isterdik. Bu IP adresinin sabit kodlandığına dair varsayım, gelecekte IP adresini başka bir kaynaktan almamız gerekiyorsa `expect`'i daha iyi hata ele alma koduna değiştirmeye zorlayacak.

### Hata Ele Alma için Yönergeler (Guidelines for Error Handling)

Kodunuzun kötü bir durumda sona bitebileceği ihtimali paniklemes tavsiye edilebilir. Bu bağlamda, bir _kötü durum_ (bad state), varsayım, garanti, sözleşme veya değişmez bozulduğunda gerçekleşir, örneğin geçersiz, çelişen veya eksik değerler kodunuza geçirildiğinde — ve aşağıdakilerden bir veya fazlasıyla:

- Kötü durum, kullanıcının verileri yanlış formatta vermesi gibi ara sıra sık sık oluşacak bir şeyin zıddinde beklenmeyen beklenmeyen şaş beklenmeyen bir şeydir.
- Bu noktadan sonraki kodunuzun bu kötü durumda olmadığına güvenmesi gerekir, her adımda problemi kontrol etmektense.
- Kullandığınız tiplerde bu bilgiyi kodlamak için iyi bir yol yoktur. Bunu ne demek istediğimizi [“Durleri ve Davranışı Tipler olarak Kodlama”][encoding]<!-- ignore --> bölümünde çalışacağımız.

Birisi kodunuzu çağırsa ve anlam ifade etmeyen değerler geçirirse, eğer ele alabiliyorsanız bir hata dönmek daha iyidir ki bu kütüphanen kullanıcısı o durumda ne yapmak istediğine karar vermesin. Ancak, devam etmenin güvensiz veya zararlı olabileceği durumlarda, en iyi seçenek `panic!` çağırmak ve kütüphanizi kullanan kişiyi kodlarındaki hatayı uyarmak onarabilirler ki geliştirmeleri esnasında düzeltebilirsiniz. Benzer şekilde, kontrolünüzde olmayan dış kod çağırdığında ve düzeltemeye yolunuz yok geçersiz bir kütü durum döndüren bir `panic!` genellikle uygundur. Ancak, başarısızlık bekleniyorsa, bir `panic!` çağrısından ziyade bir `Result` dönmek daha uygundur. Örnekler, biçim veri veri bozuk verilmiş veriye sahip olmasına dönüştürülen bir ayrıştırıcı veya bir HTTP isteği sınırlama limitine ulaştığınız belirten bir durum dönmeyi. Bu durumlarda, `Result` dönmek başarısızlığın çağıran kodun ele alması gereken bir beklenen ihtimal olduğunu belirtir.

Kodunuz, geçersiz değerlerle çağrıldığında bir kullanıcıyı riske atabilecek bir operasyon yapıyorsa, kodunuzun önce değerlerin geçerli olduğunu doğrulamalı ve değerler geçerli değilse paniklemelidir. Bu çoğun güvenlik sebeplerden dolayıdır: Geçersiz veri üzerinde operasyon yapmaya çalışmak kodunuzu açıklara karşıya getirebilir. Bu, standart kütüphanen sınırlar bellek erişimi denemeyi çalıştığınızda `panic!` çağırmasının ana sebepidir: Mevcut veri yapısına ait olmayan belleğe erişmeyi çalışmak yaygın bir güvenlik sorunudur. Fonksiyonlar genellik _sözleşmeler_ (contracts) sahiptir: Davranışları sadece girilerin özel gereksinimleri karşılandığında garanti edilir. Sözleşme bozulduğunda paniklemek mantıklidir çünkü bir sözleşme ihlali her zaman çağıran tarafındaki hatayı belirtir ve çağıran kodun açıkça ele almak istediğiniz bir hata tipi değildir. Aslında, çağıran kodun kurtarabilecek makul yolu yoktur; çağıran _programcilar_ kodlarını düzeltmeleri gerekir. Bir fonksiyonun sözleşmeler, özellikle bir ihlal bir panik nedeniyecekse, fonksiyonun API belgelerinde açıklanmalıdır.

Ancak, tüm fonksiyonlarınızda çok sayıda hata kontrolü olmak ayrıntıcı ve rahatsız edici. Şanslı olarak, Rust'ın tip sistemini (ve dolayısıyla derleyici tarafından yapılan tip kontrolünü) sizin için çok fazla kontrolü yapabilirsiniz. Eğer fonksiyonunuz parametre olarak belirli bir tip sahiptirsa, derleyicinin zaten geçerli bir değere sahip olduğunu bilerek kodunuzun mantığını yürütebilirsiniz. Örneğin, eğer bir `Option` yerine bir tibiniz varsa, programınız _hiçbirşeyi_ bekliyor değil _hiçbirseyi_. Kodunuz o zaman `Some` ve `None` değişkenleri için iki durum ele almak zorunda değil: Kesinlikle kesinlikle bir değere sahip olma durumuna sahip olacak. Fonksiyonunuza hiçbir şey geçiremeyi çalıştıran kod bile derlenmez, bu yüzden fonksiyonunuz bu durumu çalışma zamanında kontrol etmek zorunda değil. Başka bir örnek, `u32` gibi işaretsizamsız (unsigned integer) bir tibin kullanmaktır ki bu, parametrenin hiçbir zaman olumsuz olacağını sağlar.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-custom-types-for-validation"></a>

### Doğrulama için Özel Tipler (Custom Types for Validation)

Rust'ın tip sistemini kullanarak geçerli bir değere sahip olduğumuzu sağlama fikrini bir adım daha ileri götürüp ve doğrulama için özel bir tip oluşturmaya bakalım. Bölüm 2'deki tahmin oyununu (guessing game) hatırlayın ki kodumuz kullanıcının 1 ile 100 arasında bir sayıyı tahmin etmesini istemiştiydi. Kullanıcının tahminin o sayılarla arada kontrol etmiştik - sadece tahminin pozitif olduğunu doğrulamıştık. Bu durumda, sonuçlar çok ciddi deyildi: "Çok yüksek" veya "Çok düşük" çıktımız hala doğru olurdu. Ancak, kullanıcıyı geçerli tahminlere doğrulamak ve kullanıcının aralık dışında bir sayı tahmin ettiği durumla, örneğin harf yerine harfler girdiğinde farklı davranışa sahip olması kullanışlı bir iyileştirme olurdu.

Bunu yapmanın bir yolu, tahmini yalnızca `u32` yerine negatif sayılar olabilmesi için `i32` olarak ayrıştırıp sonra sayının aralıkta olup olmadığını kontrol etmek olabilir, şöyle ki:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-09-guess-out-of-range/src/main.rs:here}}
```

</Listing>

`if` ifadesi değerimizin aralıkta olup olmadığını kontrol eder, kullanıcıyı sorun hakkında bilgilendirir ve döngünün sonraki yeniden bir yineleme başlatmak için başka bir tahmin isterir. `if` ifadesinden sonra, `tahmin` ile gizli sayı arasındaki karşılaştırmalarla devam edebiliriz çünkü `tahmin` 1 ile 100 arasında.

Ancak bu ideal bir çözüm değildir: Programın kesinlikle 1 ile 100 arasında değerlerle çalışması kesinlikle kritik olsa ve bu gereksinime sahip çok sayıda fonksiyon varsa, her fonksiyonda böyle bir kontrol olmak zahmetli (ve belki performansı etkileyebilir).

Bunun yerine, ayrılmış bir modülda yeni bir tip oluşturabilir ve doğrulamaları her yerde tekrar etmek yerine bir kopyas oluşturmak için `new` fonksiyonunda koyabilirsiniz. Bu şekilde, fonksiyonlar bu yeni tipi imzalarında güvenle kullanabilir ve kendilerine aldıgıkları değerlerini güvenle kullanabilirler. Kod Listesi 9-13, `new` fonksiyonu 1 ile 100 arasında bir değer aldığında bir `Guess` kopyası oluşturacak `Guess` tipini tanımlamanın bir yolunu gösterir.

<Listing number="9-13" caption="Sadece 1 ile 100 arasında değerlerle devam edecek bir `Guess` tipi" file-name="src/guessing_game.rs">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-13/src/guessing_game.rs}}
```

</Listing>

Fark edin ki *src/guessing_game.rs* içindeki bu kod burada göstermediğimiz `mod guessing_game;` modül bildirimini *src/lib.rs* içindeki eklememiz gerekiyor ki burada göstermedik. Bu yeni modülün dosyasında, içinde bir `i32` tutan bir `value` adında alan `Guess` adında bir struct tanımlıyoruz. Bu, sayının saklanacağı yerdir.

Sonra, `Guess` üzerinde `Guess` değer kopyaları oluşturan `new` adında bir ilişil fonksiyon (associated function) uyguluyoruz. `new` fonksiyonu `i32` tipinde `value` adında bir parametre sahiptirip bir `Guess` dönmek için tanımlanmıştır. `new` fonksiyonunun gövdesindeki kod, `value`'ü 1 ile 100 arasında olduğundan emin olup olmadığını test eder. `value` bu testi geçmezse, `Guess::new`'in güvendiğiği olan sözleşmeyi ihlal edecek ve bu aralık dışında `value` ile bir `Guess` oluşturmanın programcilerin düzelmesi gereken bir hata olduğunu uyaran bir `panic!` çağırısında bulunuruz. `Guess::new`'in panikleyebileceği durumlar onunla API belgelerinde tartışılmalıdır; Bölüm 14'te oluşturacağınız API belgelerinde bir `panic!` ihtimalini belirtme belgeleme kurallarını konuşacağız. Eğer `value` bu testi geçerse, `value` parametresine ayarlanmış bir `Guess` oluşturur ve `Guess`'i dönerir.

Sonra, `self` ödünen, başka herhangi parametreyi olmayan ve bir `i32` dönen `value` adında bir yöntem uyguluyoruz. Bu türde yönteme bazen _getter_ olarak adlandırılır çünkü amacı alan verilerinden bazı verileri alıp dönmektir. Bu açık yöntem gereklidir çünkü `Guess` struct'inin `value` alanı özeldir (private). `Guess` struct'inin `value` alanının özeldir olması önemlidir çünkü `Guess` struct'i kullanan kodun `value`'i doğrudan ayarlanamaz: `guessing_game` modülünin dışındaki kod `Guess::new` fonksiyonunu kullanmak _zorunda_ bir `Guess` kopyası oluşturmalıdır, bu sayede bir `Guess`'ın `Guess::new` fonksiyonundaki koşullarla kontrol edilmemiş bir `value` sahiptirmemesini sağlar.

1 ile 100 arasında sayı alan bir parametre sahip olan veya yalnızca bu aralıkta sayı dönen bir fonksiyon o zaman imzasında parametre veya dönüş tipi olarak `i32` yerine `Guess` alacağını beyanabilir ve gövdesinde herhangi ek kontrol yapmaya ihtiyac duymaz olabilir.

## Özet (Summary)

Rust'ın hata ele alma özellikleri, daha dayanıklı kod yazmanıza yardımcı olacak şekilde tasarlanmıştır. `panic!` makrosu, programın ele alamayabileceği bir durumda olduğunu ve geçersiz yerine geçersiz veya yanlış değerlerle devam etmek yerine durmasına işaret eder. `Result` enum'i, operasyonların kodunzun kurtarabilebileceği bir şekilde başarısız olabileceğini belirtmek için Rust'ın tip sistemini kullanır. `Result`'ü kullanarak, kodunuzu çağıran koda potansiyel başarı veya başarısızlığı ele alması gerektiğini belirtebilirsiniz. Uygundur durumlarda `panic!` ve `Result`'i kullanmak, kaçınılmaz problemlere karşı karşı kodunuzu daha güvenilir hale getirecektir.

Şimdi standart kütüphanesinin `Option` ve `Result` enum'leriyle jenerikleri nasıl kullandığına ilişili faydalı yollarını gördünüz, genel tiplerin (generics) nasıl çalıştığını ve bunları kodunuzda nasıl kullanabileceğinizi konuşacağız.

[encoding]: ch18-03-oo-design-patterns.html#encoding-states-and-behavior-as-types