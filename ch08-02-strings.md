## UTF-8 Kodlanmış Metinleri Stringlerle Saklama (Storing UTF-8 Encoded Text with Strings)

Bölüm 4'te stringler hakkında konuşmuştuk ancak şimdi onları daha derinlikte bakacağız. Yeni Rustaceans genellikle stringlerde üç sebebin birleşiminden takılıp kalırlar: Rust'ın olası hataları açma eğilimi, stringlerin birçok programcının düşündüğünden daha karmaşık veri yapısını olması ve UTF-8. Bu faktörler bir araya çalışabilir ki diğer programlama dillerinden geldiğinizde zor görünebilir.

Stringleri koleksiyonlar bağlamında konuşuyoruz çünkü stringler baytların bir koleksiyonu olarak uygulanır ve bu baytlar metin olarak yorumlandığında kullanışlı işlevselliği sağlayan bazı yöntemlerle birlikte bulunur. Bu bölümde, her koleksiyon tipinin sahip olduğu `String` üzerindeki operasyonları konuşacağız, örneğin oluşturma, güncelleme ve okuma. Ayrıca `String`'in diğer koleksiyonlardan hangi şekillerde farklı olduğunu tartışacağız, yani `String`'e indekslemenin insanlar ve bilgisayarların `String` verisini yorumlaması arasındaki farklar nedeniyle nasıl karmaşık olduğunu.

<!-- Old headings. Do not remove or links may break. -->

<a id="what-is-a-string"></a>

### Stringleri Tanımlama (Defining Strings)

Önce _string_ terimiyle ne kastettiğimizi tanımlayalım. Rust'ın çekir dilinde tek bir string tipi vardır ki bu dize dilimi `str`'dir ve genellikle ödünç formunda `&str` olarak görülür. Bölüm 4'te, string dilimleri (string slices) hakkında konuşmuştuk ki bunlar başka yerlerde saklanmış bazı UTF-8 kodlanmış string verilerine referanslardır. Örneğin, string literaller, programın ikili kodunda saklanırlar ve bu yüzden string dilimlerdir.

`String` tipi, çekir dile kodlanmak yerine Rust'ın standart kütüphanesi tarafından sağlanır, büyülebilir, değiştirilebilir, sahip olunan, UTF-8 kodlanmış string tipidir. Rustaceanlar Rust'ta "stringler" dediğinde, sadece bu tiplerden birini değil, hem `String` hem de string dilimi `&str` tiplerine atıfta bulunabilirler. Bu bölüm çoğunlukla `String` hakkındadır ancak her iki tip de Rust'ın standart kütüphanesinde ağırlık kullanılır ve hem `String` hem de string dilimleri UTF-8 kodlanmıştır.

### Yeni String Oluşturma (Creating a New String)

`Vec<T>` ile mevcut olan aynı operasyonların çoğu `String` ile de mevcuttur çünkü `String` aslında bazı ekstra garantilere, kısıtlamalara ve yeteneklere sahip bir bayt vektörünün etrafına bir sarmalayıcı (wrapper) olarak uygulanmıştır. Örneğin, `Vec<T>` ve `String` ile aynı şekilde çalışan bir fonksiyon olan `new` fonksiyonu, Kod Listesi 8-11'de gösterildiği gibi.

<Listing number="8-11" caption="Yeni, boş bir `String` oluşturma">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-11/src/main.rs:here}}
```

</Listing>

Bu satır `s` adında yeni, boş bir string oluşturur ve sonra içine veri yükleyebiliriz. Sıkça, stringi başlatmak isteyeceğimiz bazı ilk verilerimiz olur. Bunun için, string literallerinin yaptığı gibi `Display` trait'ini uygulayan herhangi bir tipte mevcut olan `to_string` yöntemini kullanabiliriz. Kod Listesi 8-12 iki örnek gösterir.

<Listing number="8-12" caption="Bir string literalden `String` oluşturmak için `to_string` yöntemini kullanma">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-12/src/main.rs:here}}
```

</Listing>

Bu kod "initial contents" içeren bir string oluşturur.

Ayrıca bir string literalden `String` oluşturmak için `String::from` fonksiyonunu kullanabiliriz. Kod Listesi 8-13'teki kod, Kod Listesi 8-12'deki kodun eşdeğeridir ki `to_string` kullanır.

