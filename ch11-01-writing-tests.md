## Testleri Nasıl Yazılır (How to Write Tests)

_Testler_, test edilmeyen kodun beklenen şekilde çalıştığını doğrulayan Rust fonksiyonlarıdır. Test fonksiyonlarının gövdesi genellikle şu üç eylemi gerçekleştirir:

- Gerekli herhangi bir veriyi veya durumu ayarlayın.
- Test etmek istediğiniz kodu çalıştırın.
- Sonuçların beklediğiniz gibi olduğunu iddia edin.

Rust'ın bu eylemleri gerçekleştirmek için özel olarak sunduğu özelliklere, `test` özniteliği, birkaç makro ve `should_panic` özniteliğini içeren, bakın.

<!-- Old headings. Do not remove or links may break. -->

<a id="the-anatomy-of-a-test-function"></a>

### Test Fonksiyonlarını Yapılandırma (Structuring Test Functions)

En basit haliyle, Rust'ta bir test, `test` özniteliği ile notlanmış bir fonksiyondur. Öznitelikler, Rust kodunun parçaları hakkında meta verilerdir; Bölüm 5'te struct'larla kullandığımız `derive` özniteliği bir örnektir. Bir fonksiyonu bir test fonksiyonuna dönüştürmek için `fn`'den önceki satıra `#[test]` ekleyin. `cargo test` komutuyla testlerinizi çalıştırdığınızda, Rust notlanmış fonksiyonları çalıştıran bir test çalıştırıcı ikilisi (test runner binary) oluşturur ve her test fonksiyonunun geçip geçmediğini bildirir.

Cargo ile yeni bir kütüphane projesi oluşturduğumuzda, içinde bir test fonksiyonu olan bir test modülü bizim için otomatik olarak oluşturulur. Bu modül, testlerinizi yazmak için size bir şablon sağlar böylece yeni bir proje başlattığınızda her seferde tam yapıyı ve sözdizimini aramanıza gerek kalmaz. İstediğiniz kadar ekstra test fonksiyonu ve test modülü ekleyebilirsiniz!

Gerçekten kod test etmeden önce şablon testle deneyerek testlerin nasıl çalıştığına dair bazı yönleri keşfedeceğiz. Sonra, yazdığımız bazı kodları çağıran ve davranışının doğru olduğunu iddia eden bazı gerçek dünya testleri yazacağız.

İki sayıyı ekleyen `adder` adında yeni bir kütüphane projesi oluşturalım:

```console
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

`adder` kütüphanenizdeki _src/lib.rs_ dosyasının içeriği Kod Listesi 11-1 gibi görünmelidir.

<Listing number="11-1" file-name="src/lib.rs" caption="`cargo new` tarafından otomatik oluşturulan kod">

<!-- manual-regeneration
cd listings/ch11-writing-automated-tests
rm -rf listing-11-01
cargo new listing-11-01 --lib --name adder
cd listing-11-01
echo "$ cargo test" > output.txt
RUSTFLAGS="-A unused_variables -A dead_code" RUST_TEST_THREADS=1 cargo test >> output.txt 2>&1
git diff output.txt # commit any relevant changes; discard irrelevant ones
cd ../../..
-->

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

</Listing>

Dosya test edeceğimiz bir şeyimiz olsun diye örnek bir `add` fonksiyonu ile başlar.

Şimdilik sadece `it_works` fonksiyonuna odaklanalım. `#[test]` notunu unutmayın: Bu öznitelik bu bir test fonksiyonu olduğunu belirtir, böylece test çalıştırıcı bu fonksiyonu bir test olarak tanır. `tests` modülünde ortak senaryoları ayarlamaya veya ortak işlemleri gerçekleştirmeye yardımcı olmak için test olmayan fonksiyonlar da olabilir, bu yüzden hangi fonksiyonların test olduğunu her zaman belirtmemiz gerekir.

Örnek fonksiyon gövdesi, 2 ve 2 ile `add` çağırmanın sonucunu içeren `result`'in 4'e eşit olduğunu iddia etmek için `assert_eq!` makrosunu kullanır. Bu iddia, tipik bir testin formatına bir örnektir. Bu testin geçtiğini görmek için çalıştıralım.

