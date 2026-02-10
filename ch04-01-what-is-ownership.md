## Sahiplik Nedir?

_Sahiplik_ (Ownership), bir Rust programının belleği nasıl yönettiğini yöneten bir kurallar setidir. Tüm programların, çalışırken bir bilgisayarın belleğini kullandıkları yolu yönetmeleri gerekir. Bazı dillerde, düzenli olarak artık kullanılmayan belleği arayan bir çöp toplayıcı (garbage collector) vardır; diğer dillerde, programcı belleği açıkça ayırmalı (allocate) ve serbest bırakmalıdır (free). Rust üçüncü bir yaklaşımı kullanır: Bellek, derleyicinin kontrol ettiği bir kurallar seti ile bir sahiplik sistemi üzerinden yönetilir. Kurallardan herhangi biri ihlal edilirse, program derlenmeyecek. Sahipliğin özellikleri hiçbiri, programın çalışması sırasında onu yavaşlatmayacak.

Çünkü sahiplik birçok programcı için yeni bir kavramdır, alışmak biraz zaman alır. İyi haber şu ki, Rust ve sahiplik sisteminin kuralları konusunda daha deneyimli hale geldikçe, güvenli ve verimli kodu doğal olarak geliştirmeyi daha kolay bulacaksınız. Devam edin!

Sahipliği anladığınızda, Rust'ın özgün yaptığı özellikleri anlamak için sağlam bir temeliniz olacak. Bu bölümde, çok yaygın bir veri yapısına odaklanan örnekler üzerinden sahipliği öğreneceksiniz: dizeler.

> ### Yığın ve Yığın (Stack and Heap)
>
> Birçok programlama dili sizden çok sık yığın ve yığın (heap) düşünmenizi gerektirmez. Ancak Rust gibi bir sistem programlama dilinde, bir değerin yığın veya yığın üzerinde olup olmadığı dilin nasıl davrandığını ve neden belirli kararları vermeniz gerektiğini etkiler. Sahipliğin kısımları, bu bölümde daha sonra yığın ve yığınla ilişkili olarak açıklanacak, bu yüzden hazırlık olarak kısa bir açıklama.
>
> Yığın ve yığının her ikisi de kodunuzun çalışma zamanında kullanması için uygun olan belleğin parçalarıdır, ancak farklı şekillerde düzenlenmişlerdir. Yığın değerleri aldığı sırada saklar ve değerleri ters sırada kaldırır. Buna _son giren, ilk çıkan (LIFO)_ denir. Bir tabak yığını düşünün: Daha fazla tabak eklediğinizde, onları yığının üzerine koyarsınız ve bir tabak lazım olduğunda, üstten bir tanırsınız. Ortadan veya alttan tabak eklemek veya çıkarmak iyi çalışmaz! Veri eklemek _yığına itme_ (pushing onto the stack) ve veri kaldırmak _yığından çıkma_ (popping off the stack) olarak adlandırılır. Yığında saklanan tüm verilerin bilinen, sabit bir boyutu olmalıdır. Derleme zamanında bilinmeyen boyutta veya boyutu değişebilen veri bunun yerine yığına (heap) saklanmalıdır.
>
> Yığın daha az düzenlenmiştir: Yığına veri koyduğunuzda, belirli bir miktarda yer istersiniz. Bellek ayırıcı (memory allocator), yığına içinde yeterince büyük boş bir yer bulur, kullanımda olarak işaretler ve bir _gösterici (pointer)_ döndürür, bu konumun adresidir. Bu süreç _yığına ayırma_ (allocating on the heap) olarak adlandırılır ve bazen sadece _ayırma_ (allocating) olarak kısaltılır (yığına değer itme ayırma olarak sayılmaz). Çünkü yığına (heap) gösterici bilinen, sabit bir boyut olan, göstericiyi yığında saklayabilirsiniz, ancak gerçek veriye istediğinizde, göstericiyi takip etmelisiniz. Bir restoranda oturmuş gibi düşünün. Girdiğinizde, grubunuzdaki kişi sayısını belirtirsiniz ve ev sahibi herkesi sığan boş bir masa bulur ve sizi oraya yönlendirir. Grubunuzdaki biri geç gelirse, nerede oturduğunuzu sorabilir sizi bulmak için.
>
> Yığına itmek, yığına ayırmadan daha hızlıdır çünkü ayırıcı yeni veriyi saklamak için bir yer aramak zorunda kalmaz; bu konum her zaman yığının üstündedir. Karşılaştırmalı olarak, yığında yer ayırmak daha fazla iş gerektirir çünkü ayırıcı önce veriyi tutabilecek kadar büyük bir yer bulmalı ve sonra bir sonraki ayırma için hazırlamak için defter tutma işlemini gerçekleştirmelidir.
>
> Yığındaki verilere erişmek genellikle yığındaki verilere erişmekten daha yavaştır çünkü oraya ulaşmak için bir göstericiyi takip etmeniz gerekir. Çağdaş işlemciler bellekte daha az atlarsa daha hızlıdır. Benzetmeyi sürdürün, bir restorandaki garsonun birçok masadan sipariş aldığını düşünün. Bir masadaki tüm siparişleri almak, sonra sonraki masaya geçmek en verimli yöntemdir. A masadan bir sipariş almak, sonra B masadan bir sipariş, sonra A'dan bir tane, sonra B'den bir tane daha almak çok daha yavaş bir süreç olur. Aynı mantıkla, bir işlemci genellikle işini daha iyi yapabilir eğer uzakta olan (yığında olabilir) veriden çok uzakta olan (yığında olabilir) veriyle çalışırsa.
>
> Kodunuz bir fonksiyon çağırdığında, fonksiyona geçilen değerler (potansiyel olarak yığındaki veriyi gösterenler dahil) ve fonksiyonun yerel değişkenleri yığına itilir. Fonksiyon bittiğinde, bu değerler yığından çıkarılır.
>
> Yığındaki hangi verilerin hangi kod bölümleri tarafından kullanıldığını takip etmek, yığındaki çift veri miktarını en aza indirmek ve yerini kullanılmayan veriyi temizlemek (clean up) böylece yeriniz bitmesiniz tüm problemlerdir ki sahiplik bunları ele alır. Sahipliği anladığınızda, çok sık yığın ve yığın düşünmek zorunda kalmazsınız. Ancak sahipliğin temel amacının yığın verisini yönetmek olduğunu bilmek, onun neden böyle çalıştığını açıklamaya yardımcı olabilir.

