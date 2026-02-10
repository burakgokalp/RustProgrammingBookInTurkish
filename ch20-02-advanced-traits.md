## Gelişmiş Trait'ler (Advanced Traits)

Trait'leri ilk kez [“Trait'lerle Paylaşılan Davranışı Tanımlama”][traits]<!--
ignore --> bölümünde, Bölüm 10'da ele aldık ancak daha gelişmiş detayları tartışmadık.
Şimdi Rust hakkında daha fazla şey bildiğimize göre, detaylara girebiliriz.

<!-- Old headings. Do not remove or links may break. -->

<a id="specifying-placeholder-types-in-trait-definitions-with-associated-types"></a>
<a id="associated-types"></a>

### İlişkili Türlerle Trait Tanımlama (Defining Traits with Associated Types)

_İlişkili türler_ (associated types), bir tür yer tutucusunu (type placeholder) bir trait ile
bağlar böylece trait yöntemi tanımları, imzalarında bu yer tutucu türleri kullanabilir.
Trait'in uygulayıcısı (implementor), belirli uygulama için yer tutucu tür yerine kullanılacak somut
türü belirler. Bu şekilde, trait'i uygulana kadar o türlerin tam olarak ne olduğunu
bilmeden bazı türleri kullanan bir trait tanımlayabiliriz.

Bu bölümdeki gelişmiş özelliklerin çoğunu nadiren ihtiyaç duyulan olarak
açıkladık. İlişkili türler orta bir yerde: Kitabın geri kalanında açıklanan
özelliklerden daha nadiren kullanılıyor ancak bu bölümde tartışılan diğer birçok özellikten
daha yaygın.

İlişkili türe sahip bir trait'in bir örneği, standart kütüphanenin sağladığı `Iterator`
trait'idir. İlişkili tür `Item` adında ve `Iterator` trait'ini uygulayan türün üzerinde
yinelediği (iterating) değerlerin türünü temsil eder. `Iterator` trait'inin tanımı Kod
Listesi 20-13'ta gösterildiği gibidir.

<Listing number="20-13" caption="The definition of `Iterator` trait that has an associated type `Item`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-13/src/lib.rs}}
```

</Listing>

`Item` türü bir yer tutucudur ve `next` yönteminin tanımı, `Option<Self::Item>`
türündeki değerleri döndürdüğünü gösterir. `Iterator` trait'inin uygulayıcıları, `Item`
için somut bir tür belirler ve `next` yöntemi o somut türde bir değer içeren
bir `Option` döndürür.

İlişkili türler, sonuncunun hangi türleri ele alabileceğini belirtmeden bir fonksiyonu
tanımlamamıza izin vermesi bakımından genel türler (generics) benzer bir konsept gibi
görünebilir. İki konsept arasındaki farkı incelemek için, `Counter` adında bir
türde `Iterator` trait'ini uygulayan ve `Item` türünün `u32` olduğunu belirten bir uygulamaya
bakacağız:

<Listing file-name="src/lib.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-22-iterator-on-counter/src/lib.rs:ch19}}
```

</Listing>

Bu sözdizimi (syntax) genel türlerin sözdizimine karşılaştırılabilir gibi görünüyor.
Peki, neden Kod Listesi 20-14'ta gösterildiği gibi sadece `Iterator` trait'ini genel
türlerle tanımlamıyoruz?

