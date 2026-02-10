<!-- Old headings. Do not remove or links may break. -->

<a id="yielding"></a>

### Runtime'a Kontrolü Teslim Etme (Yielding Control to Runtime)

[“İlk Async Programımız”][async-program]<!-- ignore -->
bölümünden hatırlayın ki her await noktasında, Rust bir runtime'a görevi durdurmak ve
beklenen future hazır değilse başka birine geçmek için bir şans verir. Tersi de
doğrudur: Rust yalnızca async blokları durdurur ve bir await noktasında
kontrolü runtime'a geri verir. Await noktaları arasındaki her şey senkronize'dir.

Bu demektir ki eğer bir async bloğu içinde await noktası olmadan çok iş yaparsanız,
o future diğer future'lerin ilerleme yapmasını engeller. Bunu bazen bir future'nin diğer
future'leri _aç bırakma_ (starving) olarak duyabilirsiniz. Bazı durumlarında bu
büyük bir problem olmayabilir. Ancak, biraz pahalı bir kurulum veya uzun süren bir iş
yapıyorsanız veya belirli bir görevi sonsuza kadar yapmaya devam eden bir future'niz varsa,
kontrolü runtime'a ne zaman ve nerede geri vereceğinizi düşünmeniz gerekecek.

Aç bırkma problemini göstermek için uzun süren bir işlemi simüle edelim, sonrasında
bunu çözmeyi inceleyelim. Kod Listesi 17-14, bir `slow` fonksiyonu tanıtır.

<Listing number="17-14" caption="Using `thread::sleep` to simulate slow operations" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-14/src/main.rs:slow}}
```

</Listing>

Bu kod `trpl::sleep` yerine `std::thread::sleep` kullanır ki bu sayede
`slow` çağırmak, şu anki iş parçacığını birkaç milisaniye süre boyunca
engelleyecek. `slow`'yi hem uzun süren hem engelleyen gerçek dünydaki işlemler
yerine kullanabiliriz.

Kod Listesi 17-15'te, bir çift future'de bu tür bir CPU-bağlı işin yapılmasını
simüle etmek için `slow` kullanıyoruz.

<Listing number="17-15" caption="Calling=`slow` function to simulate slow operations" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-15/src/main.rs:slow-futures}}
```

</Listing>

Her future, çok sayıda yavaş işlem yürüttükten sonra yalnızca kontrolü runtime'a geri
verir. Bu kodu çalıştırsanız bu çıktıyı göreceksiniz:

```text
'a' started.
'a' ran for 30ms
'a' ran for 10ms
'a' ran for 20ms
'b' started.
'b' ran for 75ms
'b' ran for 10ms
'b' ran for 15ms
'b' ran for 350ms
'a' finished.
```

Kod Listesi 17-5'teki gibi, `trpl::select` kullanarak iki URL alan future'leri yarıştırdık
ancak iki future'deki `slow` çağrıları arasında hala bir aralama yoktur. `a` future,
`trpl::sleep` çağrısı beklenene kadar tüm işini yapar, sonrasında `b` future kendi
`trpl::sleep` çağrısı beklenene kadar tüm işini yapar ve nihayetinde `a` future tamamlanır.
Her iki future'nin yavaş görevleri arasında ilerleme yapabilmesi için, kontrolü runtime'a geri
verebileceğimiz await noktalarına ihtiyacımız var. Bu, bekleyebileceğimiz bir şeye
ihtiyacımız olduğu anlamına gelir!

Bu tür bir devretmenin Kod Listesi 17-15'te zaten görebiliriz: `a` future'nin sonundaki
`trpl::sleep`'i kaldırsak, `b` future _tamamen_ çalışmadan tamamlanrdı. İşlemlerin
ilerleme yapmasına izin vermek için bir başlangıç noktası olarak `trpl::sleep` fonksiyonunu
kullanmayı deneyelim, Kod Listesi 17-16'da gösterildiği gibi.

<Listing number="17-16" caption="Using `trpl::sleep` to let operations switch off making progress" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-16/src/main.rs:here}}
```

</Listing>

`slow` çağrıları arasına await noktaları ile `trpl::sleep` çağrıları ekledik. Şimdi
iki future'nin işi aralanmış:

```text
'a' started.
'a' ran for 30ms
'b' started.
'b' ran for 75ms
'a' ran for 10ms
'b' ran for 10ms
'a' ran for 20ms
'b' ran for 15ms
'a' finished.
```

`a` future, `b`'ye kontrolü vermeden önce biraz çalışmaya devam ediyor çünkü
`trpl::sleep` çağırmadan önce `slow` çağırıyor ancak bundan sonra future'ler, her biri bir
await noktasına her çarptığında birbirleriyle değişiyorlar. Bu durumda, bunu `slow` çağrısının
sonra her seferinde yaptık ancak işi en anlamlı yolla parçalayabiliriz.

Ancak burada gerçekten _uyumak_ istiyoruz: mümkün olduğunca hızlı ilerleme yapmak istiyoruz.
Sadece kontrolü runtime'a geri vermemiz gerekiyor. Bunu doğrudan `trpl::yield_now` fonksiyonunu
kullanarak yapabiliriz. Kod Listesi 17-17'de, tüm o `trpl::sleep` çağrılarını `trpl::yield_now`
ile değiştiriyoruz.

<Listing number="17-17" caption="Using=`yield_now` to let operations switch off making progress" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-17/src/main.rs:yields}}
```

</Listing>

Bu kod hem niyet hakkında daha açık hem de `sleep` kullanımından önemli derecede daha hızlı
olabilir çünkü `sleep` tarafından kullanılan zamanlayıcıların ne kadar granüler olabileceği
hakkında sınırları vardır. Kullandığımız `sleep` sürümü, örneğin, bir nanosaniyelik
bir `Duration` versek bile, her zaman en az bir milisaniye uyacak. Yine de, modern bilgisayarlar
_hızlıdır_: bir milisaniyede çok şey yapabilirler!

Bu demektir ki async, programınızın başka şeyler ne yaptığına bağlı olarak, hesaplama-bağlı
görevler için bile yararlı olabilir çünkü programın farklı parçaları arasındaki ilişkileri yapılandırmak
için kullanışlı bir araç sağlar (ancak async durum makinesinin genel kat biçiminde).
Bu _işbirlikçi çok görev_ (cooperative multitasking) bir biçimidir ki burada her future,
await noktaları üzerinden ne zaman kontrolü teslim ettiğini belirleme gücüne sahiptir. Bu
nedenle her future de çok fazla süre engellememekten sorumluluğa sahiptir. Bazı Rust tabanlı
gömülü işletim sistemlerinde bu _tek_ çok görev biçimidir!

Gerçek dünya kodunda, elbette her tek çizgide await noktaları ile fonksiyon çağrılarını
değiştirmeyeceksiniz. Bu şekilde kontrolü teslim etmek görecekle ucuz olsa da, bedava değildir.
Çoğu durumda, hesaplama-bağlı bir görevi parçalamaya çalışmak, onu önemli derecede
daha yavaş yapabilir bu yüzden bazen _genel_ performans için bir işlemin kısaca engellemesine
izin vermek daha iyidir. Kodunuzun gerçek performans darboğazlarının ne olduğunu görmek için her
zaman ölçün. Eğer eşzamanlı olmasını beklediğiniz çok işin sıral olarak gerçekleştiğini
_görüyorsaniz_, alttaki dinamik hatırlamak önemlidir!

### Kendi Async Soyutlamamızı Oluşturma (Building Our Own Async Abstractions)

Ayrıca future'leri bir araya getirerek yeni modeller oluşturabiliriz. Örneğin, zaten sahip
olduğumuz async yapı bloklarını kullanarak bir `timeout` fonksiyonu oluşturabiliriz.
Bittiğimizde, sonuç başka bir yapı bloğu olacak ki onu kullanarak daha fazla async soyutlaması
oluşturabiliriz.

Kod Listesi 17-18'de, bu `timeout` fonksiyonunun yavaş bir future ile nasıl çalışacağını beklediğimizi
gösteriyoruz.

<Listing number="17-18" caption="Using our imagined `timeout` to run a slow operation with a time limit" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-18/src/main.rs:here}}
```

</Listing>

Bunu uygulayalım! Başlamak için `timeout`'un API'sini düşünelim:

- Bunu bekleyebilmemiz için kendisi async bir fonksiyon olmalı.
- İlk parametresi çalıştırılacak bir future olmalı. Herhangi bir future ile çalışabilmesi
  için bunu generic yapabiliriz.
- İkinci parametresi beklenilecek en fazla süredir. Eğer bir `Duration` kullanırsak,
  bunu `trpl::sleep`'e geçirmeyi kolaylaştırır.
- Bir `Result` dönmeli. Eğer future başarıyla tamamlanrsa, `Result` future tarafından
  üretilen değer ile `Ok` olacak. Eğer timeout önce geçerse, `Result`, timeout'un
  beklediği süre ile `Err` olacak.

Kod Listesi 17-19'de bu bildirimi gösterir.

<Listing number="17-19" caption="Defining=`signature of=`timeout`" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-19/src/main.rs:declaration}}
```

</Listing>

Bu türler için hedeflerimizi karşılıyor. Şimdi ihtiyacımız _davranışı_ düşünelim:
geçilen future'yi süre ile yarıştırmak istiyoruz. `trpl::sleep`'i kullanarak süre'den bir
zamanlayıcı future yapabiliriz ve `trpl::select` kullanarak o zamanlayıcıyı, çağıran tarafından
geçilen future ile yürütebiliriz.

Kod Listesi 17-20'de, `trpl::select`'in sonucu eşleyerek `timeout` uyguluyoruz.

<Listing number="17-20" caption="Defining=`timeout` with=`select` and=`sleep`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-20/src/main.rs:implementation}}
```

</Listing>

`trpl::select`'in uygulaması adil değil: argümanları geçirildikleri sıra ile
her zaman pollar (diğer `select` uygulamaları rastgele hangi argümanı önce pollayacaklarını
seçer). Bu nedenle, `future_to_try`'yi `select`'e önce geçiriyoruz ki böylece `max_time`
çok kısa bir süre bile olsa tamamlama şansı alsın. Eğer `future_to_try` önce tamamlarsa,
`select`, `future_to_try`'den çıktı ile `Left` döner. Eğer zamanlayıcı önce tamamlarsa,
`select`, zamanlayıcının `()` çıktısı ile `Right` döner.

Eğer `future_to_try` başarır ve bir `Left(output)` alırsak, `Ok(output)` döneriz.
Eğer uyku zamanlayıcı geçerse ve bir `Right(())` alırsak, `()`'yi `_` ile yok sayıp yerine
`Err(max_time)` döneriz.

Böylece, iki başka async yardımcısından yapılan çalışan bir `timeout`'a sahibiz. Kodumuzu
çalıştırsak, timeout'tan sonra hata modunu yazdıracak:

```text
Failed after 2 seconds
```

Çünkü future'ler diğer future'lerle birleşir, daha küçük async yapı bloklarını kullanarak gerçkten güçlü
araçlar oluşturabilirsiniz. Örneğin, aynı yaklaşımı kullanarak timeout'ları yeniden denemelerle
birleştirebilirsiniz ve sırasıyla bunları ağ çağrıları gibi işlemlerle (örneğin
Kod Listesi 17-5'dekiler) kullanabilirsiniz.

Pratikte, doğrudan `async` ve `await` ile ve ikincil olarak `select` gibi fonksiyonlar ve
`join!` gibi makrolar ile çalışacaksınız ve en dıştaki future'lerin nasıl yürütüldüğünü kontrol
edeceksiniz.

Şimdiye kadar aynı anda çoklu future'lerle çalışmanın çok sayıda yolunu gördük. Şimdi,
_akarslar_ ile zamanla çoklu future'lerle sıral olarak nasıl çalışabileceğimize bakacağız.

[async-program]: ch17-01-futures-and-syntax.html#our-first-async-program