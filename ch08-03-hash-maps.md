## Hash Haritalarında İlişkili Değerlerle Anahtarları Saklama (Storing Keys with Associated Values in Hash Maps)

Yaygın koleksiyonlarımızın sonuncusu hash haritadır (hash map). `HashMap<K, V>` tipi `K` tipinin anahtarlarından `V` tipinin değerlerine bir haritalama (mapping) saklar ki bu _hash fonksiyonu_ kullanarak bu anahtarları ve değerleri belleğe nasıl yerleştirdiğini belirler. Çoklu programlama dilleri bu türde veri yapısını destekler ancak genellikle farklı bir isim kullanırlar, örneğin _hash_, _map_, _object_, _hash table_, _dictionary_ veya _associative array_, sadece birkaçını saymak için.

Hash haritalar, vektörlerle yapabileceğiniz gibi indeks kullanarak değil, herhangi bir tipin olabilir bir anahtarı kullanarak veriyi aramak istediğinizde kullanışlıdır. Örneğin, bir oyunuda, her takımın puanını bir hash haritada takip edebilirsiniz ki burada her bir anahtar bir takımın adı ve değerler her takımın puanıdır. Bir takım adı verildiğinde, puanını alabilirsiniz.

Bu bölümde hash haritalarının temel API'si üzerinden geçeceğiz ancak standart kütüphane tarafından `HashMap<K, V>` üzerinde tanımlanmış çok daha fazla kullanışlı yöntemler gizlidir. Her zamanki gibi, daha fazla bilgi için standart kütüphane belgelerine bakın.

### Yeni Hash Haritası Oluşturma (Creating a New Hash Map)

Boş bir hash haritası oluşturmanın bir yolu `new` kullanmak ve `insert` ile öğeler eklemektir. Kod Listesi 8-20'de, adları _Blue_ ve _Yellow_ olan iki takımın puanlarını takip ediyoruz. Mavi takımı 10 puanla başlar ve Sarı takımı 50 ile başlar.

<Listing number="8-20" caption="Yeni bir hash haritası oluşturma ve bazı anahtarlar ve değerler ekleme">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-20/src/main.rs:here}}
```

</Listing>

Standart kütüphanenin koleksiyonlar bölümünden `HashMap`'ı önce `use` etmemiz gerektiğini fark edin. Yaygın koleksiyonlarımızın üçünden, bu en az kullanılan o, bu yüzden otomatik olarak kapsama getirilen özelliklere dahil değildir. Hash haritaların standart kütüphaneden daha az desteği vardır; onları inşa etmek için yerleşik bir makro yoktur, örneğin.

Vektörlerdeki gibi, hash haritalar veriyi yığında saklar. Bu `HashMap`'ın `String` tipinde anahtarları ve `i32` tipinde değerleri vardır. Vektörlerde olduğu gibi, hash haritalar homojendir: Tüm anahtarların aynı tipi olmalı ve tüm değerlerin aynı tipi olmalıdır.

### Bir Hash Haritadaki Değerlere Erişim (Accessing Values in a Hash Map)

Bir hash haritadaki bir değeri anahtarını `get` yöntemine sağlayarak alabiliriz, Kod Listesi 8-21'de gösterildiği gibi.

<Listing number="8-21" caption="Hash haritada saklanan Mavi takımı için puana erişim">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-21/src/main.rs:here}}
```

</Listing>

Burada, `score` Mavi takımıyla ilişkilendirilmiş değere sahip olacak ve sonuç `10` olacak. `get` yöntemi bir `Option<&V>` döner; hash haritada bu anahtar için değer yoksa, `get` `None` döner. Bu program, `Option`'u `copied` çağırarak bir `Option<i32>` elde etmek ve sonra `scores`'in anahtar için bir girdisi yoksa `score`'yi sıfıra ayarlamak için `unwrap_or` çağırarak ele alır.

Bir hash haritada her bir anahtar-değer çifti üzerinde vektörlerle yaptığımız gibi bir `for` döngüsü kullanarak iterasyon yapabiliriz:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-03-iterate-over-hashmap/src/main.rs:here}}
```

Bu kod her çifti keyfi bir sırada yazdırır:

```text
Yellow: 50
Blue: 10
```

<!-- Old headings. Do not remove or links may break. -->

<a id="hash-maps-and-ownership"></a>

### Hash Haritalarda Mülkiyeti Yönetme (Managing Ownership in Hash Maps)

`Copy` trait'ini uygulayan tipler için, örneğin `i32`, değerler hash haritaya kopyalanır. `String` gibi sahip olunan değerler için, değerler taşınacak ve hash harita bu değerlerin sahibi olacak, Kod Listesi 8-22'de gösterildiği gibi.

<Listing number="8-22" caption="Anahtarların ve değerlerin eklendikten sonra hash harita tarafından sahiplenildiğini gösterme">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-22/src/main.rs:here}}
```

</Listing>

