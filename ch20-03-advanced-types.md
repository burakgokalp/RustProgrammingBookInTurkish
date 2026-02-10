## Gelişmiş Türler (Advanced Types)

Rust'un tür sisteminin daha önce bahsettiğimiz ancak henüz tartışmadığımız bazı
özellikleri var. Önce newtype'leri genel olarak tartışarak türler olarak niçin yararlı olduklarını
inceleyerek başlayacağız. Sonra, tür takma adları (type aliases) gibi newtype'lere
benzer ancak hafifçe farklı semantiğe sahip bir özellige geçeceğiz. Ayrıca `!` türünü
ve dinamik olarak boyutlandırılmış türleri (dynamically sized types) tartışacağız.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-newtype-pattern-for-type-safety-and-abstraction"></a>

### Newtype Deseni ile Tür Güvenliği ve Soyutlama (Type Safety and Abstraction with Newtype Pattern)

Bu bölüm, daha önceki [“Newtype Deseniyle Dış Trait'leri Uygulama”][newtype]<!--
ignore --> bölümünü okuduğunuzu varsayar. Newtype deseni ayrıca bugüne kadar
tartıştığımız görevlerin ötesinde yararlıdır, bunlar değerlerin statik olarak hiçbir zaman
karıştırılmamasını ve bir değerinin birimlerini göstermeyi içerir. Birimin göstermek için
newtype'leri kullanmanın bir örneğini Kod Listesi 20-16'te gördünüz: `Millimeters`
ve `Meters` struct'lerinin bir newtype'te `u32` değerlerini sardığını hatırlayın. Eğer
`Millimeters` türünde bir parametreye sahip bir fonksiyon yazsaydık, o fonksiyonu yanlışlıkla
`Meters` veya düz bir `u32` değeriyle çağırmaya çalışan bir programı derleyemeyizdiniz.

Ayrıca, bir türün bazı uygulama detaylarını soyutlamak (abstract) için newtype desenini
kullanabiliriz: Yeni tür, özel iç türün API'sinden farklı genel bir API'yi açıklabilir (expose).

Newtype'ler ayrıca iç uygulamayı gizleyebilir. Örneğin, bir kişinin ID'sini
isimleriyle ilişkilendiren `HashMap<i32, String>`'i saran bir `People` türü sağlayabiliriz.
`People` kullanan kod sadece bizim sağladığımız genel API ile etkileşir, örneğin bir
isim string'ini `People` topluluğuna (collection) ekleme yöntemi gibi; o kodun içerde
bizim isimlere bir `i32` ID'si atadığımızı bilmesine gerekmez. Newtype deseni,
Bölüm 18'deki [“Uygulama Detaylarını Gizleyen Kapsülama”][encapsulation-that-hides-implementation-details]<!--
ignore --> bölümünde tartıştığımız uygulama detaylarını gizlemek için hafif bir yoldur.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-type-synonyms-with-type-aliases"></a>

### Tür Eş Anlamları ve Tür Takma Adları (Type Synonyms and Type Aliases)

Rust, mevcut bir türe başka bir isim vermek için bir _tür takma adı_ (type alias)
tanımlama yeteneği sağlar. Bunun için `type` anahtar kelimesini kullanırız. Örneğin,
şöyleki bir takma adı `Kilometers`'ı `i32` için oluşturabiliriz:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:here}}
```

Şimdi, `Kilometers` takma adı `i32` için bir _eş anlamlıdır_ (synonym); Kod Listesi
20-16'de oluşturduğumuz `Millimeters` ve `Meters` türlerinin aksine, `Kilometers` ayrı
yeni bir tür değildir. `Kilometers` türüne sahip değerler `i32` türünün değerleriyle aynı
muamele edilecek:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:there}}
```

`Kilometers` ve `i32` aynı tür olduğu için, her iki türden değerleri ekleyebiliriz ve
`i32` parametrelerini alan fonksiyonlara `Kilometers` değerleri geçebiliriz. Ancak, bu yöntemi
kullanarak, daha önce tartıştığımız newtype deseninden aldığımız tür kontrolü faydalarını (type-checking
benefits) alamayız. Başka bir deyişle, `Kilometers` ve `i32` değerlerini bir yerde
karıştırırsak, derleyici bize bir hata vermeyecektir.

Tür eş anlamları için ana kullanım durumu tekrarı azaltmaktır. Örneğin,
şöyle uzun bir türe sahip olabiliriz:

```rust,ignore
Box<dyn Fn() + Send + 'static>
```

Bu uzun türü fonksiyon imzalarında ve tür açıklamaları olarak kod boyunca
yazmak yorucu ve hataya açık olabilir. Kod Listesi 20-25'teki gibi bir kod dolu bir projeyi
hayal edin.

<Listing number="20-25" caption="Using a long type in many places">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-25/src/main.rs:here}}
```

</Listing>

Bir tür takma adı, tekrarı azaltarak bu kodu daha yönetilebilir hale getirir. Kod
Listesi 20-26'da, uzun tür için `Thunk` adında bir takma adı tanıttık ve türün tüm
kullanımlarını daha kısa `Thunk` takma adıyla değiştirebiliriz.

<Listing number="20-26" caption="Introducing a type alias, `Thunk`, to reduce repetition">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-26/src/main.rs:here}}
```

