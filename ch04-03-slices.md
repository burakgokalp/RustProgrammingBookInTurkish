## Dilim Tipi

_Dilimler_ (slices), bir [koleksiyondaki][ch08]<!-- ignore --> ardışık öğe dizisine başvurmanıza izin verir. Bir dilim bir başvuru türüdür, bu yüzden sahipliği yoktur.

Burada küçük bir programlama problemi var: Boşluklarla ayrılmış bir kelime dizisi alan ve bu dizide bulduğu ilk kelimeyi dönen bir fonksiyon yazın. Eğer fonksiyon dizide bir boşluk bulmazsa, tüm dize tek bir kelimeden oluşmalıdır, bu yüzden tüm dize dönmelidir.

> Not: Dilimleri tanıtmak amacıyla, bu bölümde sadece ASCII varsayıyoruz; UTF-8 işleme hakkında daha kapsamlı bir tartışma [Bölüm 8'deki][strings]<!-- ignore --> "Strings ile UTF-8 Kodlanmış Metin Saklamak" bölümünde bulunmaktadır.

Dilimlerin çözeceği problemi anlamak için dilimler kullanmadan bu fonksiyonun imzasını nasıl yazacağımız üzerinde çalışalım:

```rust,ignore
fn first_word(s: &String) -> ?
```

`first_word` fonksiyonu `&String` tipinde bir parametreye sahip. Sahipliğe ihtiyacımız yok, bu yüzden bu iyidir. (İdiyomatik Rust'ta, fonksiyonlar argümanlarının sahipliğini ihtiyaç duymadıkça almazlar, ve bunun nedenleri devam ettikçe netleşecektir.) Ancak ne dönmeliyiz? Bir dizenin _bir kısmı_ hakkında konuşmanın gerçek bir yolu yoktur. Ancak, bir boşluk tarafından belirtilen kelimenin sonunun indeksini dönebiliriz. Bunu deneyelim, Kod Listesi 4-7'de gösterildiği gibi.

<Listing number="4-7" file-name="src/main.rs" caption="`String` parametresine bir bayt indeksi değeri dönen `first_word` fonksiyonu">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:here}}
```

</Listing>

Çünkü `String`'i öğe öğe geçmemiz ve bir değerin bir boşluk olup olmadığını kontrol etmemiz gerekiyor, `as_bytes` yöntemini kullanarak `String`'imizi bir bayt dizisine dönüştüreceğiz.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:as_bytes}}
```

Sonra, `iter` yöntemini kullanarak bayt dizisi üzerinde bir yineleyici (iterator) oluşturuyoruz:

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:iter}}
```

Yineleyicileri [Bölüm 13'te][ch13]<!-- ignore --> daha fazla detayla konuşacağız. Şimdilik, `iter`'in bir koleksiyondaki her öğeyi dönen bir yöntem olduğunu ve `enumerate`'in `iter`'in sonucunu sarar ve her öğeyi bir demetin parçası olarak döndürdüğünü bilin. `enumerate`'den dönen demetin ilk öğesi indekstir ve ikinci öğe öğeye bir başvurudur. Bu, indeksi kendimiz hesaplamaktan biraz daha uygundur.

Çünkü `enumerate` yöntemi bir demet döndürüyor, demeti parçalamak için desenler kullanabiliriz. Desenleri [Bölüm 6'da][ch6]<!-- ignore --> daha fazla konuşacağız. `for` döngüsünde, demetin indeksi için `i` ve demetdeki tek bayt için `&item` içeren bir desen belirtiyoruz. Çünkü `.iter().enumerate()`'dan öğeye bir başvuru alıyoruz, desen içinde `&` kullanıyoruz.

`for` döngüsü içinde, bayt değişmez sözdizimini kullanarak bir boşluğu temsil eden baytı arıyoruz. Bir boşluk bulursak, pozisyonu döndürüyoruz. Aksi takdirde, `s.len()`'i kullanarak dizenin uzunluğunu döndürüyoruz.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:inside_for}}
```

Artık dizedeki ilk kelimenin sonunun indeksini bulmanın bir yolu var, ancak bir problem var. Tek başına bir `usize` döndürüyoruz ancak bu sadece `&String` bağlamında anlamlı bir sayıdır. Başka bir deyimle, çünkü bu `String`'den ayrı bir değerdir, gelecekte hala geçerli olacağı garantilmez. Kod Listesi 4-8'deki programı düşünün, bu Kod Listesi 4-7'deki `first_word` fonksiyonunu kullanır.

