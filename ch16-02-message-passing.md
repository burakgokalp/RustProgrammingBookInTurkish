<!-- Old headings. Do not remove or links may break. -->

<a id="using-message-passing-to-transfer-data-between-threads"></a>

## Mesaj Geçiri ile İş Parçacıkları Arasında Veri Transfer Etmek (Transfer Data Between Threads with Message Passing)

Güvenli eşzamanlılığı sağlamak için giderek daha popüler bir yaklaşım mesaj geçiri, iş parçacıklarının veya aktörlerin birbirine veri içeren mesajlar göndererek iletişim kurduğu dur. Burada [Go dili belgelerindeki](https://golang.org/doc/effective_go.html#concurrency) bir slogandaki fikir:"Belleği paylaşarak iletişim kurma; bunun yerine, iletişim kurarak belleği paylaş."

Mesaj gönderen eşzamanlılığı başarmak için, Rust standart kütüphanesi kanalların bir uygulamasını sağlar. Bir _kanal_ (channel), verinin bir iş parçacığından başka birine gönderildiği genel bir programlama kavramıdır.

Bir programlama kanalını, bir suyun yönellendirilmiş bir kanalı gibi, örneğin bir akım veya nehir olarak hayal edebilirsiniz. Eğer bir nehir gibi bir kanala bir kauçuk ördeği gibi bir şey koyarsanız, suyunun aşağı akarak su yolunun sonuna seyahat edecektir.

Bir kanalın iki yarı vardır: bir verici ve bir alıcı. Verici yarısi, kauçuk ördeğiyi nehire koyduğunuz yukarı konumdur ve alıcı yarısi, kauçuk ördeğin aşağı akıda sona erdiği yerdir. Kodunuzun bir kısmı, göndermek istediğiniz veri ile verici üzerinde yöntemleri çagırır ve başka bir kısım, gelen mesajlar için alım ucunu kontrol eder. Bir kanal, verici veya alıcı yarısından biri bırakılırsa _kapalı_ olarak söylenir.

Burada, değerler oluşturup bir kanal üzerinden gönderen bir iş parçacığı ve değerler alıp yazdıran başka bir iş parçacığı olan bir programaya kadar çalısacağız. Özelliği göstermek için iş parçacıkları arasında basit değerler göndermek için bir kanal kullanacağız. Tekniğe alıştığınızda, birbirleriyle iletişim kurması gereken herhangi bir iş parçacığı için kanalları kullanabilirsiniz, örneğin bir sohbet sistemi veya çok sayıda iş parçacığının bir hesaplamanın bölümlerini yaptığı ve sonuçları bir iş parçacığına gönderdiği bir sistem.

Önce, Kod Listesi 16-6'da, bir kanal oluşturacağız ama onunla hiçbir şey yapmayacağız. Rust, kanal üzerinden hangi türde değerler göndermek istediğimizi söyleyemediği için bu henüz derlemeyeceğine dikkat edin.

<Listing number="16-6" file-name="src/main.rs" caption="Creating a channel and assigning the two halves to `tx` and `rx`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-06/src/main.rs}}
```

</Listing>

`mpsc::channel` fonksiyonunu kullanarak yeni bir kanal oluşturuyoruz; `mpsc`, _çoklu üretici, tek tüketici_ anlamına gelir. Kısaca, Rust standart kütüphanesinin kanalları uygulama biçimi şudur ki bir kanalın değerleri üreten birden fazla _gönderen_ ucu olabilir ama bu değerleri tüketen tek bir _alan_ ucu olabilir. Birbirine akan çok akımın büyük bir nehire aktnığını hayal edin: Herhangi bir akımdan aşağı gönderilen her şey nehirin sonunda bir nehire sona erecek. Şimdilik tek bir üretici ile başlayacağız ancak bu örneğin çalısmasını aldığımızda çoklu üreticiler ekleyeceğiz.

`mpsc::channel` fonksiyonu bir tuple (demet) döner, bunun birinci öğesi gönderen uc—and ikinci öğesi alan uc'dir. `_verici_` ve `_alıcı_` için geleneksel olarak `tx` ve `rx` kısaltmalar kullanılır, bu yüzden değişkenlerimizi her ucu belirtmek için böyle adlandırıyoruz. Tuple'ları yok eden bir desen kullanan bir `let` bildirimi kullanıyoruz; Bölüm 19'te `let` bildirimlerinde desen kullanımını ve yok etmeyi tartışacağız. Şimdilik bilin ki `mpsc::channel` tarafından döndürülen tuple'ın parçalarını çıkarmak için bu şekilde bir `let` bildirimi kullanmak uygun bir yaklaşımdır.

Şimdi gönderen ucu oluşturulan iş parçacığına taşıyıp ana iş parçacığı ile iletişim kurmasını sağlamak için bir dize göndermesini sağlayalım, Kod Listesi 16-7'de gösterildiği gibi. Bu, kauçuk ördeğiyi nehirin yukarısına koymak veya bir sohbet mesajını bir iş parçacığından başka birine göndermek gibidir.

<Listing number="16-7" file-name="src/main.rs" caption='Moving `tx` to a spawned thread and sending `"hi"`'>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-07/src/main.rs}}
```

</Listing>

Yine, yeni bir iş parçacığı oluşturmak için `thread::spawn` kullanıyoruz ve sonra oluşturulan iş parçacığının `tx`'yi sahiplenmesi için `move` kullanıyoruz ki böylece oluşturulan iş parçacığı `tx`'ye sahip olur. Oluşturulan iş parçacığının kanal üzerinden mesajlar göndermesi için vericiye sahip olması gerekir.

Vericinin, göndermek istediğimiz değeri alan bir `send` yöntemi vardır. `send` yöntemi bir `Result<T, E>` türü döner bu yüzden alıcı zaten bırakılırsa ve gönderilecek değer için hiçbir yer yoksa, gönderme işlemi bir hata dönecek. Bu örnekte, hatadurumunda paniklemek için `unwrap` çagırıyoruz. Ancak gerçek bir uygulamada, bunu düzgün bir şekilde ele alırdık: Bölüm 9'a gidin ve düzgün hata ele alma stratejilerini gözden geçirin.

Kod Listesi 16-8'de, ana iş parçacığında alıcıdan değer alacağız. Bu, nehirin sonunda sudan kauçuk ördeği almak veya bir sohbet mesajı almak gibidir.

<Listing number="16-8" file-name="src/main.rs" caption='Receiving the value `"hi"` in the main thread and printing it'>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-08/src/main.rs}}
```

</Listing>

Alıcının iki yararlı yöntemi vardır: `recv` ve `try_recv`. Basitliği için _alma_ anlamına gelen `recv`'ü kullanıyoruz ki bu, değer kanal üzerinden gönderilene kadar ana iş parçacığının yürütmesini engelleyip bekleyecektir. Bir değer gönderildiği anda, `recv` onu bir `Result<T, E>` içinde dönecektir. Verici kaptığında, `recv` başka değerlerin gelmeyeceğini sinyallemek için bir hata dönecektir.

`try_recv` yöntemi engellemiyor ama bunun yerine anında bir `Result<T, E>` dönecektir: Bir tane varsa mesajı içeren bir `Ok` değeri ve şu an mesaj yoksa bir `Err` değeri. `try_recv` kullanmak, mesajlar için beklerken bu iş parçacığının yapması gereken başka işleri varsa kullanışlıdır: Sıklıkla `try_recv` çağıran, varsa bir mesajı ele alan ve aksi takdirde bir süreliğe diğer işleri yapan bir döngü yazabiliriz. Sonra tekrar kontrol etmeden önce.

Basitlik için bu örnekte `recv` kullandık; ana iş parçacığının mesajlar için beklemek dışında yapması gereken başka işimiz yok, bu yüzden ana iş parçacığını engellemek uygundur.

Kod Listesi 16-8'deki kodu çalıstırdığımızda, ana iş parçacığından yazdırılan değeri göreceğiz:

```text
Got: hi
```

Mükemmel!

<!-- Old headings. Do not remove or links may break. -->

<a id="channels-and-ownership-transference"></a>

### Kanallar Üzerinden Sahiplik Transfer Etmek (Transferring Ownership Through Channels)

Sahiplik kuralları mesaj göndermede hayati bir rol oynar çünkü size güvenli, eşzamanlı kod yazmanıza yardımcı olur. Rust programlarınız boyunca sahiplik düşünerek eşzamanlı programlamadaki hataları önlemek bir avantajdır. Kanalların ve sahipliğin birlikte sorunları önlemek için nasıl çalıştığını göstermek için bir deney yapalım: Bir kanal üzerinden gönderdikten _sonra_ oluşturulan iş parçacığında bir `val` değeri kullanmaya çalışacağız. Kod Listesi 16-9'deki kodun neden izin verilmediğini görmek için derlemeyi deneyin.

<Listing number="16-9" file-name="src/main.rs" caption="Attempting to use `val` after we've sent it down the channel">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-09/src/main.rs}}
```

