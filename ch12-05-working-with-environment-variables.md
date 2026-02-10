## Ortam Değişkenleriyle Çalışma (Working with Environment Variables)

`minigrep` ikilini bir ekstra özellik ekleyerek iyileştireceğiz: kullanıcının bir ortam değişkeni üzerinden açabileceği büyük/küçük harf duyarsız arama için bir seçenek. Bu özelliiği bir komut satırı seçeneği yapabilir ve kullanıcının her uygulamak istediginde girmesini gerektirebilir, ancak bunun yerine onu bir ortam değişkeni yaparak, kullanıcılara bir ortam değişkenini bir kez ayarlama ve tüm aramalarının o terminal oturumunda büyük/küçük harf duyarsız olmasına izin veriyoruz.

<a id="writing-a-failing-test-for-the-case-insensitive-search-function"></a>

### Büyük/Küçük Harf Duyarsız Arama İçin Başarısız Bir Test Yazma (Writing a Failing Test for Case-Insensitive Search)

Önce, ortam değişkeni bir değere sahip olduğunda çağrılacak yeni bir `search_case_insensitive` fonksiyonunu `minigrep` kütüphanesine ekleyeceğiz. TDD sürecini izlemeye devam edeceğiz, bu yüzden ilk adım tekrar bir başarısız test yazmak. Yeni `search_case_insensitive` fonksiyonu için yeni bir test ekleyeceğiz ve eski testimizi `one_result`'den `case_sensitive`'e yeniden adlandıracağız ki Kod Listesi 12-20'da gösterildiği gibi iki test arasındaki farkları netleştirelim.

<Listing number="12-20" file-name="src/lib.rs" caption="Eklemek üzerinde olduğumuz büyük/küçük harf duyarsız fonksiyonu için yeni bir başarısız test ekleme">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-20/src/lib.rs:here}}
```

</Listing>

Eski testin `contents`'ini de düzenlediğimizi unutmayın. Büyük _D_ kullanan `"Duct tape."` metniyle yeni bir satır ekledik ki bu, büyük/küçük harf duyarlı bir şekilde ararken sorgu `"duct"` ile eşleşmemesi gerekir. Eski testi bu şekilde değiştirmek, zaten uygulamış olduğumuz büyük/küçük harf duyarlı arama işlevselliğini yanlışlıkla bozmayı engellemeye yardımcı olur. Bu testin şimdi geçmesi gerekir ve büyük/küçük harf duyarsız arama üzerinde çalışırızken geçmeye devam etmesi gerekir.

Büyük/küçük harf _duyarsız_ arama için yeni test sorgusu olarak `"rUsT"` kullanıyor. Eklemede olduğumuz `search_case_insensitive` fonksiyonunda, sorgu `"rUsT"` baş harfi büyük _R_ olan `"Rust:"` içeren satırla ve büyük harf _T_ olan `"Trust me."` satırıyla eşleşmesi gerekir ki her ikisi de sorgudan farklı büyük/küçük harf durumuna sahip. Bu bizim başarısız testimiz ve henüz `search_case_insensitive` fonksiyonunu tanımlamadığımız için derlenemeye başarısız olacak. Testin derlenip başarısız olduğunu görmek için, Kod Listesi 12-16'da `search` fonksiyonu için yaptığımız gibi, her zaman boş bir vektör dönen bir iskelet uygulaması eklemekten çekinmeyin.

### `search_case_insensitive` Fonksiyonu Uygulama (Implementing `search_case_insensitive` Function)

Kod Listesi 12-21'de gösterilen `search_case_insensitive` fonksiyonu `search` fonksiyonuna hemen hemen aynı olacak. Tek fark, `query`'yi ve her `line`'ı küçük harfe dönüştüreceğiz ki girdi argümanlarının büyük/küçük harf durumundan bağımsız, satırın sorguyu içerip içermediğini kontrol ettiğimizde aynı büyük/küçük harf durumunda olacaklar.

<Listing number="12-21" file-name="src/lib.rs" caption="`search_case_insensitive` fonksiyonunu karşılaştırmadan önce `query` ve `line`'ı küçük harfe dönüştürecek şekilde tanımlama">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-21/src/lib.rs:here}}
```