</Listing>

Bu kodu okumak ve yazmak çok daha kolay! Bir tür takma adı için anlamlı bir isim seçmek,
niyetinizi iletmeye de yardımcı olabilir (_thunk_, daha sonra değerlendirilecek kod için bir
sözcüktür, bu yüzden saklanan bir kapanma (closure) için uygun bir isimdir).

Tür takma adları ayrıca tekrarı azaltmak için `Result<T, E>` türüyle birlikte yaygın
olarak kullanılırlar. Standart kütüphanedeki `std::io` modülünü düşünün. Girdi/çıktı
(I/O) işlemleri genellikle işlemlerin çalışmadığı durumları ele almak için
`Result<T, E>` döndürür. Bu kütüphane mümkün tüm Girdi/Çıktı hatalarını temsil
eden `std::io::Error` struct'ına sahiptir. `std::io` içindeki birçok fonksiyon
`Result<T, E>` döndürür ki burada `E` `std::io::Error`'dir, örneğin `Write`
trait'indeki bu fonksiyonlar gibi:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-05-write-trait/src/lib.rs}}
```

`Result<..., Error>` çok tekrarlanıyor. Bu nedenle, `std::io` bu tür takma adı
bildirimine sahiptir:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:here}}
```

Bu bildirim `std::io` modülünde olduğu için, tam nitelikli takma adı `std::io::Result<T>`'yi
kullanabiliriz; yani, `E` olarak `std::io::Error` ile doldurulmuş bir `Result<T, E>`. `Write`
trait fonksiyon imzaları sonunda şöyle görünür:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:there}}
```

Tür takma adı iki yolla yardımcı olur: Kodu yazmayı daha kolay yapar _ve_ bize tüm
`std::io` boyunca tutarlı bir arayüz sağlar. Bu bir takma adı olduğu için, sadece başka bir
`Result<T, E>`'dir ki bu, onunla çalışan herhangi bir yöntemi kullanabileceğimiz anlamına gelir,
ayrıca `?` operatörü gibi özel sözdizimleriyle birlikte.

### Asla Dönmeyen Tür (The Never Type That Never Returns)

Rust'un tür teorisi argoçunda _boş tür_ (empty type) olarak bilinen `!` adında özel bir türü
vardır çünkü hiçbir değere yoktur. Biz onu _asla dönen tür_ (never type) olarak
adlandırmayı tercih ediyoruz çünkü bir fonksiyon asla dönmeyeceği zaman dönüş türü yerine
geçer. İşte bir örnek:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-07-never-type/src/lib.rs:here}}
```

Bu kod, "`bar` fonksiyonu asla döner." olarak okunur. Asla dönen fonksiyonlara
_sapılan fonksiyonlar_ (diverging functions) denir. `!` türünde değerler oluşturamayız bu yüzden
`bar` asla dönmeyebilir.

Ancak asla değerler oluşturamayacağınız bir türün ne işe yaradığı? Kod Listesi
2-5'teki, sayı tahmin oyununun bir kısmından kodunu hatırlayın; onun bir kısmını burada
Kod Listesi 20-27'de yeniden ürettik.

<Listing number="20-27" caption="A `match` with an arm that ends in `continue`">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:ch19}}
```

</Listing>

O zaman, bu kodda bazı detayları atladık. Bölüm 6'daki [“`match` Kontrol Akışı
Yapısı”][the-match-control-flow-construct]<!--
ignore --> bölümünde, `match` kollarının tümünün aynı
türü döndürmesi gerektiğini tartıştık. Bu yüzden, örneğin, şu kod çalışmaz:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-08-match-arms-different-types/src/main.rs:here}}
```