### Sahiplik Kuralları

İlk olarak, sahiplik kurallarına bir bakalım. Bunları örnekler üzerinden çalışırken aklınızda tutun:

- Rust'taki her değerin bir _sahibi_ vardır.
- Bir kerede sadece bir sahibi olabilir.
- Sahibi kapsam dışına çıktığında, değer serbest bırakılacaktır (dropped).

### Değişken Kapsamı

Şuanda temel Rust sözdizimini geçtiğimize göre, örneklerde tüm `fn main() {` kodunu dahil etmeyeceğiz, bu yüzden takip ediyorsanız, aşağıdaki örnekleri manuel olarak bir `main` fonksiyonu içine koyduğunuzdan emin olun. Sonuç olarak, örneklerimiz biraz daha öz olacak, bu bizi asıl detaylara odaklamaya izin veriyor ve örnek koddan ziyade.

Sahipliğin birinci örneği olarak, bazı değişkenlerin kapsamına bakacağız. Bir _kapsam_ (scope), bir öğenin bir program içinde geçerli olduğu bir aralıktır. Aşağıdaki değişkeni düşünün:

```rust
let s = "hello";
```

`s` değişkeni, değerin programımızın metnine kodlandığı bir dize değişmezine (string literal) atıfta eder. Değişken, bildirildiği noktadan itibaren geçerlidir ve geçerli kapsamın sonuna kadar. Kod Listesi 4-1, `s` değişkeninin geçerli olduğu yerleri açıklama notaları içeren bir program gösterir.