</Listing>

Burada, `tx.send` üzerinden gönderdikten sonra `val`'yi yazdırmaya çalışıyoruz. Buna izin vermek kötü bir fikir olur: Bir değer bir başkası iş parçacığına gönderildiği anda, başka iş parçacığı değeri değiştirebilir veya bırakabiliriz. Potansiyele olarak, başka iş parçacığının değişiklikleri tutarsız veya var olmayan veriler nedeniyle hatalara veya beklenmedik sonuçlara yol açabilir. Ancak, Kod Listesi 16-9'deki kodu derlemeyi çalıstırsanız Rust bize bir hata verecektir:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-09/output.txt}}
```

Eşzamanlılık hatamız bir derleme zamanı hatasına neden oldu. `send` fonksiyonu parametresinin sahipliğini alır ve değer alıcıya taşındığında alıcı onun sahipliğini alır. Bu, gönderdikten sonra değeri kaza kullanmamızı engeller; sahiplik sistemi her şeyin yolunda olduğunu kontrol eder.

<!-- Old headings. Do not remove or links may break. -->

<a id="sending-multiple-values-and-seeing-the-receiver-waiting"></a>

### Çoklu Değerler Gönderme (Sending Multiple Values)

Kod Listesi 16-8'deki derledi ve çalıştı ancak bize açıca kanal üzerinden birbirleriyle konuşan iki ayrı iş parçacığını olduğunu göstermedi.

Kod Listesi 16-10'da, Kod Listesi 16-8'deki kodun eşzamanlı çalıştığını kanıtlayan bazı değişiklikler yaptık: Oluşturulan iş parçacığı şimdi birden fazla mesaj gönderecek ve her mesaj arasında bir saniye duraklayacak.

<Listing number="16-10" file-name="src/main.rs" caption="Sending multiple messages and pausing between each one">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-10/src/main.rs}}
```

