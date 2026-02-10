<!-- Old headings. Do not remove or links may break. -->

<a id="using-trait-objects-that-allow-for-values-of-different-types"></a>

## Ortak Davranış Üzerinde Soyutlama Yapmak İçin Trait Nesnelerini Kullanmak (Using Trait Objects to Abstract over Shared Behavior)

Bölüm 8'de vektörlerin sınırlamalarından birinin yalnızca bir tip öğe saklayabilmesi olduğunu
belirttik. Kod Listesi 8-9'da bir çözüm oluşturduk burada tamsayıları, ondalıklı sayıları ve
metinleri tutan varyantlara sahip bir `SpreadsheetCell` enum'ı tanımladık. Bu, her hücrede farklı
tiplerde veri saklayabileceğimiz ve hücre satırını temsil eden bir vektöre sahip olabileceğimiz
anlamına gelir. Bu, değiştirilebilir öğelerimiz kodumuz derlendiğinde bildiğimiz sabit bir tipler
seti olduğunda tamamen iyi bir çözümdür.

Ancak, bazen kütüphanemizin kullanıcısının belirli bir durumda geçerli olan tipler setini
genişletmesini istiyoruz. Bunu nasıl başarabileceğimizi göstermek için, bir öğeler listesinde
geçerek, her birinin üzerinde bir `draw` yöntemi çağırarak onu ekrana çizen bir örnek grafik
kullanıcı arayüzü (GUI) aracı oluşturacağız—GUI araçları için yaygın bir teknik. `gui` adında
bir GUI kütüphanesinin yapısını içeren bir kütüphane crate'i oluşturacağız. Bu crate,
kullanıcıların kullanması için bazı tipler içerebilir, örneğin `Button` veya `TextField`. Ayrıca,
`gui` kullanıcıları çizilebilir kendi tiplerini oluşturmak isteyecekler: Örneğin, bir programcı
bir `Image` ekleyebilir, başka bir `SelectBox` ekleyebilir.

Kütüphaneyi yazma zamanında, diğer programcıların oluşturmak isteyebileceği tüm tipleri bilemeyiz
ve tanımlayamayız. Ancak `gui`'nin çok farklı tiplerin değerlerini takip etmesi gerektiğini
biliyoruz ve bu farklı tipli değerlerin her biri üzerinde bir `draw` yöntemi çağırması gerekiyor.
`draw` yöntemini çağırdığımızda tam olarak ne olacağını bilmeye gerek yok, sadece değerin bize
çağırmak için o yönteme sahip olduğunu bilmemiz gerekiyor.

Bunu mirasa sahip bir dilde yapmak için, üzerinde `draw` adında bir yöntemi olan `Component` adında
bir sınıf tanımlayabiliriz. Diğer sınıflar, örneğin `Button`, `Image` ve `SelectBox`, `Component`
'den miras alacak ve böylece `draw` yöntemini miras alacaklar. Özel davranışlarını tanımlamak için
her biri `draw` yöntemini geçersiz kılabilir ama framework tüm tipleri sanki `Component`
örnekleriymiş gibi davranabilir ve onların üzerinde `draw` çağırabilir. Ancak Rust mirasa sahip olmadığı
için, kullanıcıların kütüphaneye uyumlu yeni tipler oluşturmasına izin vermek için `gui`
kütüphanesini yapılandırmak için başka bir yola ihtiyacımız var.

### Ortak Davranış İçin Bir Trait Tanımlamak (Defining a Trait for Common Behavior)