<code class="language-rust" code_block=true number="4-1" caption="Değişken ve geçerli olduğu kapsam">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-01/src/main.rs:here}}
```

</code>

Başka bir deyimle, burada iki önemli zaman noktası var:

- `s` kapsam _içeri_ girdiğinde, geçerlidir.
- Kapsam _dışına_ çıncaya kadar geçerli kalır.

Bu noktada, kapsamlar ve değişkenlerin ne zaman geçerli olduğu arasındaki ilişki, diğer programlama dillerindeki gibidir. Şimdi bu anlayışı üzerine inşa edeceğiz, `String` tipini tanıtarak.

### `String` Tipi

Sahiplik kurallarını açıklamak için, [Bölüm 3'ün][data-types]<!-- ignore --> "Veri Tipleri" bölümünde ele aldığımız tiplerden daha karmaşık bir veri tipine ihtiyacımız var. Daha önce ele alınan tipler bilinen boyuta sahiptir, yığında saklanabilir ve kapsamları bittiğinde yığından çıkartılabilir ve başka bir kod parçası aynı değeri başka bir kapsamda kullanmak istiyorsa yeni, bağımsız bir örneği hızlı ve önemsizce kopyalanabilir. Ancak yığına (heap) saklanan veriye bakmak ve Rust'ın o veriyi ne zaman temizlemesini öğrenmesini keşfetmek istiyoruz ve `String` tipi harika bir örnektir.

`String`'in sahiplikle ilgili bölümlerine odaklanacağız. Bu yönler standart kütüphane tarafından sağlanan veya siz tarafından oluşturulan diğer karmaşık veri tiplerine de uygulanır. `String`'in sahiplik olmayan yönlerini [Bölüm 8'de][ch8]<!-- ignore --> konuşacağız.

Zaten dize değişmezlerini (string literals) gördük, burada bir dize değeri programımıza kodlanmıştır. Dize değişmezleri kullanışlıdır ancak metin kullanmak istediğimiz her durum için uygun olmayabilirler. Bir neden şudur ki bunlar değişmezdir. Başka bir neden şudur ki her dize değeri kodumuzu yazarken bilinemez: Örneğin, kullanıcı girdisini alıp saklamak istersek ne olur? İşte bu durumlar için Rust'ın `String` tipi vardır. Bu tip, yığında ayırdığı veriyi yönetir ve bu yüzden derleme zamanında bize bilinmeyen bir miktar metin saklayabilir. Aşağıdaki gibi `from` fonksiyonu kullanarak bir dize değişmezinden bir `String` oluşturabilirsiniz:

```rust
let s = String::from("hello");
```

Çift noktalı `::` operatörü, bizi `string_from` gibi bir isim kullanmak yerine bu belirli `from` fonksiyonunu `String` tipinin ad alanı altında kullanmamızı izin verir. Bu sözdizimini [Bölüm 5'in][methods]<!-- ignore --> "Yöntemler" bölümünde daha fazla konuşacağız ve [Bölüm 7'de][paths-module-tree]<!-- ignore --> "Modül Ağacındaki Bir Öğeye Başvurmak için Yollar" bölümünde modüllerle ad alanını konuşurken.

Bu tür dize _değişebilir_:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-01-can-mutate-string/src/main.rs:here}}
```

Peki, burada fark nedir? Neden `String` değişebilir ancak değişmezler olamaz? Fark, bu iki tipin bellekle nasıl başa çıkmasıdır.

### Bellek ve Ayırma

Bir dize değişmez durumunda, içerikleri derleme zamanında biliyoruz, bu yüzden metin doğrudan nihai yürütülebilir içine kodlanmıştır. Bu nedenle dize değişmezleri hızlı ve verimlidir. Ancak bu özellikler sadece dize değişmezinin değişmezliğinden gelir. Ne yazık ki, derleme zamanında boyutu bilinmeyen ve çalışma zamanında boyutu değişebilen her metin parçası için bir bellek parçasını ikiliye koyamayız.

`String` tipiyle, değişebilir, büyüyebilen bir metin parçasını desteklemek için, derleme zamanında bilinmeyen bir miktarda belleği yığına (heap) ayırmamız gerekir, içerikleri tutmak için. Bu şunları ifade eder:

- Belleği bellek ayırıcıdan çalışma zamanında istemelidir.
- `String` ile işimiz bittiğinde belleği ayırıcıya geri dönmek için bir yol ihtiyacımız var.

İlk bölüm bizim tarafımızdan yapılır: `String::from` çağırdığımızda, uygulaması ihtiyaç duyduğu belleği ister. Bu neredeyse tüm programlama dillerinde evrenseldir.

Ancak ikinci bölüm farklıdır. Bir _çöp toplayıcı (GC)_ olan dillerde, GC artık kullanılmayan belleği takip eder ve temizler ve biz düşünmek zorunda kalmayız. GC'si olmayan çoğu dilde, belleğin artık kullanılmadığını belirlemek ve açıkça serbest bırakmak için kod çağırmak bizim sorumluluğumuzdur, tıpkı istediğimiz gibi. Bunu doğru yapmak tarihsel olarak zor bir programlama problemi olmuştur. Eğer unutursak, belleği israf ederiz. Eğer çok erken yaparsak, geçersiz bir değişkenimiz olur. Eğer iki kez yaparsak, o da bir hatadır. Tam olarak bir `allocate` ile tam olarak bir `free`'yi eşleştirmemiz gerekir.