<Listing number="8-13" caption="Bir string literalden `String` oluşturmak için `String::from` fonksiyonunu kullanma">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-13/src/main.rs:here}}
```

</Listing>

Çünkü stringler çok şey için kullanılır, çok farklı genel string API'lerini kullanabiliriz ki size çok fazla seçenek sağlar. Bazıları gereksiz görünebilir ancak hepsinin yeri vardır! Bu durumda, `String::from` ve `to_string` aynı şeyi yapar, bu yüzden hangisini seçtiğiniz stil ve okunabilirlik meselesidir.

Fark edin ki stringler UTF-8 kodlanmıştır, bu yüzden içlerinde doğru şekilde kodlanmış herhangi bir veriyi dahil edebiliriz, Kod Listesi 8-14'te gösterildiği gibi.

<Listing number="8-14" caption="Farklı dillerdeki selamları stringlerde saklama">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:here}}
```

</Listing>

Bunların tamamı geçerli `String` değerleridir.

### Bir String Güncelleme (Updating a String)

Bir `String` boyutta büyüyebilir ve içeriği değişebilir, içerikleri `Vec<T>` gibi, ona daha fazla veri iterirseniz. Ek olarak, `String` değerlerini birleştirmek için rahatça `+` operatörünü veya `format!` makrosunu kullanabiliriz.

<!-- Old headings. Do not remove or links may break. -->

<a id="appending-to-a-string-with-push_str-and-push"></a>

#### `push_str` veya `push` ile Eklemek (Appending with `push_str` or `push`)

Bir string dilimini eklemek için `push_str` yöntemini kullanarak bir `String` büyütebiliriz, Kod Listesi 8-15'te gösterildiği gibi.

<Listing number="8-15" caption="Bir string dilimini `String`'e eklemek için `push_str` yöntemini kullanma">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-15/src/main.rs:here}}
```

</Listing>

Bu iki satır sonrasında, `s` `foobar` içerecek. `push_str` yöntemi bir string dilimi alır çünkü parametrenin mülkiyetini almak zorunca değil istemiyoruz. Örneğin, Kod Listesi 8-16'deki kodda, içeriğini `s1`'e ekledikten sonra `s2`'yi kullanabilmek istiyoruz.

<Listing number="8-16" caption="Bir `String`'e içeriğini ekledikten sonra bir string dilimi kullanma">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-16/src/main.rs:here}}
```

</Listing>

Eğer `push_str` yöntemi `s2`'nin mülkiyetini alsaydı, son satırda değerini yazdıramazdık. Ancak, bu kod beklediğimiz gibi çalışır!

`push` yöntemi tek bir karakteri parametre olarak alır ve onu `String`'e ekler. Kod Listesi 8-17, `push` yöntemini kullanarak bir `String`'e _l_ harfini ekler.

<Listing number="8-17" caption="Bir `String` değerine tek bir karakter eklemek için `push` kullanma">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-17/src/main.rs:here}}
```

</Listing>

Sonuç olarak, `s` `lol` içerecek.

<!-- Old headings. Do not remove or links may break. -->

<a id="concatenation-with-the--operator-or-the-format-macro"></a>

#### `+` veya `format!` ile Birleştirme (Concatenating with `+` or `format!`)

Sıkça, iki mevcut stringi birleştirmek isteyeceksiniz. Bunu yapmanın bir yolu `+` operatörünü kullanmaktır, Kod Listesi 8-18'de gösterildiği gibi.

<Listing number="8-18" caption="İki `String` değerini `+` operatörünü kullanarak yeni bir `String` değerinde birleştirme">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-18/src/main.rs:here}}
```

</Listing>