`cargo test` komutu projemizdeki tüm testleri çalıştırır, Kod Listesi 11-2'de gösterildiği gibi.

<Listing number="11-2" caption="Otomatik oluşturulan testi çalıştırmadan çıktı">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-01/output.txt}}
```

</Listing>

Cargo testi derledi ve çalıştırdı. `running 1 test` satırını görüyoruz. Sonraki satır oluşturulan test fonksiyonunun adını, `tests::it_works` adını ve bu testi çalıştırmanın sonucunun `ok` olduğunu gösterir. Genel özeti `test result: ok.` tüm testlerin geçtiğini ve `1 passed; 0 failed` okuyan kısmı geçen veya başarısız olan testlerin sayısını toplar.

Bir testi, belirli bir örnekte çalışmaması için yoksay (ignore) olarak işaretlemek mümkündür; bunu bu bölümün ilerleyen kısımlarında ["Özellikle İstenmediği Sürece Testleri Yoksayma"][ignoring]<!-- ignore --> bölümünde kapsayacağız. Burada bunu yapmadığımız için, özet `0 ignored` gösterir. Ayrıca `cargo test` komutuna bir argüman geçebiliriz ki bu sadece adı bir dizeyle eşleşen testleri çalıştırır; buna _filtreleme_ denir ve bunu ["Ada Göre Testlerin Bir Alt Kümesini Çalıştırma"][subset]<!-- ignore --> bölümünde kapsayacağız. Burada, çalıştırılan testleri filtrelemedik, bu yüzden özetin sonunda `0 filtered out` gösterir.

`0 measured` istatistiği, performansı ölçen kıyaslama testleri içindir. Kıyaslama testleri, bu yazının yazıldığına göre, sadece nightly Rust'te mevcuttur. Daha fazla bilgi için [kıyaslama testleri hakkındaki belgelemeye][bench] bakın.

Test çıktısının `Doc-tests adder` ile başlayan sonraki kısmı herhangi bir belgelem testlerinin sonuçları içindir. Henüz herhangi bir belgelem testimiz yok, ancak Rust API belgelememizde görünen herhangi bir kod örneğini derleyebilir. Bu özellik belgelerinizi ve kodunuzu senkronize tutmaya yardımcı olur! Bölüm 14'ün ["Testler Olarak Belgelem Yorumları"][doc-comments]<!-- ignore --> bölümünde belgelem testlerini nasıl yazacağımızı tartışacağız. Şimdilik `Doc-tests` çıktısını yoksayacağız.

Şimdi testi kendi ihtiyaçlarımıza göre özelleştirmeye başlayalım. Önce, `it_works` fonksiyonunun adını farklı bir ad olan `exploration` olarak değiştirin, şöyle:

<span class="filename">Dosya Adı: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/src/lib.rs}}
```

Sonra, tekrar `cargo test` çalıştırın. Çıktı artık `it_works` yerine `exploration` gösterir:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/output.txt}}
```

Şimdi başka bir test ekleyeceğiz ancak bu sefer başarısız olacak bir test yapacağız! Testler, test fonksiyonu içindeki bir şeyler panik atarsa başarısız olur. Her test yeni bir iş parçacığında (thread) çalışır ve ana iş parçacığının (main thread) bir test iş parçacığının öldüğünü gördüğünde test başarısız olarak işaretlenir. Bölüm 9'da, en basit panik atma yolunun `panic!` makrosunu çağırmak olduğunu konuştuk. `another` adında yeni bir test fonksiyonu olarak girin, böylece _src/lib.rs_ dosyanız Kod Listesi 11-3 gibi olur.

<Listing number="11-3" file-name="src/lib.rs" caption="`panic!` makrosunu çağırdığımız için başarısız olacak ikinci bir test ekleme">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-03/src/lib.rs}}
```

</Listing>

`cargo test` kullanarak testleri tekrar çalıştırın. Çıktı Kod Listesi 11-4 gibi görünmeli ki bu `exploration` testinin geçtiğini ve `another`'in başarısız olduğunu gösterir.

