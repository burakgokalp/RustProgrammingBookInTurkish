## I/O Projemizi İyileştirme (Improving Our I/O Project)

İteratörler hakkındaki bu yeni bilgiyle, Bölüm 12'deki I/O projemizi kodda bazı yerleri iteratörler kullanarak daha net ve daha kısa yaparak iyileştirebiliriz. İteratörlerin `Config::build` fonksiyonumuzun ve `search` fonksiyonumuzun uygulamasını nasıl iyileştirebileceğine bakalım.

### İteratör Kullanarak Bir `clone` Kaldırma (Removing a `clone` Using an Iterator)

Kod Listesi 12-6'te, `String` değerlerinin bir dilimini alan ve dilime indeksleyip değerleri klonlayarak bir `Config` struct örneği oluşturduktan kod ekledik ki bu, `Config` struct'ının bu değerleri sahip olmasına izin veriyordu. Kod Listesi 13-17'te, Kod Listesi 12-23'teki gibi `Config::build` fonksiyonunun uygulamasını yeniden ürettik.

<Listing number="13-17" file-name="src/main.rs" caption="Kod Listesi 12-23'ten `Config::build` fonksiyonunun yeniden üretimi">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-23-reproduced/src/main.rs:ch13}}
```

</Listing>

O zaman, verimsiz `clone` çağrılarından endişmememizi söyledik çünkü onları gelecekte kaldıracaktık. Evet, o zaman şimdi!

Burada `clone`'a ihtiyacımız vardı çünkü `args` parametresinde `String` elemanları olan bir dilimimiz var ama `build` fonksiyonu `args`'e sahip değil. Bir `Config` örneğinin mülkiyetini dönmek için, `Config` struct'ının kendi değerleri sahip olabilmesi için `query` ve `file_path` alanlarından değerleri klonlamak zorundaydık.

İteratörler hakkındaki yeni bilgimizle, `build` fonksiyonunu bir dilim ödünçlemek yerine bir iteratörün mülkiyetini parametre olarak almak üzere değiştirebiliriz. Dilimin uzunluğunu kontrol eden ve belirli yerlere indeksleyen kod yerine iteratör işlevselliğini kullanacağız. Bu, `Config::build` fonksiyonunun ne yaptığını netleştirecektir çünkü iteratör değerlere erişecek.

Bir kez `Config::build` iteratörün mülkiyetini aldığında ve ödünçleyen indeksleme işlemlerini kullanmayı durduğunda, `clone` çağırıp yeni bir ayırma yapmak yerine `String` değerlerini iteratörden `Config`'e taşıyabiliriz.

#### Döndürülen İteratörü Doğrudan Kullanma (Using the Returned Iterator Directly)

I/O projenizin _src/main.rs_ dosyasını açın, bu şöyle görünmeli:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-24-reproduced/src/main.rs:ch13}}
```

Önce, Kod Listesi 12-24'te sahip olduğumuz `main` fonksiyonunun başlangıcını, bu sefer bir iteratör kullanan kodla değiştireceğiz ki bu Kod Listesi 13-18'de. Bu, `Config::build`'i de güncellemedikçe derlenmeyecektir.

<Listing number="13-18" file-name="src/main.rs" caption="`env::args`'in dönen değerini `Config::build`'e geçme">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-18/src/main.rs:here}}
```

</Listing>

`env::args` fonksiyonu bir iteratör döner! İteratör değerlerini bir vektöre toplamak ve sonra bir dilimi `Config::build`'e geçmek yerine, şimdi `env::args`'den dönen iteratörün mülkiyetini doğrudan `Config::build`'e geçiriyoruz.

Sonra, `Config::build`'in tanımını güncellememiz gerekiyor. `Config::build`'in imzasını Kod Listesi 13-19 gibi görünmesi için değiştirelim. Bu hala derlenmeyecek çünkü fonksiyon gövdesini güncellememiz gerekiyor.

<Listing number="13-19" file-name="src/main.rs" caption="Bir iteratör beklemesi için `Config::build` imzasını güncelleme">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-19/src/main.rs:here}}
```

</Listing>

`env::args` fonksiyonu için standart kütüphane belgesi, döndürülen iteratörün tipinin `std::env::Args` olduğunu ve bu tipin `Iterator` trait'ini uyguladığını ve `String` değerleri döndürdüğünü gösterir.

`Config::build` fonksiyonunun imzasını güncelledik ki `args` parametresi `&[String]` yerine trait sınırları olan `impl Iterator<Item = String>` ile genel bir tipe sahip. `impl Trait` sözdiziminin bu kullanımı, Bölüm 10'de ["Trait'leri Parametre Olarak Kullanma"][impl-trait]<!-- ignore --> bölümünde konuştuğumuz, `args`'in `Iterator` trait'ini uygulayan ve `String` öğelerini döndüren herhangi bir tip olabileceği anlamına gelir.

`args`'in mülkiyetini aldığımızdan ve onu üzerinde yineleyerek `args`'i mutasyon edeceğimizden, `args` parametresinin belirtimine `mut` anahtar kelimesini ekleyerek onu değiştirilebilir yapabiliriz.

<a id="using-iterator-trait-methods-instead-of-indexing"></a>

#### `Iterator` Trait Yöntemlerini Kullanma (Using `Iterator` Trait Methods)

