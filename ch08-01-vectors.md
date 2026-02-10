## Vektörlerle Değer Listelerini Saklama (Storing Lists of Values with Vectors)

Bakacağımız ilk koleksiyon tipi `Vec<T>`'dir, ayrıca vektör olarak bilinir. Vektörler size tek bir veri yapısında birden fazla değeri saklamanızı sağlar ki tüm değleri bellekte birbirinin yanına koyar. Vektörler sadece aynı tipin değerlerini saklayabilirler. Bir dosyadaki metin satırları veya bir alışveriş sepetindeki öğelerin fiyatları gibi öğelerin bir listesine sahip olduğunuzda kullanışlıdırlar.

### Yeni Vektör Oluşturma (Creating a New Vector)

Yeni, boş bir vektör oluşturmak için, Kod Listesi 8-1'de gösterildiği gibi `Vec::new` fonksiyonunu çağırız.

<Listing number="8-1" caption="`i32` tipin değerlerini tutmak için yeni, boş bir vektör oluşturma">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-01/src/main.rs:here}}
```

</Listing>

Burada bir tip notasyonu (type annotation) eklediğimizi fark edin. Bu vektöre hiç değer yerleştirmediğimiz için, hangi türde öğeler saklayacağımızı Rust bilmiyor. Bu önemli bir nokta. Vektörler genel tipler (generics) kullanılarak uygulanır; kendi tiplerinizle genel tipleri Bölüm 10'de nasıl kullanacağımızı kaplayacağız. Şu an için, standart kütüphane tarafından sağlanan `Vec<T>` tipinin herhangi bir tipi tutabileceğini bilin. Belirli bir tipi tutacak bir vektör oluşturduğumuzda, açısal köşeler içinde tip belirtebiliriz. Kod Listesi 8-1'de, `v` içindeki `Vec<T>`'nin `i32` tipinin öğelerini tutacağını Rust'a söyledik.

Daha sık, ilk değerleriyle bir `Vec<T>` oluşturacaksınız ve Rust saklamak istediğiniz değerin tipini çıkaracak, bu yüzden bu tip notasyonunu nadiren yapmanız gerekir. Rust rahatça `vec!` makrosunu sağlar ki bu size verdiğiniz değerleri tutan yeni bir vektör oluşturur. Kod Listesi 8-2, `1`, `2` ve `3` değerlerini tutan yeni bir `Vec<i32>` oluşturur. Tamsayı tipi `i32`'dir çünkü bu varsayılan tamsayı tipidir, Bölüm 3'ün ["Veri Tipleri" (Data Types)][data-types]<!-- ignore --> bölümünde tartıştığımız gibi.

<Listing number="8-2" caption="Değerleri tutan yeni bir vektör oluşturma">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-02/src/main.rs:here}}
```

</Listing>

İlk `i32` değerlerini verdiğimiz için, `v`'nin tipinin `Vec<i32>` olduğunu Rust çıkarabilir ve tip notasyonu gerekli değildir. Sonra, bir vektörü nasıl değiştireceğimizi bakacağız.

### Bir Vektörü Güncelleme (Updating a Vector)

Bir vektör oluşturmak ve sonra öğeler eklemek için, Kod Listesi 8-3'te gösterildiği gibi `push` yöntemini kullanabiliriz.

<Listing number="8-3" caption="Bir vektöre değer eklemek için `push` yöntemini kullanma">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-03/src/main.rs:here}}
```

</Listing>

Herhangi bir değişken gibi, değerini değiştirebilmek istiyorsak, Bölüm 3'te tartıştığımız gibi `mut` anahtar kelimesini kullanarak değiştirilebilir yapmalısınız. İçerine koyduğumuz sayıların tamamı `i32` tipindedir ve Rust bunu veriden çıkarır, bu yüzden `Vec<i32>` notasyonuna gerek yoktur.

### Vektörlerin Öğelerini Okuma (Reading Elements of Vectors)

Bir vektörde saklanmış bir değere atıfta bulunmanın iki yolu vardır: indeksleme (indexing) veya `get` yöntemini kullanma. Aşağıdaki örneklerde, fazladan netlik için bu fonksiyonlardan dönen değerlerin tipini not ettik.

Kod Listesi 8-4, indeksleme sözdizimi (syntax) ve `get` yöntemi ile bir vektördeki bir değere erişmenin her iki yöntemini gösterir.

<Listing number="8-4" caption="Bir vektördeki öğeye erişmek için indeksleme sözdizimini ve `get` yöntemini kullanma">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-04/src/main.rs:here}}
```

</Listing>

Burada birkaç detayı fark edin. Vektörler sayı ile indekslendiği ve sıfırdan başladığı için üçüncü öğeyi almak için `2` indeks değerini kullanıyoruz. `&` ve `[]` kullanmak bize indeks değerindeki öğeye bir referans verir. `get` yöntemini bir argüman olarak geçirilen indeks ile kullandığımızda, `match` ile kullanabileceğimiz bir `Option<&T>` alırız.

