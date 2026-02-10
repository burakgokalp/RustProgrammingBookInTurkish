## İş Parçacıklarını Kullanarak Kodu Aynı Anda Çalıştırma (Using Threads to Run Code Simultaneously)

Çoğu geçerli işletim sisteminde, yürütülen bir programın kodu bir _süreçte_ (process) çalışır ve işletim sistemi birden fazla süreci aynı anda yönetir. Bir programın içinde, aynı anda çalışan bağımsız bölümleri de olabilir. Bu bağımsız bölümleri çalıştıran özelliklere _iş parçacıkları_ (threads) denir. Örneğin, bir web sunucusu aynı anda birden fazla isteğe yanıt verebilmek için birden fazla iş parçacığına sahip olabilir.

Programınızdaki hesaplamayı birden fazla iş parçacığına bölmek ve aynı anda birden fazla görevi çalıştırmak performansı artırabilir ama karmaşıklık da ekler. İş parçacıkları aynı anda çalışabildiği için, farklı iş parçacıkları üzerindeki kodunuzun bölümlerinin hangi sırada çalışacağı doğal bir garanti yoktur. Bu sorunlara yol açabilir, örneğin:

- Yarış durumları, iş parçacıklarının veriye veya kaynaklara tutarsız bir sırada eriştiği
- Kilitlenmeler, iki iş parçacığının birbirini beklediği, her iki iş parçacığının da devam etmesini engellediği
- Sadece belirli durumlarda gerçekleşen ve güvenilir bir şekilde yeniden üretmek ve düzeltmek zor olan hatalar

Rust, iş parçacıklarının kullanmanın negatif etkilerini azaltmaya çalışır ancak çok iş parçacıklı bağlamda programlama yine dikkatli düşünme gerektirir ve tek iş parçacığında çalışan programlardaki farklı bir kod yapısını gerektirir.

Programlama dilleri iş parçacıklarını birkaç farklı şekilde uygular ve çok sayıda işletim sistemi yeni iş parçacıkları oluşturmak için programlama dilinin çağırabileceği bir API sağlar. Rust standart kütüphanesi iş parçacığı uygulamasının _1:1_ modelini kullanır ki bu sayede bir program her bir dil iş parçacığı için bir işletim sistemi iş parçacığı kullanır. 1:1 modeline farklı takaslar yapan diğer iş parçacığı modellerini uygulayan kütüphaneler var. (Bir sonraki bölümde göreceğimiz Rust'un async sistemi eşzamanlılık için başka bir yaklaşım da sağlar.)

### `spawn` ile Yeni Bir İş Parçacığı Oluşturma (Creating a New Thread with `spawn`)

Yeni bir iş parçacığı oluşturmak için, `thread::spawn` fonksiyonunu çağırır ve yeni iş parçacığında çalıştırmak istediğimiz kodu içeren bir kapanış (Bölüm 13'te kapanışlardan konuştuk) geçiririz. Kod Listesi 16-1'deki örnek, ana iş parçacığından bazı metinler ve yeni iş parçacığından başka metinler yazdırır.

<Listing number="16-1" file-name="src/main.rs" caption="Ana iş parçacığı başka bir şey yazdırırken yeni bir iş parçacığı bir şey yazdırmak için oluşturma">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-01/src/main.rs}}
```

</Listing>

Rust programının ana iş parçacığının tamamlandığında oluşturulan tüm iş parçacıklarının kapatılacağına dikkat edin, çalışmayı bitirip bitirmediklerine bakılmaksızın. Bu programın çıktısı her seferinde biraz farklı olabilir ama aşağıdakine benzer olacaktır:

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

`thread::sleep` çağrıları iş parçacığının yürütmesini kısa bir süre durdurmaya zorlar ve farklı bir iş parçacığının çalışmasına izin verir. İş parçacıkları muhtemelen sıraya girecek ama bu garanti edilmez: İşletim sisteminizin iş parçacıklarını nasıl planladığına bağlı. Bu çalıştırmada, ana iş parçacığı önce yazdırdı, ancak oluşturulan iş parçacığından yazdırma bildirimi kodda önce görünüyor. Ve oluşturulan iş parçacığına `i` `9` olana kadar yazdırmasını söylememize rağmen, ana iş parçacığı kapanmadan önce sadece `5`'e kadar geldi.

Bu kodu çalıştırırsanız ve sadece ana iş parçacığından çıktı görürseniz veya herhangi bir örtüşme görmüyorsanız, işletim sisteminin iş parçacıkları arasında geçiş yapması için daha fazla fırsat yaratmak için aralıklardaki sayıları artırmayı deneyin.

<!-- Old headings. Do not remove or links may break. -->

<a id="waiting-for-all-threads-to-finish-using-join-handles"></a>

### Tüm İş Parçacıklarının Bitmesini Bekleme (Waiting for All Threads to Finish)

Kod Listesi 16-1'deki kod genellikle oluşturulan iş parçacığını, ana iş parçacığının bitmesi yüzünden erken durdurur ama iş parçacıklarının hangi sırada çalışacağının garantisi olmadığı için, oluşturulan iş parçacığının hiç çalışacağını da garanti edemeyiz!

Oluşturulan iş parçacığının çalışmaması veya erken bitmesi sorununu `thread::spawn`'in dönüş değerini bir değişkende saklayarak düzeltebiliriz. `thread::spawn`'in dönüş türü `JoinHandle<T>`'dir. `JoinHandle<T>`'e, üzerinde `join` yöntemini çağırdığımızda iş parçacığının bitmesini bekleyen sahip bir değerdir. Kod Listesi 16-2'de, Kod Listesi 16-1'de oluşturduğumuz iş parçacığının `JoinHandle<T>`'ünü nasıl kullanacağımızı ve `main` çıkmadan önce oluşturulan iş parçacığının bitmesini sağlamak için `join`'i nasıl çağıracağımızı gösterir.

<Listing number="16-2" file-name="src/main.rs" caption="İş parçacığının bitene kadar çalıştığını garanti etmek için `thread::spawn`'den `JoinHandle<T>`'ü saklama">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-02/src/main.rs}}
```