Sonra, `Config::build`'in gövdesini düzelteceğiz. `args` `Iterator` trait'ini uyguladığından, onun üzerinde `next` yöntemini çağırabileceğimizi biliyoruz! Kod Listesi 13-20, `next` yöntemini kullanmak için Kod Listesi 12-23'teki kodu günceller.

<Listing number="13-20" file-name="src/main.rs" caption="`Config::build` gövdesini iteratör yöntemlerini kullanmak için değiştirme">

```rust,ignore,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-20/src/main.rs:here}}
```

</Listing>

`env::args`'in dönen değerindeki ilk değerin programın adı olduğunu hatırlayın. Bunu yoksaymak ve bir sonraki değere ulaşmak istiyoruz, bu yüzden önce `next`'i çağırıp dönen değerle hiçbir şey yapmıyoruz. Sonra, `Config`'in `query` alanına koymak istediğimiz değeri almak için `next`'i çağırıyoruz. Eğer `next` `Some` dönerse, değeri ayıklamak için bir `match` kullanıyoruz. Eğer `None` dönerse, bu, yeterli argüman verilmediği anlamına gelir ve bir `Err` değeriyle erken dönüyoruz. `file_path` değeri için aynı şeyi yapıyoruz.

<a id="making-code-clearer-with-iterator-adapters"></a>

### İteratör Adaptörleriyle Kodu Netleştirme (Clarifying Code with Iterator Adapters)

I/O projemizdeki `search` fonksiyonunda da iteratörlerin avantajından yararlanabiliriz ki bu Kod Listesi 13-21'te Kod Listesi 12-19'teki gibi yeniden üretildi.

<Listing number="13-21" file-name="src/lib.rs" caption="Kod Listesi 12-19'ten `search` fonksiyonunun uygulaması">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:ch13}}
```

</Listing>

Bu kodu iteratör adaptör yöntemleri kullanarak daha kısa bir yolla yazabiliriz. Bunu yapmak ayrıca değiştirilebilir aracı bir `results` vektörü olmağından kaçınmamızı sağlar. Fonksiyonel programlama tarzı kodu netleştirmek için değiştirilebilir durum miktarını en aza indirmeyi tercih eder. Değiştirilebilir durumu kaldırmak gelecekteki bir iyileştirmeyi, aramayı paralellikte olmasını sağlayabilir çünkü `results` vektörüne eşzamanlı erişimi yönetmemiz gerekmeyecekti. Kod Listesi 13-22 bu değişikliği gösterir.

<Listing number="13-22" file-name="src/lib.rs" caption="`search` fonksiyonunun uygulamasında iteratör adaptör yöntemlerini kullanma">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-22/src/lib.rs:here}}
```

</Listing>

`search` fonksiyonunun amacının `contents` içindeki `query` içeren tüm satırları dönmek olduğunu hatırlayın. Bu, Kod Listesi 13-16'teki `filter` örneğine benzer, sadece `line.contains(query)`'nın `true` döndürdüğü satırları tutmak için `filter` adaptörünü kullanır. Sonra `collect` kullanarak eşleşen satırları başka bir vektöre topluyoruz. Çok daha basit! `search_case_insensitive` fonksiyonunda da iteratör yöntemlerini kullanmak için aynı değişikliği yapmaktan çekinmeyin.

Daha fazla bir iyileştirme için, `search` fonksiyonundan bir iteratör dönmek için `collect` çağrısını kaldırıp dönüş tipini `impl Iterator<Item = &'a str>` olarak değiştirerek fonksiyonu bir iteratör adaptörü yapın. Testleri de güncellemeniz gerekeceğini not edin! `minigrep` aracınızı kullanarak büyük bir dosyada bu değişiklikten önce ve sonra arama davranışındaki farkı gözlemek için arama yapın. Bu değişiklikten önce, program tüm sonuçları topladığına kadar herhangi bir sonuç yazdırmayacak ama değişiklikten sonra, sonuçlar her eşleşen satır bulunduğu gibi yazılacak çünkü `run` fonksiyonundaki `for` döngüsü iteratörün tembelliğinden yararlanabilecek.

<a id="choosing-between-loops-or-iterators"></a>

### Döngüler ve İteratörler Arasında Seçim Yapma (Choosing Between Loops and Iterators)

Bir sonraki mantıksal sorusu, kendi kodunuzda hangi tarzı seçmeniz gerektiği ve nedeni: Kod Listesi 13-21'deki orijinal uygulama veya Kod Listesi 13-22'deki iteratör kullanan sürüm (tüm sonuçları dönmekten önce topladığınızı ve döndürdüğümüzü varsayıyoruz). Çoğu Rust programcısı iteratör tarzını kullanmayı tercih eder. Başlangıçta kavraması biraz daha zordur ama bir kez çeşitli iteratör adaptörleri ve onların ne yaptığı için bir hisse alırsanız, iteratörler daha kolay anlaşılabilir olabilir. Döngü oluşturma ve yeni vektörler inşa etme gibi çeşitli bitlerle oynamak yerine, kod döngünün üst düzey amacına odaklanır. Bu, bu koda özgün kavramları, örneğin her elemanın filtreleme koşulunun geçmesi gerektiği gibi, görmeyi kolaylaştırır.

Ama iki uygulama gerçekten eşdeğer mi? Sezgisel varsayım, alt düzey döngünün daha hızlı olacağı olabilir. Performans hakkında konuşalım.

[impl-trait]: ch10-02-traits.html#trait-leri-parametre-olarak-kullanma