`string `s3` `Hello, world!` içerecek. `s1`'in eklemeden sonra geçerli olmamasının ve `s2`'ye referans kullandığımızın sebebi, `+` operatörünü kullanırdığımızda çağrılan yöntemin imzasıyla ilgilidir. `+` operatörü `add` yöntemini kullanır ki imzası şuna benzeyebilir:

```rust,ignore
fn add(self, s: &str) -> String {
```

Standart kütüphanede, genel tipler ve ilişkili tipler kullanılarak tanımlanmış `add`'i göreceksiniz. Burada, somut tiplerle değiştirmişiz ki bu biz `String` değerleriyle bu yöntemi çağırdığımızda olan şeydir. Bölüm 10'de genel tipleri konuşacağız. Bu imza bize `+` operatörünün zor noktalarını anlamak için ipuçları verir.

İlk, `s2` bir `&`'ye sahiptir ki bu ikinci stringin bir referansını ilk stringe ekliyoruz. Bu, `add` fonksiyonundaki `s` parametresinin sebebidir: Bir `String`'e sadece bir string dilimi ekleyebiliriz; iki `String` değerini birleştiremeyiz. Ama bekle - `&s2`'nin tipi `&String`'dir, `add` fonksiyonunun ikinci parametresinde belirtilen `&str` değil. O zaman Kod Listesi 8-18 neden derlenir?

`&s2`'i `add` çağrısında kullanabildiğimizin sebebi, derleyicinin `&String` argümanını bir `&str`'ye zorlayabilmesidir. `add` yöntemini çağırdığımızda, Rust referans zorlaması kullanır ki burada `&s2`'yi `&s2[..]`'e çevirir. Referans zorlamasını Bölüm 15'te daha derinlikte konuşacağız. Çünkü `add`, `s` parametresinin mülkiyetini almaz, `s2` bu operasyondan sonra hala geçerli bir `String` olacaktır.

İkinci, imzada `add`'in `self`'in mülkiyetini aldığını görebiliriz çünkü `self`'in bir `&`'si _yoktur_. Bu, Kod Listesi 8-18'deki `s1`'in `add` çağrısına taşınacağını ve o andan sonra geçerli olmayacağını anlamına gelir. Bu yüzden, `let s3 = s1 + &s2;` her iki stringi kopyalayıp yeni bir tane oluşturmuş gibi görünse de, bu ifade aslında `s1`'in mülkiyetini alır, `s2`'nin içeriğinin bir kopyasını ekler ve sonrasında sonucun mülkiyetini döner. Başka bir deyimle, çok fazla kopya yapmış gibi görünse de aslında değildir; uygulanması kopyalamaktan daha verimlidir.

Eğer çoklu stringleri birleştirmemiz gerekirse, `+` operatörünün davranışı kontrolsüzleşir:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-01-concat-multiple-strings/src/main.rs:here}}
```

Bu noktada, `s` `tic-tac-toe` olacaktır. Tüm `+` ve `"` karakterleriyle, ne olduğu görmek zordur. Stringleri daha karmaşık şekillerde birleştirmek için, bunun yerine `format!` makrosunu kullanabiliriz:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-02-format/src/main.rs:here}}
```

Bu kod da `s`'yi `tic-tac-toe` olarak ayarlar. `format!` makrosu `println!` gibi çalışır ancak ekrana çıktı yazmak yerine, içeriği olan bir `String` döner. `format!` makrosu kullanan kodun versiyonu çok daha okunakabilirdir ve `format!` makrosu tarafından oluşturulan kod referansları kullandığından bu çağrı hiçbir parametrenin mülkiyetini almaz.

### Stringlere İndekleme (Indexing into Strings)

Birçok başka programlama dilinde, bir stringdeki bireysel karakterlere indeksleme atıfta bulunmak geçerli ve yaygın bir operasyondur. Ancak, Rust'te indeksleme sözdizimini kullanarak bir `String`'in parçalarına erişmeye çalışırsanız, bir hata alırsınız. Kod Listesi 8-19'deki geçersiz kodu düşünün.

<Listing number="8-19" caption="Bir `String` ile indeksleme sözdizimini kullanmaya çalışma">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-19/src/main.rs:here}}
```

</Listing>

Bu kodu aşağıdaki hatayı sonuçlandıracaktır:

```console
{{#include ../listings/ch08-common-collections/listing-08-19/output.txt}}
```

Hata bize hikayeyi anlatır: Rust stringleri indekslemeyi desteklemez. Ancak neden desteklemiyor? Bu soruyu cevaplamak için, Rust'ın bellekte stringleri nasıl sakladığını tartışmamız gerekir.

#### İç Gösterimi (Internal Representation)

