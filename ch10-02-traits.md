<!-- Old headings. Do not remove or links may break. -->

<a id="traits-defining-shared-behavior"></a>

## Trait'lerle Paylaşılan Davranışı Tanımlama (Defining Shared Behavior with Traits)

Bir _trait_, belirli bir tipin sahip olduğu ve diğer tiplerle paylaşabileceği işlevselliği tanımlar. Trait'leri paylaşılan davranışı soyut bir şekilde tanımlamak için kullanabiliriz. _Trait sınırları_ (trait bounds) kullanarak genel bir tipin belirli bir davranışa sahip olan herhangi bir tip olabileceğini belirtebiliriz.

> Not: Trait'ler, bazı farklılıklar olmasına rağmen, genellikle diğer dillerde _arayüzler_ (interfaces) olarak adlandırılan bir özelliğe benzer.

### Bir Trait Tanımlama (Defining a Trait)

Bir tipin davranışı, o tip üzerinde çağırabileceğimiz yöntemlerden (methods) oluşur. Tüm bu tipler üzerinde aynı yöntemleri çağırabiliyorsak, farklı tipler aynı davranışı paylaşır. Trait tanımları, yöntem imzalarını birlikte gruplayarak belirli bir amacı gerçekleştirmek için gerekli davranışlar kümesini tanımlamanın bir yoludur.

Örneğin, çeşitli türlerde ve miktarlarda metin tutan birden fazla struct'ımız olduğunu söyleyelim: belirli bir konumda doslanmış bir haber hikayesini tutan bir `NewsArticle` struct'ı ve en fazla 280 karakterlik bir metin ve yeni bir gönderi, bir yeniden gönderi (repost) veya başka bir gönderiye bir yanıt olup olmadığını belirten meta veri tutan bir `SocialPost`.

`NewsArticle` veya `SocialPost` örneğinde depolanmış olabilecek verilerin özetlerini görüntüleyebilen `aggregator` adlı bir medya toplayıcı (aggregator) kütüphane crate'i oluşturmak istiyoruz. Bunu yapmak için her tipten bir özete ihtiyacımız var ve bir örnekte `summarize` yöntemini çağırarak bu özet isteyeceğiz. Kod Listesi 10-12, bu davranışı ifade eden genel bir `Summary` trait tanımını gösterir.

<Listing number="10-12" file-name="src/lib.rs" caption="Bir `summarize` yöntemi tarafından sağlanan davranıştan oluşan `Summary` trait'i">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-12/src/lib.rs}}
```

</Listing>

Burada, `trait` anahtar kelimesini kullanarak bir trait bildiriyoruz ve sonra trait'in adını, bu durumda `Summary`. Ayrıca trait'i `pub` olarak bildiriyoruz, böylece bu crate'e bağımlı olan crate'ler de bu trait'i kullanabilir, birkaç örnekte göreceğimiz gibi. Küme parantezleri içinde, bu trait'i uygulayan tiplerin davranışlarını tanımlayan yöntem imzalarını bildiriyoruz, bu durumda `fn summarize(&self) -> String`.

Yöntem imzasından sonra, küme parantezleri içinde bir uygulama sağlamak yerine, bir noktalı virgül kullanıyoruz. Bu trait'i uygulayan her tipin yöntem gövdesi için kendi özel davranışını sağlaması gerekir. Derleyici, `Summary` trait'ine sahip herhangi bir tipin tam olarak bu imzada tanımlanmış `summarize` yöntemine sahip olacağını zorlayacaktır.

Bir trait gövdesinde birden fazla yöntem olabilir: Yöntem imzaları satır başına bir tane listelenir ve her satır bir noktalı virgülle biter.

### Bir Tipte Trait Uygulama (Implementing a Trait on a Type)

Şimdi `Summary` trait'inin yöntemlerinin istenen imzalarını tanımladığımıza göre, bunları medya toplayıcımızdaki tiplerde uygulayabiliriz. Kod Listesi 10-13, `NewsArticle` struct'ında `summarize`'nin dönüş değerini oluşturmak için başlık, yazar ve konumu kullanan bir `Summary` trait uygulamasını gösterir. `SocialPost` struct'ı için ise, gönderi içeriğinin zaten 280 karakter ile sınırlı olduğu varsayarak, `summarize`'yi kullanıcı adı ve ardından gönderinin tüm metni olarak tanımlıyoruz.

<Listing number="10-13" file-name="src/lib.rs" caption="`NewsArticle` ve `SocialPost` tiplerinde `Summary` trait'inin uygulanması">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-13/src/lib.rs:here}}
```