`insert` çağrısıyla hash haritaya taşındıktan sonra `field_name` ve `field_value` değişkenlerini kullanamayız.

Eğer hash haritaya değerlere referanslar eklersek, değerler hash haritaya taşınmaz. Referansların işaret ettiği değerler en az hash harita geçerli olduğu kadar geçerli olmalı. Bu sorunları hakkında Bölüm 10'daki ["Referansları Ömürlerle Geçerliğini Doğrulama" (Validating References with Lifetimes)][validating-references-with-lifetimes]<!-- ignore --> bölümünde daha fazlasını konuşacağız.

### Bir Hash Haritası Güncelleme (Updating a Hash Map)

Anahtar ve değer çiftlerinin sayısı büyülebilir olsa da, her benzersiz anahtara bir zamanda sadece tek bir değer ilişkilendirilebilir (ama tersi değil: Örneğin, hem Mavi takımı hem de Sarı takımı `scores` hash haritada depolanmış `10` değerine sahip olabilir).

Bir hash haritadaki veriyi değiştirmek istediğinizde, bir anahtarın zaten atanmış bir değeri olduğunda durumu nasıl ele alacağınızı karar vermeniz gerekir. Eski değeri yeni değerle değiştirebilirsiniz, eski değeri tamamen ihmal ederek. Eski değeri tutabilir ve yeni değeri ihmal edebilirsiniz, sadece yeni değeri eğer anahtar _yoksa_ değerle ekleyebilirsiniz. Veya eski değeri ve yeni değeri birleştirebilirsiniz. Her birini nasıl yapacağımıza bakalım!

#### Bir Değeri Geçersiz Kılma (Overwriting a Value)

Eğer bir hash haritaya bir anahtar ve bir değer ekleriz ve sonra aynı anahtarı farklı bir değerle ekleriz, o anahtarla ilişkilendirilmiş değer değiştirilecek. Kod Listesi 8-23 `insert`'i iki kere çağırmasına rağmen, hash harita sadece bir anahtar-değer çifti içerecek çünkü Mavi takımının anahtarı için değeri her iki kere ekliyoruz.

<Listing number="8-23" caption="Belirli bir anahtarla saklanan bir değeri değiştirme">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-23/src/main.rs:here}}
```

</Listing>

Bu kod `{"Blue": 25}` yazdıracak. `10`'un orijinal değeri geçersiz kılındı.

<!-- Old headings. Do not remove or links may break. -->

<a id="only-inserting-a-value-if-the-key-has-no-value"></a>

#### Bir Anahtar Değer Sahip Değilse Sadece Anahtar ve Değer Ekleme (Adding a Key and Value Only If a Key Isn't Present)

Belirli bir anahtarın hash haritada zaten değeri olup olmadığını kontrol etmek ve sonra şu eylemleri almak yaygındır: Eğer anahtar hash haritada varsa, mevcut değer olduğu şekilde kalmalı; eğer anahtar yoksa, onu ve değerini eklemeli.

Hash haritaların buna özel bir API'si vardır ki `entry` denir ve kontrol etmek istediğiniz anahtarı parametre olarak alır. `entry` yönteminin dönüş değeri muhtemelen mevcut olan veya olmayan bir değeri temsil eden `Entry` adında bir enum'dir. Sarı takımının anahtarının bir değeriyle ilişkilendirilmiş olup olmadığını kontrol etmek istediğimizi söyleyin. Eğer yoksa, `50` değerini eklemek istiyoruz ve Mavi takımı için de aynı. `entry` API'sini kullanarak, kod Kod Listesi 8-24 gibi görünecek.

<Listing number="8-24" caption="Anahtarın zaten bir değeri yoksa sadece eklemek için `entry` yöntemini kullanma">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-24/src/main.rs:here}}
```

</Listing>

`Entry` üzerindeki `or_insert` yöntemi, karşılık gelen `Entry` anahtarı için eğer anahtar varsa o anahtarın değeri için değiştirilebilir referans dönmek ve eğer yoksa, parametreyi bu anahtar için yeni değer olarak eklemek ve yeni değere değiştirilebilir referans dönmek üzere tanımlanmıştır. Bu teknik mantığı kendimiz yazmaktan çok daha temizdir ve ek olarak, ödünç kontrolcüsüyle daha iyi çalışır.

Kod Listesi 8-24'ü çalıştırmak `{"Yellow": 50, "Blue": 10}` yazdırır. `entry`'ye ilk çağrı, Sarı takımının `50` değeriyle anahtarı ekleyecek çünkü Sarı takımı zaten bir değere sahip değil. `entry`'ye ikinci çağrı hash haritayı değiştirmeyecek çünkü Mavi takımı zaten `10` değerine sahip.

#### Eski Değere Dayalı Bir Değeri Güncelleme (Updating a Value Based on Old Value)