<Listing number="11-4" caption="Bir test geçtiğinde ve bir test başarısız olduğunda test sonuçları">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-03/output.txt}}
```

</Listing>

<!-- manual-regeneration
rg panicked listings/ch11-writing-automated-tests/listing-11-03/output.txt
check that line number of panic matches line number in the following paragraph
 -->

`ok` yerine, `test tests::another` satırı `FAILED` gösterir. Bireysel sonuçlar ve özet arasında iki yeni bölüm görünür: İlk bölüm her test başarısızlığı için ayrıntılı nedeni gösterir. Bu durumda, `tests::another`'ın _src/lib.rs_ dosyasında 17. satırda `Make this test fail` mesajıyla panik attığı için başarısız olduğu ayrıntılarını alıyoruz. Sonraki bölüm sadece başarısız olan tüm testlerin adlarını listeler ki bu, çok fazla test ve çok fazla ayrıntılı başarısız test çıktısı olduğunda kullanışlıdır. Başarısız bir testin adını sadece o testi çalıştırmak için kullanabiliriz böylece onu daha kolay hata ayıklamasını (debug) yapabiliriz; testleri çalıştırmanın yolları hakkında ["Testlerin Nasıl Çalıştığı Kontrol Etme"][controlling-how-tests-are-run]<!-- ignore --> bölümünde daha fazla konuşacağız.

Özet satırı sonda görüntülenir: Genel olarak, test sonucumuz `FAILED`. Bir test geçti ve bir test başarısız oldu.

Farklı senaryolarda test sonuçlarının neye benzediğini gördüğünüze göre, artık `panic!` dışında testlerde kullanışlı olan birkaç makroya bakalım.

<!-- Old headings. Do not remove or links may break. -->

<a id="checking-results-with-the-assert-macro"></a>

### Sonuçları `assert!` ile Kontrol Etme (Checking Results with `assert!`)

Standart kütüphane tarafından sağlanan `assert!` makrosu, testteki bir koşulun `true` olarak değerlendirildiğinden emin olmak istediğinizde kullanışlıdır. `assert!` makrosuna bir Boolean olarak değerlenen bir argüman veririz. Değer `true` ise hiçbir şey olmaz ve test geçer. Değer `false` ise, `assert!` makrosu testi başarısız yapmak için `panic!` çağırır. `assert!` makrosunu kullanmak kodumuzun istediğimiz şekilde çalıştığını kontrol etmemize yardımcı olur.

Bölüm 5'te, Kod Listesi 5-15'de, burada Kod Listesi 11-5'te tekrarlanan bir `Rectangle` struct'ı ve bir `can_hold` yöntemini kullandık. Bu kodu _src/lib.rs_ dosyasına koyun, sonra `assert!` makrosunu kullanarak onun için bazı testler yazalım.

<Listing number="11-5" file-name="src/lib.rs" caption="Bölüm 5'ten `Rectangle` struct'ı ve onun `can_hold` yöntemi">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-05/src/lib.rs}}
```

</Listing>

`can_hold` yöntemi bir Boolean döndürür ki bu `assert!` makrosu için mükemmel bir kullanım durumudur. Kod Listesi 11-6'de, genişliği 8 ve yüksekliği 7 olan bir `Rectangle` örneği oluşturarak ve genişliği 5 ve yüksekliği 1 olan başka bir `Rectangle` örneğini tutabileceğini iddia ederek `can_hold` yöntemini kullanmayı test eden bir test yazarız.

<Listing number="11-6" file-name="src/lib.rs" caption="Daha büyük bir dikdörtgenin gerçekten daha küçük bir dikdörtgeni tutup tutmadığını kontrol eden `can_hold` için bir test">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-06/src/lib.rs:here}}
```

</Listing>

`tests` modülünün içindeki `use super::*;` satırını unutmayın. `tests` modülü Bölüm 7'de ["Modül Ağacındaki Bir Öğeye Atıf Yollar İçin"][paths-for-referring-to-an-item-in-the-module-tree]<!-- ignore --> bölümünde kapsadığımız normal görünürlük kurallarını takip eden normal bir modüldür. `tests` modülü bir iç modül olduğu için, dış modülde test edilen kodu iç modülün kapsamına getirmemiz gerekir. Burada bir joker (glob) kullanıyoruz, böylece dış modülde tanımladığımız her şey bu `tests` modülü için mevcuttur.

Testimizi `larger_can_hold_smaller` olarak adlandırdık ve ihtiyacımız olan iki `Rectangle` örneği oluşturduk. Sonra, `assert!` makrosunu çağırdık ve `larger.can_hold(&smaller)` çağırmanın sonucunu ona geçtik. Bu ifadenin `true` döndürmesi beklenir, böylece testimiz geçmelidir. Hadi bulalım!

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-06/output.txt}}
```