</Listing>

Bir tipe trait uygulamak, düzenli yöntemleri uygulamaya benzer. Fark, `impl`'den sonra uygulamak istediğimiz trait adını, ardından `for` anahtar kelimesini ve sonra trait'i uygulamak istediğimiz tipin adını koyuyoruz. `impl` bloğu içinde, trait tanımının tanımladığı yöntem imzalarını koyuyoruz. Her imzadan sonra noktalı virgül eklemek yerine, küme parantezleri kullanıyoruz ve trait'in yöntemlerinin belirli bir tip için sahip olmasını istediğimiz özel davranışla yöntem gövdesini dolduruyoruz.

Şimdi kütüphane `NewsArticle` ve `SocialPost` üzerinde `Summary` trait'ini uyguladığına göre, crate'in kullanıcıları `NewsArticle` ve `SocialPost` örneklerinde trait yöntemlerini düzenli yöntemleri çağırdığımız gibi çağırabilir. Tek fark, kullanıcının tiplerin yanı sıra trait'i de kapsama (scope) getirmesidir. İşte ikili (binary) bir crate'in bizim `aggregator` kütüphane crate'ini nasıl kullanabileceğine dair bir örnek:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-01-calling-trait-method/src/main.rs}}
```

Bu kod `1 new post: horse_ebooks: of course, as you probably already know, people` yazdırır.

`aggregator` crate'ine bağımlı olan diğer crate'ler de kendi tiplerinde `Summary` uygulamak için `Summary` trait'ini kapsamaya getirebilir. Not etmeniz gereken bir kısıtlama, bir trait'i bir tipe sadece trait veya tip veya her ikisi de crate'mize yerel ise uygulayabiliriz. Örneğin, `SocialPost` tipinin bizim `aggregator` crate'mize yerel olduğu için `aggregator` crate işlevselliğinin bir parçası olarak `SocialPost` gibi özel bir tipte `Display` gibi standart kütüphane trait'lerini uygulayabiliriz. Ayrıca, `Summary` trait'inin bizim `aggregator` crate'mize yerel olduğu için `aggregator` crate'mizde `Vec<T>` üzerinde `Summary` uygulayabiliriz.

Ancak dış tiplerde dış trait'leri uygulayamayız. Örneğin, `Display` ve `Vec<T>`'nin ikisi de standart kütüphanede tanımlandığı ve bizim `aggregator` crate'mize yerel olmadığı için `aggregator` crate'miz içinde `Vec<T>` üzerinde `Display` trait'ini uygulayamayız. Bu kısıtlama, _tutarlılık_ (coherence) denilen ve daha spesifik olarak _yetim kural_ (orphan rule) denilen bir özelliğin bir parçasıdır, bu şekilde adlandırılır çünkü üst tip mevcut değildir. Bu kural, diğer insanların kodunun sizin kodunuzu bozamayacağını ve tam tersini sağlar. Kural olmadan, iki crate aynı tip için aynı trait'i uygulayabilir ve Rust hangi uygulamayı kullanacağını bilemezdi.

<!-- Old headings. Do not remove or links may break. -->

<a id="default-implementations"></a>

### Varsayılan Uygulamaları Kullanma (Using Default Implementations)

Bazen trait içindeki yöntemlerin bazılarının veya tümünün her tipteki tüm yöntemler için uygulamalar zorunlu tutmak yerine varsayılan davranışa sahip olması kullanışlıdır. Sonra, bir tipe trait uyguladığımızda, her yöntemin varsayılan davranışını koruyabilir veya geçersiz kılabiliriz.

Kod Listesi 10-14'te, Kod Listesi 10-12'de yaptığımız gibi sadece yöntem imzasını tanımlamak yerine, `Summary` trait'inin `summarize` yöntemi için bir varsayılan dize belirtiyoruz.

<Listing number="10-14" file-name="src/lib.rs" caption="`summarize` yönteminin varsayılan uygulamasına sahip `Summary` trait'inin tanımlanması">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-14/src/lib.rs:here}}
```

</Listing>

`NewsArticle` örneklerini özetlemek için varsayılan uygulamayı kullanmak için, `impl Summary for NewsArticle {}` boş bir `impl` bloğu belirtiyoruz.

