## Yöntemler (Methods)

Yöntemler fonksiyonlara benzerdir: Onları `fn` anahtar kelimesi ve bir isimle bildiririz, parametreleri ve bir dönüş değeri olabilir ve başka bir yerden yöntem çağrıldığında çalışacak bazı kod içerirler. Fonksiyonlardan farklı olarak, yöntemler bir yapının (veya bir numaralandırmanın veya bir özellik nesnesinin ki bunları sırasıyla [Bölüm 6'da][enums]<!-- ignore --> ve [Bölüm 18'de][trait-objects]<!-- ignore --> kapsamlıyacağız) bağlamı içinde tanımlanır ve ilk parametreleri her zaman `self`'tir ki bu yapının yöntemin çağrıldığı örneğini temsil eder.

<!-- Old headings. Do not remove or links may break. -->

<a id="defining-methods"></a>

### Yöntem Sözdizimi (Method Syntax)

`Rectangle` örneği parametresi olan `area` fonksiyonunu değiştirelim ve bunun yerine `Rectangle` yapısı üzerinde tanımlanan bir `area` yöntemi yapalım, Kod Listesi 5-13'te gösterildiği gibi.

<Listing number="5-13" file-name="src/main.rs" caption="`Rectangle` yapısı üzerinde bir `area` yöntemi tanımlama">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-13/src/main.rs}}
```

</Listing>

`Rectangle`'in bağlamı içinde bir fonksiyon tanımlamak için, `Rectangle` için bir `impl` (uygulama) bloğu başlatıyoruz. Bu `impl` bloğu içindeki her şey `Rectangle` tipiyle ilişkilendirilir. Sonra, `area` fonksiyonunu `impl` kıvramalı parantezlerine taşıyoruz ve imzadaki ilk (ve bu durumda tek) parametreyi `self` yapıyoruz ve gövdede her yerde. `main`'de, `area` fonksiyonunu çağırdığımız ve `rect1`'i argüman olarak geçirdiğimiz yerde, bunun yerine yöntem sözdizimini kullanarak `Rectangle` örneğimiz üzerinde `area` yöntemini çağırabiliriz. Yöntem sözdizimi örneğinden sonra gelir: Bir nokta ve ardından yöntem adı, parantezler ve herhangi bir argüman ekliyoruz.

`area`'nın imzasında, `rectangle: &Rectangle` yerine `&self` kullanıyoruz. `&self` aslında `self: &Self` kısaltmasıdır. Bir `impl` bloğu içinde, `Self` tipi `impl` bloğu için olan tip için bir takma addır. Yöntemlerin ilk parametresi için `Self` tipinde `self` adında bir parametre olması gerekir, bu yüzden Rust bunu sadece ilk parametre yerinde `self` ismiyle kısaltmanıza izin verir. `self` kısaltmasının önünde `&` kullanmaya devam ettiğimize dikkat edin ki bu yöntemin `Self` örneğini ödünçlediğini (borrows) belirtir, `rectangle: &Rectangle`'de yaptığımız gibi. Yöntemler `self`'in sahipliğini alabilir, `self`'i değişmez olarak ödünçleyebilir, burada yaptığımız gibi, veya `self`'i değiştirilebilir olarak ödünçleyebilir, tıpkı herhangi bir diğer parametrede olduğu gibi.

Burada `&self`'i fonksiyon sürümünde `&Rectangle` kullandığımız aynı nedenle seçtik: Sahiplik almak istemiyoruz ve sadece yapıdaki veriyi okumak istiyoruz, onu yazmak değil. Eğer yöntemin yaptığı bir parçası olarak yöntemin çağrıldığı örneği değiştirmek isteseydik, ilk parametre olarak `&mut self` kullanırdık. Yalnızca `self`'i ilk parametre olarak kullanarak örneğin sahipliğini alan bir yönteme sahip olmak nadirdir; bu teknik genellikle yöntem `self`'i başka bir şeye dönüştürdüğünde ve dönüşümden sonra çağıranın orijinal örneği kullanmasını engellemek istediğinizde kullanılır.

Yöntemleri fonksiyonlar yerine kullanmanın ana nedeni, yöntem sözdizimi sağlamaya ek olarak, her yöntemin imzasında `self`'in tipini tekrarlamak zorunda olmamaya ek olarak, organizasyondur. Örneğinin yapabileceği tüm şeyleri tek bir `impl` bloğuna koyduk ve gelecekteki kodumuzu kullanıcılara sağladığımız kütüphanenin çeşitli yerlerinde `Rectangle`'ın yeteneklerini aramalar zorunda bırakmadık.

Bir yönteme yapının alanlarından biriyle aynı ismi verebileceğimizi fark edin. Örneğin, `Rectangle` üzerinde de `width` adında bir yöntem tanımlayabiliriz:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-06-method-field-interaction/src/main.rs:here}}
```

</Listing>