<Listing number="4-8" file-name="src/main.rs" caption="`first_word` fonksiyonunu çağırmadan sonucu saklamak ve sonra `String` içeriğini değiştirmek">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-08/src/main.rs:here}}
```

</Listing>

Bu program herhangi bir hatasız derlenir ve `s.clear()` çağırdıktan sonra `word` kullanırsak da öyle olur. Çünkü `word`, `s`'nin durumuyla hiçbir şekilde bağlı değil, `word` hala `5` değerini içerir. Bu `5` değerini `s` değişkeniyle kullanıp ilk kelimeyi çıkarmaya çalışabiliriz ancak bu bir hatadır çünkü `s`'nin içerikleri `word` içinde `5` kaydettiğimizden beri değişti.

`word` içindeki indeksin `s`'deki veriyle senkronize olmasından endişelenmek can sıkıcı ve hataya eğilimli! Eğer bir `second_word` fonksiyonu yazarsak, bu indeksleri yönetmek daha da kırılgandır. İmzası şöyle görünmeli:

```rust,ignore
fn second_word(s: &String) -> (usize, usize) {
```

Şimdi bir başlangıç _ve_ bir bitiş indeksi takip ediyoruz ve özel bir durumundaki verilerden hesaplanan ancak o durumla hiçbir şekilde bağlı olmayan daha fazla değerimiz var. Senkronize tutulması gereken üç ilgisiz değişkenimiz var.

Şanslıyız ki Rust'ın bu probleme bir çözümü var: dize dilimleri.

### Dize Dilimleri (String Slices)

Bir _dize dilimi_ (string slice), bir `String`'in öğelerinin ardışık dizisine bir başvurudur ve şöyle görünür:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-17-slice/src/main.rs:here}}
```

Tüm `String`'e bir başvuru yerine, `hello` `String`'in bir kısmına bir başvurudur, ekstra `[0..5]` biti ile belirtilir. Dilimleri köşeli parantez içinde bir aralık belirterek oluşturuyoruz, `[baslangic_indeks..bitis_indeks]`, burada _`baslangic_indeks`_ dilimdeki ilk pozisyondur ve _`bitis_indeks`_ dilimdeki son pozisyondan birdir. Dahili olarak, dilim veri yapısı dilimin başlangıç pozisyonunu ve dilimin uzunluğunu saklar ki bu _`bitis_indeks`_ eksi _`baslangic_indeks`_'a karşılık gelir. Bu yüzden, `let world = &s[6..11];` durumunda, `world` `s`'nin indeks 6'daki bayta işaret eden ve `5` uzunluk değerine sahip bir dilim olacaktır.

Şekil 4-7 bunu bir diyagramda gösterir.

<img alt="Üç tablo: yığın verisini temsil eden s tablosu, yığındaki dize verisi tablosundaki indeks 0'daki bayta işaret ediyor. Üçüncü tablo world diliminin yığın verisini temsil ediyor, 5 uzunluk değerine sahip ve yığın veri tablosunun bayt 6'sına işaret ediyor." src="img/trpl04-07.svg" class="center" style="width: 50%;" />

<span class="caption">Şekil 4-7: Bir `String`'in bir kısmına atıfta eden bir dize dilimi</span>

Rust'ın `..` aralık sözdizimi ile, indeks 0'dan başlamak istiyorsanız, iki noktadan önceki değeri bırakabilirsiniz. Başka bir deyimle, bunlar eşittir:

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

Aynı tokenla, diliminiz `String`'in son baytını içeriyorsa, sondaki sayıyı bırakabilirsiniz. Bu şu demektir ki bunlar eşittir:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

Ayrıca her iki değeri bırakarak tüm dizenin dilimini alabilirsiniz. Bu yüzden, bunlar eşittir:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> Not: Dize dilimi aralık indeksleri geçerli UTF-8 karakter
> sınırlarında oluşmalıdır. Çok baytlı bir karakterin ortasında bir dize dilimi
> oluşturmaya çalışırsanız, programınız bir hatayla çıkacaktır.

Tüm bu bilgiler aklımızda tutularak, `first_word`'u bir dilim dönecek şekilde yeniden yazalım. "Dize dilimi"ni belirten tip `&str` olarak yazılır:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-18-first-word-slice/src/main.rs:here}}
```

</Listing>

Kelimenin sonunun indeksini Kod Listesi 4-7'de yaptığımız şekilde, ilk boşluk oluşumunu arayarak alıyoruz. Bir boşluk bulduğumuzda, dizenin başlangıcını ve boşluğun indeksini başlangıç ve bitiş indeksleri olarak kullanarak bir dize dilimi döndürüyoruz.

Şimdi `first_word` çağırdığımızda, temel verilere bağlı tek bir değer geri alıyoruz. Değer dilimin başlangıç noktasına bir başvurudan ve dilimdeki öğe sayısından oluşur.

Bir dilim dönmek `second_word` fonksiyonu için de çalışacaktır:

```rust,ignore
fn second_word(s: &String) -> &str {
```

Artık karıştırması çok daha zor olan basit bir API'miz var çünkü derleyici `String`'e yapılan başvuruların geçerli kalmasını sağlayacaktır. Kod Listesi 4-8'deki programdaki hatayı, ilk kelimenin sonunun indeksini aldık ancak sonra dizeyi temizledik yani indeksimiz geçersiz olanı anımsayın? Bu kod mantıksal olarak yanlıştı ancak hiçbir anlık hata göstermedi. Problemler ilk kelime indeksini boşaltılmış bir dizeyle kullanmaya çalıştırmayı devam ettikçe daha sonra gösterirdi. Dilimler bu hatayı imkansız kılar ve kodumuzla bir problemimiz olduğunu çok daha erken bize bildirir. `first_word`'un dilim versiyonunu kullanmak bir derleme zamanı hatası atacak:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/src/main.rs:here}}
```

</Listing>

İşte derleyici hatası:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/output.txt}}
```

Ödünç alma kurallarından anımsayın ki bir şeye değişmez bir başvurumuz varsa, aynı zamanda bir değiştirilebilir başvuru da alamayız. Çünkü `clear` `String`'i kısaltmak ihtiyacı var, bir değiştirilebilir başvuru almak zorundadır. `clear` çağırdıktan sonraki `println!`, `word` içindeki başvuruyu kullanıyor, bu yüzden değişmez başvuru o noktada hala aktif olmalıdır. Rust, `clear` içindeki değiştirilebilir başvuruyu ve `word` içindeki değişmez başvuruyu aynı anda var olmayı yasaklar ve derleme başarısız olur. Rust sadece API'mizi daha kullanışlı hale getirmekle kalmamış, aynı zamanda derleme zamanında tüm bir hata sınıfını ortadan kaldırmıştır!

<!-- Old headings. Do not remove or links may break. -->

<a id="string-literals-are-slices"></a>

#### Dize Değişmezleri Dilimler Olarak (String Literals as Slices)

Dize değişmezlerinin ikili içinde saklandığı hakkında konuştuğumuzu anımsayın. Artık dilimler hakkında bildiğimize göre, dize değişmezlerini düzgünce anlayabiliriz:

```rust
let s = "Hello, world!";
```

Burada `s`'nin tipi `&str`'dir: İkili içindeki o belirli noktaya işaret eden bir dilimdir. Bu aynı zamanda dize değişmezlerinin neden değişmez olduğu; `&str` bir değişmez başvurusudur.

#### Dize Dilimlerini Parametre Olarak (String Slices as Parameters)

Değişmezlerin ve `String` değerlerinin dilimlerini alabileceğinizi bilmek bizi `first_word` üzerinde bir daha iyileştirmeye götürür ve bu da imzasıdır:

```rust,ignore
fn first_word(s: &String) -> &str {
```

Daha deneyimli bir Rustacean Kod Listesi 4-9'da gösterilen imzayı yazar çünkü bu bize hem `&String` değerleri hem de `&str` değerleri için aynı fonksiyonu kullanmamıza izin verir.

<Listing number="4-9" caption="`first_word` fonksiyonunu `s` parametresinin tipi için bir dize dilimi kullanarak iyileştirmek">

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:here}}
```

</Listing>

Eğer bir dize dilimiz varsa, bunu doğrudan geçebiliriz. Eğer bir `String`'imiz varsa, `String`'in dilimini veya `String`'e bir başvuru geçebiliriz. Bu esneklik, [Bölüm 15'in][deref-coercions]<!-- ignore --> "Fonksiyonlarda ve Yöntemlerde Deref Zorlamalarını Kullanmak" bölümünde konuşacağımız bir özellik olan deref zorlamalarının (deref coercions) avantajını kullanır.