</Listing>

Bu seferde, oluşturulan iş parçacığının ana iş parçacığına göndermek istediğimiz dizelerin bir vektörü var. Onlar üzerinde yineleme yaparak her birini ayrı ayrı gönderiyoruz ve her birinin arasında, bir saniye `Duration` değeri ile `thread::sleep` fonksiyonunu çağırarak duraklıyoruz.

Ana iş parçacığında, artık `recv` fonksiyonunu açıca çagmıyoruz: Bunun yerine, `rx`'yi bir yineleyici olarak ele alıyoruz. Alınan her değer için, onu yazdırıyoruz. Kanal kaptığında, yineleme sona erecek.

Kod Listesi 16-10'deki kodu çalıstırdığınızda, her satır arasında bir saniyelik bir duraklama ile aşağıdaki çıktıyı görmelisiniz:

```text
Got: hi
Got: from
Got: to
Got: thread
```

Ana iş parçacığındaki `for` döngüsünde duraklayan veya geciktiren kodumuz olmadığı için, ana iş parçacığının oluşturulan iş parçacığından değerler almak için beklediğini söyleyebiliriz.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-multiple-producers-by-cloning-the-transmitter"></a>

### Çoklu Üreticiler Oluşturma (Creating Multiple Producers)

Önce `mpsc`'nin _çoklu üretici, tek tüketici_ için bir kısaltma olduğunu belirttik. Şimdi `mpsc`'i kullanmak için ve Kod Listesi 16-10'deki kodu genişleterek değerleri aynı alıcıya gönderen çoklu iş parçacıkları oluşturalım. Bunu, vericiyi klonlayarak yapabiliriz, Kod Listesi 16-11'de gösterildiği gibi.

<Listing number="16-11" file-name="src/main.rs" caption="Sending multiple messages from multiple producers">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-11/src/main.rs:here}}
```

</Listing>

Bu seferde, ilk oluşturulan iş parçacığını oluşturmadan önce, verici üzerinde `clone` çağırıyoruz. Bu, ilk oluşturulan iş parçacığına geçirebileceğimiz yeni bir verici verecektir. Orijinal vericiyi ikinci oluşturulan iş parçacığına geçiriyoruz. Bu, birbirlerine farklı mesajlar gönderen iki iş parçacığı verir.

Kodu çalıstırdığınızda, çıktınız şöyle görünebilir:

```text
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: you
Got: thread
```

Sisteminize bağlı olarak başka bir sırada değerler görebilirsiniz. Bu, eşzamanlıyı ilginç kilan yanı sıra zor kılan şeydir. `thread::sleep` ile deneyip, farklı iş parçacıklarına çeşitli değerler verirseniz, her çalıştırma daha belirsiz (nondeterministic) olacak ve her seferinde farklı çıktü üretecektir.

Şimdi kanalların nasıl çalıştığına baktık, eşzamanlının farklı bir yöntemine bakalım.