Rust farklı bir yol izler: Sahibi olan değişken kapsam dışına çıktığında bellek otomatik olarak döndürülür. Kod Listesi 4-1'deki kapsam örneğinin bir versiyonu, dize değişmezi yerine bir `String` kullanarak işte:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-02-string-scope/src/main.rs:here}}
```

`String`'imizin ihtiyaç duyduğu belleği ayırıcıya döndürebileceğimiz doğal bir nokta var: `s` kapsam dışına çıktığında. Bir değişken kapsam dışına çıktığında, Rust bizim için özel bir fonksiyon çağırır. Bu fonksiyon [`drop`][drop]<!-- ignore --> olarak adlandırılır ve `String`'in yazarının belleği dönmek için kod koyabileceği yerdir. Rust kapanan kıvramalı parantezde `drop`'ü otomatik olarak çağırır.

> Not: C++'de, bir öğenin yaşam süresinin sonunda kaynakları serbest bırakma bu desen bazen _Kaynak Edinimi Başlatmadır (Resource Acquisition Is Initialization - RAII)_ olarak adlandırılır. C++'de RAII desenleri kullandıysanız Rust'taki `drop` fonksiyonu size tanıdık gelecektir.

Bu desen, Rust kodunun yazıldığı yola derin bir etkiye sahiptir. Şimdi basit görünebilir ancak daha karmaşık durumlarda, yığına (heap) ayırdığımız veriyi birden fazla değişken kullanmak istediğimizde, kodun davranışı beklenmedik olabilir. Şimdi bu durumlardan bazılarını keşfedelim.

<a id="ways-variables-and-data-interact-move"></a>

#### Değişkenler ve Veri Taşıma ile Etkileşimi (Move)

Birden fazla değişken, Rust'ta aynı veriyle farklı şekillerde etkileşebilirler. Kod Listesi 4-2, bir tamsayı kullanan bir örnek gösterir.

<code class="language-rust" code_block=true number="4-2" caption="`x` değişkenin tamsayı değerini `y`'ye atama">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-02/src/main.rs:here}}
```

</code>

Bunu ne yapıyor tahmin edebiliriz: "`5` değerini `x`'e bağla; sonra, `x`'teki değerin bir kopyasını oluştur ve `y`'ye bağla." Artık iki değişkenimiz var, `x` ve `y`, ve her ikisi de `5`'e eşit. Bu gerçekten olan şeydir çünkü tamsayılar bilinen, sabit boyutta basit değerlerdir ve bu iki `5` değeri yığına itilir.