</Listing>

Önce, `query` dizesini küçük harfe dönüştürüyoruz ve onu orijinal `query`'yi gölgeleyen (shadowing) aynı adda bir değişkende saklıyoruz. `query` üzerinde `to_lowercase` çağırmak gerekir ki kullanıcının sorgusu `"rust"`, `"RUST"`, `"Rust"` veya `"rUsT"` olmasından bağımsız, sorguyu `"rust"` gibi ele alacağız ve büyük/küçük harfe duyarsız olacağız. `to_lowercase` temel Unicode'yi ele alacak olsa da, %100 doğru olmayacak. Gerçek bir uygulama yazsaydık, burada biraz daha iş yapmamız gerekirdi, ancak bu bölüm ortam değişkenleri hakkında, Unicode değil, bu yüzden orada bırakacağız.

`query`'un şimdi bir dize değişmezi (string literal) değil bir `String` olduğuna dikkat edin çünkü `to_lowercase` çağırmak mevcut veriyi referans etmek yerine yeni veri oluşturur. Örneğin, `query` `"rUsT"` olsaydı: O dize değişmezinde kullanmak için küçük harfli `u` veya `t` yoktur, bu yüzden `"rust"` içeren yeni bir `String` tahsis etmek zorundayız. `query`'yu şimdi bir argüman olarak `contains` yöntemine geçirdiğimizde, `contains`'in imzası bir dize dilimi almak için tanımlandığı için bir ampersand eklememiz gerekiyor.

Sonra, tüm karakterleri küçük harfe dönüştürmek için her `line` üzerinde `to_lowercase` çağrısını ekliyoruz. Şimdi `line`'ı ve `query`'yi küçük harfe dönüştürdüğümüze göre, sorgunun büyük/küçük harf durumu ne olursa olsun eşleşmeler bulacağız.

Bu uygulamanın testleri geçip geçmediğini görelim:

```console
{{#include ../listings/ch12-an-io-project/listing-12-21/output.txt}}
```

Harika! Geçtiler. Şimdi `run` fonksiyonundan yeni `search_case_insensitive` fonksiyonunu çağıralım. Önce, büyük/küçük harf duyarlı ve büyük/küçük harf duyarsız arama arasında geçiş yapmak için `Config` struct'ına bir konfigürasyon seçeneği ekleyeceğiz. Bu alanı henüz hiçbir yerde ilklendirmemiz çünkü derleyici hatalarına neden olacak:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/main.rs:here}}
```

Bir Booleean tutan `ignore_case` alanı ekledik. Sonra, Kod Listesi 12-22'da gösterildiği gibi, `ignore_case` alanının değerini kontrol etmeyi ve `search` fonksiyonunu mu yoksa `search_case_insensitive` fonksiyonunu mu çağıracağına karar vermesi için `run` fonksiyonuna ihtiyaç duyuyoruz. Bu henüz derlenmeyecek.

<Listing number="12-22" file-name="src/main.rs" caption="`config.ignore_case`'taki değere dayanarak `search` veya `search_case_insensitive` çağırma">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/main.rs:there}}
```

</Listing>

Son olarak, bir ortam değişkeni kontrol etmemiz gerekiyor. Ortam değişkenleriyle çalışmak için fonksiyonlar standart kütüphanenin `env` modülündedir ki _src/main.rs_'in üstünde zaten kapsamda. Kod Listesi 12-23'da gösterildiği gibi, `IGNORE_CASE` adında bir ortam değişkeni için herhangi bir değer ayarlanıp ayarlanmadığını görmek için `env` modülünden `var` fonksiyonunu kullanacağız.

<Listing number="12-23" file-name="src/main.rs" caption="`IGNORE_CASE` adında bir ortam değişkeninde herhangi bir değer olup olmadığını kontrol etme">

```rust,ignore,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-23/src/main.rs:here}}
```

</Listing>