Rust size bir öğeye atıfta bulunmak için bu iki yolu sağlar ki böylece mevcut öğelerin aralığının dışındaki bir indeks değerini kullanmaya çalıştığınızda programın nasıl davranacağını seçebilirsiniz. Örnek olarak, beş öğeli bir vektörümüz olduğunda ve sonra her tekniği kullanarak indeks 100'deki bir öğeye erişmeye çalıştığımızda ne olduğunu görelim, Kod Listesi 8-5'te gösterildiği gibi.

<Listing number="8-5" caption="Beş öğeli bir vektörde indeks 100'deki öğeye erişmeye çalışma">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-05/src/main.rs:here}}
```

</Listing>

Bu kodu çalıştırdığımızda, ilk `[]` yöntemi programın paniklemesine (panic) neden olacaktır çünkü var olmayan bir öğeye atıfta bulunur. Bu yöntem, vektörün sonundan geçen bir öğeye erişme denemesi varsa programın çökmesini istediğinizda en iyidir.

`get` yöntemine vektörün dışındaki bir indeks geçirildiğinde, paniklemeden `None` döner. Normal şartlar altında vektörün aralığının dışındaki bir öğeye erişmenin ara sıra olabileceği bu yöntemi kullanırdınız. Kodunuz daha sonra Bölüm 6'da tartıştığımız gibi `Some(&öğe)` veya `None` olduğuna sahip mantıkla ele almalıdır. Örneğin, indeks bir kişinin bir sayı girmesinden gelebilir. Eğer yanlışlıkla çok büyük bir sayı girerler ve program `None` değeri alırsa, kullanıcıya mevcut vektörde kaç öğe olduğunu söyleyip geçerli bir değer girmeleri için başka bir şans verebilirsiniz. Bu, bir yazım hatası nedeniyle programın çökmesinden daha kullanıcı dostudur!

Program geçerli bir referansa sahiptir, ödünç kontrolcüsü bu referansın ve vektörün içeriğindeki diğer tüm referansların geçerli kalmasını sağlamak için mülkiyet ve ödünç kurallarını (Bölüm 4'te kaplanan) uygular. Aynı kapsamada değiştirilebilir ve değişmez referanslara sahip olamayacağınız kuralını hatırlayın. Bu kural Kod Listesi 8-6'da uygulanır ki burada bir vektördeki ilk öğeye değişmez bir referans tutuyoruz ve sonuna bir öğe eklemeye çalışıyoruz. Fonksiyonun sonrasında bu öğeye atıfta bulunmaya da çalışırsak bu program çalışmaz.

<Listing number="8-6" caption="Bir öğeye referans tutarken bir vektöre bir öğe ekleme denemesi">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-06/src/main.rs:here}}
```

</Listing>

Bu kodu derlemek bu hatayı sonuçlandıracaktır:

```console
{{#include ../listings/ch08-common-collections/listing-08-06/output.txt}}
```

Kod Listesi 8-6'deki kod çalışması gerektiğini görünüyor: İlk öğeye referansın vektörün sonundaki değişikliklerle neden ilgilenmesi gerekir mi? Bu hata vektörlerin çalışma yolundan kaynaklanır: Çünkü vektörler bellekte değerleri birbirinin yanına koyar, vektörün sonuna yeni bir öğe eklemek, vektörün şu an depolandığı yerde tüm öğeleri birbirinin yanına koymak için yeterli yer yoksa, yeni bellek ayırmayı ve eski öğeleri yeni alana kopyalamayı gerektirebilir. Bu durumda, ilk öğeye referans tahsis edilmiş belleki işaret edecektir. Ödünç kuralları programların bu durumla sonlanmasını önler.

> Not: `Vec<T>` tipinin uygulanma detayları hakkında daha fazlası için, ["Rustonomicon" (The Rustonomicon)][nomicon]<!-- ignore --> bölümüne bakın.

### Vektördeki Değerler Üzerinde İterasyon (Iterating Over Values in a Vector)

Bir vektördeki her öğeye sırayla erişmek için, tek tekine erişmek için indeksleri kullanmak yerine tüm öğeler aracılığı iterasyon yapacağız. Kod Listesi 8-7, `i32` değerleri vektöründeki her öğeye değişmez referanslar almak ve yazdırmak için bir `for` döngüsünü nasıl kullanacağımızı gösterir.

<Listing number="8-7" caption="Bir `for` döngüsünü kullanarak bir vektördeki her öğeyi yazdırma">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-07/src/main.rs:here}}
```

</Listing>

Ayrıca bir değiştirilebilir vektördeki her öğeye değiştirilebilir referanslar aracılığı iterasyon yapabiliriz ki tüm öğeleri değiştirmemizi sağlar. Kod Listesi 8-8'deki `for` döngüsü her öğeye `50` ekleyecek.

<Listing number="8-8" caption="Bir vektördeki öğelere değiştirilebilir referanslar aracılığı iterasyon">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-08/src/main.rs:here}}
```

