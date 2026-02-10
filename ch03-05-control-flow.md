## Kontrol Akışı

Bir koşulun `true` olup olmadığına bağlı olarak bazı kod çalıştırma yeteneği ve bir koşulun `true` olduğu sürece bazı kodu tekrar tekrar çalıştırma yeteneği çoğu programlama dilindeki temel yapı bloklarıdır. Rust kodunun çalıştırma akışını kontrol etmenize izin veren en yaygın yapılar `if` ifadeleri ve döngülerdir.

### `if` İfadeleri

Bir `if` ifadesi koşullara bağlı olarak kodunuzu dallandırmanıza izin verir. Bir koşul sağlarsınız ve sonra, "Eğer bu koşul karşılanırsa, bu kod bloğu çalıştır. Eğer koşul karşılanmazsa, bu kod bloğu çalıştırma."

`if` ifadesini keşfetmek için _branches_ adında _projects_ dizininzde yeni bir proje oluşturun. _src/main.rs_ dosyasında, aşağıdakileri girin:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/src/main.rs}}
```

Tüm `if` ifadeleri `if` anahtar kelimesi ile başlar, ardından bir koşul gelir. Bu durumda, koşul `number` değişkeninin 5'den küçük bir değere sahip olup olmadığını kontrol eder. Koşul `true` ise çalıştıracağı kod bloğu küme parantezleri içinde hemen koşuldan sonra yerleştiririz. `if` ifadelerindeki koşullarla ilişkili kod blokları bazen _kollar_ (arms) olarak adlandırılır, [Bölüm 2'deki "Tahmini Gizli Sayıyla Karşılaştırma"][comparing-the-guess-to-the-secret-number]<!-- ignore --> bölümünde tartıştığımız `match` ifadelerindeki kollar gibi.

İsteğe bağlı olarak, ayrıca bir `else` ifadesi ekleyebiliriz, bu programın koşul `false` değerlendirdiğinde çalıştırması için alternatif bir kod bloğu sağlamayı seçtik. Bir `else` ifadesi sağlamazsanız ve koşul `false` ise, program `if` bloğu atlayacak ve bir sonraki koda devam edecek.

Bu kodu çalıştırmayı deneyin; aşağıdaki çıktıyı görmelisiniz:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/output.txt}}
```

`number` değişkeninin değerini koşulu `false` yapacak bir değere değiştirmeyi ve ne olduğunu görmeyi deneyelim:

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/src/main.rs:here}}
```

Programi tekrar çalıştırın ve çıktıya bakın:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/output.txt}}
```

Bu koddaki koşulun bir `bool` _zorunlu olarak_ olması da not etmeye değerdir. Koşul bir `bool` değilse, bir hata alacaksınız. Örneğin, aşağıdaki kodu çalıştırmayı deneyin:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/src/main.rs}}
```

Bu seferde `if` koşulu `3` değerine değerlenir ve Rust bir hata atar:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/output.txt}}
```

Hata, Rust'ın bir `bool` beklediğini ancak bir tamsayı aldığını gösterir. Ruby ve JavaScript gibi dillerin aksine, Rust Boolean olmayan tipleri Boolean'ye otomatik olarak dönüştürmeye çalışmaz. Açık olmalısınız ve her zaman `if`'e koşulu olarak bir Boolean sağlamalısınız. Örneğin, `if` kod bloğu sadece bir sayı `0`'a eşit değilken çalışmasını istiyorsanız, `if` ifadesini aşağıdaki gibi değiştirebiliriz:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-29-if-not-equal-0/src/main.rs}}
```

Bu kodu çalıştırmak `number was something other than zero` yazdıracak.

#### `else if` ile Birden Fazla Koşulu İşleme

Bir `else if` ifadesinde `if` ve `else`'i birleştirerek birden fazla koşul kullanabilirsiniz. Örneğin:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/src/main.rs}}
```

Bu programın alabileceği dört olası yol vardır. Bunu çalıştırdıktan sonra, aşağıdaki çıktıyı görmelisiniz:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/output.txt}}
```

Bu program çalıştığında, her `if` ifadesini sırayla kontrol eder ve koşulu `true`'ya değerlendiren ilk gövdeyi çalıştırır. 6'nin 2'ye bölünebilse bile `number is divisible by 2` çıktısını görmeyiz, ayrıca `number is not divisible by 4, 3, or 2` metnini `else` bloğundan görmeyiz. Bunun nedeni, Rust sadece ilk `true` koşulu için bloğu çalıştırır ve birini bulduğunda, geri kalanları kontrol etmez.

Çok fazla `else if` ifadesi kullanmak kodunuzu karıştırabilir, bu yüzden bir fazlasanız varsa, kodunuzu yeniden düzenlemeyi isteyebilirsiniz. Bölüm 6 bu durumlar için Rust'ta `match` adında güçlü bir dallandırma yapısı anlatır.

#### `let` Deyiminde `if` Kullanımı

Çünkü `if` bir ifadedir, onu bir değişkene sonucu atamak için `let` deyiminin sağ tarafında kullanabiliriz, Kod Listesi 3-2'deki gibi.

<Listing number="3-2" file-name="src/main.rs" caption="Bir `if` ifadesinin sonucunu bir değişkene atama">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-02/src/main.rs}}
```