Bu koddaki `guess`'in türü bir tamsayı _ve_ bir string olmak zorundaydı ve Rust
`guess`'in sadece bir türe sahip olmasını gerektirir. Peki, `continue` ne döner? Biz nasıl
Kod Listesi 20-27'deki bir kol `continue` ile biterken bir koldan `u32` dönebildik?

Tahmin edebileceğiniz gibi, `continue`'in `!` değeri vardır. Yani, Rust `guess`'in türünü
hesapladığında, her iki match koluna bakar: biri `u32` değeriyle ve diğeri `!` değeriyle.
`!` asla bir değer sahibi olamayacağı için, Rust `guess`'in türünün `u32` olduğuna karar verir.

Bu davranışı resmi olarak tanımlama yolu şudur ki `!` türündeki ifadeler
herhangi bir diğer türe zorlanabilir (coerced). Bu `match` kolunu `continue` ile bitirmeye
izin veriyorum çünkü `continue` bir değer dönmüyor; bunun yerine, kontrolü döngünün başına
geri hareket ettiriyor (moves control back) bu yüzden `Err` durumunda, `guess`'e asla bir değer
atamayız.

Asla dönen türü (never type), `panic!` makrosuyla da yararlıdır. `Option<T>`
değerlerinde değer üretmek veya bu tanımla ile paniklemek için çağırdığımız `unwrap` fonksiyonunu
hatırlayın:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-09-unwrap-definition/src/lib.rs:here}}
```

Bu kodda, Kod Listesi 20-27'deki `match`'tekiyle aynı şey olur: Rust `val`'ın `T`
türüne ve `panic!`'in `!` türüne sahip olduğunu görür bu yüzden genel `match` ifadesinin sonucu
`T`'dir. Bu kod çalışır çünkü `panic!` bir değer üretmez; programı sonlandırır. `None`
durumunda, `unwrap`'den bir değer dönmeyeceğiz bu yüzden bu kod geçerlidir.

`!` türüne sahip nihai bir ifade bir döngüdür:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-10-loop-returns-never/src/main.rs:here}}
```

Burada, döngü asla biter bu yüzden `!`, ifadenin değeridir. Ancak, bir `break` dahil
etseydik doğru olmazdı çünkü `break`'e ulaştığında döngü sonlanacaktı.

### Dinamik Olarak Boyutlandırılmış Türler ve `Sized` Trait (Dynamically Sized Types and `Sized` Trait)

Rust, türleri hakkında belirli detayları bilmeye ihtiyaç duyar, örneğin belirli bir türdeki bir değer için
ne kadar bellek ayıracağı. Bu, tür sisteminin bir köşesini başlangıçta biraz kafa karıştırıyor
hale getiriyor: _dinamik olarak boyutlandırılmış türler_ (dynamically sized types) konsepti. Bazen
_DST'ler_ (DSTs) veya _boyutsuz türler_ (unsized types) olarak da adlandırılır, bu türler
boyutunu sadece çalışma zamanında (runtime) bilebileceğimiz değerlerle kod yazmamıza izin verir.