Burada, örneklerin `width` alanındaki değer `0`'dan büyükse `true` dönen ve değer `0`'sa `false` dönen bir `width` yöntemi yapmayı seçiyoruz: Aynı ismin yöntemin içinde bir alanı herhangi bir amaç için kullanabiliriz. `main`'de, `rect1.width`'i parantezlerle takip ettiğimizde, Rust `width` yöntemini kastettiğimizi biliyor. Parantezleri kullanmadığımızda, Rust `width` alanını kastettiğini biliyor.

Genellikle ama her zaman değil, bir yönteme bir alanla aynı ismi verdiğimizde sadece alandaki değeri döndürmesini ve başka bir şey yapmamasını istiyoruz. Bu gibi yöntemler _getter_'lar olarak adlandırılır ve bazı başka dillerin yaptığı gibi Rust yapı alanları için otomatik olarak uygulamaz. Getter'lar kullanışlıdır çünkü alanı özel (private) yapabilirsiniz ancak yöntemi genel (public) yapabilirsiniz ve böylece tipin genel API'sinin bir parçası olarak o alana salt okunabilir erişim izin verirsiniz. Genel ve özelin ne olduğunu ve bir alan veya yöntemi genel veya özel olarak nasıl atayacağınızı [Bölüm 7'de][public]<!-- ignore --> konuşacağız.

> ### `->` Operatörü Nerede?
>
> C ve C++'ta yöntemleri çağırmak için iki farklı operatör kullanılır: Bir nesne üzerinde doğrudan yöntem çağırıyorsanız `.` kullanırsınız ve bir nesnenin işaretçisi üzerinde yöntem çağırıyorsanız ve işaretçiyi önce çözmek istiyorsanız `->` kullanırsınız. Başka bir deyimle, eğer `object` bir işaretçi ise, `object->something()`, `(*object).something()`'a benzer.
>
> Rust'ın `->` operatörüne eşdeğer bir şey yoktur; aksine, Rust _otomatik başvurma ve çözme_ adında bir özelliği vardır. Yöntemleri çağırmak bu davranışa sahip Rust'ta az yerlerden biridir.
>
> İşte nasıl çalıştığı: `object.something()` ile bir yöntem çağırdığınızda, Rust otomatik olarak `&`, `&mut` veya `*` ekler böylece `object` yöntemin imzasıyla eşleşir. Başka bir deyimle, şunlar aynıdır:
>
> <!-- CAN'T EXTRACT SEE BUG https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust
> # #[derive(Debug,Copy,Clone)]
> # struct Point {
> #     x: f64,
> #     y: f64,
> # }
> #
> # impl Point {
> #    fn distance(&self, other: &Point) -> f64 {
> #        let x_squared = f64::powi(other.x - self.x, 2);
> #        let y_squared = f64::powi(other.y - self.y, 2);
> #
> #        f64::sqrt(x_squared + y_squared)
> #    }
> # }
> # let p1 = Point { x: 0.0, y: 0.0 };
> # let p2 = Point { x: 5.0, y: 6.5 };
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> İlk çok daha temiz görünüyor. Bu otomatik başvurma davranışı yöntemlerin net bir alıcısı olması nedeniyle çalışır—`self`'in tipi. Alıcı ve bir yöntemin ismi verildiğinde, Rust yöntemin okuduğunu (`&self`), değiştirdiğini (`&mut self`) veya tükettiğini (`self`) kesin olarak çıkarabilir. Rust'ın yöntem alıcıları için ödünçlemeyi örtük yapması sahipliği pratikte ergonomik yapmanın büyük bir parçasıdır.

### Daha Fazla Parametreli Yöntemler

Yöntemleri kullanmayı `Rectangle` yapısı üzerinde ikinci bir yöntem uygulayarak pratik edelim. Bu sefer `Rectangle` örneğinin başka bir `Rectangle` örneğini almasını ve ikinci `Rectangle` `self` (ilk `Rectangle`) içinde tamamen sığabiliyorsa `true` döndürmesini istiyoruz; aksi takdirde, `false` döndürmelidir. Yani, `can_hold` yöntemini tanımladıktan sonra, Kod Listesi 5-14'te gösterilen programı yazmak istiyoruz.

