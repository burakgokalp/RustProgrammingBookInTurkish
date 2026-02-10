## Gelişmiş Fonksiyonlar ve Kapanmalar (Advanced Functions and Closures)

Bu bölüm, fonksiyonlar ve kapanmalar (closures) ile ilgili bazı gelişmiş özellikleri
inceliyor, bunlar fonksiyon işaretçileri ve kapanmaları döndürmeyi içerir.

### Fonksiyon İşaretçileri (Function Pointers)

Kapanmaları fonksiyonlara nasıl geçeceğimizi tartıştık; ayrıca düzenli fonksiyonları da
fonksiyonlara geçebilirsiniz! Bu teknik, yeni bir kapanma tanımlamak yerine zaten tanımladığınız bir
fonksiyonu geçmek istediğinizde yararlıdır. Fonksiyonlar `fn` türüne (küçük harfli _f_),
`Fn` kapanma trait'i ile karıştırılmaması gereken, zorlanır (coerce). `fn` türü
_fonksiyon işaretçisi_ (function pointer) olarak adlandırılır. Fonksiyon işaretçileriyle
fonksiyonları geçmek, fonksiyonları diğer fonksiyonlara argüman olarak kullanmanıza izin verir.

Bir parametrenin bir fonksiyon işaretçisi olduğunu belirtme sözdizimi kapanmaların sözdizimine
benzerdir, Kod Listesi 20-28'te gösterildiği gibi, burada parametresine 1 ekleyen
`add_one` adında bir fonksiyon tanımladık. `do_twice` fonksiyonu iki parametre alır: `i32`
parametresi alan ve `i32` döndüren herhangi bir fonksiyona fonksiyon işaretçisi ve bir `i32` değeri.
`do_twice` fonksiyonu fonksiyon `f`'i iki kez çağırır, ona `arg` değerini geçer, sonra iki
fonksiyon çağırma sonucunu bir araya ekler. `main` fonksiyonu `add_one` ve `5` argümanlarıyla
`do_twice`'i çağırır.

<Listing number="20-28" file-name="src/main.rs" caption="Using `fn` type to accept a function pointer as an argument">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-28/src/main.rs}}
```

</Listing>

Bu kod `The answer is: 12` yazdırır. `do_twice` içindeki `f` parametresinin bir `i32`
türünde bir parametre alan ve `i32` döndüren bir `fn` olduğunu belirtiyoruz. Sonra `do_twice`'nin
gövdesinde `f`'i çağırabiliriz. `main` içinde, `do_twice`'in ilk argümanı olarak fonksiyon adı
`add_one`'i geçebiliriz.

Kapanmaların aksine, `fn` bir trait yerine bir türdür bu yüzden `Fn` trait'lerinden birini
trait sınırı olarak genel bir tür parametresi bildirmek yerine doğrudan `fn`'i parametre türü olarak belirtiyoruz.

Fonksiyon işaretçileri kapanma trait'lerinin tüm üçünü (`Fn`, `FnMut` ve `FnOnce`) uygular,
bu, bir kapanma bekleyen bir fonksiyon için argüman olarak her zaman bir fonksiyon işaretçisi geçebileceğiniz
anlamına gelir. Fonksiyonlarınızın fonksiyonları veya kapanmaları kabul etmesi için genel bir tür ve kapanma
trait'lerinden biri kullanarak fonksiyonlar yazmak en iyisidir.

Yine de, sadece `fn`'i kabul etmek ve kapanmaları kabul etmemek isteyeceğiniz bir örnek şudur:
kapanmaları olmayan harici kodla arayüz oluşturduğunuzda: C fonksiyonları argüman olarak fonksiyonları
kabul edebilir ancak C'de kapanmalar yoktur.

Satır içi tanımlanan bir kapanma veya adlandırılmış bir fonksiyon kullanabileceğiniz bir örnek olarak,
standart kütüphanede `Iterator` trait'i tarafından sağlanan `map` yönteminin kullanımına bakalım.
`map` yöntemini sayıların bir vektörünü string'lerin vektörüne dönüştürmek için kullanmak için, bir kapanma
kullanabiliriz, Kod Listesi 20-29'teki gibi.

<Listing number="20-29" caption="Using a closure with `map` method to convert numbers to strings">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-29/src/main.rs:here}}
```

</Listing>

Veya kapanma yerine `map` için argüman olarak bir fonksiyon adlandırabiliriz. Kod Listesi
20-30 bunun nasıl görüneceğini gösterir.

<Listing number="20-30" caption="Using a `String::to_string` function with `map` method to convert numbers to strings">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-30/src/main.rs:here}}
```

</Listing>

`to_string` adında birden çok fonksiyon olduğu için [“Gelişmiş Trait'ler”][advanced-traits]<!--
ignore --> bölümünde tartıştığımız tam nitelikli sözdizimi kullanmalıyımızı not edin.

Burada, `Display` uygulayan herhangi bir tür için standart kütüphanenin uyguladığı `ToString`
trait'inde tanımlanan `to_string` fonksiyonunu kullanıyoruz.

Bölüm 6'daki [“Enum Değerleri”][enum-values]<!--
ignore --> bölümünden tanımladığımız her enum varyantının
adının ayrıca bir başlatıcı fonksiyonu haline geldiğini hatırlayın. Bu başlatıcı fonksiyonları,
kapanma trait'lerini uygulayan fonksiyon işaretçileri olarak kullanabiliriz, bu, kapanmaları
alan yöntemler için argüman olarak başlatıcı fonksiyonlarını belirleyebileceğimiz anlamına gelir,
Kod Listesi 20-31'de görüldüğü gibi.

<Listing number="20-31" caption="Using an enum initializer with `map` method to create a `Status` instance from numbers">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-31/src/main.rs:here}}
```