`gui`'ye sahip olmasını istediğimiz davranışı uygulamak için, üzerinde `draw` adında bir yöntemi
olan `Draw` adında bir trait tanımlayacağız. Sonra, bir trait nesnesi alan bir vektör tanımlayabiliriz.
Bir _trait nesnesi_ hem belirlediğimiz trait'i uygulayan bir tipin örneğine hem de çalışma zamanında
o tip üzerinde trait yöntemlerini aramak için kullanılan bir tabloya işaret eder. Bir trait nesnesini
bir referans veya `Box<T>` akıllı işaretçisi gibi bir tür işaretçi belirleyerek, sonra `dyn`
anahtar kelimesini ve sonra ilgili trait'i belirleyerek oluşturuyoruz. (Trait nesnelerinin neden bir
işaretçi kullanması gerektiğinden Bölüm 20'de [“Dinamik Olarak Boyutlandırılan Tipler ve
`Sized` Trait”][dynamically-sized]<!-- ignore --> bölümünde konuşacağız.) Trait nesnelerini bir generic
veya somut tip yerine kullanabiliriz. Trait nesnesi kullandığımız her yerde, Rust'ın tip sistemi
derleme zamanında bu bağlamda kullanılan herhangi bir değerin trait nesnesinin trait'ini uyguladığından
emin olacak. Sonuç olarak, derleme zamanında tüm olası tipleri bilmemize gerek yoktur.

Rust'ta, diğer dillerin nesnelerinden ayırt etmek için struct'ları ve enum'ları "nesneler" olarak
adlandırmaktan kaçındığımızı belirttik. Bir struct veya enum'da, struct alanlarındaki veri ve
`impl` bloklarındaki davranış ayrılır, oysa diğer dillerde veri ve davranış bir kavramda birleştirilmiş
olur genellikle nesne olarak etiketlenir. Trait nesneleri diğer dillerdeki nesnelerden farklıdır çünkü
bir trait nesnesine veri ekleyemeyiz. Trait nesneleri diğer dillerdeki nesneler kadar genel olarak
faydalı değildir: Onların belirli amacı ortak davranış üzerinden soyutlamayı mümkün kılmaktır.

Kod Listesi 18-3, üzerinde `draw` adında bir yöntemi olan `Draw` adında bir trait'i nasıl
tanımlayacağınızı gösterir.

<Listing number="18-3" file-name="src/lib.rs" caption="Definition of the `Draw` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-03/src/lib.rs}}
```

</Listing>

Bu sözdizimi Bölüm 10'da trait'leri nasıl tanımladığımız hakkında tartışmalarımızdan tanıdık
gelmeli. Sıradan yeni bir sözdizimi geliyor: Kod Listesi 18-4, `components` adında bir vektör
tutan `Screen` adında bir struct tanımlar. Bu vektör `Box<dyn Draw>` tipindedir, bu bir trait
nesnesidir; bir `Box` içinde `Draw` trait'ini uygulayan herhangi bir tipin yerine geçer.

<Listing number="18-4" file-name="src/lib.rs" caption="Definition of the `Screen` struct with a `components` field holding a vector of trait objects that implement `Draw` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-04/src/lib.rs:here}}
```

</Listing>

`Screen` struct'ı üzerinde, her birinin `components` üzerinde `draw` yöntemini çağıracak `run`
adında bir yöntem tanımlayacağız, bu Kod Listesi 18-5'te gösterildiği gibi.

<Listing number="18-5" file-name="src/lib.rs" caption="A `run` method on `Screen` that calls the `draw` method on each component">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-05/src/lib.rs:here}}
```

</Listing>

Bu, trait sınırları ile generic bir tip parametresi kullanan bir struct tanımlamaktan farklı çalışır.
Bir generic tip parametresi bir seferde yalnızca bir somut tip ile değiştirilebilir, oysa trait
nesneleri çalışma zamanında trait nesnesi yerine çoklu somut tipleri dolmaya izin verir. Örneğin,
Kod Listesi 18-6'da olduğu gibi, bir generic tip ve trait sınırı kullanarak `Screen` struct'ını
tanımlayabilirdik.

<Listing number="18-6" file-name="src/lib.rs" caption="An alternate implementation of the `Screen` struct and its `run` method using generics and trait bounds">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-06/src/lib.rs:here}}
```

</Listing>

Bu bizi tüm bileşenleri `Button` tipinde veya tümü `TextField` tipinde olan bir `Screen` örneği ile
sınırlar. Sadece homojen koleksiyonlarınız olacaksa, generics ve trait sınırlarını kullanmak
daha iyidir çünkü tanımlar derleme zamanında somut tipleri kullanmak için monomorfiye edilir.

Diğer yandan, trait nesnelerini kullanan bir yöntem ile, bir `Screen` örneği hem bir `Box<Button>`
hem de bir `Box<TextField>` içeren bir `Vec<T>` tutabilir. Bu nasıl çalıştığına bakalım, sonra çalışma
zamanı performans etkilerini konuşacağız.