`NewsArticle` üzerinde `summarize` yöntemini doğrudan artık tanımlamamıza rağmen, bir varsayılan uygulama sağladık ve `NewsArticle`'ın `Summary` trait'ini uyguladığını belirttik. Sonuç olarak, hala `NewsArticle` örneğinde `summarize` yöntemini şu şekilde çağırabiliriz:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-02-calling-default-impl/src/main.rs:here}}
```

Bu kod `New article available! (Read more...)` yazdırır.

Varsayılan bir uygulama oluşturmak, Kod Listesi 10-13'teki `SocialPost` üzerinde `Summary` uygulaması hakkında hiçbir şeyi değiştirmemizi gerektirmez. Bunun nedeni, varsayılan bir uygulamayı geçersiz kılmanın sözdiziminin, varsayılan uygulamaya sahip olmayan bir trait yöntemini uygulama sözdizimiyle aynı olmasıdır.

Varsayılan uygulamalar, diğer yöntemlerin varsayılan uygulamaya sahip olmamasına rağmen aynı trait içindeki diğer yöntemleri çağırabilir. Bu şekilde, bir trait çok fazla kullanışlı işlevsellik sağlayabilir ve uygulayıcıların sadece küçük bir kısmını belirtmesini gerektirebilir. Örneğin, `Summary` trait'ini uygulanması gereken bir `summarize_author` yöntemine sahip olacak şekilde ve ardından `summarize_author` yöntemini çağıran varsayılan bir uygulamaya sahip bir `summarize` yöntemi tanımlayabiliriz:

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:here}}
```

Bu `Summary` sürümünü kullanmak için, bir tipe trait uyguladığımızda sadece `summarize_author` tanımlamamız gerekir:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:impl}}
```

`summarize_author` tanımladıktan sonra, `SocialPost` struct'ının örneklerinde `summarize` çağırabiliriz ve `summarize`'nin varsayılan uygulaması sağladığımız `summarize_author` tanımını çağırır. `summarize_author` uyguladığımız için, `Summary` trait'i daha fazla kod yazmamızı gerektirmeden bize `summarize` yönteminin davranışını verdi. İşte bunun neye benzediği:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/main.rs:here}}
```

Bu kod `1 new post: (Read more from @horse_ebooks...)` yazdırır.

Aynı yöntemin geçersiz kılan uygulamasından varsayılan uygulamayı çağırmanın mümkün olmadığını unutmayın.

<!-- Old headings. Do not remove or links may break. -->

<a id="traits-as-parameters"></a>

### Trait'leri Parametre Olarak Kullanma (Using Traits as Parameters)

Artık trait'leri nasıl tanımlayacağınızı ve uygulayacağınızı bildiğinize göre, birçok farklı tipi kabul eden fonksiyonları tanımlamak için trait'leri nasıl kullanabileceğimizi keşfedebiliriz. Kod Listesi 10-13'te `NewsArticle` ve `SocialPost` tiplerinde uyguladığımız `Summary` trait'ini kullanarak, `item` parametresinde `Summary` trait'ini uygulayan bazı tipin `summarize` yöntemini çağıran bir `notify` fonksiyonu tanımlayacağız. Bunu yapmak için, şu şekilde `impl Trait` sözdizimini kullanıyoruz:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-04-traits-as-parameters/src/lib.rs:here}}
```

`item` parametresi için somut bir tip yerine, `impl` anahtar kelimesini ve trait adını belirtiyoruz. Bu parametre, belirtilen trait'i uygulayan herhangi bir tipi kabul eder. `notify` gövdesinde, `Summary` trait'inden gelen herhangi bir yöntemi, örneğin `summarize`, `item` üzerinde çağırabiliriz. `NewsArticle` veya `SocialPost`'un herhangi bir örneğini geçirerek `notify` çağırabiliriz. `String` veya `i32` gibi başka bir tiple fonksiyonu çağıran kod derlenmez, çünkü bu tipler `Summary` uygulamıyor.

<!-- Old headings. Do not remove or links may break. -->

<a id="fixing-the-largest-function-with-trait-bounds"></a>

#### Trait Sınırı Sözdizimi (Trait Bound Syntax)

`impl Trait` sözdizimi basit durumlar için çalışır ancak aslında _trait sınırı_ (trait bound) denilen daha uzun bir form için sözdizim şekeri (syntax sugar)dir; şöyle görünür:

```rust,ignore
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

Bu uzun form önceki bölümdeki örneğe eşittir ancak daha ayrıntılıdır. Trait sınırlarını genel tip parametresi bildiriminden sonra ve açılı ayraçlar içinde, iki nokta üstüste koyarak yerleştiriyoruz.