</Listing>

Tutucu üzerinde `join` çağırmak, tutucu tarafından temsil edilen iş parçacığı sonlandırılana kadar çalışan iş parçacığını engeller. Bir iş parçacığını _engellemek_ (blocking), iş parçacığının çalışmasını yapmasından veya çıkmasından engellendiği anlamına gelir. `join` çağrısını ana iş parçacığının `for` döngüsünden sonra koyduğumuz için, Kod Listesi 16-2'yi çalıştırmak buna benzer çıktı üretmelidir:

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

İki iş parçacığı sıralı bir şekilde devam ediyor ama ana iş parçacığı `handle.join()` çağrısı yüzünden bekliyor ve oluşturulan iş parçacığı bitene kadar sona ermiyor.

Fakat `main` içinde `for` döngüsünden önce `handle.join()`'i şu şekilde taşıdığımızda ne olduğunu görelim:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/no-listing-01-join-too-early/src/main.rs}}
```

</Listing>

Ana iş parçacığı oluşturulan iş parçacığının bitmesini bekleyecek ve sonra kendi `for` döngüsünü çalıştıracak ki çıktı artık örtüşmeyecek, burada gösterildiği gibi:

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

Küçük detaylar, örneğin `join`'in nerede çağrıldığı gibi, iş parçacıklarınızın aynı anda çalışıp çalışmamasını etkileyebilir.

### İş Parçacıkları ile `move` Kapanışlarını Kullanma (Using `move` Closures with Threads)

Sıklıkla `thread::spawn`'e geçirilen kapanışlarla `move` anahtar kelimesini kullanacağız çünkü kapanış o zaman ortamından kullandığı değerleri sahiplenecek ve böylece bu değerlerin sahipliğini bir iş parçacığından başka birine transfer edecek. Bölüm 13'teki ["Referansları Yakalama veya Sahipliği Taşıma"][capture]<!-- ignore --> bölümünde, kapanışlar bağlamında `move`'i tartıştık. Şimdi `move` ve `thread::spawn` arasındaki etkileşime daha yoğunlaşacağız.

Kod Listesi 16-1'de `thread::spawn`'e geçirdiğimiz kapanışın hiçbir argüman almadığına dikkat edin: Oluşturulan iş parçacığının kodunda ana iş parçacığından herhangi bir veri kullanmıyoruz. Oluşturulan iş parçacığında ana iş parçacığından veri kullanmak için, oluşturulan iş parçacığının kapanışı ihtiyaç duyduğu değerleri yakalamalı. Kod Listesi 16-3, ana iş parçacığında bir vektör oluşturma ve oluşturulan iş parçacığında kullanma girişini gösterir. Ancak, bir an içinde göreceğiniz gibi bu henüz çalışmayacak.

<Listing number="16-3" file-name="src/main.rs" caption="Ana iş parçacığı tarafından oluşturulan bir vektörü başka bir iş parçacığında kullanma girişimi">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-03/src/main.rs}}
```