Şimdi `String` versiyonuna bakalım:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-03-string-move/src/main.rs:here}}
```

Bu çok benzer görünüyor, bu yüzden çalıştığı yolun aynı olduğunu varsayabiliriz: Yani, ikinci satır `s1`'teki değerin bir kopyasını oluşturup `s2`'ye bağlayacaktır. Ancak bu tam olarak olan şey değildir.

Şekil 4-1'e bakın ve `String`'in örtü altında ne olduğunu görün. Bir `String`, solda gösterilen üç bölümden oluşur: dizenin içeriklerini tutan belleğe gösterici, bir uzunluk ve bir kapasite. Bu veri grubu yığında saklanır. Sağda ise, içerikleri tutan yığındaki bellek vardır.

<img alt="İki tablo: ilk tablo s1'in yığındaki temsilini içeriyor, onun uzunluğunu (5), kapasitesini (5) ve ikinci tablodaki ilk değere göstericiyi içeriyor. İkinci tablo yığındaki dize verisinin bayt bayt temsilini içeriyor." src="img/trpl04-01.svg" class="center" style="width: 50%;" />

<span class="caption">Şekil 4-1: `s1`'e bağlı `"hello"` değerini tutan bir `String`'in bellekteki temsili</span>

Uzunluk, `String`'in içeriklerinin şu anda kullandığı bellek miktarıdır, bayt olarak. Kapasite, `String`'in ayırıcıdan aldığı toplam bellek miktarıdır, bayt olarak. Uzunluk ve kapasite arasındaki fark önemlidir ancak bu bağlamda değil, bu yüzden şimdilik kapasiteyi yok saymak iyidir.

`s1`'i `s2`'ye atadığımızda, `String` verisi kopyalanır, yani yığındaki göstericiyi, uzunluğu ve kapasiteyi kopyalarız. Göstericinin işaret ettiği yığındaki veriyi kopyalamayız. Başka bir deyimle, bellekteki veri temsili Şekil 4-2 gibidir.

<img alt="Üç tablo: s1 ve s2 tabloları sırasıyla yığındaki bu dizeleri temsil ediyor ve her ikisi yığındaki aynı dize verisine işaret ediyor." src="img/trpl04-02.svg" class="center" style="width: 50%;" />

<span class="caption">Şekil 4-2: `s1`'in gösterici, uzunluk ve kapasitesinin kopyasına sahip `s2` değişkenin bellekteki temsili</span>

Temsil, Şekil 4-3 gibidir değil, bu Rust yığın veriyi kopyalasıydıysa bellek nasıl görünürdü. Eğer Rust bunu yapsaydı, yığındaki veri büyükse çalışma zamanı performansı açısından `s2 = s1` işlemi çok maliyetli olabilir.

<img alt="Dört tablo: s1 ve s2'nin yığın verilerini temsil eden iki tablo ve her ikisi kendi dize verisi kopyasına yığındaki işaret ediyor." src="img/trpl04-03.svg" class="center" style="width: 50%;" />

<span class="caption">Şekil 4-3: `s2 = s1`'in yapabileceği başka bir olasılık eğer Rust yığın veriyi de kopyalasıydı</span>

Daha önce, bir değişken kapsam dışına çıktığında, Rust otomatik olarak `drop` fonksiyonunu çağırır ve o değişkenin yığın belleğini temizlediğini söyledik. Ancak Şekil 4-2, her iki veri göstericisinin aynı konuma işaret ettiğini gösterir. Bu bir problemdir: `s2` ve `s1` kapsam dışına çıktığında, her ikisi de aynı belleği serbest bırakmayı deneyecektir. Bu _iki kat serbest bırakma_ (double free) hatası olarak bilinir ve daha önce bahsettiğimiz bellek güvenliği hatalarından biridir. Belleği iki kez serbest bırakmak bellek bozulmasına yol açabilir ki bu potansiyel olarak güvenlik açıklarına yol açabilir.

Bellek güvenliğini sağlamak için, `let s2 = s1;` satırından sonra, Rust `s1`'i artık geçersiz olarak kabul eder. Bu nedenle, `s1` kapsam dışına çıktığında Rust'ın hiçbir şeyi serbest bırakmasına gerek yoktur. `s2` oluşturulduktan sonra `s1`'i kullanmaya çalıştığınızda ne olacağını kontrol edin; çalışmayacak:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-04-cant-use-after-move/src/main.rs:here}}
```

Rust sizin geçersiz bir başvuruyu (reference) kullanmanızı önlediği için şuna benzer bir hata alacaksınız:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-04-cant-use-after-move/output.txt}}
```

Diğer dillerle çalışırken _sığ kopya_ (shallow copy) ve _derin kopya_ (deep copy) terimlerini duysanız, veriyi kopyalamadan gösterici, uzunluk ve kapasiteyi kopyalama kavramı muhtemelen sığ kopya yapmaya benzer. Ancak Rust aynı zamanda ilk değişkeni geçersiz kıldığından, sığ kopya olarak adlandırılmak yerine, bir _taşıma_ (move) olarak bilinir. Bu örnekte, `s1`'in `s2`'ye _taşındığını_ söyleriz. Yani, gerçekte olan şey Şekil 4-4'te gösterilmiştir.

<img alt="Üç tablo: s1 ve s2 tabloları sırasıyla yığındaki bu dizeleri temsil ediyor ve her ikisi yığındaki aynı dize verisine işaret ediyor. s1 tablosu gri çünkü s1 artık geçerli değil; sadece s2 yığın verisine erişmek için kullanılabilir." src="img/trpl04-04.svg" class="center" style="width: 50%;" />

<span class="caption">Şekil 4-4: `s1` geçersiz kılındıktan sonraki bellekteki temsil</span>

Bu probleminizi çözer! Sadece `s2` geçerli olduğu için, kapsam dışına çıktığında o tek başına belleği serbest bırakacaktır ve işimiz bitti.

Ek olarak, bu tarafından ima edilen bir tasarım seçimi var: Rust verinizin "derin" kopyalarını otomatik olarak asla oluşturmaz. Bu nedenle, herhangi bir _otomatik_ kopyalama çalışma zamanı performansı açısından maliyetli olduğu varsayılabilir.

#### Kapsam ve Atama

Kapsamlama, sahiplik ve belleğin `drop` fonksiyonu üzerinden serbest bırakılması arasındaki ilişki için tersi de doğrudur. Mevcut bir değişkene tamamen yeni bir değer atadığınızda, Rust `drop`'ü çağırır ve orijinal değerinin belleğini hemen serbest bırakacaktır. Örneğin bu kodu düşünün:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-04b-replacement-drop/src/main.rs:here}}
```