`impl Trait` sözdizimi uygundur ve basit durumlarda daha kısa kod sağlar, tam trait sınırı sözdizimi ise diğer durumlarda daha fazla karmaşıklık ifade edebilir. Örneğin, `Summary` uygulayan iki parametremiz olabilir. Bunu `impl Trait` sözdizimiyle yapmak şöyle görünür:

```rust,ignore
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```

Eğer bu fonksiyonun `item1` ve `item2`'nin farklı tiplerde olmasına (her iki tip de `Summary` uyguladığı sürece) izin vermesini istiyorsak `impl Trait` kullanmak uygundur. Ancak her iki parametrenin aynı tipte olmasını zorlamak istiyorsak, bir trait sınırı kullanmalıyız, şöyle:

```rust,ignore
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

`item1` ve `item2` parametrelerinin tipi olarak belirtilen genel tip `T`, fonksiyonu `item1` ve `item2` için argüman olarak geçirilen değerin somut tipinin aynı olması gerektiği şekilde kısıtlar.

<!-- Old headings. Do not remove or links may break. -->

<a id="specifying-multiple-trait-bounds-with-the--syntax"></a>

#### `+` Sözdizimiyle Çoklu Trait Sınırları (Multiple Trait Bounds with the `+` Syntax)

Birden fazla trait sınırı da belirtebiliriz. `notify` fonksiyonunun `item` üzerinde `summarize` yanında görüntüleme formatlamasını (display formatting) da kullanmasını istediğimizi söyleyelim: `notify` tanımında `item`'ın hem `Display` hem de `Summary` uygulaması gerektiğini belirtiyoruz. Bunu `+` sözdizimini kullanarak yapabiliriz:

```rust,ignore
pub fn notify(item: &(impl Summary + Display)) {
```

`+` sözdizimi genel tipler üzerinde trait sınırlarıyla da geçerlidir:

```rust,ignore
pub fn notify<T: Summary + Display>(item: &T) {
```

İki trait sınırı belirtilen `notify` gövdesi `summarize` çağırabilir ve `item`'ı formatlamak için `{}` kullanabilir.

#### `where` Tümceleriyle Daha Net Trait Sınırları (Clearer Trait Bounds with `where` Clauses)

Çok fazla trait sınırı kullanmanın dezavantajları vardır. Her genel tipin kendi trait sınırları vardır, bu yüzden birden fazla genel tip parametresi olan fonksiyonlar fonksiyon adı ve parametre listesi arasında çok fazla trait sınırı bilgisi içerebilir ve bu da fonksiyon imzasını okumayı zorlaştırır. Bu nedenle, Rust fonksiyon imzasından sonra bir `where` tümcesi içinde trait sınırlarını belirtmek için alternatif sözdizime sahiptir. Böylece, bunu yazmak yerine:

```rust,ignore
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

bir `where` tümcesi kullanabiliriz, şöyle:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-07-where-clause/src/lib.rs:here}}
```

Bu fonksiyonun imzası daha az karışık: Fonksiyon adı, parametre listesi ve dönüş tipi birbirine yakındır, çok fazla trait sınırları olmayan bir fonksiyona benzer.

### Trait Uygulayan Tipleri Döndürme (Returning Types That Implement Traits)

Bir trait uygulayan bir tipten bir değer döndürmek için dönüş pozisyonunda da `impl Trait` sözdizimini kullanabiliriz, burada gösterildiği gibi:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-05-returning-impl-trait/src/lib.rs:here}}
```

Dönüş tipi için `impl Summary` kullanarak, `returns_summarizable` fonksiyonunun somut tipi adlandırmadan `Summary` trait'ini uygulayan bazı tipi döndürdüğünü belirtiyoruz. Bu durumda, `returns_summarizable` bir `SocialPost` döndürüyor, ancak bu fonksiyonu çağıran kodun bunu bilmesine gerek yok.

Dönüş tipini sadece uyguladığı trait ile belirtme yeteneği, Bölüm 13'te kapsayacağımız kapamalar (closures) ve yineleyiciler (iterators) bağlamında özellikle kullanışlıdır. Kapamalar ve yineleyiciler sadece derleyicinin bildiği veya belirtmek çok uzun olan tipler oluşturur. `impl Trait` sözdizimi, çok uzun bir tip yazmak zorunda kalmadan bir fonksiyonun `Iterator` trait'ini uygulayan bazı tip döndürdüğünü kısa şekilde belirtmenize izin verir.

