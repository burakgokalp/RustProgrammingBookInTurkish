## Yolları `use` Anahtar Kelimesiyle Kapsama Getirme (Bringing Paths into Scope with `use` Keyword)

Fonksiyonları çağırmak için yolları yazmak elverişsiz ve tekrarlı hissettirebilir. Kod Listesi 7-7'de, `add_to_waitlist` fonksiyonuna mutlak veya göreceli yol seçtiksek, her seferinde `add_to_waitlist`'i çağırmak istediğimizde `front_of_house` ve `hosting`'i de belirtmeliydik. Şanslıyız ki, bu süreci basitleştirmenin bir yolu var: Bir kapsamada `use` anahtar kelimesiyle bir yola kısayol oluşturabiliriz ve sonra kapsamadaki başka her yerde daha kısa adı kullanabiliriz.

Kod Listesi 7-11'de, `eat_at_restaurant` fonksiyonunun kapsamına `crate::front_of_house::hosting` modülünü getiriyoruz ki `eat_at_restaurant` içinde `add_to_waitlist` fonksiyonunu çağırmak için sadece `hosting::add_to_waitlist` belirtmemiz gerekir.

<Listing number="7-11" file-name="src/lib.rs" caption="Bir modülü `use` ile kapsama getirme">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-11/src/lib.rs}}
```

</Listing>

Bir kapsamada `use` ve bir yol eklemek, bir dosya sisteminde sembolik bir bağlantı oluşturmaya benzer. Kafa kökünde `use crate::front_of_house::hosting` ekleyerek, `hosting` artık o kapsamada geçerli bir addır, tıpkı `hosting` modülü kafa kökünde tanımlanmış gibi. `use` ile kapsama getirilen yollar başka herhangi bir yol gibi gizliliği kontrol eder.

Fark edin ki `use` sadece `use`'nin olduğu belirli bir kapsamada kısayol oluşturur. Kod Listesi 7-12, `eat_at_restaurant` fonksiyonunu `use` ifadesinden farklı olan `customer` adında yeni bir alt modüle taşır, bu yüzden fonksiyon gövdesi derlenmez.

<Listing number="7-12" file-name="src/lib.rs" caption="Bir `use` ifadesi sadece olduğu kapsamada uygulanır.">

```rust,noplayground,test_harness,does_not_compile,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-12/src/lib.rs}}
```

</Listing>

Derleyici hatası gösterir ki kısayol artık `customer` modülü içinde uygulanmaz:

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-12/output.txt}}
```

Ayrıca `use`'nin artık kapsamada kullanılmadığı hakkında bir uyarının olduğunu fark edin! Bu sorunu çözmek için, `use`'i de `customer` modülü içine taşıyın veya alt `customer` modülü içinde `super::hosting` ile ebeveyn modülündeki kısayola atıfta bulunun.

### İdiyomatik `use` Yolları Oluşturma (Creating Idiomatic `use` Paths)

Kod Listesi 7-11'de, `use crate::front_of_house::hosting` belirttik ve sonra `eat_at_restaurant` içinde `hosting::add_to_waitlist` çağırdık, Kod Listesi 7-13'teki gibi aynı sonucu elde etmek için `use` yolunu `add_to_waitlist` fonksiyonuna kadar tamamen belirtmek yerine neden merak etmiş olabilirsiniz.

