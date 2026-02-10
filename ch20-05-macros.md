## Makrolar (Macros)

Bu kitap boyunca `println!` gibi makrolar kullandık ancak bir makronun ne olduğunu ve nasıl
çalıştığını tam olarak incelemedik. _Makro_ terimi, Rust'taki bir özellik ailesini ifade eder
— `macro_rules!` ile bildirimsel (declarative) makrolar ve üç tür yordamsal (procedural) makro:

- `derive` özelliği ile struct'lara ve enum'lara eklenen kodu belirleyen özel `#[derive]` makroları
- Herhangi bir öğede kullanılabilen özel özellikler (attributes) tanımlayan özellik-benzeri (attribute-like) makrolar
- Argüman olarak belirtilen tokenler üzerinde çalışan fonksiyon çağırma gibi görünen fonksiyon-benzeri (function-like) makrolar

Bunların her birini sırayla konuşacağız ancak önce, fonksiyonlarımız olduğunda neden makrolara
ihtiyacımız olduğuna bakalım.

### Makrolar ve Fonksiyonlar Arasındaki Fark (The Difference Between Macros and Functions)

Temelde, makrolar diğer kodları yazan kod yazmanın bir yoludur ki bu _metaprogramlama_ olarak bilinir.
Ek C'de `derive` özelliğini tartıştık ki bu, sizin için çeşitli trait'lerin uygulamasını
üretir. Ayrıca `println!` ve `vec!` makrolarını kitap boyunca kullandık.
Bu makroların tümü sizin manuel olarak yazdığınız koddan daha fazla kod üretmek için _genişler_ (expand).

Metaprogramlama, yazmanız ve bakımını yapmanız gereken kod miktarını azaltmak için yararlıdır,
bu ayrıca fonksiyonların rollerinden biridir. Ancak, makroların fonksiyonların sahip olmadığı bazı
ek güçleri vardır.

Bir fonksiyon imzası (signature), fonksiyonun sahip olduğu parametre sayısını ve türünü
bildirmelidir. Makrolar ise diğer taraftan değişken sayıda parametre alabilir: Tek
argümanla `println!("hello")` veya iki argümanla `println!("hello {}", name)` çağırabiliriz.
Ayrıca, makrolar derleyicinin (compiler) kodun anlamını yorumlamasından önce genişler, bu yüzden
bir makro, örneğin, belirli bir türde bir trait uygulayabilir. Bir fonksiyon bunu yapamaz çünkü
çalışma zamanında (runtime) çağırılır ve bir trait derleme zamanında uygulanmalıdır.

Fonksiyon yerine bir makro uygulamanın dezavantajı, makro tanımlarının fonksiyon tanımlarından
daha karmaşık olmasıdır çünkü Rust kodunu yazan Rust kodunu yazıyorsunuz. Bu dolaylılık (indirection)
yüzünden, makro tanımları genel olarak fonksiyon tanımlarından okumak, anlamak ve bakım yapmak
daha zordur.

Makrolar ve fonksiyonlar arasındaki diğer önemli fark şudur ki makroları tanımlamalı veya onları
kapsama (scope) sokmalıyız, onları bir dosyada çağırmadan _önce_; buna karşın fonksiyonları
her yerde tanımlayabilir ve her yerde çağırabilirsiniz.

<!-- Old headings. Do not remove or links may break. -->

<a id="declarative-macros-with-macro_rules-for-general-metaprogramming"></a>

### Genel Metaprogramlama İçin Bildirimsel Makrolar (Declarative Macros for General Metaprogramming)

Rust'taki en yaygın kullanılan makro formu _bildirimsel makro_ (declarative macro) dur. Bunlar
bazen "örnek ile makrolar" (macros by example), "`macro_rules!` makroları" veya sadece düz "makrolar"
olarak da adlandırılır. Temellerinde, bildirimsel makrolar, sizin bir Rust `match` ifadesine benzer bir şey yazmanıza
izin verir. Bölüm 6'da tartıştığı gibi, `match` ifadeleri, bir ifade alır, ifadenin
sonuç değerini desenlerle (patterns) karşılaştırır ve sonra eşleşen desenle ilişkili kodu çalıştırır.
Makrolar da belirli bir kodla ilişkili olan desenlere bir değer karşılaştırır: Bu durumda,
değer makroya geçilen gerçek (literal) Rust kaynak kodudur; desenler, o kaynak kodunun yapısıyla
karşılaştırılır; ve eşleştiğinde, her desenle ilişkili kod, makroya geçilen kodu
yerine koyar. Bunların tümü derleme sırasında olur.