Burada, `ignore_case` adında yeni bir değişken oluşturuyoruz. Değerini ayarlamak için, `env::var` fonksiyonunu çağırıyoruz ve ona `IGNORE_CASE` ortam değişkeninin adını geçiyoruz. `env::var` fonksiyonu bir `Result` döndürür ki eğer ortam değişkeni herhangi bir değere ayarlanmışsa başarılı `Ok` varyantında ortam değişkeninin değerini içerir. Eğer ortam değişkeni ayarlanmamışsa `Err` varyantını döndürür.

`Result` üzerinde `is_ok` yöntemini kullanarak ortam değişkenisinin ayarlanıp ayarlanmadığını kontrol ediyoruz ki bu, programın büyük/küçük harf duyarsız bir arama yapması gerektiği anlamına gelir. Eğer `IGNORE_CASE` ortam değişkenisi herhangi bir şeye ayarlanmamışsa, `is_ok` `false` dönecektir ve program büyük/küçük harf duyarlı bir arama yapacaktır. Ortam değişkeninin _değerine_ değil, sadece ayarlanıp ayarlanmadığına dikkat ediyoruz, bu yüzden `unwrap`, `expect` veya `Result` üzerinde gördüğümüz diğer yöntemlerin herhangi birini kullanmak yerine `is_ok`'i kontrol ediyoruz.

`ignore_case` değişkenindeki değeri `Config` örneğine geçiriyoruz ki `run` fonksiyonu o değeri okuyup Kod Listesi 12-22'da uygulamış olduğumuz gibi `search_case_insensitive`'ı mı yoksa `search`'ü çağırmaya karar verebilsin.

Deneyelim! Önce, ortam değişkenisi ayarlanmadan ve `to` sorgusuyla programımızı çalıştıralım ki bu küçük harfli tüm _to_ kelimesini içeren her satırla eşleşmeli:

```console
{{#include ../listings/ch12-an-io-project/listing-12-23/output.txt}}
```

Görünüşe göre hala çalışıyor! Şimdi programı `IGNORE_CASE` `1`'e ayarlanmış ancak aynı `to` sorgusuyla çalıştıralım:

```console
$ IGNORE_CASE=1 cargo run -- to poem.txt
```

PowerShell kullanıyorsanız, ortam değişkenisini ayarlama ve programı ayır komutlar olarak çalıştırmanız gerekecektir:

```console
PS> $Env:IGNORE_CASE=1; cargo run -- to poem.txt
```

Bu `IGNORE_CASE`'i shell oturumunun geri kalanı için kalıcı hale getirecek. `Remove-Item` cmdlet'iyle kaldırılabilir:

```console
PS> Remove-Item Env:IGNORE_CASE
```

Büyük harfli olan _To_ içeren satırları almalıyız:

<!-- manual-regeneration
cd listings/ch12-an-io-project/listing-12-23
IGNORE_CASE=1 cargo run -- to poem.txt
can't extract because of environment variable
-->

```console
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

Mükemmel, ayrıca _To_ içeren satırları da aldık! Bizim `minigrep` programımız şimdi bir ortam değişkeniyle kontrol edilen büyük/küçük harf duyarsız arama yapabiliyor. Şimdi komut satırı argümanları veya ortam değişkenlerini kullanarak ayarlanan seçenekleri nasıl yöneteceğinizi biliyorsunuz.

Bazı programlar aynı konfigürasyon için argümanlar _ve_ ortam değişkenlerine izin verir. Bu durumlarda, programlar biri veya diğerinin öncelik almasına karar verir. Kendi başınıza biraz daha alıştırma olarak, büyük/küçük harf duyarsılılığı bir komut satırı argümanı veya bir ortam değişkeni üzerinden kontrol etmeyi deneyin. Program bir tanesi büyük/küçük harf duyarlı ve diğer tanesi büyük/küçük harf duyarsız olacak şekilde çalıştırılırsa, komut satırı argümanının mı yoksa ortam değişkeninin mi öncelik almasına karar verin.

`std::env` modülü ortam değişkenleriyle uğraşmak için çok daha fazla yararlı özellik içerir: Mevcut olanları görmek için belgelendirmesini kontrol edin.