<Listing number="7-13" file-name="src/lib.rs" caption="`use` ile `add_to_waitlist` fonksiyonunu kapsama getirme ki bu idiyomatik değildir">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-13/src/lib.rs}}
```

</Listing>

Hem Kod Listesi 7-11 hem de Kod Listesi 7-13 aynı görevi tamamlasa da, Kod Listesi 7-11 `use` ile bir fonksiyonu kapsama getirmenin idiyomatik yoludur. Fonksiyonun ebeveyn modülünü `use` ile kapsama getirmek, fonksiyonu çağırırken ebeveyn modülü belirtmeyi gerektirir. Fonksiyonu çağırırken ebeveyn modülünü belirtmek, fonksiyonun yerel olarak tanımlanmadığını açık yaparken tam yolun tekrarını en aza indirir. Kod Listesi 7-13'deki kod `add_to_waitlist`'in nerede tanımlandığı konusunda belirsizdir.

Öte yandan, yapılar, enumler ve diğer ögeler `use` ile getirilirken, tam yol belirtmek idiyomatiktir. Kod Listesi 7-14, standart kütüphanenin `HashMap` yapısını bir ikili kafanın kapsamına idiyomatik yolla getirmeyi gösterir.

<Listing number="7-14" file-name="src/main.rs" caption="`HashMap`'ü idiyomatik bir yolla kapsama getirme">

```rust
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-14/src/main.rs}}
```

</Listing>

Bu idiyomun arkasında güçlü bir sebep yok: Bu sadece ortaya çıkmış bir konvansiyondur ve insanlar bu şekilde Rust kodunu okuyup yazmaya alışmışlardır.

Bu idiyomun istisnası, `use` ifadeleriyle aynı adda sahip iki ögeyi kapsama getiriyorsanız, çünkü Rust buna izin vermez. Kod Listesi 7-15, aynı adda ancak farklı ebeveyn modüllere sahip iki `Result` tipini nasıl kapsama getireceğinizi ve onlara nasıl atıfta bulunacağınızı gösterir.

<Listing number="7-15" file-name="src/lib.rs" caption="Aynı adda sahip iki tipi aynı kapsamaya getirmek onların ebeveyn modüllerini kullanmayı gerektirir.">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-15/src/lib.rs:here}}
```

</Listing>

Görebileceğiniz gibi, ebeveyn modüllerini kullanmak iki `Result` tipini ayırtır. Bunun yerine `use std::fmt::Result` ve `use std::io::Result` belirtseydik, aynı kapsamada iki `Result` tipi olurdu ve Rust `Result` kullandığımızda hangisini kastettiğimizi bilmezdi.

### `as` Anahtar Kelimesiyle Yeni Adlar Sağlama (Providing New Names with `as` Keyword)

Aynı adda sahip iki tipi `use` ile aynı kapsamaya getirme sorunun başka bir çözümü var: Yoldan sonra, tip için `as` ve yeni bir yerel adı veya _alias_ belirtebiliriz. Kod Listesi 7-16, iki `Result` tipinden birini `as` kullanarak yeniden adlandırarak Kod Listesi 7-15'teki kodu yazmanın başka bir yolunu gösterir.

<Listing number="7-16" file-name="src/lib.rs" caption="Bir tip `use` ile kapsama getirildiğinde `as` anahtar kelimesiyle yeniden adlandırma">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-16/src/lib.rs:here}}
```

</Listing>

İkinci `use` ifadesinde, `std::io::Result` tipi için yeni ad `IoResult` seçtik ki bu, kapsamaya getirdiğimiz `std::fmt`'den `Result` ile çakışmaz. Kod Listesi 7-15 ve Kod Listesi 7-16 idiyomatik olarak düşünülmektedir, bu yüzden seçim size kalmış!

### `pub use` ile Yeniden Dışa Aktarma (Re-exporting Names with `pub use`)

Bir adı `use` anahtar kelimesiyle kapsama getirdiğimizde, ad ithal ettiğimiz kapsama özeldir. Bu kapsamın dışındaki kodun o ada, sanki o kapsamada tanımlanmış gibi, atıfta bulunmasını sağlamak için, `pub` ve `use`'i birleştirebiliriz. Bu teknik _yeniden dışa aktarma_ (re-exporting) olarak adlandırılır çünkü bir ögeyi kapsama getiriyoruz ancak ayrıca bu ögeyi başkalarının kapsamaya getirmeleri için erişilebilir yapıyoruz.

Kod Listesi 7-17, kök modüldeki `use`'i `pub use` olarak değiştirilmiş Kod Listesi 7-11'deki kodu gösterir.

<Listing number="7-17" file-name="src/lib.rs" caption="Bir adı `pub use` ile yeni bir kapsamadan kullanılabilir yapma">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-17/src/lib.rs}}
```

</Listing>