Bir makro tanımlamak için, `macro_rules!` yapısını kullanırsınız. `vec!` makrosunun nasıl tanımlandığına
bakarak `macro_rules!` nasıl kullanılacağını inceleyelim. Bölüm 8, `vec!` makrosunu kullanarak belirli
değerlerle yeni bir vektör nasıl oluşturabileceğimizi kapsadı. Örneğin, aşağıdaki makro üç tamsayı içeren
yeni bir vektör oluşturur:

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

Ayrıca `vec!` makrosunu iki tamsayının vektörünü veya beş string diliminin vektörünü
yapmak için de kullanabilirdik. Önceden değerlerin sayısını veya türünü bilemeyeceğimiz için bir fonksiyonla
aynısını yapamazdık.

Kod Listesi 20-35, `vec!` makrosunun biraz basitleştirilmiş tanımını gösterir.

<Listing number="20-35" file-name="src/lib.rs" caption="A simplified version of `vec!` macro definition">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-35/src/lib.rs}}
```

</Listing>

> Not: Standart kütüphanedeki `vec!` makrosunun gerçek tanımı, önden doğru miktarda
> bellek ayırmak (pre-allocate) için kod içerir. Bu kod örneği basit yapmak için buraya
> dahil etmediğimiz bir optimizasyon.

`#[macro_export]` açıklaması (annotation), bu makronun, makronun tanımlandığı crate, kapsama
sokulduğunda her zaman kullanılabilir yapılması gerektiğini belirtir. Bu açıklama olmadan, makro kapsama
sokulamaz.

Sonra, `macro_rules!` ile makro tanımını başlatırız ve tanımlamakta olduğumuz makronun adını
ünlem işareti _olmadan_. Bu durumda ad `vec`, makro tanımının gövdesini gösteren süslü parantezlerle
takip eder.

`vec!` gövdesindeki yapı, bir `match` ifadesinin yapısına benzer. Burada, desen `( $( $x:expr ),* )`
ile bir kol ve ardından `=>` ve bu desenle ilişkili kod bloğu var. Eğer desen eşleşirse,
ilişkili kod bloğu yayılır (emitted). Bu makroda bu sadece desen olduğu için, eşleşmenin sadece bir
geçerli yolu vardır; diğer desenler bir hataya neden olur. Daha karmaşık makrolar birden fazla
kola sahip olacaktır.

Makro tanımlarındaki geçerli desen sözdizimi, Bölüm 19'da kapsanan desen sözdiziminden farklıdır çünkü
makro desenleri Rust kod yapısına karşı değerlere karşı karşılaştırılır. Kod Listesi 20-29'daki desen parçalarının
ne anlama geldiğine bakalım; tam makro desen sözdizimi için [Rust Referansı][ref]'e bakın.

Önce, tüm deseni saran bir parantez seti kullanırız. Bir dolar işareti (`$`) kullanarak
desene eşleşen Rust kodunu içerecek makro sisteminde bir değişken bildiririz. Dolar işareti,
bu, normal bir Rust değişkeni yerine bir makro değişkeni olduğunu netleştirir. Sonra, parantez seti
gelir ki bu, yedekleme kodunda kullanmak için parantez içinde eşleşen değerleri yakalar. `$()`
içinde `$x:expr` vardır ki bu, herhangi bir Rust ifadesiyle eşleşir ve ifadenin adını `$x` olarak
verir.

`$()`'yi izleyen virgül, `$()`'deki kodla eşleşen kodun her örneği arasında gerçek bir
virgül ayıracı karakterinin bulunması gerektiğini belirtir. `*`, deseni önüne ne gelirse
sıfır veya daha fazla kez eşleştiğini belirtir.

Bu makroyu `vec![1, 2, 3]; ile çağırdığımızda, `$x` deseni üç kez, `1`, `2`
ve `3` ifadeleriyle eşleşir.

Şimdi bu kolla ilişkili kod gövdesindeki desene bakalım: `$()*` içindeki `temp_vec.push()`,
desenin kaç kez eşleştiğine bağlı olarak sıfır veya daha fazla kez, desenle eşleşen her parça için
üretilir. `$x`, eşleşen her ifadeyle değiştirilir. Bu makroyu `vec![1, 2, 3]; ile çağırdığımızda,
bu makro çağrısını yerine koyan kod şöyle olacaktır:

```rust,ignore
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

Herhangi bir türde herhangi bir sayıda argüman alan ve belirtilen öğeleri içeren bir vektör oluşturmak
için kod üreten bir makro tanımladık.

Makro yazmak hakkında daha fazla bilgi edinmek için, çevrimiçi belgelere veya [Daniel Keep tarafından
başlatılan ve Lukas Wirth tarafından sürdürülen "The Little Book of Rust
Macros"][tlborm] gibi diğer kaynaklara danışın.

### Özelliklerden Kod Üretmek İçin Yordamsal Makrolar (Procedural Macros for Generating Code from Attributes)

Makroların ikinci formu yordamsal makrodur ki bu, bir fonksiyona (ve bir yordam/prosedür türüne)
daha çok davranır. _Yordamsal makrolar_ (procedural macros), bazı kodu girdi olarak alır, o kod
üzerinde çalışır ve çıktı olarak bazı kod üretir, desenlere karşılaştırarak ve kodu diğer kodla yerine koymak yerine
bildirimsel makroların yaptığı gibi. Yordamsal makroların üç türü: özel `derive`, özellik-benzeri
(attribute-like) ve fonksiyon-benzeri (function-like), ve tümü benzer şekilde çalışır.

Yordamsal makrolar oluştururken, tanımlar özel bir crate türüyle kendi crate'lerinde bulunmalıdır.
Bu, gelecekte ortadan kaldırmayı umduğumuz karmaşık teknik nedenlerle ilgilidir. Kod Listesi
20-36'de, bir yordamsal makro nasıl tanımlanacağını gösteriyoruz ki burada `some_attribute`, belirli bir
makro çeşidini kullanmak için bir yer tutucusu (placeholder).

<Listing number="20-36" file-name="src/lib.rs" caption="An example of defining a procedural macro">

```rust,ignore
use proc_macro::TokenStream;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

</Listing>

Yordamsal makro tanımlayan fonksiyon, bir `TokenStream`'ı girdi olarak alır ve bir `TokenStream`
çıktısı olarak üretir. `TokenStream` türü, Rust ile dahil olan `proc_macro` crate'ince tanımlanır ve
token dizisini temsil eder. Bu, makronun çekirde: Makronun üzerinde çalıştığı kaynak kodu girdi
`TokenStream`'i oluşturur ve makronun ürettiği kod çıktı `TokenStream`'dir. Fonksiyon ayrıca kendisine
bağlı bir özelliğe (attribute) sahiptir ki bu, hangi türde yordamsal makro oluşturduğumuzu belirtir.
Aynı crate'de birden fazla türde yordamsal makro sahibi olabiliriz.

Farklı türde yordamsal makrolara bakalım. Özel `derive` makrosuyla başlayacağız ve sonra diğer
formları farklı yapan küçük farklılıkları açıklayacağız.

<!-- Old headings. Do not remove or links may break. -->

<a id="how-to-write-a-custom-derive-macro"></a>

### Özel `derive` Makroları (Custom `derive` Macros)

`HelloMacro` adında bir trait ve `hello_macro` adında bir ilişkili fonksiyon tanımlayan `hello_macro`
adında bir crate oluşturalım. Kullanıcıların türlerinin her biri için `HelloMacro` trait'ini uygulamalarını
yapmalarını beklemek yerine, kullanıcıların türlerine `#[derive(HelloMacro)]` ile not alarak
`hello_macro` fonksiyonunun varsayılan uygulamasını elde etmeleri için bir yordamsal makro sağlayacağız. Varsayılan
uyulama, `Hello, Macro! My name is TypeName!` yazacak ki burada `TypeName`, trait'in tanımlandığı türün
adıdır. Başka bir deyişle, kullanıcılarımızın Kod Listesi 20-37'deki gibi bir kodumuzu crate'ımızı
kullanarak yazmasına izin veren bir crate yazacağız.