Bir `String` bir `Vec<u8>` üzerindeki bir sarmalayıcıdır. Kod Listesi 8-14'teki doğru şekilde UTF-8 kodlanmış string örneklerimizden bazılarına bakalım. İlk, bu tane:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:spanish}}
```

Bu durumda, `len` 4 olacak ki bu `Hola"` stringini saklayan vektörün 4 bayt uzunluğunda olduğu anlamına gelir. Bu harflerden her biri UTF-8'de kodlandığında 1 bayt alır. Ancak şu satır sizi şaşırtabilir (bu stringin büyük Kiril harfı _Ze_ ile başladığını ve sayı 3 olmadığını fark edin):

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:russian}}
```

Eğer size stringin ne kadar uzun olduğunu sorsanız, 12 diyebilirsiniz. Aslında, Rust'ın cevabı 24'tür: Bu, "Здравствуйте" stringini UTF-8'de kodlamak için gereken bayt sayısıdır çünkü bu stringteki her Unicode skaler değeri depolamak için 2 bayt depolama alır. Bu nedenle, bir stringin baytlarına indeksleme her zaman geçerli bir Unicode skaler değeriyle eşleşmeyecektir. Bunu göstermek için, şu geçersiz Rust kodunu düşünün:

```rust,ignore,does_not_compile
let hello = "Здравствуйте";
let answer = &hello[0];
```

`answer`'in ilk harf olan `З` olmayacağını zaten biliyorsunuz. UTF-8'de kodlandığında, `З`'nin ilk baytı `208` ve ikincisi `151`, bu yüzden `answer`'in aslında `208` olması gerektiğini görünse de `208` kendi başına geçerli bir karakter değildir. `208`'i dönmek muhtemelen bu stringin ilk harfini soran bir kullanıcının isteyeceği değildir; ancak bu, Rust'ın bayt indeksi 0'da sahip olduğu tek veridir. Kullanıcılar genellikle, string sadece Latin harfler içerse bile dönen bayt değerini istemezler: Eğer `&"hi"[0]` geçerli kod olsa ve bayt değeri dönerse, `h` yerine `104` dönerdi.

Cevap, o zaman, beklenmeyen bir değer dönmeyi ve geliştirme sürecinde hemen keşfedilmeyen hatalara neden olmasını önlemektir çünkü Rust bu kodu hiçbir şekilde derlemez ve erken yanlış anlaşılmaları önler.

<!-- Old headings. Do not remove or links may break. -->

<a id="bytes-and-scalar-values-and-grapheme-clusters-oh-my"></a>

#### Baytlar, Skaler Değerler ve Grapheme Kümeler (Bytes, Scalar Values, and Grapheme Clusters)

UTF-8 hakkında başka bir nokta şudur ki stringlere Rust'ın perspektifinden bakmanın aslında üç alakalı yolu vardır: baytlar olarak, skaler değerler olarak ve grapheme kümeleri olarak (bizim _harfler_ diye adlandıracağımıza en yakın şey).

Eğer Devanagari betiğinde yazılmış "नमस्ते" Hindi kelimesine bakarsak, `u8` değerleri vektörü olarak şöyle görünen bir şey olarak saklanır:

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164,
224, 165, 135]
```

Bu 18 bayttir ve bu bilgisayarların sonunda bu veriyi nasıl sakladığıdır. Onları Rust'ın `char` tipinin olduğu Unicode skaler değerleri olarak bakarsak, bu baytlar şöyle görünür:

```text
['न', 'म', 'स', '्', 'त', 'े']
```

Burada altı `char` değeri var ancak dördüncü ve altıncısı harf değil: Onlar kendi başlarına anlamları olmayan diyakritiklerdir. Son olarak, onları grapheme kümeleri olarak bakarsak, insanların Hindi kelimesini oluşturduğunu söyleyeceği dört harf elde ederiz:

```text
["न", "म", "स्", "ते"]
```

Rust bilgisayarların depoladığı ham string verisini yorumlama şeklinde farklı yollar sunar ki böylece her program ihtiya duyduğu yorumlamayı seçebilir, verinin hangi insan dilinde olduğu önemli değildir.

Rust'un bir `String`'e indeksleyip bir karakter almak için bize izin vermemesinin son sebebi şudur ki indeksleme operasyonlarının her zaman sabit zaman(O(1)) alması beklenir. Ancak `String` ile bu performansı garanti etmek mümkün değildir çünkü Rust geçerli karakterlerin kaç tane olduğunu belirlemek için baştan sonuna kadar içeriğini gezmesi gerekirdi.