</Listing>

Kapanış `v`'i kullandığı için, `v`'yi yakalayacak ve onu kapanışın ortamının bir parçası yapacak. Çünkü `thread::spawn` bu kapanışı yeni bir iş parçacığında çalıştırıyor, yeni iş parçacığında içinde `v`'ye erişebilmeliyiz. Ancak bu örneği derlediğimizde, şu hatayı alıyoruz:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-03/output.txt}}
```

Rust `v`'yi nasıl yakalayacağını _çıkararımını_ yapar ve çünkü `println!` sadece `v`'ye bir referansa ihtiyaç duyuyor, kapanış `v`'yi ödünç almaya çalışır. Ancak, bir sorun var: Rust, oluşturulan iş parçacığının ne kadar süre çalışacağını söyleyemiyor, bu yüzden `v`'ye referansın her zaman geçerli olup olmayacağını bilmiyor.

Kod Listesi 16-4, `v`'ye geçerli olmayan bir referans olma olasılığı daha yüksek olan bir senaryo sağlar.

<Listing number="16-4" file-name="src/main.rs" caption="Kapanışın `v`'ye bir referans yakalamaya çalıştığı ve ana iş parçacığının `v`'yi bıraktığı bir iş parçacığı">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-04/src/main.rs}}
```

</Listing>

Rust'in bu kodu çalıştırmamıza izin verdiğini düşünürsek, oluşturulan iş parçacığının hiç çalışmadan hemen arkaya konma olasılığı olur. Oluşturulan iş parçacığı içinde `v`'ye bir referans vardır ama ana iş parçacığı Bölüm 15'te tartıştığımız `drop` fonksiyonunu kullanarak hemen `v`'yi bırakır. Sonra, oluşturulan iş parçacığı yürütmeyi başladığında, `v` artık geçerli değil, ona referans da geçersiz. Hayır!

Kod Listesi 16-3'teki derleyici hatasını düzeltmek için, hata mesajının tavsiyesini kullanabiliriz:

```text
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++
```

Kapanıştan önce `move` anahtar kelimesini ekleyerek, kapanışı değerleri ödünç almak yerine kullandığı değerleri sahiplenmesini zorlarız. Kod Listesi 16-3'e Kod Listesi 16-5'te gösterilen değişiklik, istediğimiz gibi derleyecek ve çalışacaktır.

<Listing number="16-5" file-name="src/main.rs" caption="Kapanışın kullandığı değerleri sahiplenmesini zorlamak için `move` anahtar kelimesini kullanma">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-05/src/main.rs}}
```

</Listing>

Kod Listesi 16-4'teki, ana iş parçacığının `drop` çağırdığı kodu aynı şeyi düzeltmek için çalışmaya ikna edilebilirsiniz. Ancak, bu düzeltme çalışmayacak çünkü Kod Listesi 16-4'ün yapmaya çalıştığı farklı bir nedenle yasaktır. Eğer kapanışa `move` eklerdik, `v`'yi kapanışın ortamına taşırız ve ana iş parçacığında üzerinde artık `drop` çağıramazdık. Bunun yerine şu derleyici hatasını alırdık:

```console
{{#include ../listings/ch16-fearless-concurrency/output-only-01-move-drop/output.txt}}
```

Rust'un sahiplik kuralları bizi tekrar kurtardı! Kod Listesi 16-3'ten bir hata aldık çünkü Rust muhafazakardı ve iş parçacığı için sadece `v`'yi ödünç alıyordu ki bu demektir ki ana iş parçacığı teorik olarak oluşturulan iş parçacığının referansını geçersiz kılabilir. Rust'a `v`'nin sahipliğini oluşturulan iş parçacığına taşımasını söyleyerek, ana iş parçacığının artık `v`'yi kullanmayacağını garanti ediyoruz. Eğer Kod Listesi 16-4'ü aynı şekilde değiştirirsek, ana iş parçacığında `v`'yi kullanmaya çalıştığımızda sahiplik kurallarını ihlal ediyoruz. `move` anahtar kelimesi, Rust'un muhafazakar ödünç alma varsayılanını geçersiz kılar; sahiplik kurallarını ihlal etmemize izin vermez.

Şimdi iş parçacıklarının ne olduğunu ve iş parçacığı API tarafından sağlanan yöntemleri ele aldık, iş parçacıklarını kullanabileceğimiz bazı durumlara bakalım.

[capture]: ch13-01-closures.html#capturing-references-or-moving-ownership