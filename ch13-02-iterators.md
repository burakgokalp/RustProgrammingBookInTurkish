## İteratörlerle Öğe Serisini İşleme (Processing a Series of Items with Iterators)

İteratör deseni, bir öğe serisinin her birini sırayla bazı görevleri gerçekleştirmenize izin verir. İteratör, her bir öğe üzerinde yineleme mantığından ve serinin ne zaman bittiğini belirlemekten sorumludur. İteratörler kullandığınızda, bu mantığı kendiniz tekrar uygulamak zorunda değilsiniz.

Rust'te iteratörler _tembeldir_, bu onların iteratörü kullanmak için onu tüketen yöntemleri çağırana kadar etki göstermediği anlamına gelir. Örneğin, Kod Listesi 13-10'teki kod, `Vec<T>` üzerinde tanımlanan `iter` yöntemini çağırarak `v1` vektöründeki öğelerin üzerinde bir iteratör oluşturur. Bu kod tek başına hiçbir şey faydalı yapmaz.

<Listing number="13-10" file-name="src/main.rs" caption="Bir iteratör oluşturma">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-10/src/main.rs:here}}
```

</Listing>

İteratör `v1_iter` değişkeninde saklanıyor. Bir kez bir iteratör oluşturduktan sonra, onu çeşitli şekillerde kullanabiliriz. Kod Listesi 3-5'te, her bir öğesi üzerinde bazı kodları çalıştırmak için bir `for` döngüsü kullanarak bir dizinin üzerinde yinelendik. Perde altında, bu örtük bir iteratör oluşturdu ve sonra onu tüketti, ancak şimdiye kadar bunun tam olarak nasıl çalıştığı hakkında konuştuk.

Kod Listesi 13-11'deki örnekte, iteratörün oluşturmasını `for` döngüsünde iteratörün kullanımından ayırıyoruz. `for` döngüsü `v1_iter`'teki iteratör kullanılarak çağrıldığında, iteratördeki her eleman döngünün bir yinelemesinde kullanılır ki bu her değeri yazdırır.

<Listing number="13-11" file-name="src/main.rs" caption="Bir `for` döngüsünde bir iteratör kullanma">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-11/src/main.rs:here}}
```

</Listing>

Standart kütüphaneleri tarafından sağlanan iteratörler olmayan dillerde, aynı işlevselliği yazmak muhtemelen index 0'da bir değişken başlatarak, o değişkeni vektöre indeksleyerek bir değer almak için kullanarak ve vektördeki öğelerin toplam sayısına ulaştığında döngüde değişken değerini arttırarak yazarlardınız.

İteratörler tüm bu mantığı sizin için ele alır, potansiyel olarak karışabileceğiniz tekrarlayan kodu azaltır. İteratörler, sadece indeksleyebileceğiniz veri yapıları gibi değil, birçok farklı türde serilerle aynı mantığı kullanmanız için daha fazla esneklik verir. İteratörlerin bunu nasıl yaptığını inceleyelim.

### `Iterator` Trait'i ve `next` Yöntemi (The `Iterator` Trait and `next` Method)

Tüm iteratörler standart kütüphanede tanımlanan `Iterator` adında bir trait'ini uygular. Trait'in tanımı şöyle görünür:

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // varsayılan uygulamaları olan yöntemler atlandı
}
```

Bu tanımın yeni bir sözdizimi kullandığına dikkat edin: `type Item` ve `Self::Item`, bu trait ile bir ilişkili tip tanımlıyorlar. İlişkili tipleri Bölüm 20'de derinlemesine konuşacağız. Şimdilik bilmeniz gereken tek şey, bu kodun `Iterator` trait'ini uygulamanın `Item` tipini de tanımlamanızı gerektirdiğini ve bu `Item` tipinin `next` yönteminin dönüş tipinde kullanıldığını söylüyor. Başka bir deyişle, `Item` tipi iteratörden dönen tip olacaktır.

`Iterator` trait'i sadece uygulayıcıların tek bir yöntemi tanımlamasını gerektirir: `next` yöntemi, bu iteratörün her seferinde bir öğesini, `Some`'de sarılmış döner ve yineleme bittiğinde, `None` döner.

İteratörler üzerinde doğrudan `next` yöntemini çağırabiliriz; Kod Listesi 13-12, bir vektörden oluşturulan bir iteratör üzerindeki `next`'e tekrarlayan çağrılardan hangi değerlerin döndüğünü gösterir.

<Listing number="13-12" file-name="src/lib.rs" caption="Bir iteratör üzerinde `next` yöntemini çağırma">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-12/src/lib.rs:here}}
```