Geçti! Şimdi başka bir test ekleyelim, bu sefer daha küçük bir dikdörtgenin daha büyük bir dikdörtgeni tutamayacağını iddia ederek:

<span class="filename">Dosya Adı: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/src/lib.rs:here}}
```

Bu durumda `can_hold` fonksiyonunun doğru sonucu `false` olduğu için, onu `assert!` makrosuna geçirmeden önce sonucu reddetmemiz gerekir. Sonuç olarak, testimiz `can_hold` `false` dönerse geçer:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/output.txt}}
```

Geçen iki test! Şimdi kodumuza bir hata (bug) getirdiğimizde test sonuçlarımıza ne olduğunu görelim. `can_hold` yönteminin uygulamasını genişlikleri karşılaştırırken büyüktür işaretini (`>`) küçüktür işaretiyle (`<`) değiştirerek değiştireceğiz:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/src/lib.rs:here}}
```

Şimdi testleri çalıştırmak şu sonuçları üretir:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/output.txt}}
```

Testlerimiz hatayı yakaladı! Çünkü `larger.width` `8` ve `smaller.width` `5`, `can_hold` içindeki genişliklerin karşılaştırması artık `false` döndürüyor: 8, 5'ten küçük değildir.

<!-- Old headings. Do not remove or links may break. -->

<a id="testing-equality-with-the-assert_eq-and-assert_ne-macros"></a>

### `assert_eq!` ve `assert_ne!` ile Eşitlik Test Etme (Testing Equality with `assert_eq!` and `assert_ne!`)

İşlevselliği doğrulamanın yaygın bir yolu, test edilen kodun sonucunu ve kodun döndürmesini beklediğiniz değeri arasındaki eşitliği test etmektir. Bunu `assert!` makrosunu kullanarak ve `==` operatörünü kullanan bir ifadeyi ona geçirerek yapabilirsiniz. Ancak bu, standart kütüphanenin bu testi daha kullanışlı bir şekilde gerçekleştirmek için bir çift makro—`assert_eq!` ve `assert_ne!`—sağladığı kadar yaygın bir testtir. Bu makrolar sırasıyla iki argümanı eşitlik veya eşitsizlik için karşılaştırır. Ayrıca, iddia başarısız olursa iki değeri yazdırır ki bu, testin _neden_ başarısız olduğunu görmeyi kolaylaştırır; aksine, `assert!` makrosu sadece `==` ifadesi için `false` değeri aldığını, `false` değerine yol açan değerleri yazdırmaz.

Kod Listesi 11-7'de, parametresine `2` ekleyen `add_two` adında bir fonksiyon yazıyoruz ve sonra bu fonksiyonu `assert_eq!` makrosunu kullanarak test ediyoruz.

<Listing number="11-7" file-name="src/lib.rs" caption="`assert_eq!` makrosunu kullanarak `add_two` fonksiyonunu test etme">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-07/src/lib.rs}}
```

</Listing>

Geçtiğini kontrol edelim!

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-07/output.txt}}
```

`add_two(2)` çağırmanın sonucunu tutan `result` adında bir değişken oluşturduk. Sonra, `result` ve `4`'ü `assert_eq!` makrosuna argümanlar olarak geçtik. Bu test için çıktı satırı `test tests::it_adds_two ... ok` ve `ok` metni testimizin geçtiğini gösterir!

Şimdi `assert_eq!`'nin başarısız olduğunda neye benzediğini görmek için kodumuza bir hata getirelim. `add_two` fonksiyonunun uygulamasını `3` eklemek yerine değiştirin:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/src/lib.rs:here}}
```

Testleri tekrar çalıştırın:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/output.txt}}
```