Bu değişiklikten önce, dış kodun `add_to_waitlist` fonksiyonunu `restaurant::front_of_house::hosting::add_to_waitlist()` yolunu kullanarak çağırması gerekirdi ki bu ayrıca `front_of_house` modülünün `pub` olarak işaretlenmesini gerektirirdi. Şimdi bu `pub use`, `hosting` modülünü kök modülden yeniden dışa aktardığı için, dış kodun yerine `restaurant::hosting::add_to_waitlist()` yolunu kullanabilir.

Kodunuzun iç yapısı, kodunuzu çağıran programcıların alan hakkında düşüneceği şekilden farklı olduğunda yeniden dışa aktarma kullanışlıdır. Örneğin, bu restoran metaforumunda, restoranı işleten kişiler "ön bölüm" ve "arka bölüm" düşünüyorlar. Ancak restoranı ziyaret eden müşteriler muhtemelen restoranın parçalarını bu terimlerle düşüneceklerdir. `pub use` ile, kodumuzu bir yapıyla yazabilir ancak farklı bir yapı açığa çıkarabiliriz. Bunu yapmak, kütüphanede çalışan programcılar ve kütüphaneyi çağıran programcılar için kütüphanemizi iyi organize eder. `pub use`'in başka bir örneğine ve kafanızın belgelerini nasıl etkilediğine Bölüm 14'te ["Uygun Genel API Dışa Aktarma" (Exporting a Convenient Public API)][ch14-pub-use]<!-- ignore --> bölümünde bakacağız.

### Dış Paketleri Kullanma (Using External Packages)

Bölüm 2'de, rastgele sayılar almak için `rand` adında bir dış paket kullanan bir tahmin oyunu projesi programladık. Projemizde `rand` kullanmak için, bu satırı _Cargo.toml_'e ekledik:

<!-- `rand` kullanılan sürümü güncellerken, bu dosyalarda kullanılan `rand` sürümünü de güncelleyn ki hepsi eşlensin:
* ch02-00-guessing-game-tutorial.md
* ch14-03-cargo-workspaces.md
-->

<Listing file-name="Cargo.toml">

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:9:}}
```

</Listing>

_Cargo.toml_'de `rand`'i bağımlılık olarak eklemek, Cargo'ya `rand` paketini ve [crates.io](https://crates.io/) adresinden herhangi bir bağımlılığı indirmesini ve `rand`'i projemizde erişilebilir yapmasını söyler.

Sonra, `rand` tanımlarını paketimizin kapsamına getirmek için, kafa adı `rand` ile başlayan bir `use` satırı ve kapsamaya getirmek istediğimiz ögeleri listeledik. Bölüm 2'de ["Rastgele Sayı Üretme" (Generating a Random Number)][rand]<!-- ignore --> bölümünde hatırlayın, `Rng` trait'ini kapsamaya getirdik ve `rand::thread_rng` fonksiyonunu çağırdık:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:ch07-04}}
```

Rust topluluğunun üyeleri [crates.io](https://crates.io/) adresinde birçok paketi erişilebilir yaptılar ve herhangi birini paketinize getirmek bu aynı adımları içerir: Paketinizin _Cargo.toml_ dosyasında onları listelemek ve `use` kullanarak onların kafasından ögeleri kapsama getirmek.

Fark edin ki standart `std` kütüphanesi de paketimizin dışında olan bir kafadır. Standart kütüphane Rust diliyle birlikte gönderildiği için, `std` içermek için _Cargo.toml_ değiştirmemize gerekmez. Ancak oradan ögeleri paketimizin kapsamına getirmek için `use` kullanarak ona atıfta bulunmamız gerekir. Örneğin, `HashMap` ile bu satırı kullanırdık:

```rust
use std::collections::HashMap;
```

Bu standart kütüphane kafası adda `std` ile başlayan bir mutlak yoldur.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-nested-paths-to-clean-up-large-use-lists"></a>

### İç İçe Yolları Kullanarak `use` Listelerini Temizleme (Using Nested Paths to Clean Up `use` Lists)

Eğer aynı kafa veya aynı modülde tanımlanmış birçok ögeyi kullanıyorsanız, her ögeyi kendi satırında listelemek dosyalarımızda çok sayısal yer alabilir. Örneğin, Kod Listesi 2-4'teki tahmin oyununda sahip olduğumuz bu iki `use` ifadesi `std`'den ögeleri kapsamaya getirir:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-01-use-std-unnested/src/main.rs:here}}
```

</Listing>

Bunun yerine, aynı ögeleri bir satırda kapsama getirmek için iç içe yollar kullanabiliriz. Bunu, yolun ortak kısmını belirtip, ardından iki nokta ve sonra yolların farklı kısımlarının etrafında süslü parantez kullanarak yaparız, Kod Listesi 7-18'de gösterildiği gibi.

<Listing number="7-18" file-name="src/main.rs" caption="Aynı öneki sahip çoklu ögeleri kapsamaya getirmek için bir iç içe yol belirtme">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-18/src/main.rs:here}}
```