</Listing>

`number` değişkeni, `if` ifadesinin sonucuna bağlı bir değere bağlanacak. Bu kodu çalıştırın ve ne olduğunu görmeyin:

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-02/output.txt}}
```

Kod blokları içindeki son ifadeye değerlendirdiklerini ve sayıların kendileri de ifadeler olduğunu hatırlayın. Bu durumda, tüm `if` ifadesinin değeri hangi kod bloğu çalıştığına bağlıdır. Bu, `if` kolunun ve `else` kolunun olası sonuçlarının aynı tipte olması gerektiği anlamına gelir; Kod Listesi 3-2'de, `if` kolunun ve `else` kolunun sonuçları `i32` tamsayılarıdır. Tipler eşleşmezse, aşağıdaki örnekteki gibi, bir hata alacaksınız:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/src/main.rs}}
```

Bu kodu derlemeye çalıştığımızda bir hata alacağız. `if` ve `else` kollarının uyumsuz değer tipleri var ve Rust programdaki sorunun tam olarak nerede bulunacağını gösterir:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/output.txt}}
```

`if` bloğundaki ifade bir tamsayıya değerlenir ve `else` bloğundaki ifade bir dizeye değerlenir. Bu çalışmaz, çünkü değişkenlerin tek bir tipi olmalı ve Rust'ın derleme zamanında `number` değişkeninin ne tip olduğunu belirlice şekilde bilmesi gerekir. `number`'in tipini bilmek, derleyiciye `number`'ı kullandığımız her yerde tipin geçerli olduğunu doğrulamamıza izin verir. `number`'in tipi sadece çalışma zamanında belirlenecek olsaydı, derleyici daha karmaşık olur ve kodu için daha az garanti sağlardı çünkü herhangi bir değişken için çok fazla hipotetik tip takip etmeleri gerekirdi.

### Döngüler ile Tekrar

Bir kod bloğunu bir kereden fazla çalıştırmak genellikle yararlıdır. Bu görev için, Rust birkaç _döngü_ sağlar, bu döngü gövdesi içindeki kodu baştan sona kadar çalıştırır ve sonra hemen başlangıca geri döner. Döngülerle denemek için _loops_ adında yeni bir proje yapalım.

Rust'ın üç tür döngüsü vardır: `loop`, `while` ve `for`. Her birini deneyelim.

#### `loop` ile Kod Tekrarı

`loop` anahtar kelimesi Rust'a bir kod bloğunu sonsuza kadar veya açıkça durmasını söyleyene kadar tekrar tekrar çalıştırmasını söyler.

Bir örnek olarak, _loops_ dizinizdeki _src/main.rs_ dosyasını şöyle görünmesini değiştirin:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-loop/src/main.rs}}
```

Bu programı çalıştırdığımızda, programi manuel olarak durdurana kadar tekrar tekrar `again!` yazdırıldığını göreceksiniz. Çoğu terminal, sürekli bir döngüye sıkışmış bir programi kesmek için <kbd>ctrl</kbd>-<kbd>C</kbd> klavye kısayolunu destekler. Bunu deneyin:

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-32-loop
cargo run
CTRL-C
-->

```console
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.08s
     Running `target/debug/loops`
again!
again!
again!
^Cagain!
```

`^C` sembolü <kbd>ctrl</kbd>-<kbd>C</kbd>'ye bastığınızı temsil eder.

Kod kesme sinyalini aldığı noktada `again!` kelimesini yazılmış olarak görebilirsiniz veya göremeyebilirsiniz, bu kodun o noktada döngünün nerede olduğuna bağlıdır.

Şanslı olarak, Rust ayrıca kod kullanarak bir döngüden çıkmak için bir yol sağlar. Döngüden ne zaman durdurması gerektiğini söylemek için `break` anahtar kelimesini döngü içine yerleştirebilirsiniz. Bölüm 2'deki tahmin oyununda [Doğru Tahminden Sonra Çıkma][quitting-after-a-correct-guess]<!-- ignore --> bölümünde kullanıcı doğru sayı tahmin ederek oyunu kazandığında programdan çıkmak için bunu yapmıştığımızı hatırlayın.