Testlerimiz hatayı yakaladı! `tests::it_adds_two` testi başarısız oldu ve mesaj bize başarısız olan iddianın `left == right` olduğunu ve `left` ve `right` değerlerinin ne olduğunu söylüyor. Bu mesaj bize hata ayıklamayı başlatmada yardımcı olur: `left` argümanı, `add_two(2)` çağırmanın sonucunu aldığımız yer, `5` idi ancak `right` argümanı `4` idi. Bir sürü devam eden çok fazla testimiz olduğu zaman bunun özellikle yardımcı olabileceğini hayal edebilirsiniz.

Bazı dillerde ve test çerçevelerinde, eşitlik iddia fonksiyonlarına geçirilen parametrelere `expected` ve `actual` denir ve argümanları belirttiğimiz sıralama önemlidir. Ancak Rust'ta buna `left` ve `right` denir ve beklediğimiz değeri ve kodun ürettiği değeri belirttiğimiz sıralama önemli değildir. Bu testteki iddianı `assert_eq!(4, result)` olarak yazabiliriz ki bu `` assertion `left == right` failed `` gösteren aynı başarısız mesajını sonuçlandıracaktır.

`assert_ne!` makrosu, ona verdiğimiz iki değer eşit değilse geçer ve eşitse başarısız olur. Bu makro, bir değerin ne _olacağından_ emin değiliz ancak değerin kesinlikle _olmaması gerektiğinden_ emin olduğumuz durumlar için en kullanışlıdır. Örneğin, girdisini bir şekilde değiştirmeyi garanti eden ancak girdinin değiştirilme şeklinin testleri hangi haftada çalıştığına bağlı olan bir fonksiyonu test ediyorsak, iddia etmenin en iyi şeyi fonksiyonun çıktısının girdiye eşit olmaması olabilir.

Yüzey altında, `assert_eq!` ve `assert_ne!` makroları sırasıyla `==` ve `!=` operatörlerini kullanır. İddialar başarısız olduğunda, bu makrolar argümanlarını hata ayıklama biçiminde (debug formatting) yazdırır ki bu, karşılaştırılan değerlerin `PartialEq` ve `Debug` trait'lerini uygulamalarını gerektirdiği anlamına gelir. Tüm temel tipler ve standart kütüphanenin çoğu tipi bu trait'leri uygular. Kendi tanımladığınız struct'lar ve enum'lar için, bu tiplerin eşitliğini iddia etmek için `PartialEq` uygulamanız gerekir. Ayrıca, iddia başarısız olduğunda değerleri yazdırmak için `Debug` uygulamanız gerekir. Her iki trait türetilebilir (derivable) trait olduğu için, Bölüm 5'teki Kod Listesi 5-12'de bahsedildiği gibi, bu genellikle struct veya enum tanımınıza `#[derive(PartialEq, Debug)]` notunu eklemek kadar basittir. Bu ve diğer türetilebilir trait'ler hakkında daha fazla ayrıntı için Ek C, ["Türetilebilir Trait'ler,"][derivable-traits]<!-- ignore --> bölümüne bakın.

### Özel Başarısızlık Mesajları Ekleme (Adding Custom Failure Messages)

`assert!`, `assert_eq!` ve `assert_ne!` makrolarına başarısızlık mesajıyla birlikte yazdırılacak özel bir mesaj da ekleyebilirsiniz. Gerekli argümanlardan sonra belirtilen herhangi bir argüman `format!` makrosuna (Bölüm 8'deki ["`+` veya `format!` ile Birleştirme"][concatenating]<!--
ignore --> bölümünde tartışıldığı gibi) geçirilir böylece `{}` yer tutucuları (placeholders) içeren bir biçim dizesi ve bu yer tutuculara gidecek değerleri geçebilirsiniz. Özel mesajlar, bir iddianın ne anlama geldiğini belgelemek için kullanışlıdır; bir test başarısız olduğunda, kodun sorunun ne olduğu hakkında daha iyi bir fikiriniz olur.