Ancak, yalnızca tek bir tip döndürüyorsanız `impl Trait` kullanabilirsiniz. Örneğin, dönüş tipi `impl Summary` olarak belirtilen bir `NewsArticle` veya bir `SocialPost` döndüren şu kod çalışmaz:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-06-impl-trait-returns-one-type/src/lib.rs:here}}
```

Bir `NewsArticle` veya bir `SocialPost` döndürmek, derleyicide `impl Trait` sözdiziminin nasıl uygulandığı hakkındaki kısıtlamalar nedeniyle izin verilmez. Bölüm 18'deki ["Trait Nesnelerini Paylaşılan Davranış Üzerinde Soyutlama İçin Kullanma"][trait-objects]<!-- ignore --> bölümünde bu davranışı sahip bir fonksiyonu nasıl yazacağımızı kapsayacağız.

### Trait Sınırlarını Kullanarak Koşullu Olarak Yöntem Uygulama (Using Trait Bounds to Conditionally Implement Methods)

Genel tip parametreleri kullanan bir `impl` bloğuyla bir trait sınırı kullanarak, belirtilen trait'leri uygulayan tipler için koşullu olarak yöntemler uygulayabiliriz. Örneğin, Kod Listesi 10-15'teki `Pair<T>` tipi her zaman `Pair<T>`'nin yeni bir örneğini döndüren `new` fonksiyonu uygular (Bölüm 5'teki ["Yöntem Sözdizimi"][methods]<!-- ignore --> bölümünden `Self`'in `impl` bloğunun tipi için bir tip takma adı olduğunu, bu durumda `Pair<T> olduğunu hatırlayın). Ancak bir sonraki `impl` bloğunda, `Pair<T>` sadece `cmp_display` yöntemini uygular, eğer iç tipi `T` karşılaştırmayı etkinleştiren `PartialOrd` trait'ini _ve_ yazdırmayı etkinleştiren `Display` trait'ini uygularsa.

<Listing number="10-15" file-name="src/lib.rs" caption="Trait sınırlarına bağlı olarak genel tipte yöntemlerin koşullu uygulanması">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-15/src/lib.rs}}
```

</Listing>

Ayrıca, başka bir trait uygulayan herhangi bir tip için bir trait'i koşullu olarak uygulayabiliriz. Trait sınırlarını karşılayan herhangi bir tipte bir trait'in uygulamalarına _genel uygulamalar_ (blanket implementations) denir ve Rust standart kütüphanesinde yaygın olarak kullanılır. Örneğin, standart kütüphane `Display` trait'ini uygulayan herhangi bir tip üzerinde `ToString` trait'ini uygular. Standart kütüphanedeki `impl` bloğu bu koda benzer:

```rust,ignore
impl<T: Display> ToString for T {
    // --snip--
}
```

Standart kütüphanenin bu genel uygulamaya sahip olması nedeniyle, `Display` trait'ini uygulayan herhangi bir tip üzerinde `ToString` trait'i tarafından tanımlanan `to_string` yöntemini çağırabiliriz. Örneğin, tamsayıların `Display` uyguladığı için tamsayıları bunları gibi karşılık gelen `String` değerlerine dönüştürebiliriz:

```rust
let s = 3.to_string();
```

Genel uygulamalar trait'in dokümantasyonunun "Implementors" bölümünde görünür.

Trait'ler ve trait sınırları, tekrarlamayı azaltmak için genel tip parametrelerini kullanan ancak derleyiciye genel tipin belirli bir davranışa sahip olmasını istediğimizi de belirten kod yazmamıza izin verir. Derleyici daha sonra trait sınırı bilgisini, kodumuzla kullanılan tüm somut tiplerin doğru davranışı sağladığını kontrol etmek için kullanabilir. Dinamik olarak yazılan dillerde, bir yöntemi tanımlamayan bir tipte bir yöntem çağırırsak çalışma zamanında bir hata alırdık. Ancak Rust bu hataları derleme zamanına taşır, böylece kodumuzun çalışabilmesi için sorunları düzeltmeye zorunlu kalırız. Ayrıca, çalışma zamanında davranış kontrol eden kod yazmamıza gerek yoktur, çünkü bunu derleme zamanında zaten kontrol ettik. Bunu yapmak, jeneriklerin esnekliğinden vazgeçmeden performansı iyileştirir.

[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[methods]: ch05-03-method-syntax.html#method-syntax