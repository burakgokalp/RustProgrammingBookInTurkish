<!-- Old headings. Do not remove or links may break. -->
<a id="developing-the-librarys-functionality-with-test-driven-development"></a>

## Test Tabanlı Geliştirme ile İşlevselli Ekleme (Adding Functionality with Test-Driven Development)

Şimdi arama mantığını `main` fonksiyonundan ayrı _src/lib.rs_'de olduğumuza göre, kodumuzun çekirdek işlevselliği için testler yazmak çok daha kolay. Komut satırından ikilimizi çağırmak zorunda kalmadan, çeşitli argümanlarla fonksiyonları doğrudan çağırabilir ve dönen değerleri kontrol edebiliriz.

Bu bölümde, `minigrep` programına test tabanlı geliştirme (TDD) sürecini kullanarak arama mantığı ekleyeceğiz ve şu adımları izleyeceğiz:

1. Beklediğiniz bir nedenden dolayı başarısız olacak bir test yazın ve başarısız olduğunu doğrulamak için çalıştırın.
2. Yeni testi geçirmek için yeterince kod yazın veya değiştirin.
3. Yeni eklediğiniz veya değiştirdiğiniz kodu yerefactor edin ve testlerin geçmeye devam ettiğinden emin olun.
4. 1. adımdan tekrar edin!

Yazılım yazmak için birçok yolun sadece biri olmasına rağmen, TDD kod tasarımını sürdürebilir. Testi geçiren kodu yazmadan önce test yazmak sürec boyunca yüksek test kapsamı korumaya yardımcı olur.

Gerçekten dosya içeriklerinde sorgu dizesi için arama yapan ve sorguyla eşleşen satırların listesini üreten işlevselliğinin uygulamasını test-sürüteceğe yapacağız. Bu işlevselliği `search` adında bir fonksiyonda ekleyeceğiz.

### Başarısız Bir Test Yazma (Writing a Failing Test)

_src/lib.rs_'de, [Bölüm 11][ch11-anatomy]<!-- ignore -->'te yaptığımız gibi bir test fonksiyonu ile bir `tests` modülü ekleyeceğiz. Test fonksiyonu `search` fonksiyonuna sahip olmasını istediğimiz davranışı belirler: Bir sorgu ve aranacak metni alacak ve metinden sadece sorguyu içeren satırları dönecektir. Kod Listesi 12-15 bu testi gösterir.

<Listing number="12-15" file-name="src/lib.rs" caption="Sahip olduğumuzu işlevselliği için `search` fonksiyonuna bir başarısız test oluşturma">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-15/src/lib.rs:here}}
```

</Listing>

Bu test `"duct"` dizesi için arar. Aradığımız metin üç satır, sadece biri `"duct"` içerir (açılış çift tırnaçtaki eğik çizginin Rust'a bu dize değişmezinin başlangıcında yeni satır karakteri koymasını söylediğine dikkat edin). `search` fonksiyonundan dönen değerinin sadece beklediğimiz satırı içerdiğini iddia ediyoruz.

Bu testi çalıştırırsak, şu anda başarısız olacak çünkü `unimplemented!` makrosu "not implemented" mesajıyla panik atar. TDD prensiplerine uygun olarak, `search` fonksiyonunu her zaman boş bir vektör dönecek şekilde tanımlayarak, fonksiyonu çağırdığından panik atmamak için yeterince kod eklemenin küçük bir adımını atacağız ki bu Kod Listesi 12-16'da gösterilir. Sonra, test derlenir ve `"safe, fast, productive."` satırını içeren bir vektörle eşleşmediği için başarısız olur.

<Listing number="12-16" file-name="src/lib.rs" caption="`search` fonksiyonunu çağırmak panik atmaması için yeterince tanımlama">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-16/src/lib.rs:here}}
```

</Listing>

Şimdi `search`'un imzasında neden açık bir ömür `'a` tanımlamamız ve bu ömürü `contents` argümanı ve dönen değerle kullanmamız gerektiğini tartışalım. [Bölüm 10][ch10-lifetimes]<!-- ignore -->'te ömür parametrelerinin hangi argümanın ömrünün dönen değerinin ömrüne bağlı olduğunu belirlediğini hatırlayın. Bu durumda, dönen vektörün `contents` argümanının dilimlerine referans eden dize dilimlerini içerdiğini (sorgu argümanının değil) belirliyoruz.

Başka bir deyişle, `search` fonksiyonu tarafından dönen verinin `search` fonksiyonuna `contents` argümanında geçilen verinin yaşam kadar yaşamasını Rust'a söyluyoruz. Bu önemli! Bir dilim _tarafından_ referans edilen verinin referansın geçerli olması için geçerli olması gerekir; eğer derleyici `contents` yerine `query`'den dize dilimleri yaptığımızı varsarsa, güvenlik kontrolünü yanlış yapar.

Ömür notasyonlarını unutur ve bu fonksiyonu derlemeye çalışırsak, şu hatayı alacağız:

```console
{{#include ../listings/ch12-an-io-project/output-only-02-missing-lifetimes/output.txt}}
```

Rust, iki parametreden hangisini çıktı için ihtiyaç duyduğumuzu bilmiyor, bu yüzden ona açıkça söylememiz gerekiyor. Yardım metninin tüm parametreler ve çıktı türü için aynı ömür parametresini belirlemeyi önerdiğine dikkat edin ki bu yanlıştır! Çünkü `contents`, tüm metnimizi içeren parametredir ve onun eşleşen parçalarını dönmek istiyoruz, `contents`'in ömür sözdizimini kullanarak dönen değere bağlanması gereken tek parametre olduğunu biliyoruz.

Başka programlama dilleri imzada argümanları dönen değerlere bağlamayı gerektirmez, ancak bu uygulama zamanla daha kolay olacaktır. Bunu Bölüm 10'daki [`"Ömürlerle Referansları Doğrulama"`][validating-references-with-lifetimes]<!-- ignore --> bölümündeki örneklerle karşılaştırmak isteyebilirsiniz.

### Testi Geçiren Kod Yazma (Writing Code to Pass the Test)

Şu anda, testimiz başarısız çünkü her zaman boş bir vektör dönüyoruz. Bunu düzeltmek ve `search`'u uygulamak için, programımız şu adımları izlemeli:

1. İçeriklerin her satırı boyunca yinele.
2. Satırın sorgu dizesini içerip içermediğini kontrol et.
3. İeriyorsa, döndüğümüz değerler listesine ekle.
4. İermiyorsa, hiçbir şey yapma.
5. Eşleşen sonuçlar listesini dön.

Her adımı boyunca çalışalım, satırlar boyunca yinelemeyle başlayalım.

#### `lines` Yöntemiyle Satırlar Boyunca Yineleme (Iterating Through Lines with `lines` Method)

Rust'in, dizeyi satır satır yineleme için yararlı, uygun şekilde `lines` adlandırılmış bir yöntemi var ki bu Kod Listesi 12-17'de gösterildiği gibi çalışır. Buna henüz derlenmeyeceğine dikkat edin.

<Listing number="12-17" file-name="src/lib.rs" caption="`contents`'taki her satır boyunca yineleme">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-17/src/lib.rs:here}}
```

</Listing>

`lines` yöntemi bir iteratör döndürür. İteratörleri [Bölüm 13][ch13-iterators]<!-- ignore -->'te ayrıntılı olarak tartışacağız. Ancak [Kod Listesi 3-5][ch3-iter]<!-- ignore -->'te bir iteratörü bu şekilde kullandığınızı hatırlayın ki orada bir `for` döngüsüyle bir koleksiyondaki her öğe üzerinde bazı kod çalıştırmıştık.

#### Sorgu İçin Her Satırı Arama (Searching Each Line for `Query`)

Sonra, geçerli satırın sorgu dizesini içerip içermediğini kontrol edeceğiz. Şanslısına, dizecilerin bunu bizim için yapan `contains` adında yararlı bir yöntemi var! `search` fonksiyonunda `contains` yöntemine bir çağrı ekle, bu Kod Listesi 12-18'da gösterildiği gibi. Buna henüz derlenmeyeceğine dikkat edin.

<Listing number="12-18" file-name="src/lib.rs" caption="Satırın `query`'taki dizeyi içerip içermediğini görmek için işlevsellik ekleme">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-18/src/lib.rs:here}}
```

</Listing>

Şu anda, işlevsellik yüklüyoruz. Kodun derlenmesi için, fonksiyon imzasında belirlediğimiz gibi gövdeden bir değer dönmemiz gerekiyor.

#### Eşleşen Satırları Saklama (Storing Matching Lines)

Bu fonksiyonu bitirmek için, dönmek istediğimiz eşleşen satırları saklamanın bir yoluna ihtiyaç duyuyoruz. Bunun için, `for` döngüsünden önce değiştirilebilir bir vektör yapabilir ve `line`'ı vektörde saklamak için `push` yöntemini çağırabiliriz. `for` döngüsünden sonra, vektörü döneriz ki bu Kod Listesi 12-19'da gösterilir.

<Listing number="12-19" file-name="src/lib.rs" caption="Onları dönebilmek için eşleşen satırları saklama">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:here}}
```

</Listing>

Şimdi `search` fonksiyonu sadece `query`'u içeren satırları dönmeli ve testimiz geçmeli. Testi çalıştıralım:

```console
{{#include ../listings/ch12-an-io-project/listing-12-19/output.txt}}
```

Testimiz geçti, yani çalıştığını biliyoruz!

Bu noktada, testleri geçerek aynı işlevselliği korumak için `search` fonksiyonunun uygulamasını yerefactor etme fırsatlarını düşünebiliriz. `search` fonksiyonundaki kod pek kötü değil, ancak iteratörlerin bazı yararlı özelliklerinden faydalanmıyor. Bunu [Bölüm 13][ch13-iterators]<!-- ignore -->'te bu örneğe döneceğiz ki orada iteratörleri ayrıntılı olarak keşfedeceğiz ve onu nasıl iyileştireceğe bakacağız.

Şimdi tüm program çalışmalı! Deneyelim, önce Emily Dickinson şiirinden tam bir satır dönecek bir kelimeyle: _frog_.

```console
{{#include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/output.txt}}
```

Harika! Şimdi birden fazla satırı eşleşen bir kelimeyle, örneğin _body_ ile deneyelim:

```console
{{#include ../listings/ch12-an-io-project/output-only-03-multiple-matches/output.txt}}
```

Ve son olarak, şiirin hiçbir yerinde olmayan bir kelime için, örneğin _monomorphization_ için hiçbir satır almadığımızdan emin olalım:

```console
{{#include ../listings/ch12-an-io-project/output-only-04-no-matches/output.txt}}
```

Mükemmel! Klasik bir aracın kendi mini versiyonunu inşa ettik ve uygulamaların nasıl yapılandırılması gerektiği hakkında çok şey öğrendik. Ayrıca dosya girdi ve çıktısı, ömürler, test ve komut satırı çözme hakkında biraz da öğrendik.

Bu projeyi tamamlamak için, ortam değişkenleriyle nasıl çalışacağınızı ve standart hataya nasıl yazdıracağınızı kısaca göstereceğiz ki her ikisi de komut satırı programları yazarken yararlıdır.

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[ch11-anatomy]: ch11-01-writing-tests.html#the-anatomy-of-a-test-function
[ch10-lifetimes]: ch10-03-lifetime-syntax.html
[ch3-iter]: ch03-05-control-flow.html#looping-through-a-collection-with-for
[ch13-iterators]: ch13-02-iterators.html