</Listing>

`v1_iter`'in değiştirilebilir yapmamız gerektiğine dikkat edin: Bir iteratör üzerinde `next` yöntemini çağırmak, iteratörün serinin neresinde olduğunu izlemek için kullandığı iç durumu değiştirir. Başka bir deyişle, bu kod iteratörü _tüketir_ veya kullanır. `next`'e yapılan her çağrı iteratörden bir öğeyi yer. `for` döngüsü kullandığımızda `v1_iter`'in değiştirilebilir yapmamıza gerekmediydi çünkü döngü `v1_iter`'in mülkiyetini aldı ve onu perde altında değiştirilebilir yaptı.

Ayrıca `next`'e yapılan çağrılardan aldığımız değerlerin vektördeki değerlere değişmez referanslar olduğuna dikkat edin. `iter` yöntemi değişmez referanslar üzerinde bir iteratör üretir. Eğer `v1`'in mülkiyetini alan ve sahip olunan değerleri döndüren bir iteratör oluşturmak istiyorsak, `iter` yerine `into_iter`'i çağırabiliriz. Benzer şekilde, eğer değiştirilebilir referanslar üzerinde yinelemek istiyorsak, `iter` yerine `iter_mut`'u çağırabiliriz.

### İteratörü Tüketen Yöntemler (Methods That Consume Iterator)

`Iterator` trait'inin standart kütüphane tarafından sağlanan varsayılan uygulamalarla birkaç farklı yöntemi vardır; bu yöntemleri `Iterator` trait'i için standart kütüphane API belgelerine bakarak öğrenebilirsiniz. Bu yöntemlerden bazıları tanımlarında `next` yöntemini çağırır ki bu yüzden `Iterator` trait'ini uygularken `next` yöntemini uygulamanız gerekir.

`next`'i çağıran yöntemlere _tüketen adaptörler_ denir çünkü onları çağırmak iteratörü kullanır. Bir örnek, `sum` yöntemidir ki bu iteratörün mülkiyetini alır ve öğeler üzerinde `next`'i tekrarlayarak çağırarak yinelendi, böylece iteratörü tüketir. Yinelendiği sırada, her öğeyi bir çalışan toplama ekler ve yineleme tamamlandığında toplamı döner. Kod Listesi 13-13, `sum` yönteminin kullanımını gösteren bir test içerir.