Başlangıçta `s` adında bir değişken bildirir ve onu `"hello"` değerine sahip bir `String`'e bağlarız. Sonra, hemen `"ahoy"` değerine sahip yeni bir `String` oluşturur ve onu `s`'ye atarız. Bu noktada, orijinal değeri tutan hiçbir şey yoktur. Şekil 4-5 şimdi yığın ve yığın verisini göstermektedir:

<img alt="Yığında dize değerini temsil eden tek tablo, yığındaki ikinci dize verisine (ahoy) işaret ediyor, orijinal dize verisi (hello) gri çünkü artık erişilemiyor." src="img/trpl04-05.svg" class="center" style="width: 50%;" />

<span class="caption">Şekil 4-5: Orijinal değer tümüyle değiştirildikten sonraki bellekteki temsil</span>

Orijinal dize bu yüzden hemen kapsam dışına çıkar. Rust onda `drop` fonksiyonunu çalıştıracaktır ve belleği hemen serbest bırakılacak. Sonunda değeri yazdırdığımızda, o `"ahoy, world!"` olacaktır.

<a id="ways-variables-and-data-interact-clone"></a>

#### Değişkenler ve Veri Klonlama ile Etkileşimi (Clone)

Eğer `String`'in yığın (heap) verisini _derinlemesini_ kopyalamak, sadece yığın verisini değil istersek, `clone` adında yaygın bir yöntem kullanabiliriz. Yöntem sözdizimini Bölüm 5'te konuşacağız ancak yöntemler birçok programlama dilinde yaygın bir özellik olduğu için, muhtemelen daha önce görmüşsünüz.

İşte `clone` yönteminin eylemde olduğu bir örnek:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-05-clone/src/main.rs:here}}
```

Bu tamamen iyi çalışır ve Şekil 4-3'teki davranışı açıkça üretir, orada yığın verisi _kopyalanır_.

`clone` çağrısı gördüğünüzde, bazı keyfi kodun yürütüldüğünü ve bu kodun maliyetli olabileceğini bilirsiniz. Bu, farklı bir şeyin olduğunu görsel bir göstergesidir.

#### Sadece Yığın Verisi: Kopya (Copy)

Henüz konuşmadığımız başka bir kıvrım (wrinkle) var. Tamsayılar kullanan bu kod - Kod Listesi 4-2'den gösterilen kısmı - çalışır ve geçerlidir:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-06-copy/src/main.rs:here}}
```

Ancak bu kod, just öğrendiğimizle çelişkiyor görünüyor: `clone` çağrımız yok ancak `x` hala geçerli ve `y`'ye taşınmadı.

Sebebi şudur ki derleme zamanında bilinen boyutu olan tipler, tamsayılar gibi, tamamen yığında saklanırlar bu yüzden gerçek değerlerin kopyaları hızlıca yapılır. Bu şu anlama gelir ki `y` değişkenini oluşturduktan sonra `x`'in geçerli olmasını önlememiz için hiçbir neden yoktur. Başka bir deyimle, derin ve sığ kopyalama arasında fark yoktur, bu yüzden `clone` çağırmak normal sığ kopyalamadan farklı bir şey yapmaz ve onu atlayabiliriz.

Rust'ın `Copy` özelliği (trait) adında özel bir notasyonu vardır ki yığında saklanan tiplere yerleştirebilir, tamsayılar gibi (özellikleri [Bölüm 10'de][traits]<!-- ignore --> daha fazla konuşacağız). Bir tip `Copy` özelliğini uygularsa, onu kullanan değişkenler taşınmaz ancak önemsizce kopyalanırlar, başka bir değişkene atamadan sonra hala geçerli kalırlar.