Tahmin oyununda ayrıca `continue` kullandık, bu bir döngüde programı döngünün bu yenilemesinde kalan herhangi bir kodu atlaması ve bir sonraki yenilemeye geçmesini söyler.

#### Döngülerden Değer Dönme

Bir `loop`'ın kullanımlarından biri, başarısız başarabilecek bir işlemi yeniden denemektir, örneğin bir iş parçasının işini tamamladığını kontrol etmek. Ayrıca o işlemin sonucunu döngüden dışarıdaki koda geçmeniz gerekebilir. Bunu yapmak için, döngüyü durdurmak için kullandığınız `break` ifadesinden sonra döndürmek istediğiniz değeri ekleyebilirsiniz; o değer döngüden dışarı dönecek böylece onu kullanabilirsiniz, burada gösterildiği gibi:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-33-return-value-from-loop/src/main.rs}}
```

Döngüden önce, `counter` adında bir değişken bildiririz ve onu `0`'a başlatırız. Sonra, döngüden dönen değeri tutmak için `result` adında bir değişken bildiririz. Döngünün her yenilemesinde, `counter` değişkenine `1` ekleriz ve sonra `counter`'ın `10`'a eşit olup olmadığını kontrol ederiz. Eğer eşitse, `break` anahtar kelimesini `counter * 2` değeriyle kullanırız. Döngüden sonra, değeri `result`'e atayan deyimi sonlandırmak için bir noktalı virgül kullanırız. Son olarak, `result`'teki değeri yazdırırız, bu durumda `20`'dir.

Ayrıca bir döngünün içinden `return` edebilirsiniz. `break` sadece mevcut döngüden çıkar, ancak `return` her zaman mevcut fonksiyondan çıkar.

<!-- Old headings. Do not remove or links may break. -->
<a id="loop-labels-to-disambiguate-between-multiple-loops"></a>

#### Döngü Etiketleri ile Belirsizliği Giderme

Eğer döngüler içinde döngüleriniz varsa, `break` ve `continue` o noktadaki en iç döngüye uygulanır. Bir döngüye isteğe bağlı bir _döngü etiketi_ belirleyebilir ve sonra `break` veya `continue` ile kullanarak bu anahtar kelimelerin en içteki döngüye yerine etiketli döngüye uygulanmasını belirtebilirsiniz. Döngü etiketleri tek tırnakla başlamalıdır. İşte iki iç içe döngülü olan bir örnek:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/src/main.rs}}
```

Dış döngü `'counting_up` etiketine sahiptir ve 0'dan 2'ye kadar yukarı sayar. Etiketi olmayan iç döngü 10'dan 9'ye kadar aşağı sayar. Etiketi belirtmeyen ilk `break` sadece iç döngüden çıkar. `break 'counting_up';` deyimi dış döngüden çıkaracaktır. Bu kod şunu yazdırır:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/output.txt}}
```

<!-- Old headings. Do not remove or links may break. -->
<a id="conditional-loops-with-while"></a>

#### `while` ile Koşullu Döngüler Basitleştirme

Bir program genellikle bir döngü içinde bir koşulu değerlendirmeye ihtiyaç duyacaktır. Koşul `true` iken, döngü çalışır. Koşul `true` olmayı bırakdığında, program `break` çağırır, döngüyü durdurur. Bu davranışı `loop`, `if`, `else` ve `break` kombinasyonu kullanarak uygulamak mümkündür; şimdi bir programda, isterseniz bunu deneyebilirsiniz. Ancak, bu desen o kadar yaygındır ki Rust bunun için yerleşik bir dil yapısına, `while` döngüsü adında sahiptir. Kod Listesi 3-3'te, her seferde aşağı sayar ve programı üç kez döndürerek döngü kullanırız, sonra, döngüden sonra bir mesaj yazdırırız ve çıkarız.

<Listing number="3-3" file-name="src/main.rs" caption="Bir koşulun `true` değerlendirdiği sürece bir `while` döngüsü kullanarak kod çalıştırmak">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-03/src/main.rs}}
```

</Listing>

Bu yapı, `loop`, `if`, `else` ve `break` kullanırsanız gerekli olacak birçok yuva iç içmeyi ortadan kaldırır ve daha açıktır. Koşul `true` değerlendirdiğinde, kod çalışır; aksi takdirde, döngüden çıkar.

#### `for` ile Bir Koleksiyon Geçme

Bir dizi gibi bir koleksiyonun elemanları üzerinde dönmek için `while` yapısını kullanmayı seçebilirsiniz. Örneğin, Kod Listesi 3-4'teki döngü `a` dizisindeki her elemanı yazdırır.