Hash haritalar için başka bir yaygın kullanım durumu bir anahtarın değerini aramak ve sonra eski değere dayalı olarak güncellemektir. Örneğin, Kod Listesi 8-25, bazı metinde her kelimenin kaç kez göründüğünü sayan kod gösterir. Kelimeler anahtarları olarak ve o kelimeyi kaç kez gördüğümüzü takip etmek için artırmak için değer kullanan bir hash harita kullanıyoruz. Eğer bir kelimeyi gördüğümüz ilk kereyse, önce `0` değerini ekleyeceğiz.

<Listing number="8-25" caption="Kelimeler ve sayıları saklayan bir hash harita kullanarak kelimelerin sayısını sayma">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-25/src/main.rs:here}}
```

</Listing>

Bu kod `{"world": 2, "hello": 1, "wonderful": 1}` yazdıracak. Aynı anahtar-değer çiftlerini farklı bir sırada yazdırmış görebilirsiniz: ["Bir Hash Haritadaki Değerlere Erişim" (Accessing Values in a Hash Map)][access]<!-- ignore --> bölümünden hatırlayın ki bir hash haritada iterasyon keyfi bir sırada gerçekleşir.

`split_whitespace` yöntemi, `text` içindeki değerin boşluklarla ayrılmış alt dilimleri üzerinde bir iterator döner. `or_insert` yöntemi belirtilen anahtar için değere değiştirilebilir referans (`&mut V`) döner. Burada, değiştirilebilir referansı `count` değişkeninde saklıyoruz, bu yüzden o değere atamak için önce asterisk (`*`) kullanarak `count`'i referans kaldırmamız gerekir. Değiştirilebilir referans, `for` döngüsünün sonunda kapsamadan çıkar, bu yüzden tüm bu değişiklikler güvenlidir ve ödünç kuralları tarafından izin verilir.

### Hash Fonksiyonları (Hashing Functions)

Varsayılan olarak, `HashMap` _SipHash_ adında bir hash fonksiyonu kullanır ki hash tablolarını içeren hizmet reddi (DoS) saldırılarına karşı direnç sağlayabilir[^siphash]<!-- ignore -->. Bu mevcut olan en hızlı hash algoritması değildir ancak performansın düşüşüyle gelen daha iyi güvenlik değiş-değişi buna değer. Kodunuzu profile edersen ve varsayılan hash fonksiyonun sizin amaçlarınız için çok yavaş bulursanız, farklı bir hasher belirterek başka bir fonksiyona geçebilirsiniz. Bir _hasher_, `BuildHasher` trait'ini uygulayan bir tiptir. Trait'leri ve onları nasıl uygulayacağımızı Bölüm 10'de konuşacağız.[Chapter 10][traits]<!-- ignore --> Kendi hasher'ınızı sıfırdan uygulamak zorunda değilsiniz; [crates.io](https://crates.io/)<!-- ignore --> adresinde, çok yaygın hash algoritmaları uygulayan hasher'ları sağlayan diğer Rust kullanıcıları tarafından paylaşılan kütüphaneler vardır.

[^siphash]: [https://en.wikipedia.org/wiki/SipHash](https://en.wikipedia.org/wiki/SipHash)

## Özet

Vektörler, stringler ve hash haritalar programlarda veriyi saklamak, erişmek ve değiştirmek için gereken çok sayıda işlevsellik sağlar. Şimdi çözmeniz için donanımlı olmanız gereken bazı egzersizler:

1. Tamsayı listesine verildiğinde, bir vektör kullanın ve ortalamayı (sıralandığında, orta
   pozisyondaki değer) ve modunu (en sık görünen değer; bir hash harita burada kullanışlı
   olacak) döner.
2. Stringleri Pig Latin'e çevirin. Her kelimenin ilk ünsesi kelimenin sonuna taşınır ve _ay_
   eklenir, bu yüzden _first_ _irst-fay_ olur. Ünlü ile başlayan kelimeler sonuna _hay_
   eklenir (_apple_ _apple-hay_ olur). UTF-8 kodlaması detaylarını akılda tutun!
3. Bir hash harita ve vektörleri kullanarak, bir kullanıcıya bir şirketin bir bölümüne çalışan
   adlarını eklemesine izin veren bir metin arabirim oluşturun; örneğin, "Add Sally to
   Engineering" veya "Add Amir to Sales." Sonra, kullanıcıya bir bölümdeki tüm kişilerin veya
   departmana göre tüm kişilerin alfabetik sıralanmış listesini almasına izin verin.

Standart kütüphane API belgeleri vektörlerin, stringlerin ve hash haritaların sahip olduğu, bu egzersizler için kullanışlı olacak yöntemleri tarif eder!

İşlemlerin başarısız olabileceği daha karmaşık programlara giriyoruz, bu yüzden hata ele almayı konuşmak için mükemmel bir zaman. Şimdi bunu yapacağız!

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[access]: #accessing-values-in-a-hash-map
[traits]: ch10-02-traits.html