</Listing>

Daha büyük programlarda, aynı kafa veya modülden birçok ögeyi iç içe yollar kullanarak kapsama getirmek, gerekli ayrı `use` ifadeleri sayısını çok azaltabilir!

Bir yolun herhangi bir seviyesinde iç içe yol kullanabiliriz ki bu, bir alt yolu paylaşan iki `use` ifadesini birleştirdiğinizde kullanışlıdır. Örneğin, Kod Listesi 7-19 iki `use` ifadesini gösterir: biri `std::io`'yi kapsamaya getirir diğeri ise `std::io::Write`'i kapsamaya getirir.

<Listing number="7-19" file-name="src/lib.rs" caption="İki `use` ifadesi ki biri diğerinin bir alt yoludur">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-19/src/lib.rs}}
```

</Listing>

Bu iki yolun ortak kısmı `std::io`'dir ve bu tam ilk yoldur. Bu iki yolu bir `use` ifadesine birleştirmek için, iç içe yolda `self` kullanabiliriz, Kod Listesi 7-20'da gösterildiği gibi.

<Listing number="7-20" file-name="src/lib.rs" caption="Kod Listesi 7-19'deki yolları bir `use` ifadesinde birleştirme">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-20/src/lib.rs}}
```

</Listing>

Bu satır `std::io` ve `std::io::Write`'i kapsamaya getirir.

<!-- Old headings. Do not remove or links may break. -->

<a id="the-glob-operator"></a>

### Glob Operatörüyle Ögeleri İthale Etmek (Importing Items with Glob Operator)

Bir yolda tanımlanmış tüm genel ögeleri kapsama getirmek istiyorsak, o yolu `*` glob operatörü takip ederek belirtebiliriz:

```rust
use std::collections::*;
```

Bu `use` ifadesi, `std::collections`'da tanımlanmış tüm genel ögeleri mevcut kapsamaya getirir. Glob operatörü kullanırken dikkatli olun! Glob hangi isimlerin kapsamada olduğunu ve programınızda kullanılan bir adın nerede tanımlandığını söylemeyi zorlaştırabilir. Ek olarak, bir bağımlılık tanımlarını değiştirirse, ithal ettiğiniz de değişir ki bu, bağımlılıği yükselttiğinizde derleyici hatalarına yol açabilir, örneğin bağımlılık aynı kapsamada sizin tanımlamanız ile aynı adda sahip bir tanımlama eklerse.

Glob operatörü genellikle test sırasında her şeyi test altındaki `tests` modülüne getirmek için kullanılır; Bölüm 11'de ["Nasıl Test Yazılır" (How to Write Tests)][writing-tests]<!-- ignore --> bölümünde bunun hakkında konuşacağız. Glob operatörü bazen bir önüdeş (prelude) deseni parçası olarak da kullanılır; Bu desen hakkında daha fazla bilgi için [standart kütüphane belgelerine](../std/prelude/index.html#other-preludes)<!-- ignore --> bakın.

[ch14-pub-use]: ch14-02-publishing-to-crates-io.html#exporting-a-convenient-public-api
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
[writing-tests]: ch11-01-writing-tests.html#how-to-write-tests