Örneğin, isme göre insanları selamlayan bir fonksiyonumuz olduğunu ve fonksiyona geçtiğimiz ismin çıktıda göründüğünü test etmek istediğimizi söyleyin:

<span class="filename">Dosya Adı: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-05-greeter/src/lib.rs}}
```

Bu program için gereksinimler henüz anlaşılmadı ve selamlamanın başındaki `Hello` metninin değişeceğinden oldukça eminiz. Gereksinimler değiştiğinde testi güncellemek zorunda kalmayacağımıza karar verdik, bu yüzden `greeting` fonksiyonundan dönen değere tam eşitlik kontrol etmek yerine, çıktının girdi parametresinin metnini içerdiğini sadece iddia edeceğiz.

Şimdi `greeting`'i `name`'i hariç tutacak şekilde değiştirerek bu koda bir hata getirelim ve varsayılan test başarısızlığının neye benzediğini görelim:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/src/lib.rs:here}}
```

Bu testi çalıştırmak şu sonuçları üretir:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/output.txt}}
```

Bu sonuç sadece iddianın başarısız olduğunu ve iddianın hangi satırda olduğunu gösterir. Daha kullanışlı bir başarısızlık mesajı, `greeting` fonksiyonundan gelen değeri yazdıracaktır. `greeting` fonksiyonundan aldığımız gerçek değerle doldurulan yer tutuculu bir biçim dizesinden oluşan özel bir başarısızlık mesajı ekleyelim:

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/src/lib.rs:here}}
```

Şimdi testi çalıştırdığımızda, daha bilgilendirici bir hata mesajı alacağız:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/output.txt}}
```

Test çıktısında gerçekten aldığımız değeri görebiliriz ki bu, beklediğimiz ne olduğunu yerine ne olduğunu hata ayıklamada yardımcı olur.

### `should_panic` ile Panikleri Kontrol Etme (Checking for Panics with `should_panic`)

Dönüş değerlerini kontrol etmenin yanı sıra, kodumuzun hata durumlarını beklediğimiz şekilde ele aldığından emin olmak önemlidir. Örneğin, Bölüm 9'de Kod Listesi 9-13'te oluşturduğumuz `Guess` tipini düşünün. `Guess` kullanan başka kod, `Guess` örneklerinin sadece 1 ve 100 arasındaki değerleri içereceğini garantiye bağlıdır. Bu aralığın dışındaki bir değerle bir `Guess` örneği oluşturma girişiminin panik atacağından emin olan bir test yazabiliriz.

Bunu test fonksiyonumuza `should_panic` özniteliğini ekleyerek yaparız. Fonksiyon içindeki kod panik atarsa test geçer; fonksiyon içindeki kod panik atmazsa test başarısız olur.

Kod Listesi 11-8, beklediğimizde `Guess::new`'in hata durumlarının gerçekleştiğini kontrol eden bir test gösterir.

<Listing number="11-8" file-name="src/lib.rs" caption="Bir koşulun bir `panic!`'e neden olacağını test etme">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-08/src/lib.rs}}
```

</Listing>

`#[test]` özniteliğinden sonra ve uyguladığı test fonksiyonundan önce `#[should_panic]` özniteliğini yerleştiriyoruz. Bu test geçtiğinde sonucu bakalım:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-08/output.txt}}
```

İyi görünüyor! Şimdi kodumuza, değeri 100'den büyükse `new` fonksiyonunun panik atacağı koşulunu kaldırarak bir hata getirelim:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/src/lib.rs:here}}
```

Kod Listesi 11-8'deki testi çalıştırdığımızda, bu başarısız olacak:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/output.txt}}
```

Bu durumda çok kullanışlı bir mesaj alamıyoruz ancak test fonksiyonuna baktığımızda, `#[should_panic]` ile notlandığını görüyoruz. Aldığımız başarısızlık, test fonksiyonundaki kodun panik atmadığı anlamına gelir.

`should_panic` kullanan testler kesinsiz olabilir. `should_panic` testi, beklediğimizden farklı bir nedenle test panik atsa bile geçer. `should_panic` testlerini daha hassas hale getirmek için, `should_panic` özniteliğine isteğe bağlı `expected` parametresini ekleyebiliriz. Test kafesi, başarısızlık mesajının sağlanan metni içerdiğinden emin olacaktır. Örneğin, Kod Listesi 11-9'de `Guess` için değiştirilen kodu düşünün ki `new` fonksiyonu değerin çok küçük veya çok büyük olmasına göre farklı mesajlarla panik atar.