### Stringleri Dilimleme (Slicing Strings)

Bir stringi indekslemek genellikle kötü bir fikirdir çünkü string-indeksleme operasyonunun hangi dönüş tipi olması gerektiği açık değil: bir bayt değeri, bir karakter, bir grapheme kümesi veya bir string dilimi mi? Eğer gerçekten indeksleri kullanarak string dilimleri oluşturmanız gerekirse, bu yüzden Rust size daha spesifik olmanızı ister.

Tek bir sayıyla `[]` kullanmak yerine, belirli baytları içeren bir string dilimi oluşturmak için `[]` aralığı kullanabilirsiniz:

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

Burada, `s` stringin ilk 4 baytini içeren bir `&str` olacak. Daha önce, bu karakterlerin her birinin 2 bayt olduğu bahsetmiştik ki bu `s` `Зд` olacaktır.

Eğer bir karakterin baytlarının sadece bir kısmını dilimlemek için `&hello[0..1]` gibi bir şeyle çalışsaydık, Rust, bir vektörde geçersiz bir indekse erişildiğinde olduğu gibi çalışma zamanında panikleyecektir:

```console
{{#include ../listings/ch08-common-collections/output-only-01-not-char-boundary/output.txt}}
```

Aralıklarla string dilimleri oluştururken dikkat kullanmalısınız çünkü bunu yapmak programınızın çökmesine neden olabilir.

<!-- Old headings. Do not remove or links may break. -->

<a id="methods-for-iterating-over-strings"></a>

### Stringler Üzerinde İterasyon (Iterating Over Strings)

String parçaları üzerinde operasyon yapmanın en iyi yolu, karakterler mi yoksa baytlar mı istediğiniz konusunda açık olmaktır. Bireysel Unicode skaler değerleri için `chars` yöntemini kullanın. "Зд" üzerinde `chars` çağırmak iki `char` tipinde değer döner ve sonuç üzerinden iterasyon yaparak her öğeye erişebilirsiniz:

```rust
for c in "Зд".chars() {
    println!("{c}");
}
```

Bu kodu aşağıdaki yazdırır:

```text
З
д
```

Alternatif olarak, `bytes` yöntemi her bir ham baytı döner ki bu sizin alanınız için uygun olabilir:

```rust
for b in "Зд".bytes() {
    println!("{b}");
}
```

Bu kodu bu stringi oluşturan 4 baytı yazdırır:

```text
208
151
208
180
```

Ancak geçerli Unicode skaler değerlerinin birden fazla bayttan oluşabileceğini hatırlayın.

Devanagari betiğinde olduğu gibi stringlerden grapheme kümeleri almak karmaşıktır, bu yüzden bu işlevsellik standart kütüphane tarafından sağlanmaz. [crates.io](https://crates.io/)<!-- ignore --> adresinde ihtiya duyduğunuz işlevsellik için paketler mevcuttur.

<!-- Old headings. Do not remove or links may break. -->

<a id="strings-are-not-so-simple"></a>

### Stringlerin Karmaşıklarını Yönetme (Handling Complexities of Strings)

Özetle, stringler karmaşıktır. Farklı programlama dilleri bu karmaşıklığı programcıya nasıl sunma konusunda farklı seçimler yapar. Rust, tüm Rust programları için `String` verisinin doğru işlenmesini varsayılan davranış olarak seçmiştir ki bu, programcıların UTF-8 verisini önden daha fazla düşünmelerini gerektirir. Bu değiş-değiş, diğer programlama dillerinde görünenden daha fazla string karmaşıklığını açığa çıkarır ancak size, geliştirme yaşam döngüsünün ilerleyen kısımlarda ASCII olmayan karakterleri içeren hataları yönetmek zorunda kalmanızı önler.

İyi haber şu ki standart kütüphane `String` ve `&str` tipleri üzerine inşa edilmiş çok işlevsellik sunar ki bu karmaşık durumları doğru şekilde yönetmenize yardım eder. `contains` yöntemi gibi bir stringde arama yapmak ve `replace` yöntemi gibi bir stringin bir kısmını başka bir stringle değiştirmek için kullanışlı yöntemler gibi standart kütüphanenin sunduğu şeylerden emin olmak için belgelere göz atmayı unutmayın.

Biraz daha az karmaşıklık bir şeye geçelim: hash haritaları!