<Listing number="20-37" file-name="src/main.rs" caption="The code a user of our crate will be able to write when using our procedural macro">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-37/src/main.rs}}
```

</Listing>

Bu kod, işimiz bittiğinde `Hello, Macro! My name is Pancakes!` yazacak. İlk adım, şöyleki yeni bir
kütüphane crate'i yapmak:

```console
$ cargo new hello_macro --lib
```

Sonra, Kod Listesi 20-38'de, `HelloMacro` trait'ini ve onun ilişkili fonksiyonunu tanımlayacağız.

<Listing file-name="src/lib.rs" number="20-38" caption="A simple trait that we will use with `derive` macro">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-38/hello_macro/src/lib.rs}}
```

</Listing>

Bir trait'imiz ve onun fonksiyonumuz var. Bu noktada, crate kullanıcımızı aranan işlevselliği elde etmek için
trait'i uygulayabilir, Kod Listesi 20-39'deki gibi.

<Listing number="20-39" file-name="src/main.rs" caption="How it would look if users wrote a manual implementation of `HelloMacro` trait">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-39/pancakes/src/main.rs}}
```

</Listing>

Ancak, `hello_macro` ile kullanmak istedikleri her tür için uygulama bloğu yazmaları gerekirdi; biz,
bu işi yapmalarından onları kurtarmak istiyoruz.

Ayrıca, `hello_macro` fonksiyonunu trait'in uygulandığı türün adını yazacak varsayılan uygulama olarak henüz
sağlayamıyoruz: Rust, yansıma yeteneklerine sahip değil bu yüzden çalışma zamanında (runtime) türün adına bakamaz.
Kod üretmek için derleme zamanında (compile time) bir makroya ihtiyacımız var.

Sıradaki adım, yordamsal makro tanımlamaktır. Bu yazı zamanında, yordamsal makrolar kendi crate'lerinde
olmalıdır. Sonunda, bu kısıtlama kaldırılabilir. Crate'leri ve makro crate'lerini yapılandırma geleneği şöyledir:
`foo` adında bir crate için, özel `derive` yordamsal makro crate'i `foo_derive` adlandırılır.
`hello_macro` projemizin içinde `hello_macro_derive` adında yeni bir crate başlatalım:

```console
$ cargo new hello_macro_derive --lib
```

İki crate'ımız sıkı sıkıya ilişkili bu yüzden yordamsal makro crate'ini `hello_macro` crate'imizin
dizininde oluşturuyoruz. `hello_macro` içindeki trait tanımını değiştirirsek, `hello_macro_derive` içindeki
yordamsal makro uygulamasını da değiştirmemiz gerekecek. İki crate ayrı ayrı yayınlanmalı ve bu crate'leri
kullanan programcıların ikisini de bağımlılık (dependency) olarak eklemeleri ve her ikisini de kapsama
sokmaları gerekecek. Bunun yerine, `hello_macro` crate'inin `hello_macro_derive`'i bağımlılık olarak kullanmasını
ve yordamsal makro kodunu tekrar dışa aktarmasını (re-export) sağlayabilirdik. Ancak, projemizi
yapılandırdığımız yol, programcıların `derive` işlevselliğini istemeler bile `hello_macro` kullanmasına
izin verir.

`hello_macro_derive` crate'ini bir yordamsal makro crate'i olarak bildirmemiz gerekiyor. Birazdan göreceğiniz gibi
`syn` ve `quote` crate'lerinden işlevsellik de ihtiyacımız var, bu yüzden onları bağımlılık olarak eklememiz
gerekiyor. `hello_macro_derive` için _Cargo.toml_ dosyasına şunları ekleyin:

<Listing file-name="hello_macro_derive/Cargo.toml">

```toml
{{#include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/Cargo.toml:6:12}}
```

</Listing>

Yordamsal makroyu tanımlamaya başlamak için, Kod Listesi 20-40'teki kodu `hello_macro_derive` crate'iniz için
_src/lib.rs_ dosyasına yerleştirin. Bu kod, `impl_hello_macro` fonksiyonu için bir tanım eklemeden derlenmez.

<Listing number="20-40" file-name="hello_macro_derive/src/lib.rs" caption="Code that most procedural macro crates will require in order to process Rust code">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/src/lib.rs}}
```

</Listing>

Kodu `hello_macro_derive` fonksiyonuna ve `impl_hello_macro` fonksiyonuna böldüğümüze dikkat edin ki
ilki `TokenStream`'i ayrıştırmadan (parsing) sorumlu, ikincisi sözdizimi ağacını (syntax tree) dönüştürmekten
sorumlu: Bu, bir yordamsal makro yazmayı daha uygun hale getirir. Dış fonksiyondaki kod (bu durumda
`hello_macro_derive`), göreceğiniz veya oluşturacağınız hemen hemen her yordamsal makro crate'inde aynı olacaktır.
İç fonksiyonun gövdesinde (bu durumda `impl_hello_macro`) belirleyeceğiniz kod, yordamsal makronuzun
amacına bağlı olarak farklı olacaktır.

Üç yeni crate tanıttık: `proc_macro`, [`syn`][syn]<!--
ignore --> ve [`quote`][quote]<!--
ignore -->. `proc_macro` crate'i Rust ile gelir bu yüzden _Cargo.toml_'da bağımlılık olarak eklememize gerekmedi.
`proc_macro` crate'i, bizim kodumuzdan Rust kodunu okumamıza ve manipüle etmemize izin veren derleyicinin API'sidir.

`syn` crate'i, Rust kodunu bir stringden, üzerinde işlemler gerçekleştirebileceğimiz bir veri yapısına
ayrıştırır (parses). `quote` crate'i, `syn` veri yapılarını tekrar Rust koduna döndürür. Bu crate'ler,
handle etmek isteyebileceğiniz herhangi bir Rust kodunu ayrıştırmayı çok daha basit hale getirir: Rust kodu için tam bir
ayrıştırıcı (parser) yazmak basit bir görev değildir.

Kütüphanemizin bir kullanıcısı bir türde `#[derive(HelloMacro)]` belirttiğinde `hello_macro_derive` fonksiyonu
çağırılacak. Bu mümkündür çünkü burada `hello_macro_derive` fonksiyonunu `proc_macro_derive` ile notladık ve
trait adımızla eşleşen `HelloMacro` adını belirttik; bu, çoğu yordamsal makronun takip ettiği bir gelenektir.

`hello_macro_derive` fonksiyonu önce `input`'u bir `TokenStream`'den, üzerinde işlemler yapabileceğimiz bir veri
yapısına dönüştürür. İşte `syn`'nin devreye girdiği yer burasıdır. `syn` içindeki `parse` fonksiyonu
bir `TokenStream` alır ve ayrıştırılmış Rust kodunu temsil eden bir `DeriveInput` struct döndürür. Kod Listesi
20-41, ayrıştırdığımız `struct Pancakes;` string kodundan aldığımız `DeriveInput` struct'ının ilgili
kısımlarını gösterir.

<Listing number="20-41" caption="The `DeriveInput` instance we get when parsing code that has our macro's attribute in Listing 20-37">

```rust,ignore
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

</Listing>

Bu struct'ın alanları, ayrıştırdığımız Rust kodunun, `Pancakes`'in `ident`'ine (kimlik/identifier, yani ad)
sahip bir birim struct olduğunu gösteriyor. Her türlü Rust kodunu tanımlamak için bu struct'ta daha fazla alan var;
daha fazla bilgi için [`syn` belgelerindeki `DeriveInput` için][syn-docs] bakın.

Yakında, oluşturmak istediğimiz yeni Rust kodunu inşa edeceğimiz `impl_hello_macro` fonksiyonunu tanımlayacağız.
Ancak yapmadan önce, not edelim ki `derive` makromuz için çıktı da bir `TokenStream`'dir. Döndürülen `TokenStream`,
crate kullanıcılarımızın yazdığı koda eklenir böylece crate'lerini derlediklerinde, bizim değiştirilmiş `TokenStream`'de
sağladığımız ek işlevsellik elde ederler.

`hello_macro_derive` fonksiyonunun, `syn::parse` fonksiyonuna çağrı burada başarısız olursa
paniklemesini (panic) sağlamak için `unwrap` çağırdığımızı fark etmiş olabilirsiniz. Hatalarda yordamsal makromuzun
paniklemesi gerekir çünkü `proc_macro_derive` fonksiyonlarının `Result` yerine `TokenStream` döndürmesi
gerekir, yordamsal makro API'sine uygun olmak için. Bu örneği `unwrap` kullanarak basitleştirdik; üretim kodunda,
neyin yanlış gittiğine daha özel hata mesajları sağlamak için `panic!` veya `expect` kullanmalısınız.

Şimdi, işaretlenmiş Rust kodunu bir `TokenStream`'dan bir `DeriveInput` örneğine dönüştürmek için kodumuz var,
işaretlenen türde `HelloMacro` trait'ini uygulayan kod üretelim, Kod Listesi 20-42'de gösterildiği gibi.

<Listing number="20-42" file-name="hello_macro_derive/src/lib.rs" caption="Implementing `HelloMacro` trait using parsed Rust code">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-42/hello_macro/hello_macro_derive/src/lib.rs:here}}
```

</Listing>

İşaretlenen türün adını (kimliğini) içeren bir `Ident` struct örneği `ast.ident` kullanarak elde ederiz.
Kod Listesi 20-41'deki struct gösterir ki Kod Listesi 20-37'deki kod üzerinde `impl_hello_macro` fonksiyonunu
çalıştırdığımızda, elde edeceğimiz `ident`'in, `"Pancakes"` değerine sahip bir `ident` alanı olacak. Böylece,
Kod Listesi 20-42'deki `name` değişkeni, yazıldığında `"Pancakes"` string'ini olacaktır, Kod Listesi
20-37'deki struct'ın adı.

`quote!` makrosu, döndürmek istediğimiz Rust kodunu tanımlamamıza izin verir. Derleyici,
`quote!` makrosunun yürütülmesinin doğrudan sonucundan farklı bir şey bekler bu yüzden bunu bir `TokenStream`'e
dönüştürmemiz gerekir. Bunu `into` yöntemini çağırarak yaparız ki bu, ara temsili tüketir ve
gereken `TokenStream` türünde bir değer döndürür.

`quote!` makrosu ayrıca bazı çok havalı şablonlama mekanikaları sağlar: `#name` girebiliriz ve
`quote!` onu `name` değişkenindeki değerle yerine koyacak. Düzenli makroların çalışma yoluna benzer bazı tekrarlama
yapabilirsiniz. Kapsamlı bir giriş için [`quote` crate belgelerine][quote-docs] bakın.

Yordamsal makromuzun, kullanıcı işaretlediği türde `HelloMacro` trait'imizin uygulamasını üretmesini istiyoruz,
bunu `#name` kullanarak elde edebiliriz. Trait uygulamasının bir `hello_macro` fonksiyonu vardır ki onun gövdesi,
sağlamak istediğimiz işlevselliği içerir: `Hello, Macro! My name is` ve sonra işaretlenen türün adını yazdırmak.

Burada kullanılan `stringify!` makrosu, Rust'ta yerleşik (built-in). Bir Rust ifadesi alır, örneğin
`1 + 2`, ve derleme zamanında ifadesini bir string gerçek (literal), örneğin `"1 + 2"`'ye dönüştürür. Bu,
`format!` veya `println!`'den farklıdır ki bunlar ifadeyi değerlendirip sonucu bir `String`'e dönüştüren
makrolardır. `#name` girdisinin harfiyen yazdırılacak bir ifade olma ihtimali vardır bu yüzden `stringify!`
kullanıyoruz. `stringify!` kullanmak, ayrıca `#name`'i derleme zamanında bir string gerçek (literal)'e dönüştürerek
bir ayırmadan (allocation) kurtarır.

Bu noktada, `cargo build`, hem `hello_macro` hem `hello_macro_derive` içinde başarıyla tamamlanmalı. Kod Listesi
20-37'deki kodu bağlayalım (hook up) ve yordamsal makroyu aksiyonda görelim! _projects_ dizininizde
`cargo new pancakes` kullanarak yeni bir ikili (binary) proje oluşturun. `pancakes` crate'inin _Cargo.toml_'ında
`hello_macro` ve `hello_macro_derive`'i bağımlılık olarak eklememiz gerekiyor. [crates.io](https://crates.io/)<!--
ignore --> için `hello_macro` ve `hello_macro_derive` sürümlerinizi yayınlıyorsanız, bunlar normal bağımlılıklar olacaktır;
değilse, onları aşağıdaki gibi `path` bağımlılıkları olarak belirleyebilirsiniz:

```toml
{{#include ../listings/ch20-advanced-features/no-listing-21-pancakes/pancakes/Cargo.toml:6:8}}
```

Kod Listesi 20-37'dekini _src/main.rs_'a koyun ve `cargo run` çalıştırın: `Hello, Macro! My name is Pancakes!`
yazacak. Yordamsal makrodan gelen `HelloMacro` trait uygulaması, `pancakes` crate'inin onu uygulamasına
gerek kalmadan dahil edildi; `#[derive(HelloMacro)]` trait uygulamasını ekledi.

Sonra, diğer türde yordamsal makroların özel `derive` makrolarından nasıl farklılaştığına inceleyelim.

### Özellik-Benzeri Makrolar (Attribute-Like Macros)

Özellik-benzeri (attribute-like) makrolar, özel `derive` makrolarına benzer ancak `derive` özelliği için kod
üretmek yerine, yeni özellikler (attributes) oluşturmanıza izin verirler. Ayrıca daha esnektirler: `derive`
sadece struct'lar ve enum'ler için çalışır; özellikler diğer öğelere de uygulanabilir, örneğin fonksiyonlara.
İşte bir özellik-benzeri makro kullanmanın bir örneği. Bir web uygulaması çerçevesi kullanırken fonksiyonları
notlayan `route` adında bir özelliğe sahip olduğunuzu söyleyin:

```rust,ignore
#[route(GET, "/")]
fn index() {
}
```

Bu `#[route]` özelliği, çerçeve tarafından bir yordamsal makro olarak tanımlanacaktır. Makro tanımlama
fonksiyonun imzası şöyle görünür:

```rust,ignore
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
}
```

Burada, `TokenStream` türünde iki parametremiz var. İlk, özelliğin içeriği içindir: `GET, "/"` kısmı.
İkinci, özelliğin eklendiği öğenin gövdesidir: bu durumda `fn index() {}` ve fonksiyon gövdesinin
geri kalanı.

Bunun dışında, özellik-benzeri makrolar, özel `derive` makrolarıyla aynı şekilde çalışır: `proc-macro`
crate türüne sahip bir crate oluşturursunuz ve istediğiniz kodu üreten bir fonksiyon uygularsınız!

### Fonksiyon-Benzeri Makrolar (Function-Like Macros)

Fonksiyon-benzeri makrolar, fonksiyon çağırma gibi görünen makroları tanımlar. `macro_rules!`
makrolarına benzer şekilde, fonksiyonlardan daha esnektirler; örneğin, bilinmeyen sayıda argüman alabilirler.
Ancak, `macro_rules!` makroları, daha önce [“Genel Metaprogramlama İçin Bildirimsel
Makrolar”][decl]<!--
ignore --> bölümünde tartıştığımız benzer-match sözdizimi kullanarak tanımlanabilir.
Fonksiyon-benzeri makrolar bir `TokenStream` parametresi alır ve tanımları, diğer iki tür yordamsal
makronun yaptığı gibi, o `TokenStream`'i Rust kodu kullanarak manipüle eder. Bir fonksiyon-benzeri makronun örneği,
şöyle çağırılabilecek bir `sql!` makrosudur:

```rust,ignore
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

Bu makro, içindeki SQL ifadesini ayrıştırır ve sözdizimsel olarak doğru olduğunu kontrol eder ki bu, bir
`macro_rules!` makrosunun yapabileceğinden çok daha karmaşık işlemedir. `sql!` makrosu şöyle tanımlanacaktır:

```rust,ignore
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
}
```

Bu tanım, özel `derive` makrosunun imzasına benzer: Parantez içindeki tokenleri alır ve üretmek istediğimiz
kodu döndürür.

## Özet (Summary)

Vay! Artık, sık kullanmayacağınız ancak çok belirli durumlarda kullanılabilir olmasını bileceğiniz bazı Rust
özelliklerinizi araç kutunuzda (toolbox) var. Çeşitli karmaşık konular tanıttık böylece hata mesajı önerilerinde
veya diğer insanların kodunda karşılaştığınızda, bu konseptları ve sözdizimlerini tanıyabileceksiniz. Bu bölümü,
sizin çözümlere yönlendirecek bir referans olarak kullanın.

Sonra, kitap boyunca tartıştığımız her şeyi pratiğe koyacağız ve bir projenin daha fazlasını
yapacağız!

[ref]: ../reference/macros-by-example.html
[tlborm]: https://veykril.github.io/tlborm/
[syn]: https://crates.io/crates/syn
[quote]: https://crates.io/crates/quote
[syn-docs]: https://docs.rs/syn/2.0/syn/struct.DeriveInput.html
[quote-docs]: https://docs.rs/quote
[decl]: #declarative-macros-with-macro_rules-for-general-metaprogramming