<Listing number="5-14" file-name="src/main.rs" caption="Henüz yazılmamış `can_hold` yöntemini kullanma">

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-14/src/main.rs}}
```

</Listing>

Beklenen çıktısı şu gibi görünecek çünkü `rect2`'nin her iki boyutu `rect1`'in boyutlarından küçük, ancak `rect3` `rect1`'den daha geniş:

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

Bir yöntem tanımlamak istediğimizi biliyoruz, bu yüzden `impl Rectangle` bloğu içinde olacak. Yöntemin adı `can_hold` olacak ve başka bir `Rectangle`'ın değişmez bir ödünçsünü (immutable borrow) parametre olarak alacak. Yöntemi çağıran koda bakarak parametrenin hangi tip olacağını söyleyebiliriz: `rect1.can_hold(&rect2)` `&rect2`'yi geçer ki bu `rect2`'ye bir `Rectangle` örneği olan değişmez ödünçsüdür. Bu mantıklıdır çünkü sadece `rect2`'yi okumamız gerekir (yazmak değil ki değiştirilebilir ödünç gerektirecektir) ve `main`'in `rect2`'nin sahipliğini korumasını istiyoruz böylece `can_hold` yöntemini çağırdıktan sonra tekrar kullanabiliriz. `can_hold`'ın dönüş değeri bir Boolean olacak ve uygulama `self`'in genişliğinin ve yüksekliğinin diğer `Rectangle`'ın genişliğinden ve yüksekliğinden daha büyük olup olmadığını kontrol edecek. Kod Listesi 5-13'teki `impl` bloğuna yeni `can_hold` yöntemini ekleyelim, Kod Listesi 5-15'te gösterildiği gibi.

<Listing number="5-15" file-name="src/main.rs" caption="`Rectangle` üzerinde `can_hold` yöntemini uygulama ki başka bir `Rectangle` örneğini parametre olarak alır">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-15/src/main.rs:here}}
```

</Listing>

Bu kodu Kod Listesi 5-14'teki `main` fonksiyonuyla çalıştırdığımızda, istediğimiz çıktıyı alacağız. Yöntemler `self` parametresinden sonra imzaya eklediğimiz çok sayıda parametre alabilir ve bu parametreler fonksiyonlardaki parametreler gibi çalışır.

### İlişkili Fonksiyonlar (Associated Functions)

Bir `impl` bloğu içinde tanımlanan tüm fonksiyonlar _ilişkili fonksiyonlar_ olarak adlandırılır çünkü `impl`'den sonra adlandırılan tiple ilişkilidirler. İlk parametreleri olarak `self`'i olmayan (ve böylece yöntem olmayan) ilişkili fonksiyonlar tanımlayabiliriz çünkü çalışmak için tipin bir örneğine ihtiyaçları yoktur. Zaten bu gibi bir fonksiyon kullandık: `String` tipi üzerinde tanımlanan `String::from` fonksiyonu.

Yöntem olmayan ilişkili fonksiyonlar genellikle yapının yeni bir örneğini dönecek olan kurucular için kullanılır. Bunlar genellikle `new` adı verilir ancak `new` özel bir isim değildir ve dile dahili değildir. Örneğin, bir boyut parametresi ve bunu hem genişlik hem yükseklik olarak kullanan bir `square` adında ilişkili fonksiyon sağlamayı seçebiliriz böylece kare `Rectangle` oluşturmak daha kolay olur aynı değeri iki kez belirtmek zorunda kalmaz:

<span class="filename">Dosya adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-03-associated-functions/src/main.rs:here}}
```

Dönüş tipindeki ve fonksiyon gövdesindeki `Self` anahtar kelimeleri `impl` anahtar kelimesinden sonra gelen tip için takma addır ki bu durumda `Rectangle`'tır.

Bu ilişkili fonksiyonu çağırmak için, `::` sözdizimini yapı adı ile kullanıyoruz; `let sq = Rectangle::square(3);` bir örnektir. Bu fonksiyon yapı tarafından adlandırılmıştır: `::` sözdizimi hem ilişkili fonksiyonlar hem de modüller tarafından oluşturulan ad alanları için kullanılır. [Bölüm 7'de][modules]<!-- ignore --> modülleri konuşacağız.

### Birden Fazla `impl` Bloğu

Her yapının birden fazla `impl` bloğuna sahip olması izin verilir. Örneğin, Kod Listesi 5-15 Kod Listesi 5-16'da gösterilen koda eşittir ki her yöntemi kendi `impl` bloğunda vardır.

<Listing number="5-16" caption="Birden fazla `impl` bloğu kullanarak Kod Listesi 5-15'i yeniden yazma">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-16/src/main.rs:here}}
```

</Listing>

Burada bu yöntemleri birden fazla `impl` bloğuna ayırmak için hiçbir neden yok ancak bu geçerli sözdizimidir. Birden fazla `impl` bloğunun kullanışlı olduğu bir durumu Bölüm 10'da göreceğiz ki orada genel tipler ve özellikleri konuşuyoruz.

## Özet

Yapılar alanınız için anlamına gelen özel tipler oluşturmanızı sağlar. Yapıları kullanarak, ilgili veri parçalarını birbirine bağlı tutabilirsiniz ve her parçayı adlandırarak kodunuzu net yapabilirsiniz. `impl` bloklarında, tipinizle ilişkili fonksiyonları tanımlayabilirsiniz ve yöntemler yapılarınızın örneklerinin sahip olduğu davranışı belirtmenize izin veren bir tür ilişkili fonksiyondur.

Ancak yapılar özel tip oluşturmanın tek yolu değil: Araç kutunuza başka bir araç eklemek için Rust'ın enum (numaralandırma) özelliğine dönelim.

[enums]: ch06-00-enums.html
[trait-objects]: ch18-02-trait-objects.md
[public]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html