### Trait'i Uygulamak (Implementing the Trait)

Şimdi `Draw` trait'ini uygulayan bazı tipler ekleyeceğiz. `Button` tipini sunacağız. Yine, gerçekten
bir GUI kütüphanesi uygulamak bu kitabın kapsamının ötesindedir, bu yüzden `draw` yönteminin
gövdesinde hiçbir kullanışlı uygulama olmayacak. Uygulamanın neye benzeyebileceğini hayal etmek
için, bir `Button` struct'ı Kod Listesi 18-7'de gösterildiği gibi `width`, `height` ve `label`
alanlarına sahip olabilir.

<Listing number="18-7" file-name="src/lib.rs" caption="A `Button` struct that implements the `Draw` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-07/src/lib.rs:here}}
```

</Listing>

`Button` üzerindeki `width`, `height` ve `label` alanları diğer bileşenlerdeki alanlardan farklı
olacak; örneğin, `TextField` tipi bu aynı alanlara artı bir `placeholder` alanına sahip olabilir.
Ekrana çizmek istediğimiz tiplerin her biri `Draw` trait'ini uygulayacak ama o belirli tipi nasıl
çizeceğini tanımlamak için `draw` yönteminde farklı kod kullanacak, `Button`'ın burada yaptığı gibi
(bahsedildiği gibi gerçek GUI kodu olmadan). `Button` tipi, örneğin, kullanıcının butona tıkladığında
ne olduğuna ilişkin yöntemler içeren ek bir `impl` bloğu olabilir. Bu tür yöntemler `TextField` gibi
tiplere uygulanmayacaktır.

Kütüphanemizi kullanan biri `width`, `height` ve `options` alanlarına sahip `SelectBox` struct'ını
uygulamaya karar verirse, Kod Listesi 18-8'de gösterildiği gibi `SelectBox` tipinde de `Draw`
trait'ini uygulayacaklar.

<Listing number="18-8" file-name="src/main.rs" caption="Another crate using `gui` and implementing the `Draw` trait on a `SelectBox` struct">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-08/src/main.rs:here}}
```

</Listing>

Kütüphanemizin kullanıcısı artık bir `Screen` örneği oluşturmak için `main` fonksiyonlarını
yazabilirler. `Screen` örneğine, her birini bir `Box<T>` içine koyarak bir `SelectBox` ve bir
`Button` ekleyebilirler böylece trait nesnesi olurlar. Sonra `Screen` örneği üzerinde `run` yöntemini
çağırabilirler, bu da her birinin bileşenlerinin üzerinde `draw` çağıracak. Kod Listesi 18-9 bu
uygulamayı gösterir.

<Listing number="18-9" file-name="src/main.rs" caption="Using trait objects to store values of different types that implement the same trait">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-09/src/main.rs:here}}
```

</Listing>

Kütüphaneyi yazdığımızda, birinin `SelectBox` tipi ekleyebileceğini bilemedik ama `Screen`
uygulamamız yeni tip üzerinde çalışabildi ve onu çizdi çünkü `SelectBox` `Draw` trait'ini
uygular, bu da `draw` yöntemini uygular anlamına gelir.

Bu kavram—bir değerin somut tipinden ziyade bir değerin yanıt verdiği mesajlarla ilgilenmek—dinamik
tipli dillerdeki _ördek yazılımı_ (duck typing) kavramına benzer: Eğer bir ördek gibi yürür ve bir
ördek gibi vaklar, o zaman bir ördek olmalı! Kod Listesi 18-5'te `Screen` üzerindeki `run`
uygulamasında, `run`'un her bileşenin somut tipinin ne olduğunu bilmeye ihtiyacı yok. Bir bileşenin
bir `Button` örneği mi yoksa bir `SelectBox` mi olduğunu kontrol etmez, sadece bileşen üzerinde
`draw` yöntemini çağırır. `components` vektöründeki değerlerin tipi olarak `Box<dyn Draw>`
belirleyerek, `Screen`'in üzerine `draw` yöntemini çağırabileceğimiz değerlere ihtiyaç duyduğunu
tanımladık.

Trait nesnelerini ve Rust'ın tip sistemini kullanarak ördek yazılımı kullanan koda benzer kod
yazmanın avantajı, çalışma zamanında bir değerin belirli bir yöntemi uygulayıp uygulamadığını kontrol
etmemize veya bir değer bir yöntemi uygulamıyorsa ama biz yine de çağırırsak hatalar almamıza
ihtiyacımız olmamasıdır. Eğer değerler trait nesnelerinin ihtiyaç duyduğu trait'leri uygulamıyorsa,
Rust kodumuzu derlemeyecek.

Örneğin, Kod Listesi 18-10, bir `String` bileşen olarak `Screen` oluşturmaya çalışırsak ne olacağını
gösterir.

<Listing number="18-10" file-name="src/main.rs" caption="Attempting to use a type that doesn't implement the trait object's trait">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-10/src/main.rs}}
```