<Listing number="13-13" file-name="src/lib.rs" caption="İteratördeki tüm öğelerin toplamını almak için `sum` yöntemini çağırma">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-13/src/lib.rs:here}}
```

</Listing>

`sum`'a çağrıdan sonra `v1_iter`'i kullanmamıza izin verilmez çünkü `sum`, onu çağırdığımız iteratörün mülkiyetini alır.

### Başka İteratörler Üreten Yöntemler (Methods That Produce Other Iterators)

_İteratör adaptörleri_ `Iterator` trait'i üzerinde tanımlanan yöntemlerdir ki iteratörü tüketmezler. Bunun yerine, özgün iteratörün bazı yönlerini değiştirerek farklı iteratörler üretirler.

Kod Listesi 13-14, öğeler üzerinde yinelendiği sırada her öğeye çağırmak için bir kapanış alan `map` iteratör adaptör yöntemini çağırmanın bir örneğini gösterir. `map` yöntemi değiştirilmiş öğeleri üreten yeni bir iteratör döner. Buradaki kapanış, vektördeki her öğenin 1 ile arttırılacağı yeni bir iteratör oluşturur.

<Listing number="13-14" file-name="src/main.rs" caption="Yeni bir iteratör oluşturmak için `map` iteratör adaptörünü çağırma">

```rust,not_desired_behavior
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-14/src/main.rs:here}}
```

</Listing>

Ancak, bu kod bir uyarı üretir:

```console
{{#include ../listings/ch13-functional-features/listing-13-14/output.txt}}
```

Kod Listesi 13-14'teki kod hiçbir şey yapmaz; belirlediğimiz kapanış hiçbir zaman çağrılmaz. Uyarı bize bunun neden olduğunu hatırlatır: İteratör adaptörleri tembel, ve burada iteratörü tüketmemiz gerekiyor.

Bu uyarıyı düzeltmek ve iteratörü tüketmek için, `collect` yöntemini kullanacağız ki onu Kod Listesi 12-1'de `env::args` ile kullandık. Bu yöntem iteratörü tüketir ve sonuçta değerleri bir koleksiyon veri tipine toplar.

Kod Listesi 13-15'te, `map`'e çağrıdan dönen iteratör üzerinde yinelemeye başlamadan sonuçları bir vektöre topluyoruz. Bu vektör, sonunda özgün vektörden her öğeyi, 1 ile arttırılmış olarak içerecek.

<Listing number="13-15" file-name="src/main.rs" caption="Yeni bir iteratör oluşturmak için `map` yöntemini çağırma, ve sonra yeni iteratörü tüketmek ve bir vektör oluşturmak için `collect` yöntemini çağırma">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-15/src/main.rs:here}}
```

</Listing>

`map` bir kapanış aldığından, her öğede gerçekleştirmek istediğimiz herhangi bir işlemi belirleyebiliriz. Bu, kapanışların `Iterator` trait'in sağladığı yineleme davranışını yeniden kullanarak bazı davranışları nasıl özelleştirmenize izin verdiğini harika bir örnektir.

Okunabilir bir yolla karmaşık eylemleri gerçekleştirmek için iteratör adaptörlerine birden fazla çağrı zincirleyebilirsiniz. Ancak tüm iteratörler tembel olduğundan, iteratör adaptörlerine yapılan çağrılardan sonuçları almak için tüketen adaptör yöntemlerinden birini çağırmak zorundasınız.

<a id="using-closures-that-capture-their-environment"></a>

### Kapsamlarını Yakalayan Kapanışlar (Closures That Capture Their Environment)

Birçok iteratör adaptörü argüman olarak kapanışlar alır ve genellikle iteratör adaptörlerine argüman olarak belirleyeceğimiz kapanışlar, kapsamlarını yakalayan kapanışlar olacak.

Bu örnek için, bir kapanış alan `filter` yöntemini kullanacağız. Kapanış iteratörden bir öğeyi alır ve bir `bool` döner. Eğer kapanış `true` dönerse, değer `filter` tarafından üretilen yinelemeye dahil edilecek. Eğer kapanış `false` dönerse, değer dahil edilmeyecek.

Kod Listesi 13-16'te, `shoe_size` değişkenini kapsamından yakalayan bir kapanışla `filter` kullanıyoruz ki bu `Shoe` struct örneklerinden oluşan bir koleksiyon üzerinde yinelensin. Bu sadece belirtilen boyuttaki ayakkabıları dönecek.

<Listing number="13-16" file-name="src/lib.rs" caption="`shoe_size`'i yakalayan bir kapanışla `filter` yöntemini kullanma">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-16/src/lib.rs}}
```

</Listing>

`shoes_in_size` fonksiyonu, bir ayakkabı vektörünün mülkiyetini ve bir ayakkabı boyutunu parametre olarak alır. Dönen bir vektör sadece belirtilen boyuttaki ayakkabıları içerir.

`shoes_in_size`'in gövdesinde, vektörün mülkiyetini alan bir iteratör oluşturmak için `into_iter`'i çağırıyoruz. Sonra, sadece kapanışın `true` döndürdüğü öğeleri içeren yeni bir iteratöre adapt etmek için `filter`'i çağırıyoruz.

Kapanış kapsamdan `shoe_size` parametresini yakalar ve değeri her bir ayakkabının boyutuyla karşılaştırır, sadece belirtilen boyuttaki ayakkabıları tutuyor. Son olarak, `collect`'i çağırmak adapt edilmiş iteratör tarafından dönen değerleri fonksiyon tarafından dönen bir vektöre toplar.

Test, `shoes_in_size`'i çağırdığımızda, sadece belirlediğimiz değerle aynı boyutta olan ayakkabıları aldığımızı gösteriyor.