Rust, tip veya herhangi bir parçası `Drop` özelliğini uygulamışsa, onu `Copy` notasyonuyla işaretlemeye bize izin vermeyecek. Eğer tip bir değerin kapsam dışına çıktığında özel bir şey olması gerekir ve o tipe `Copy` notasyonunu eklersek, derleme zamanı hatası alacağız. Kendi tipinize `Copy` notasyonunu eklemek için özelliği uygulamayı öğrenmek için, [Ek C'deki][derivable-traits]<!-- ignore --> "Türetilebilir Özellikler" bölümüne bakın.

Peki, hangi tipler `Copy` özelliğini uygular? Verilen tipin belgelendirmesinden emin olabilirsiniz ancak genel kural olarak, herhangi bir basit skaler değer grubu `Copy`'i uygulayabilir ve herhangi bir ayırma gerektiren veya bir kaynak formu olan bir şey `Copy`'i uygulayabilir. İşte `Copy`'i uygulayan bazı tipler:

- Tüm tamsayı tipleri, örneğin `u32`.
- Boolean tip, `bool`, `true` ve `false` değerleriyle.
- Tüm kayan nokta tipleri, örneğin `f64`.
- Karakter tip, `char`.
- Tüpler (tuples), eğer sadece `Copy`'i uygulayan tipler içerirlerse. Örneğin,
  `(i32, i32)` `Copy`'i uygular ancak `(i32, String)` uygulamaz.

### Sahiplik ve Fonksiyonlar

Bir değeri bir fonksiyona geçmek mekanizması, onu bir değişkene atamakla benzerdir. Bir değişkeni bir fonksiyona geçmek, atamada olduğu gibi taşıyacaktır veya kopyalayacaktır. Kod Listesi 4-3, değişkenlerin nerede kapsam içine ve dışına çıktıklarını gösteren açıklamalarla birlikte bir örnek sahiptir.

<code class="language-rust" code_block=true number="4-3" caption="Sahiplik ve kapsam açıklamalı fonksiyonlar">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-03/src/main.rs}}
```

</code>

`takes_ownership` çağrısından sonra `s`'yi kullanmaya çalışsaydik, Rust bir derleme zamanı hatası atardı. Bu statik kontroller bizi hatalardan korur. `main` koduna `s` ve `x` kullanan kod ekleyin ve onları nerede kullanabileceğinizi ve sahiplik kurallarının sizi ne yapmaktan ne önlediğini görün.

### Dönüş Değerleri ve Kapsam

Değerleri dönmek (returning values) de sahipliği aktarabilir. Kod Listesi 4-4, bazı değer dönen bir fonksiyon örneği gösterir, Kod Listesi 4-3'teki ile benzer açıklamalarla birlikte.

<code class="language-rust" code_block=true number="4-4" caption="Dönüş değerlerinin sahipliğini aktarma">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-04/src/main.rs}}
```

</code>

Bir değişkenin sahipliği her kerede aynı kalıbı takip eder: Değer başka bir değişkene atadığınızda onu taşır. Yığındaki veriyi içeren bir değişken kapsam dışına çıktığında, verinin sahipliği başka bir değişkene taşınmadıkça, `drop` tarafından temizlenir.

Bu çalışırken, her fonksiyonla sahiplik alıp sonra sahipliği dönmek biraz can sıkıcı. Bir fonksiyonun bir değeri kullanmasını ancak sahiplik almamasını istemeyelim ne olur? Geçtikleri her şeyi geri geçmek zorunda olmamız da çok can sıkıcıdır, ayrıca fonksiyonun gövdesinden kaynaklanan ve dönmek istediğimiz herhangi bir veri de.

Rust, Kod Listesi 4-5'te gösterildiği gibi bir tüple kullanarak birden fazla değer dönmeye izin verir.

<code class="language-rust" code_block=true number="4-5" caption="Parametrelerin sahipliğini dönmek">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-05/src/main.rs}}
```

</code>

Ancak bu çok fazla tören ve yaygın olması gereken bir kavram için çok fazla iş. Şanslıyız ki Rust, sahipliği aktarmadan bir değeri kullanmak için bir özelliğe sahiptir: başvurular (references).

[data-types]: ch03-02-data-types.html#data-types
[ch8]: ch08-02-strings.html
[traits]: ch10-02-traits.html
[derivable-traits]: appendix-03-derivable-traits.html
[methods]: ch05-03-method-syntax.html#methods
[paths-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
[drop]: ../std/ops/trait.Drop.html#tymethod.drop