<Listing number="3-4" file-name="src/main.rs" caption="Bir `while` döngüsü kullanarak bir koleksiyonun her elemanı geçme">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-04/src/main.rs}}
```

</Listing>

Burada, kod dizideki elemanlar üzerinden yukarı sayar. İndeksi `0`'da başlar ve sonra dizinin final indeksine ulaşana kadar döngüler (bu, `index < 5`'in artık `true` olmama noktasıdır). Bu kodu çalıştırmak dizideki her elemanı yazdıracak:

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-04/output.txt}}
```

Beklendiği gibi, dizideki beş dizi değeri terminalda görünür. `index` bir noktada `5` değerine ulaşacak olsa bile, döngü diziden altıncı bir değeri almayı denemeden önce çalıştırmayı durdurur.

Ancak, bu yaklaşım hataya açıktır; indeks değeri veya test koşulu yanlışsa programın panic yapmasına neden olabilir. Örneğin, `a` dizisinin tanımını dört eleman içerecek şekilde değiştirsensiz ancak koşulu `while index < 4` olarak güncellemeyi unutursanız, kod panic yapacak. Ayrıca yavaştır, çünkü derleyici her döngü yenilemesinde indeksin dizinin sınırları içinde olup olmadığını kontrol eden çalışma zamanı kodu ekler ve bu döngünün her yenilemesinde dizinin uzunluğuyla indeksi kıyaslamak için çalışma zamanı kodunu ekler ve bu döngünün her yenilemesinde dizinin uzunluğuyla indeksi karşılaştırmak gerekir.

Daha kısa bir alternatif olarak, bir `for` döngüsü kullanabilir ve bir koleksiyondaki her öğe için bazı kod çalıştırabilirsiniz. Bir `for` döngüsü Kod Listesi 3-5'teki koda benzer.

<Listing number="3-5" file-name="src/main.rs" caption="Bir `for` döngüsü kullanarak bir koleksiyonun her elemanı geçme">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-05/src/main.rs}}
```

</Listing>

Bu kodu çalıştırdığımızda, Kod Listesi 3-4'tekindeki çıktı ile aynı çıktıyı göreceksiniz. Daha önemlis olarak, artık kodun güvenliğini arttırdık ve dizinin sonunun geçmesinden veya yeterince gitmemekten ve bazı öğeleri atlamaktan kaynaklanabilecek hataların şansını ortadan kaldırdık. `for` döngülerinden üretilen makine kodu daha verimli de olabilir çünkü döngünün her yenilemesinde indeksin dizinin uzunluğuyla kıyaslanmasına gerekmez.

`for` döngüsünü kullandığınızda, dizideki değer sayısını değiştirmeniz durumda Kod Listesi 3-4'teki yöntemi kullandığınızda unutmanız gerekecek olan diğer hiçbir kodu değiştirmeniz gerekmez.

`for` döngülerinin güvenliği ve kısalığı, onları Rust'ta en yaygın kullanılan döngü yapısı yapar. Kod Listesi 3-3'teki `while` döngüsünü kullanan geri sayım örneği gibi, bazı kodu belirli bir kerede çalıştırmanız gereken durumlarda bile, çoğu Rustacean `for` döngüsü kullanırdı. Bunu yapmanın yolu, standart kütüphane tarafından sağlanan ve bir sayıdan başlayıp başka sayının önce bitene kadar tüm sayıları sırayla üreten bir `Range` kullanmak olacaktır.

İşte `for` döngüsü ve henüz konuşmadığımız başka bir yöntem olan `rev`'i kullanarak bir geri sayımın nasıl görünebileceği:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-34-for-range/src/main.rs}}
```

Bu kod daha güzel, değil mi?

## Özet

Yaptınız! Bu büyük bir bölümdü: Değişkenler, skaler ve bileşik veri tipleri, fonksiyonlar, yorumlar, `if` ifadeleri ve döngüler hakkında öğrendiniz! Bu bölümde tartışılan kavramlarla pratik yapmak için şu programları yapmayı deneyin:

- Fahrenheit ve Celsius arasında sıcaklık dönüştürme.
- n*th Fibonacci sayısını üretin.
- "The Twelve Days of Christmas" Noel şarkısı "On İki Gününün İkrami," tekrarlanma avantajından yararlanarak şarkının sözlerini yazdırın.

Harekete geçmeye hazır olduğunuzda, diğer programlama dillerde yaygın olarak bulunmayan Rust'taki bir kavramdan konuşacağız: sahiplik.

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[quitting-after-a-correct-guess]: ch02-00-guessing-game-tutorial.html#quitting-after-a-correct-guess