</Listing>

`String` `Draw` trait'ini uygulamadığı için bu hatayı alacağız:

```console
{{#include ../listings/ch18-oop/listing-18-10/output.txt}}
```

Bu hata bize, ya `Screen`'e geçirmeyi düşünmediğimiz bir şey geçirdiğimizi ve bu yüzden farklı
bir tip geçirmemiz gerektiğini ya da `Screen`'in üzerinde `draw` çağırabilmesi için `String`
üzerinde `Draw` uygulamamız gerektiğini bilirir.

<!-- Old headings. Do not remove or links may break. -->

<a id="trait-objects-perform-dynamic-dispatch"></a>

### Dinamik Dispatch Yapmak (Performing Dynamic Dispatch)

Bölüm 10'da [“Generics Kullanan Kodun Performansı”][performance-of-code-using-generics]<!--
ignore --> bölümünde, `compiler` tarafından generics üzerinde yapılan monomorfiyeleme süreci hakkında
tartışmamızı hatırlayın: `Compiler`, generic tip parametresi yerine kullandığımız her somut tip
için fonksiyonların ve yöntemlerin generic olmayan uygulamalarını oluşturur. Monomorfiyeleme sonucu
olan kod _statik dispatch_ yapıyor, bu derleme zamanında hangi yöntemi çağırdığınızı bilmek demek.
Bu _dinamik dispatch_'a (dynamic dispatch) zıttır, bu derleme zamanında hangi yöntemi çağırdığınızı
bilmenin mümkün olmadığı demektir. Dinamik dispatch durumlarında, `compiler` çalışma zamanında hangi
yöntemi çağıracağını bilecek kod oluşturur.

Trait nesnelerini kullandığımızda, Rust dinamik dispatch kullanmak zorundadır. `Compiler`, trait
nesnelerini kullanan kodla kullanılabilecek tüm tipleri bilemez, bu yüzden hangi yöntemin hangi tip
üzerinde uygulandığını çağırmayı bilmez. Bunun yerine, çalışma zamanında, hangi yöntemi çağırmayı
bilebilmek için trait nesnesi içindeki işaretçileri kullanır. Bu arama, statik dispatch'te olmayan bir
çalışma zamanı maliyeti oluşturur. Dinamik dispatch ayrıca `compiler`'ın bir yöntemin kodunu
satır içine yerleştirmeyi seçmesini engeller, bu da bazı optimizasyonları engeller ve Rust dinamik
dispatch'i nerede ve nerede kullanamayacağınız hakkında bazı kurallara sahiptir, buna _dyn
uyumluluğu_ (dyn compatibility) denir. Bu kurallar bu tartışmanın kapsamının ötesindedir ama
bunlar hakkında [referansta][dyn-compatibility]<!-- ignore --> daha fazla okuyabilirsiniz. Ancak, Kod
Listesi 18-5'te yazdığımız kodda ekstra esneklik elde ettik ve Kod Listesi 18-9'te destekleyebildik,
bu yüzden göz önünde bulundurulacak bir değiş tokuştur.

[performance-of-code-using-generics]: ch10-01-syntax.html#performance-of-code-using-generics
[dynamically-sized]: ch20-03-advanced-types.html#dynamically-sized-types-and-the-sized-trait
[dyn-compatibility]: https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility