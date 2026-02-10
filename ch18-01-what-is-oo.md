## Nesne Yönelimli Dillerin Özellikleri (Characteristics of Object-Oriented Languages)

Programlama topluluğunda bir dilin nesne yönelimli olması için hangi özelliklere sahip olması
gerektiği konusunda bir konsensüs yoktur. Rust, OOP dahil olmak üzere çok programlama paradigmasından
etkilenmiştir; örneğin, Bölüm 13'te fonksiyonel programlamadan gelen özellikleri inceledik.
Tartışmalı olarak, OOP dilleri belirli ortak özellikleri paylaşır—yani nesneler, kapsülleme ve
miras. Bu özelliklerin her birinin ne anlama geldiğine ve Rust'ın bunu destekleyip desteklemediğine
bakalım.

### Nesneler Veri ve Davranış İçerir (Objects Contain Data and Behavior)

Erich Gamma, Richard Helm, Ralph Johnson ve John Vlissides tarafından yazılan ve _Tasarım
Kalıpları: Yeniden Kullanılabilir Nesne Yönelimli Yazılımın Öğeleri_ (Design Patterns: Elements
of Reusable Object-Oriented Software) (Addison-Wesley, 1994), gayri resmi olarak _The Gang
of Four_ kitabı olarak adlandırılan kitap, nesne yönelimli tasarım kalıplarının bir kataloğudur.
OOP'yi şu şekilde tanımlar:

> Nesne yönelimli programlar nesnelerden oluşur. Bir **nesne** hem veriyi hem de o veri üzerinde
> çalışan prosedürleri paketler. Prosedürler genellikle **yöntemler** veya **işlemler** olarak
> adlandırılır.

Bu tanımı kullanarak, Rust nesne yönelimlidir: Struct'lar ve enum'lar veriye sahiptir ve
`impl` blokları struct'lar ve enum'lar üzerinde yöntemler sağlar. Yöntemleri olan struct'lar ve
enum'lar nesneler _olarak adlandırılmasa bile_, The Gang of Four'ün nesneler tanımına göre aynı
işlevselliği sağlarlar.

### Uygulama Detaylarını Gizleyen Kapsülleme (Encapsulation That Hides Implementation Details)

OOP ile yaygın olarak ilişkilendirilen başka bir yön _kapsülleme_ fikridir, bu da bir nesnenin
uygulama detaylarının o nesneyi kullanan koddan erişilebilir olmadığı anlamına gelir. Bu nedenle,
bir nesneyle etkileşmenin tek yolu onun genel API'si üzerinden; nesneyi kullanan kod nesnenin
içlerine girip doğrudan veriyi veya davranışı değiştirememelidir. Bu, programcının nesneyi kullanan
kodu değiştirmeye gerek kalmadan nesnenin içlerini değiştirmesini ve yeniden düzenlemesini
(sağlar).

Bölüm 7'de kapsüllemeyi nasıl kontrol edeceğimizi tartıştık: Kodumuzdaki hangi modüllerin,
türlerin, fonksiyonların ve yöntemlerin genel olması gerektiğine karar vermek için `pub` anahtar
kelimesini kullanabiliriz ve varsayılan olarak diğer her şey özeldir. Örneğin, `i32` değerlerinden
oluşan bir vektör içeren bir alana sahip olan bir `AveragedCollection` struct'ı tanımlayabiliriz.
Struct ayrıca vektördeki değerlerin ortalamasını içeren bir alana sahip olabilir, bu da ortalamanın
herkesin istediğinde talep üzerine hesaplanması gerektiği anlamına gelmez. Başka bir deyişle,
`AveragedCollection` bizim için hesaplanan ortalamayı önbellekleyecek. Kod Listesi 18-1,
`AveragedCollection` struct'ının tanımını içerir.

<Listing number="18-1" file-name="src/lib.rs" caption="An `AveragedCollection` struct that maintains a list of integers and the average of the items in the collection">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-01/src/lib.rs}}
```

</Listing>

Struct `pub` olarak işaretlenmiş böylece diğer kod onu kullanabilir, ancak struct içindeki alanlar
özel kalır. Bu durumda bu önemlidir çünkü listeye bir değer eklendiğinde veya kaldırıldığında
ortalamanın da güncellenmesini sağlamak istiyoruz. Bunu, Kod Listesi 18-2'de gösterildiği gibi,
struct üzerinde `add`, `remove` ve `average` yöntemlerini uygulayarak yapıyoruz.

<Listing number="18-2" file-name="src/lib.rs" caption="Implementations of the public methods `add`, `remove`, and `average` on `AveragedCollection`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-02/src/lib.rs:here}}
```

</Listing>

Genel yöntemler `add`, `remove` ve `average`, `AveragedCollection` örneğindeki verilere erişmenin veya
değiştirmenin tek yollarıdır. `add` yöntemi veya `remove` yöntemi kullanılarak bir öğe `list`'e
eklendiğinde veya kaldırıldığında, her birinin uygulamaları özel olan ve `average` alanını güncellemeyi
de işleyen `update_average` yöntemini çağırır.

`list` ve `average` alanlarını özel bırakıyoruz böylece dış kodun doğrudan `list` alanına öğe
eklemesi veya kaldırması için bir yol yoktur; aksi takdirde, `list` değiştiğinde `average` alanı
senkronize olmayabilir. `average` yöntemi `average` alanındaki değeri döndürür, dış kodun
`average` okumasına ancak değiştirmesine izin vermez.

`AveragedCollection` struct'ının uygulama detaylarını kapsüllediğimiz için, veri yapısı gibi
önemleri gelecekte kolayca değiştirebiliriz. Örneğin, `list` alanı için `Vec<i32>` yerine bir
`HashSet<i32>` kullanabiliriz. `add`, `remove` ve `average` genel yöntemlerinin imzaları aynı kaldığı
sürece, `AveragedCollection` kullanan kodun değişmesi gerekmez. Bunun yerine `list`'i genel yapmış
olsaydık, bu gerekli olmazdı: `HashSet<i32>` ve `Vec<i32>` öğe ekleme ve kaldırma için farklı
yöntemlere sahiptir, bu yüzden dış kod doğrudan `list`'i değiştiriyorsa muhtemelen değiştirmek
zorunda kalırdı.

Eğer kapsülleme, bir dilin nesne yönelimli olarak kabul edilmesi için gerekli bir yönse, Rust bu
gereksinimi karşılar. Kodun farklı kısımları için `pub` kullanıp kullanmamak seçeneği, uygulama
detaylarının kapsüllenmesini sağlar.

### Bir Tip Sistemi Olarak Miras ve Kod Paylaşımı (Inheritance as a Type System and as Code Sharing)

_Miras_, bir nesnenin başka bir nesnenin tanımından öğeler miras alabileceği böylece üst nesnenin
verilerini ve davranışlarını yeniden tanımlamadan kazanabileceği bir mekanizmadır.

Eğer bir dilin nesne yönelimli olması için mirasa sahip olması gerekiyorsa, Rust böyle bir dil değildir.
Üst struct'ın alanlarını ve yöntem uygulamalarını miras alan bir struct tanımlamanın bir yolu yoktur,
bir makro kullanmadan.

Ancak, programlama araçlarınızda mirasa alışkınsanız, Rust'ta diğer çözümleri kullanabilirsiniz,
mirasa ulaşmanızın sebebine bağlı olarak.

Mirası seçmek için iki ana nedeniniz olur. Biri kodun yeniden kullanımı için: Bir tip için belirli bir
davranışı uygulayabilirsiniz ve miras size farklı bir tip için o uygulamayı yeniden kullanmayı sağlar.
Bunu Rust kodunda varsayılan trait yöntem uygulamalarını kullanarak sınırlı bir şekilde
yapabilirsiniz, bunu Kod Listesi 10-14'te `Summary` trait'i üzerinde `summarize` yönteminin varsayılan
uygulamasını eklediğimizde gördük. `Summary` trait'ini uygulayan her tip `summarize` yöntemini ekstra
kod olmadan kullanabilir. Bu, bir üst sınıfın bir yönteminin bir uygulamasına ve miras alan bir alt
sınıfın da yönteminin uygulamasına sahip olmasıyla benzerdir. `Summary` trait'ini uyguladığımızda
`summarize` yönteminin varsayılan uygulamasını da geçersiz kılabiliriz, bu da bir alt sınıfın üst
sınıftan miras aldığı bir yöntemin uygulamasını geçersiz kılmasıyla benzerdir.

Mirası kullanmanın diğer nedeni tip sistemiyle ilgilidir: Bir alt tipin üst tipin olduğu yerlerde
kullanılabilmesini sağlamak için. Buna da _polimorfizm_ denir, bu da belirli özellikleri paylaşıyorsa
çalışma zamanında çoklu nesneler birbirinin yerine kullanılabilir.

> ### Polimorfizm (Polymorphism)
>
> Çoğu insan için polimorfizm miras ile eş anlamlıdır. Ancak aslında çoklu tipteki veriyle
> çalışabilen koda başvuran daha genel bir kavramdır. Miras için bu tipler genellikle alt sınıflardır.
>
> Bunun yerine Rust, farklı olası tipler üzerinde soyutlama yapmak için generics ve bu tiplerin
> ne sağlaması gerektiğine kısıtlamalar koymak için trait sınırları kullanır. Buna bazen _sınırlı
> parametrik polimorfizm_ (bounded parametric polymorphism) denir.

Rust miras sunmayı seçmemiş olarak farklı bir değiş tokuşlar seti seçmiştir. Miras genellikle
gereğinden daha fazla kod paylaşma riski taşır. Alt sınıflar her zaman üst sınıflarının tüm özelliklerini
paylaşmamalı ama mirasla yapacaktır. Bu, bir programın tasarımını daha az esnek hale getirebilir.
Ayrıca, alt sınıflarda yöntemler çağırma olasılığını da sunar ki bu mantıklı değildir veya
yöntemlerin alt sınıfa uygulanmadığı için hatalara neden olabilir. Ayrıca, bazı diller sadece _tek
miras_ (single inheritance) izin verir (bir alt sınıfın yalnızca bir sınıftan miras alabileceği
anlamına gelir), bu da bir programın tasarımının esnekliğini daha da kısıtlar.

Bu nedenlerle, Rust miras yerine çalışma zamanında polimorfizmi elde etmek için trait nesnelerini
kullanmaya farklı bir yaklaşım seçmiştir. Trait nesnelerinin nasıl çalıştığına bakalım.