<Listing number="20-14" caption="A hypothetical definition of `Iterator` trait using generics">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-14/src/lib.rs}}
```

</Listing>

Fark şudur ki genel türler kullandığımızda, Kod Listesi 20-14'teki gibi, her
uyulamada türleri açıklamalıyız (annotate); çünkü `Iterator<String>`'i `Counter` için veya
diğer herhangi bir tür için de uygulayabiliriz, `Counter` için `Iterator`'ın birden fazla
uygulamasına sahip olabiliriz. Başka bir deyişle, bir trait'in bir genel parametresi olduğunda,
bir tür için birden fazla kez uygulanabilir, her seferinde genel tür parametrelerinin somut
türlerini değiştirerek. `Counter` üzerinde `next` yöntemini kullandığımızda, hangi
`Iterator` uygulamasını kullanmak istediğimizi belirtmek için tür açıklamaları sağlamalıyız.

İlişkili türlerle, türleri açıklamamıza gerek yok çünkü bir tür için bir trait'i birden fazla
kez uygulayamayız. Kod Listesi 20-13'teki ilişkili türleri kullanan tanım ile, `impl
Iterator for Counter` sadece bir tane olabileceği için `Item`'ın hangi tür olacağını sadece
bir kez seçebiliriz. `Counter` üzerinde `next` çağırdığımız her yerde `u32` değerlerinin bir
iterator'ı istediğimizi belirtmemize gerek yok.

İlişkili türler ayrıca trait'in sözleşmesinin bir parçası haline gelir: Trait'in
uyulayıcıları, ilişkili tür yer tutucusu için bir tür sağlamalı. İlişkili türler
genellikle türün nasıl kullanılacağını açıklayan bir isme sahiptir ve ilişkili türü API
belgelerinde belgelemek iyi bir uygulamadır.

<!-- Old headings. Do not remove or links may break. -->

<a id="default-generic-type-parameters-and-operator-overloading"></a>

### Varsayılan Genel Tür Parametreleri ve Operatör Aşırı Yükleme Kullanma (Using Default Generic Parameters and Operator Overloading)

Genel tür parametreleri kullandığımızda, genel tür için varsayılan bir somut tür
belirleyebiliriz. Bu, trait'in uygulayıcılarının varsayılan tür işliyorsa somut bir
tür belirtme gereksinimini ortadan kaldırır. Varsayılan türü `<PlaceholderType=ConcreteType>`
sözdizimi ile genel bir tür bildirirken belirtirsiniz.

Bu tekniğin yararlı olduğu durumlardan büyük bir örneği _operatör aşırı yükleme_
(operator overloading)dir ki bu, belirli durumlarda bir operatörün (örneğin `+`) davranışını
özelleştirirsiniz.

Rust kendi operatörlerinizi oluşturmanıza veya keyfi operatörleri aşırı yüklemenize izin
vermez. Ancak `std::ops` içinde listelenen işlemleri ve ilgili trait'leri operatörle
ilişkili trait'leri uygulayarak aşırı yükleyebilirsiniz. Örneğin, Kod Listesi 20-15'te,
iki `Point` örneğini eklemek için `+` operatörünü aşırı yüklüyoruz. Bunu bir
`Point` struct'ı üzerinde `Add` trait'ini uygulayarak yapıyoruz.

<Listing number="20-15" file-name="src/main.rs" caption="Implementing the `Add` trait to overload the `+` operator for `Point` instances">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-15/src/main.rs}}
```

</Listing>

`add` yöntemi iki `Point` örneğinin `x` değerlerini ve iki `Point` örneğinin `y`
değerlerini ekler ve yeni bir `Point` oluşturur. `Add` trait'in `add` yönteminden
dönen türü belirleyen `Output` adında bir ilişkili türü vardır.

Bu koddaki varsayılan genel tür `Add` trait'inin içindedir. İşte tanımı:

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

Bu kod genel olarak tanıdık görünmeli: bir yöntemi ve bir ilişkili tür olan bir trait.
Yeni kısım `Rhs=Self`: Bu sözdizimi _varsayılan tür parametreleri_ olarak adlandırılır.
`Rhs` genel tür parametresi ("right-hand side"nın kısaltması), `add` yöntemindeki
`rhs` parametresinin türünü tanımlar. `Add` trait'ini uyguladığımızda `Rhs` için bir
somut tür belirtmezsek, `Rhs`'nin türü varsayılan olarak `Self`'e dönecektir ki bu
`Add`'ı uyguladığımız tür olacaktır.

`Point` için `Add` uyguladığımızda, iki `Point` örneği eklemek istediğimiz için
`Rhs` için varsayılanı kullandık. `Rhs` türünü varsayılan kullanmak yerine
özelleştirmek istediğimiz `Add` trait'ini uygulayan bir örneğe bakalım.

Farklı birimlerde değer tutan iki struct'ımız var: `Millimeters` ve `Meters`. Mevcut bir
türü başka bir struct'ta ince sarmalama (wrapping), [“Newtype Deseniyle Dış Trait'leri
Uygulama”][newtype]<!--
ignore --> bölümünde daha detaylı olarak açıkladığımız bir _newtype deseni_
olarak bilinir. Milimetre değerlerini metre değerlerine eklemek istiyoruz ve `Add` uygulamasının
dönüşümü doğru yapmasını istiyoruz. `Millimeters` için `Meters`'i `Rhs` olarak
kullanarak `Add` uygulayabiliriz, Kod Listesi 20-16'da gösterildiği gibi.

<Listing number="20-16" file-name="src/lib.rs" caption="Implementing the `Add` trait on `Millimeters` to add `Millimeters` and `Meters`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-16/src/lib.rs}}
```

</Listing>

`Millimeters` ve `Meters`'i eklemek için, varsayılan olan `Self` yerine `Rhs` tür
parametresinin değerini ayarlamak için `impl Add<Meters>` belirtiyoruz.

Varsayılan tür parametrelerini iki ana şekilde kullanacaksınız:

1. Mevcut kodu bozmadan bir türü genişletmek için
2. Çoğu kullanıcının ihtiyaç duymayacağı belirli durumlarda özelleştirmeye izin vermek için

Standart kütüphanenin `Add` trait'i ikinci amacın bir örneğidir: Genellikle iki
benzer tür eklersiniz ancak `Add` trait'i ötesinde özelleştirme yeteneği sağlar. `Add`
trait tanımında bir varsayılan tür parametresi kullanmak, çoğu zaman ekstra parametre belirtmenize
gerekmediği anlamına gelir. Başka bir deyişle, biraz uygulama boilerplate'i gerekmez ki bu
trait'i kullanmayı kolaylaştırır.

İlk amaç ikinciye benzer ancak ters yönde: Mevcut bir trait'e bir tür parametresi eklemek
isterseniz, ona bir varsayılan verebilirsiniz böylece mevcut uygulama kodunu bozmadan
trait'in işlevselliğini genişletmesine izin verir.

<!-- Old headings. Do not remove or links may break. -->

<a id="fully-qualified-syntax-for-disambiguation-calling-methods-with-the-same-name"></a>
<a id="disambiguating-between-methods-with-the-same-name"></a>

### Aynı İsimli Yöntemler Arasında Ayrım Yapma (Disambiguating Between Identically Named Methods)

Rust'ta bir trait'in başka bir trait'in yöntemiyle aynı isme sahip bir yöntemi olmasını
engelleyen bir şey yoktur, ayrıca Rust tek bir türde her iki trait'i uygulamanızı da engellemez.
Ayrıca, trait'lerden yöntemlerle aynı isme sahip bir yöntemi doğrudan bir tür üzerinde
uyulamak da mümkündür.

Aynı isimli yöntemleri çağırırken, hangisini kullanmak istediğinizi Rust'a söylemeniz
gerekecek. Kod Listesi 20-17'deki kodu düşünün ki burada iki trait tanımladık: `Pilot`
ve `Wizard`, her ikisi de `fly` adında bir yönteme sahip. Sonra `fly` adında bir yöntemi
doğrudan üzerinde uygulanan `Human` adında bir türde her iki trait'i uyguluyoruz. Her
`fly` yöntemi farklı bir şey yapıyor.

<Listing number="20-17" file-name="src/main.rs" caption="Two traits are defined to have a `fly` method and are implemented on `Human` type, and a `fly` method is implemented on `Human` directly.">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-17/src/main.rs:here}}
```

</Listing>

Bir `Human` örneğinde `fly` çağırdığımızda, derleyici doğrudan tür üzerinde uygulanan
yöntemi çağırmaya varsayılan (default) olarak çalışır, Kod Listesi 20-18'de gösterildiği gibi.

<Listing number="20-18" file-name="src/main.rs" caption="Calling `fly` on an instance of `Human`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-18/src/main.rs:here}}
```

</Listing>

Bu kodu çalıştırmak `*waving arms furiously*` yazdıracak ki bu Rust'ın doğrudan `Human` üzerinde
uyulanan `fly` yöntemini çağırdığını gösterir.

Ya da `Wizard` trait'inden `fly` yöntemlerini çağırmak için, hangi `fly` yöntemini
kastettiğimizi belirtmek için daha açık sözdizimini kullanmamız gerekir. Kod Listesi 20-19 bu sözdizimini
gösterir.

<Listing number="20-19" file-name="src/main.rs" caption="Specifying which trait's `fly` method we want to call">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-19/src/main.rs:here}}
```

</Listing>

Yöntem adından önce trait adını belirtmek, hangi `fly` uygulamasını çağırmak istediğimizi
Rust'a netleştirir. Ayrıca `Human::fly(&person)` yazabiliriz ki bu, Kod Listesi 20-19'ta
kullandığımız `person.fly()` ile eşdeğerdir ancak ayırmamıza gerekmiyorsa bu biraz daha
uzun yazmak olur.

Bu kodu çalıştırmak şunları yazdırır:

```console
{{#include ../listings/ch20-advanced-features/listing-20-19/output.txt}}
```

Çünkü `fly` yöntemi bir `self` parametresi alır, eğer ikisi de bir _trait_'i uygulayan
iki _türümüz_ olsaydı, Rust, hangi trait uygulamasını kullanacağını `self`'in türüne
dayalı anlayabilirdi.

Ancak, yöntem olmayan ilişkili fonksiyonlar `self` parametresine sahip değildir. Aynı
fonksiyon ismine sahip yöntem olmayan fonksiyonları tanımlayan birden çok tür veya trait olduğunda,
Rust tam olarak nitelikli sözdizimi (fully qualified syntax) kullanmadığınız sürece her zaman hangi
türü kastettiğinizi bilemez. Örneğin, Kod Listesi 20-20'de, tüm bebek köpeklere Spot
adını vermek isteyen bir hayvan sığınağı (animal shelter) için bir trait oluşturuyoruz. `Animal`
adında bir trait yapıyoruz ki bu, `baby_name` adında ilişkili yöntem olmayan bir fonksiyona sahip.
`Dog` struct'ı `Animal` trait'ini uygular ki biz ayrıca `Dog` üzerinde doğrudan ilişkili
yöntem olmayan bir `baby_name` fonksiyonu sağlarız.

<Listing number="20-20" file-name="src/main.rs" caption="A trait with an associated function and a type with an associated function of the same name that also implements the trait">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-20/src/main.rs}}
```

</Listing>

Tüm köpek yavrularına Spot adını veren kodu, `Dog` üzerinde tanımlanan ilişkili fonksiyon
olan `baby_name` içinde uygularız. `Dog` türü ayrıca tüm hayvanların sahip olduğu özellikleri
açıklayan `Animal` trait'ini uygular. Bebek köpeklere puppy denir ve bu, `Dog` üzerinde
`Animal` trait'inin uygulamasında `Animal` trait'iyle ilişkili `baby_name` fonksiyonunda ifade edilir.

`main` içinde, `Dog::baby_name` fonksiyonunu çağırıyoruz ki bu, `Dog` üzerinde doğrudan
tanımlanan ilişkili fonksiyonu çağırır. Bu kod şunları yazdırır:

```console
{{#include ../listings/ch20-advanced-features/listing-20-20/output.txt}}
```

Bu çıktı istediğimiz değil. `Dog` üzerinde uyguladığımız `Animal` trait'inin parçası olan
`baby_name` fonksiyonunu çağırmak istiyoruz ki böylece kod `A baby dog is called a puppy` yazdırır.
Kod Listesi 20-19'te kullandığımız trait adını belirtme tekniği burada yardımcı olmaz;
`main`'i Kod Listesi 20-21'deki koda değiştirirsek, bir derleme hatası alacağız.

<Listing number="20-21" file-name="src/main.rs" caption="Attempting to call the `baby_name` function from the `Animal` trait, but Rust doesn't know which implementation to use">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-21/src/main.rs:here}}
```

</Listing>

Çünkü `Animal::baby_name` bir `self` parametresine sahip değil ve `Animal` trait'ini uygulayan
diğer türler olabilir, Rust kullanmak istediğimiz `Animal::baby_name` uygulamasının hangisi olduğunu
anlayamaz. Bu derleyici hatasını alacağız:

```console
{{#include ../listings/ch20-advanced-features/listing-20-21/output.txt}}
```

Ayrım yapmak ve Rust'a başka bir tür için `Animal` uygulaması yerine `Dog` için `Animal` uygulamasını
kullanmak istediğimizi söylemek için, tam olarak nitelikli sözdizimini (fully qualified syntax) kullanmamız
gerekiyor. Kod Listesi 20-22 tam olarak nitelikli sözdizimini nasıl kullanacağımızı gösterir.

<Listing number="20-22" file-name="src/main.rs" caption="Using fully qualified syntax to specify that we want to call the `baby_name` function from the `Animal` trait as implemented on `Dog`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-22/src/main.rs:here}}
```

</Listing>

Rust'a köşeli parantez içinde bir tür açıklaması sağlıyoruz ki bu, `Dog` üzerinde uygulanan
`Animal` trait'inden `baby_name` yöntemini çağırmak istediğimizi gösterir, bunu bu fonksiyon
çağrısı için `Dog` türünü bir `Animal` olarak ele almak istediğimizi söyleyerek yapıyoruz. Bu kod
artık istediğimizi yazdırır:

```console
{{#include ../listings/ch20-advanced-features/listing-20-22/output.txt}}
```

Genel olarak, tam olarak nitelikli sözdizimi (fully qualified syntax) şöyle tanımlanır:

```rust,ignore
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

Yöntem olmayan ilişkili fonksiyonlar için bir `receiver` olmaz: Diğer argümanların listesi
olur. Fonksiyon veya yöntem çağırdığınız her yerde tam olarak nitelikli sözdizimini
kullanabilirsiniz. Ancak, Rust programdaki diğer bilgilerden anlayabileceği bu sözdiziminin herhangi
bir bölümünü atlamasına izin verilir. Aynı ismi kullanan birden çok uygulama var ve Rust hangisini
çağırmak istediğinizi anlamaya yardıma ihtiyaç duyduğu durumlarda sadece bu daha ayrıntılı sözdizimini
kullanmanız gerekir.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-supertraits-to-require-one-traits-functionality-within-another-trait"></a>

### Supertrait'leri Kullanma (Using Supertraits)

Bazen, başka bir trait'e bağlı olan bir trait tanımı yazabilirsiniz: Bir türün ilk trait'i uygulaması
için, türün ikinci trait'i de uygulamasını istersiniz. Bunu yaparsınız ki trait tanımınız ikinci
trait'in ilişkili maddelerini kullanabilir. Trait tanımınızın güvendiği trait, trait'inizin
_supertrait'i_ olarak adlandırılır.

Örneğin, `OutlinePrint` adında bir trait yapmak istediğimizi söyleyin ki bu, verilen bir değer
yıldızlarla çerçevelenmiş şekilde basacak bir `outline_print` yöntemine sahip. Yani, standart
kütüphane trait'ini `Display` uygulayan ve `(x, y)` sonucunu veren bir `Point` struct'ı
verilirsek, `x` için `1` ve `y` için `3` değerine sahip bir `Point` örneğinde
`outline_print` çağırırsak, şunları yazdırmalı:

```text
**********
*        *
* (1, 3) *
*        *
**********
```

`outline_print` yönteminin uygulamasında, `Display` trait'in işlevselliğini kullanmak istiyoruz.
Bu yüzden, `OutlinePrint` trait'inin, `Display`'ı da uygulayan ve `OutlinePrint`'in ihtiyaç duyduğu
işlevselliği sağlayan türler için çalışacağını belirtmemiz gerekir. Bunu trait tanımında
`OutlinePrint: Display` belirterek yapabiliriz. Bu teknik, trait'e bir trait sınırı (trait bound)
eklemeye benzer. Kod Listesi 20-23, `OutlinePrint` trait'inin bir uygulamasını gösterir.

<Listing number="20-23" file-name="src/main.rs" caption="Implementing the `OutlinePrint` trait that requires functionality from `Display`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-23/src/main.rs:here}}
```

</Listing>

`OutlinePrint`'ın `Display` trait'ini gerektirdiğini belirttiğimiz için, `Display` uygulayan herhangi
tür için otomatik olarak uygulanmış olan `to_string` fonksiyonunu kullanabiliriz. `to_string`'ı
kullanmaya çalışırsak ve trait adından sonra iki nokta ekleyip `Display` trait'ini belirtmezsek, `&Self`
türü için `to_string` adında bir yöntemin geçerli kapsamda bulunmadığını söyleyen bir hata alırız.

`Display` uygulamayan bir türde, örneğin `Point` struct'ında `OutlinePrint` uygulamaya çalıştığımızda
ne olacağını görelim:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/src/main.rs:here}}
```

</Listing>

Gerektiğine rağmen uygulanmadığını söyleyen bir hata alırız:

```console
{{#include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/output.txt}}
```

Bunu düzeltmek için, `Point` üzerinde `Display` uygularız ve `OutlinePrint`'ın gerektirdiği
kısıtı karşılarız, şöyle:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-03-impl-display-for-point/src/main.rs:here}}
```

</Listing>

Sonra, `Point` üzerinde `OutlinePrint` trait'ini uygulamak başarıyla derlenecek ve bir `Point` örneğinde
yıldızların çerçevesi içinde göstermesi için `outline_print` çağırabiliriz.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-newtype-pattern-to-implement-external-traits-on-external-types"></a>
<a id="using-the-newtype-pattern-to-implement-external-traits"></a>

### Newtype Deseniyle Dış Trait'leri Uygulama (Implementing External Traits with the Newtype Pattern)

Bölüm 10'daki [“Bir Türde Trait Uygulama”][implementing-a-trait-on-a-type]<!--
ignore --> bölümünde, trait'i bir tür üzerinde sadece şu durumda uygulayabileceğimizi söyleyen yetim kuralından
(orphan rule) bahsettik: trait ya da tür, ya da her ikisi de crate'imize yerel ise.
Newtype desenini kullanarak bu kısıtlamayı aşmak mümkündür ki bu, bir tuple struct'ta yeni bir
tür oluşturmayı içerir. (Tuple struct'ları Bölüm 5'teki [“Tuple Struct'larla Farklı Türler
Oluşturma”][tuple-structs]<!--
ignore --> bölümünde ele aldık.) Tuple struct'ta bir alanı olacak ve
trait'ini uygulamak istediğimiz tür için ince bir sarmalayıcı (wrapper) olacak. Sonra,
sarmalayıcı tür crate'imize yerel olacak ve sarmalayıcı üzerinde trait uygulayabiliriz. _Newtype_,
Haskell programlama dilinden gelen bir terimdir. Bu deseni kullanmanın çalışma zamanı (runtime) performans cezası
yoktur ve sarmalayıcı tür derleme zamanında elide edilir.

Bir örnek olarak, `Vec<T>` üzerinde `Display` uygulamak istediğimizi söyleyin ki yetim kuralı bizi
doğrudan yapmaktan engeller çünkü `Display` trait'i ve `Vec<T>` türü crate'ımızın dışında tanımlanmıştır.
Bir `Vec<T>` örneği tutan bir `Wrapper` struct yapabiliriz; sonra, `Wrapper` üzerinde `Display`
uygulayabilir ve `Vec<T>` değerini kullanabiliriz, Kod Listesi 20-24'te gösterildiği gibi.

<Listing number="20-24" file-name="src/main.rs" caption="Creating a `Wrapper` type around `Vec<String>` to implement `Display`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-24/src/main.rs}}
```

</Listing>

`Display` uygulaması, `Wrapper` bir tuple struct olduğu ve `Vec<T>` tuple'da 0 indeksteki
öğe olduğu için iç `Vec<T>`'ye erişmek için `self.0` kullanır. Sonra, `Wrapper` üzerinde
`Display` trait'inin işlevselliğini kullanabiliriz.

Bu tekniği kullanmanın dezavantajı şudur ki `Wrapper` yeni bir türdür bu yüzden tuttuğu değerin
yöntemlerine sahip değildir. `Vec<T>`'nin tüm yöntemlerini `Wrapper` üzerinde doğrudan uygulamalıyız
böylece yöntemler `self.0`'a delege eder ki bu bize `Wrapper`'ı tam olarak `Vec<T>` gibi
ele almamızı sağlar. Eğer yeni türün iç türün sahip olduğu her yönteme sahip olmasını
istiyorsak, iç türü döndüren `Wrapper` üzerinde `Deref` trait'ini uygulamak bir çözüm olacaktır
(Bölüm 15'teki [“Akıllı İşaretçileri Normal Referanslar Gibi Ele Alma”][smart-pointer-deref]<!--
ignore --> bölümünde `Deref` trait'ini uygulamayı tartıştık). Eğer `Wrapper` türünün iç türün tüm yöntemlerine
sahip olmasını istemiyorsanız—örneğin, `Wrapper` türünün davranışını kısıtlamak için—istediğimiz
yöntemleri manuel olarak uygulamalıyız.

Bu newtype deseni, trait'ler dahil olmadığında bile yararlıdır. Odağı değiştirelim ve Rust'un
tür sistemiyle etkileşim için bazı gelişmiş yollara bakalım.

[newtype]: ch20-02-advanced-traits.html#implementing-external-traits-with-the-newtype-pattern
[implementing-a-trait-on-a-type]: ch10-02-traits.html#implementing-a-trait-on-a-type
[traits]: ch10-02-traits.html
[smart-pointer-deref]: ch15-02-deref.html#treating-smart-pointers-like-regular-references
[tuple-structs]: ch05-01-defining-structs.html#creating-different-types-with-tuple-structs