<Listing number="11-9" file-name="src/lib.rs" caption="Belirli bir alt dize içeren bir panik mesajı ile bir `panic!` test etme">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-09/src/lib.rs:here}}
```

</Listing>

Bu test geçecek çünkü `should_panic` özniteliğinin `expected` parametresine koyduğumuz değer, `Guess::new` fonksiyonunun panik attığı mesajının bir alt dizesidir. Beklediğimiz tüm panik mesajını belirtebilirdik ki bu durumda `Guess value must be less than or equal to 100, got 200` olurdu. Ne belirtmeyi seçtiğiniz, panik mesajının ne kadar benzersiz veya dinamik olduğuna ve testinizi ne kadar hassas istediğinize bağlıdır. Bu durumda, panik mesajının bir alt dizesi, test fonksiyonundaki kodun `else if value > 100` durumunu çalıştırdığından emin olmak için yeterlidir.

`expected` mesajına sahip bir `should_panic` testinin başarısız olduğunda ne olduğunu görmek için, `if value < 1` ve `else if value > 100` bloklarının gövdelerini değiştirerek kodumuza tekrar bir hata getirelim:

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/src/lib.rs:here}}
```

Bu sefer, `should_panic` testini çalıştırdığımızda, bu başarısız olacak:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/output.txt}}
```

Başarısızlık mesajı, bu testin beklediğimiz gibi panik attığını ancak panik mesajının beklenen `less than or equal to 100` dizesini içermediğini gösterir. Bu durumda aldığımız panik mesajı `Guess value must be greater than or equal to 1, got 200` idi. Şimdi hatamızın nerede olduğunu çözmeye başlayabiliriz!

### Testlerde `Result<T, E>` Kullanma (Using `Result<T, E>` in Tests)

Şimdiye kadar olan tüm testlerimiz başarısız olduğunda panik atarlar. `Result<T, E>` kullanan testler de yazabiliriz! İşte Kod Listesi 11-1'den, `Result<T, E>` kullanmak için yeniden yazılmış ve panik atmak yerine bir `Err` döndüren bir test:

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-10-result-in-tests/src/lib.rs:here}}
```

`it_works` fonksiyonu artık `Result<(), String>` dönüş tipine sahip. Fonksiyon gövdesinde, `assert_eq!` makrosunu çağırmak yerine, test geçtiğinde `Ok(())` ve test başarısız olduğunda içinde bir `String` olan bir `Err` döndürüyoruz.

Testleri bir `Result<T, E>` döndürecek şekilde yazmak, test gövdesinde soru işareti operatörünü kullanmanızı sağlar ki bu, içindeki herhangi bir işlem bir `Err` değişkeni dönerse başarısız olması gereken testler yazmanın kullanışlı bir yoludur.

`Result<T, E>` kullanan testlerde `#[should_panic]` notunu kullanamazsınız. Bir işlemin bir `Err` değişkeni döndürdüğünü iddia etmek için, `Result<T, E>` değerinde soru işareti operatörünü kullanmayın. Bunun yerine, `assert!(value.is_err())` kullanın.

Artık testleri yazmanın birkaç yolunu bildiğinize göre, testlerimizi çalıştırdığımızda ne olduğunu ve `cargo test` ile kullanabileceğimiz farklı seçenekleri keşfedelim.

[concatenating]: ch08-02-strings.html#concatenating-with--or-format
[bench]: ../unstable-book/library-features/test.html
[ignoring]: ch11-02-running-tests.html#ignoring-tests-unless-specifically-requested
[subset]: ch11-02-running-tests.html#running-a-subset-of-tests-by-name
[controlling-how-tests-are-run]: ch11-02-running-tests.html#controlling-how-tests-are-run
[derivable-traits]: appendix-03-derivable-traits.html
[doc-comments]: ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests
[paths-for-referring-to-an-item-in-the-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html