Bir fonksiyonu bir `String`'e başvuru yerine bir dize dilimi alacak şekilde tanımlamak, hiçbir işlevselliği kaybetmeden API'mizi daha genel ve kullanışlı hale getirir:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:usage}}
```

</Listing>

### Diğer Dilimler (Other Slices)

Tahmin edebileceğiniz gibi, dize dilimleri dizelere özeldir. Ancak daha genel bir dilim tipi de vardır. Bu diziyi düşünün:

```rust
let a = [1, 2, 3, 4, 5];
```

Bir dizenin bir kısmına atıfta etmek isteyebileceğimiz gibi, bir dizinin bir kısmına atıfta etmek isteyebiliriz. Bunu şöyle yaparız:

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

Bu dilimin `&[i32]` tipi vardır. Dize dilimleriyle aynı şekilde çalışır, ilk öğeye bir başvuru ve bir uzunluk saklayarak. Bu tür dilimi Bölüm 8'de vektörleri konuşurken tüm diğer koleksiyonlar için kullanacaksınız.

## Özet

Sahiplik, ödünç alma ve dilim kavramları Rust programlarında derleme zamanında bellek güvenliği sağlar. Rust dili size diğer sistem programlama dilleriyle aynı şekilde bellek kullanımınız üzerinde kontrol verir. Ancak, verinin sahibinin sahibi kapsam dışına çıktığında bu veriyi otomatik olarak temizlemesi anlamında bu kontrolü almak için ekstra kod yazmanıza ve hata ayıklamanıza gerek yok.

Sahiplik, Rust'ın diğer birçok parçasının nasıl çalıştığını etkiler, bu yüzden kitabın geri kalanı boyunca bu kavramları daha fazla konuşacağız. Bölüm 5'e geçelim ve veriyi parçalarını bir `struct`'ta gruplamaya bakalım.

[ch13]: ch13-02-iterators.html
[ch6]: ch06-02-match.html#patterns-that-bind-to-values
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[deref-coercions]: ch15-02-deref.html#using-deref-coercions-in-functions-and-methods
[ch08]: ch08-00-common-collections.html