</Listing>

Burada, `Status::Value` başlatıcı fonksiyonunu kullanarak `map`'in çağrıldığı aralıktaki her `u32`
değeri kullanarak `Status::Value` örnekleri oluşturuyoruz. Bazı insanlar bu stili tercih eder ve bazıları
kapanmaları kullanmayı tercih eder. Aynı koda derlenirler bu yüzden hangi stil size daha netse onu kullanın.

### Kapanmaları Döndürme (Returning Closures)

Kapanmalar trait'ler tarafından temsil edilir, bu, kapanmaları doğrudan döndüremeyeceğiniz
anlamına gelir. Bir trait dönmek isteyebileceğiniz çoğu durumda, trait'i uygulayan somut
bir türü fonksiyonun dönüş değeri olarak kullanabilirsiniz. Ancak, genellikle bunu kapanmalarla
yapamazsınız çünkü dönebilir somut türe sahip değiller; kapanma kapsamından (scope) herhangi bir
değer yakalarsa, dönüş türü olarak fonksiyon işaretçisi `fn` kullanmanıza izin verilmez, örneğin.

Bunun yerine, normal olarak Bölüm 10'da öğrendiğimiz `impl Trait` sözdizimini kullanacaksınız.
`Fn`, `FnOnce` ve `FnMut` kullanarak herhangi bir fonksiyon türünü dönebilirsiniz. Örneğin, Kod
Listesi 20-32'deki kod gayet güzel bir şekilde derlenecek.

<Listing number="20-32" caption="Returning a closure from a function using the `impl Trait` syntax">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-32/src/lib.rs}}
```

</Listing>

Ancak, Bölüm 13'teki [“Kapanma Türlerini Çıkarımı ve Açıklama”
][closure-types]<!--
ignore --> bölümünde belirttiğimiz gibi, her kapanma ayrıca kendi ayrı türüdür. Aynı
imza (signature) ancak farklı uygulamalara sahip birden çok fonksiyonla çalışmanız gerekiyorsa, onlar için
bir trait nesnesi (trait object) kullanmanız gerekecek. Kod Listesi 20-33'teki gibi bir kod yazarsanız ne
olacağını düşünelim.

<Listing file-name="src/main.rs" number="20-33" caption="Creating a `Vec<T>` of closures defined by functions that return `impl Fn` types">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-33/src/main.rs}}
```

</Listing>

Burada iki fonksiyonumuz var: `returns_closure` ve `returns_initialized_closure`, ikisi de
`impl Fn(i32) -> i32` döndürüyor. Aynı türü uygulamalarına rağmen döndürdükleri
kapanmaların farklı olduğunu not edin. Bunu derlemeye çalışırsak, Rust bize çalışmayacağını bildirir:

```text
{{#include ../listings/ch20-advanced-features/listing-20-33/output.txt}}
```

Hata mesajı bize her `impl Trait` döndürdüğümüzde, Rust'ın bizim için oluşturduğu ayrıntıların
detaylarına bakamadığımız veya Rust'ın oluşturacağı türü tahmin edip kendimiz yazamayacağımız
benzersiz bir _opak tür_ (opaque type) oluşturduğunu söyler. Bu yüzden, bu fonksiyonlar
aynı trait'i uygulayan kapanmaları döndürse bile, `Fn(i32) -> i32`, Rust'ın her biri için oluşturduğu
opak türler farklıdır. (Bu, Bölüm 17'deki [“`Pin` Türü ve `Unpin`
Trait”][future-types]<!--
ignore -->'te gördüğümüz gibi, aynı çıkış türüne sahip olsalar bile
farklı async blokları için farklı somut türler ürettiğine benzer.) Bu sorunun çözümünü artık birkaç kez
gördük: Bir trait nesnesi kullanabiliriz, Kod Listesi 20-34'teki gibi.

<Listing number="20-34" caption="Creating a `Vec<T>` of closures defined by functions that return `Box<dyn Fn>` so that they have the same type">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-34/src/main.rs:here}}
```

</Listing>

Bu kod gayet güzel bir şekilde derlenecektir. Trait nesneleri hakkında daha fazlası için, Bölüm
18'deki [“Trait Nesnelerini Paylaşılan Davranışı Soyutlamak İçin Kullanma”
][trait-objects]<!--
ignore --> bölümüne bakın.

Sonra, makrolara bakalım!

[advanced-traits]: ch20-02-advanced-traits.html#advanced-traits
[enum-values]: ch06-01-defining-an-enum.html#enum-values
[closure-types]: ch13-01-closures.html#closure-type-inference-and-annotation
[future-types]: ch17-03-more-futures.html
[trait-objects]: ch18-02-trait-objects.html