</Listing>

Değiştirilebilir referansın atıfta bulduğu değeri değiştirmek için, `+=` operatörünü kullanmadan önce `i` içindeki değere ulaşmak için `*` referans kaldırma (dereference) operatörünü kullanmamız gerekir. Referans kaldırma operatörü hakkında Bölüm 15'in ["Değeri Takip Eden Referans" (Following the Reference to a Value)][deref]<!-- ignore --> bölümünde daha fazlası konuşacağız.

Vektörü değişmez veya değiştirilebilir olarak iterasyon yapmak güvenlidir çünkü ödünç kontrolcüsünün kuralları sayesinde. Kod Listesi 8-7 ve Kod Listesi 8-8'deki `for` döngüsü gövdelerinde öğeler eklemeye veya kaldırmaya çalışsaydık, Kod Listesi 8-6'deki kodla aldığımız hataya benzeyen bir derleyici hatası alırdık. `for` döngüsü tarafından tutulan vektör referansı tüm vektörün eş zamanlı değiştirilmesini önler.

### Çoklu Tipleri Saklamak İçin Bir Enum Kullanma (Using an Enum to Store Multiple Types)

Vektörler sadece aynı tipin değerlerini saklayabilirler. Bu elverişsiz olabilir; farklı tipin öğeler listesini saklamak için kesin kullanım durumları vardır. Şanslıyız ki bir enumin varyantları aynı enum tipi altında tanımlanır, bu yüzden farklı tipin öğelerini temsil etmek için bir tip gerektiğimizde bir enum tanımlayıp kullanabiliriz!

Örneğin, bir tablonun bir satırından değerler almak istediğimizi söyleyin ki satırdaki bazı sütunlar tamsayılar, bazı kayan noktalı sayılar ve bazı dizeler içerir. Farklı değer tiplerini tutan varyantları olan bir enum tanımlayabiliriz ve tüm enum varyantları aynı tip olarak düşünülür: o enumin. Sonra, o enumi tutmak için bir vektör oluşturabiliriz ve nihayetinde farklı tipleri tutabiliriz. Bunu Kod Listesi 8-9'de gösterdik.

<Listing number="8-9" caption="Bir vektörde farklı tiplerin değerlerini saklamak için bir enum tanımlama">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-09/src/main.rs:here}}
```

</Listing>

Rust'ın derleme zamanında vektörde hangi tipler olacağını bilmesi gerekir ki her öğeyi saklamak için yığında ne kadar belleğin gerektiğini tam olarak bilsin. Ayrıca bu vektörde hangi tiplerin izin verildiği konusunda açık olmalıyız. Eğer Rust'un bir vektörün herhangi bir tipi tutmasına izin verseydi, vektördeki öğeler üzerinde yapılan operasyonlardan bir veya daha fazla tipin hataya neden olma şansı olurdu. Bir enum artı bir `match` ifadesi kullanmak, Rust'ın Bölüm 6'da tartıştığımız gibi derleme zamanında her olası durumu ele almasını sağlar.

Bir vektörde saklamak için bir programın çalışma zamanında alacağı tiplerin kapsamlı setini bilmiyorsanız, enum tekniği çalışmaz. Bunun yerine, bir trait nesnesi (trait object) kullanabilirsiniz ki bunu Bölüm 18'de kaplayacağız.

Vektörleri kullanmanın en yaygın yollarından bazılarını tartıştığımıza göre, standart kütüphane tarafından `Vec<T>` üzerinde tanımlanan çoklu kullanışlı yöntemler için tüm [API belgelerini][vec-api]<!-- ignore --> gözden geçtiğinizden emin olun. Örneğin, `push`'a ek olarak, `pop` yöntemi son öğeyi kaldırır ve döner.

### Bir Vektörü Düşürmek Ögelerini Düşürür (Dropping a Vector Drops Its Elements)

Herhangi diğer `struct` gibi, vektör kapsamadan çıktığında serbest bırakılır, Kod Listesi 8-10'de not edildiği gibi.

<Listing number="8-10" caption="Vektörün ve öğelerinin nerede düşürüldüğünü gösterme">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-10/src/main.rs:here}}
```

</Listing>

Vektör düştürüldüğinde, tüm içeriği de düşürülmüş olur ki bu, tuttuğu tamsayıların temizleneceği anlamına gelir. Ödünç kontrolcüsü, vektörün içeriğine olan tüm referansların sadece vektörün kendisi geçerliyken kullanıldığını sağlar.

Şimdi sonraki koleksiyon tipine geçelim: `String`!

[data-types]: ch03-02-data-types.html#data-types
[nomicon]: ../nomicon/vec/vec.html
[vec-api]: ../std/vec/struct.Vec.html
[deref]: ch15-02-deref.html#following-the-pointer-to-the-value-with-the-dereference-operator