Dinamik olarak boyutlandırılmış bir tür olan `str`'nin detaylarına inelim ki bunu
kitap boyunca kullandık. Evet, `&str` değil ama kendi başına `str`, bir DST'dir. Birçok
durumda, örneğin bir kullanıcı tarafından girilen metni saklarken, string'in ne kadar uzun olduğunu
çalışma zamanına kadar bilemeyiz. Bu, `str` türünde bir değişken oluşturamayacağımızı ve
`str` türünde bir parametre alamayacağımızı anlamına gelir. Çalışmayan şu kodu düşünün:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-11-cant-create-str/src/main.rs:here}}
```

Rust, belirli bir türdeki herhangi bir değer için ne kadar bellek ayırması gerektiğini bilmeli
ve bir türdeki tüm değerler aynı bellek miktarını kullanmalıdır. Eğer Rust bize bu kodu yazmamıza
izin verseydi, bu iki `str` değeri aynı miktarda yer kaplaması gerekirdi. Ancak onların farklı
uzunlukları var: `s1` 12 bayt depolamaya ihtiyaç duyar ve `s2` 15 bayt gerektirir.
Bu yüzden, dinamik olarak boyutlandırılmış bir türü tutan bir değişken oluşturmak mümkün değildir.

Peki, ne yapmalıyız? Bu durumda, cevabı zaten biliyorsunuz: `s1` ve `s2`'nin türünü `str`
yerine string dilimi (`&str`) yapıyoruz. Bölüm 4'teki [“String Dilimleri”][string-slices]<!--
ignore --> bölümünden hatırlayın ki dilim veri yapısı sadece dilimin başlangıç konumunu ve
uzunluğunu saklar. Bu yüzden, `&T` `T`'in bulunduğu bellek adresini depolayan tek bir
değerdir ancak bir string dilimi _iki_ değerdir: `str`'nin adresi ve uzunluğu. Bu şekilde,
bir string dilim değerinin boyutunu derleme zamanında bilebiliriz: Bu, bir `usize`'nin uzunluğunun iki
katıdır. Yani, ne kadar uzun olursa olsun, bir dilimin boyutunu her zaman biliriz. Genel olarak,
dinamik olarak boyutlandırılmış türlerin Rust'ta kullanıldığı budur: Dinamik bilginin boyutunu
saklayan ekstra bir metadata parçasına sahiptirler. Dinamik olarak boyutlandırılmış türlerin altın kuralı
şudur ki dinamik olarak boyutlandırılmış türleri her zaman bir türde referans (pointer) arkasına
koymalıyız.

`str`'i pointer'ın tüm türleriyle birleştirebiliriz: örneğin, `Box<str>` veya
`Rc<str>`. Aslında, bunu daha önce gördünüz ancak farklı bir dinamik olarak boyutlandırılmış tür ile:
trait'ler. Her trait, trait'in adını kullanarak referans verebileceğimiz dinamik olarak
boyutlandırılmış bir türdür. Bölüm 18'deki [“Trait Nesnelerini Paylaşılan Davranışı
Soyutlamak İçin Kullanma”][using-trait-objects-to-abstract-over-shared-behavior]<!--
ignore --> bölümünde,
trait'leri trait nesneleri (trait objects) olarak kullanmak için, onları bir referans arkasına koymamız
gerektiğini söyledik, örneğin `&dyn Trait` veya `Box<dyn Trait>` (`Rc<dyn Trait>` da çalışır).

DST'lerle çalışmak için, Rust bir türün boyutunun derleme zamanında bilinip bilinmediğini
belirlemek için `Sized` trait'ini sağlar. Bu trait, boyutu derleme zamanında bilinen her şey için
otomatik olarak uygulanır. Ek olarak, Rust, her genel fonksiyona `Sized` üzerinde dolaylı bir
sınır (bound) ekler. Yani, şu genel fonksiyon tanımı:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-12-generic-fn-definition/src/lib.rs}}
```

aslında şunu yazmışsınız gibi muamele edilir:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-13-generic-implicit-sized-bound/src/lib.rs}}
```

Varsayılan olarak, genel fonksiyonlar yalnızca derleme zamanında bilinen boyutu olan türlerde
çalışacak. Ancak, şu özel sözdizimini kullanarak bu kısıtlamayı gevşetebilirsiniz:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-14-generic-maybe-sized/src/lib.rs}}
```

`?Sized` üzerinde bir trait sınırı, "`T` `Sized` olabilir veya olmayabilir," anlamına gelir ve bu
gösterim, genel türlerin derleme zamanında bilinen bir boyuta sahip olması gerektiği varsayılanı geçersiz
kılıyor (overrides). Bu anlamdaki `?Trait` sözdizimi sadece `Sized` için kullanılabilir, diğer
trait'ler için değil.

Ayrıca `t` parametresinin türünü `T`'den `&T`'ye değiştirdiğimizi not edin. Çünkü
tür `Sized` olmayabilir, onu bir türde referans arkasında kullanmamız gerekir. Bu durumda, bir
referans seçtik.

Sonra, fonksiyonlar ve kapanmalar (closures) hakkında konuşacağız!

[encapsulation-that-hides-implementation-details]: ch18-01-what-is-oo.html#encapsulation-that-hides-implementation-details
[string-slices]: ch04-03-slices.html#string-slices
[the-match-control-flow-construct]: ch06-02-match.html#the-match-control-flow-construct
[using-trait-objects-to-abstract-over-shared-behavior]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[newtype]: ch20-02-advanced-traits.html#implementing-external-traits-with-the-newtype-pattern