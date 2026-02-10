<!-- Old headings. Do not remove or links may break. -->

<a id="concurrency-with-async"></a>

## Async ile Eşzamanlılık Uygulama (Applying Concurrency with Async)

Bu bölümde, Bölüm 16'de iş parçacıkları ile ele aldığımız aynı eşzamanlılık
sorunlarn bazılarına async uygulayacağız. Çünkü orada çok ana fikir hakkında konuştuk,
bu bölümde iş parçacıkları ve futures arasındaki ne farklı odaklanacağız.

Çoğu durumda, async kullanarak eşzamanlılık için kullanılan API'lar iş
parçacıkları için kullanılanlara çok benzerdir. Diğer durumlarda, sonuç olarak oldukça
farklı olurlar. API'lar iş parçacıkları ve async arasında _benzer_ görünseler bile,
genellikle farklı davranışlara sahip olurlar—ve neredeyse her zaman farklı performans
karakteristiklerine sahiptirler.

<!-- Old headings. Do not remove or links may break. -->

<a id="counting"></a>

### `spawn_task` ile Yeni Bir Görev Oluşturma (Creating a New Task with `spawn_task`)

[“spawn ile Yeni Bir İş Parçacığı Oluşturma”][thread-spawn]<!-- ignore -->
bölümünde Bölüm 16'de ele aldığımız ilk işlem iki ayrı iş parçacığında
yukarı sayma idi. Bunu async kullanarak yapalım. `trpl` crate, `thread::spawn`
API'sine çok benzer görünen bir `spawn_task` fonksiyonu ve `thread::sleep` API'sinin
async sürümü olan bir `sleep` fonksiyonu sağlar. Bunları birleştirerek sayma örneğini
uyulayabiliriz, Kod Listesi 17-6'da gösterildiği gibi.

<Listing number="17-6" caption="Creating a new task to print one thing while main task prints something else" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-06/src/main.rs:all}}
```

</Listing>

Başlangıç noktamız olarak, üst düzey fonksiyonumuz async olabilmesi için `main`
fonksiyonumuzu `trpl::block_on` ile ayarladık.

> Not: Bölümde bu noktadan itibaren, her örnek `main` içinde
> `trpl::block_on` ile bu tam sarmalama kodunu içerecek bu yüzden
> genellikle bunu atlayacağız tıpkı `main` ile yaptığımız gibi.
> Kodunuza dahil etmeyi unutmayın!

Sonra o bloğun içinde iki döngü yazıyoruz, her biri bir `trpl::sleep`
çağrısı içeriyor ki bu bir sonraki mesajı göndermeden önce yarım saniye (500 milisaniye) bekler.
Biri döngüsünü `trpl::spawn_task` gövdesine koyuyoruz diğeri ise üst düzey
bir `for` döngüsü. Ayrıca `sleep` çağrılarından sonra bir `await` ekliyoruz.

Bu kod iş parçacığı tabanlı uygulamaya benzer davranıyor—inanırken kendi
terminalinizde mesajların farklı bir sırada görünebileceği gerçeği dahil:

```text
hi number 1 from second task!
hi number 1 from first task!
hi number 2 from first task!
hi number 2 from second task!
hi number 3 from first task!
hi number 3 from second task!
hi number 4 from first task!
hi number 4 from second task!
hi number 5 from first task!
```

Bu sürüm, main async bloğu gövdesindeki `for` döngüsü bitince hemen
durdurur çünkü `spawn_task` tarafından üretilen görev `main` fonksiyonu bittiğinde kapatılır.
Eğer onun görevin bitişine kadar çalışmasını istiyorsanız, ilk görevin bitmesini beklemek
için bir join handle kullanmanız gerekecek. İş parçacıkları ile, iş parçacığı bitene kadar
"kilitlemek" için `join` yöntemini kullandık. Kod Listesi 17-7'de, görev
handle'i bir future olduğu için aynı şeyi yapmak için `await` kullanabiliriz.
Dönüş türü bir `Result` olduğu için, onu bekledikten sonra da paketi açıyoruz.

<Listing number="17-7" caption="Using `await` with a join handle to run a task to completion" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-07/src/main.rs:handle}}
```

</Listing>

Bu güncellenmiş sürüm _iki_ döngü de bitinceye kadar çalışır:

```text
hi number 1 from second task!
hi number 1 from first task!
hi number 2 from first task!
hi number 2 from second task!
hi number 3 from first task!
hi number 3 from second task!
hi number 4 from first task!
hi number 4 from second task!
hi number 5 from first task!
hi number 6 from first task!
hi number 7 from first task!
hi number 8 from first task!
hi number 9 from first task!
```

Şimdiye kadar, async ve iş parçacıkları bize benzer sonuçlar veriyor gibi,
sadece farklı sözdizimi ile: join handle üzerinde `join` çağırmak yerine
`await` kullanmak ve `sleep` çağrılarını beklemek.

Büyük fark şudur ki bunu yapmak için başka bir işletim sistemi iş parçacığı
üretemememize gerek yok. Aslında, burada bir görev üretmemize bile gerek yok. Çünkü async
blokları anonim future'lara derlenir, her döngüyü bir async bloğa koyabiliriz ve
`trpl::join` fonksiyonu kullanarak her ikisini de bitişe kadar çalıştırabiliriz.

Bölüm 16'de [Tüm İş Parçacıklarının Bitmesini Bekleme][join-handles]<!-- ignore -->
bölümünde, `std::thread::spawn` çağırdığınızda dönen `JoinHandle` türü üzerinde
`join` yöntemini nasıl kullanacağımızı gösterdik. `trpl::join` fonksiyonu
benzerdir ancak futures için. İki future verdiğinizde, geçtiğiniz her iki future de
_bitiğinde_ onların çıktısını içeren bir demet üreten tek yeni bir future üretir.
Bu nedenle, Kod Listesi 17-8'de, `fut1` ve `fut2` bitmesini beklemek için
`trpl::join` kullanıyoruz. `fut1` ve `fut2`'yi beklemiyoruz, bunun yerine
`trpl::join` tarafından üretilen yeni future'yi bekliyoruz. Çıktısını yok sayıyoruz
çünkü bu sadece iki birim değerini içeren bir demet.

<Listing number="17-8" caption="Using `trpl::join` to await two anonymous futures" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-08/src/main.rs:join}}
```

</Listing>

Bunu çalıştırdığımızda, her iki future'nin bitişe kadar çalıştığını görüyoruz:

```text
hi number 1 from first task!
hi number 1 from second task!
hi number 2 from first task!
hi number 2 from second task!
hi number 3 from first task!
hi number 3 from second task!
hi number 4 from first task!
hi number 4 from second task!
hi number 5 from first task!
hi number 6 from first task!
hi number 7 from first task!
hi number 8 from first task!
hi number 9 from first task!
```

Şimdi, her seferde tam aynı sırayı göreceksiniz ki bu iş parçacıkları ile ve
Kod Listesi 17-7'deki `trpl::spawn_task` ile gördüğümüzden çok farklıdır.
Bunun nedeni `trpl::join` fonksiyonunun _adil_ (fair) olmasıdır, yani her future'yi
eşit sıklıkla kontrol eder, aralarında değişir ve diğeri hazır olduğu halde birinin
önne koşmasına izin vermez. İş parçacıkları ile, işletim sistemi hangi iş parçacığını
kontrol edeceğini ve ne kadar süre çalışmasına izin vereceğini karar verir. Async Rust ile,
runtime hangi görevi kontrol edeceğine karar verir. (Pratikte, ayrıntılar karmaşık
çünkü bir async runtime eşzamanlılığı yönetmesinin bir parçası olarak işletim sistemi iş
parçacıklarını kapa kullanabilir, bu yüzden adaleti garanti etmek bir runtime için daha fazla
iş olabilir—ancak yine de mümkündür!) Runtime'lar herhangi belirli işlem için adaleti
garantisi zorunlu değildir ve genellikle adaleti isteyip istemediğinizi seçmenizi
sağlayan farklı API'lar sunarlar.

Future'leri beklemek üzerine bu varyasyonlardan bazılarını deneyin ve ne
yaptıklarını görün:

- Döngülerden birinden veya her ikisinden async bloğu çıkarın.
- Her async bloğunu tanımladıktan hemen bekleyin.
- Yalnızca ilk döngüyü bir async bloğu ile sasar ve ikinci döngü
  gövdesinden sonra sonucan future'yi bekleyin.

Ekstra bir meydan okuma için, kodu çalıştırmadan önce her durumda çıktının ne
olacağını anlamaya çalışabilir misiniz?

<!-- Old headings. Do not remove or links may break. -->

<a id="message-passing"></a>
<a id="counting-up-on-two-tasks-using-message-passing"></a>

### Mesaj Geçirme ile İki Görev Arasında Veri Gönderme (Sending Data Between Two Tasks Using Message Passing)

Future'ler arasında veri paylaşmak da tanıdık olacak: tekrar mesaj geçiri
kullanacağız ancak bu kez async sürümü olan türler ve fonksiyonlarla. Bölüm
16'deki [Mesaj Geçirme ile İş Parçacıkları Arasında Veri Transfer][message-passing-threads]<!-- ignore -->
bölümünde yaptığımızdan biraz farklı bir yol alacağız ki bu, iş parçacığı tabanlı ve
future tabanlı eşzamanlılık arasındaki bazı temel farkları gösterelim. Kod Listesi 17-9'da,
tek bir async bloğuyla başlayacağız—açık ayrı bir görev üretmeyen ayrı bir iş
parçacığı ürettiğimiz gibi.

<Listing number="17-9" caption="Creating an async channel and assigning two halves to=`tx` and=`rx`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-09/src/main.rs:channel}}
```

</Listing>

Burada, Bölüm 16'de iş parçacıkları ile kullandığımız çoklu-üretici,
tek-tüketici kanalı API'sinin async sürümü olan `trpl::channel` kullanıyoruz.
API'nin async sürümü iş parçacığı tabanlı sürümden biraz farklıdır: değiştirilemez
bir `rx` alıcı yerine değiştirilebilir bir alıcı kullanır ve `recv` yöntemi doğrudan değer
üretmek yerine beklememiz gereken bir future üretir. Şimdi gönderici üzerinden alıcıya mesajlar
gönderebiliriz. Ayrı bir iş parçacığı veya bile bir görev üretmenize gerek olmadığını fark edin;
sadece `rx.recv` çağrısını beklememiz gerekiyor.

`std::mpsc::channel` içindeki senkronize `Receiver::recv` yöntemi bir mesaj
alana kadar engeller. `trpl::Receiver::recv` yöntemi engellemez çünkü asenkrondur.
Bunun yerine engelleyip, ya bir mesaj alınana veya kanalın gönderici tarafı kapanana kadar
kontrolü runtime'a geri verir. Buna karşılık, `send` çağrısını beklemiyoruz çünkü
engellemez. Engellemesine gerek yok çünkü gönderdiğimiz kanal sınırsız (unbounded).

> Not: Bu tüm async kodunun bir `trpl::block_on` çağrısındaki bir async
> bloğunda çalıştığından, içindeki her şey engellemekten kaçınabilir. Ancak,
> _dışında_ olan kod `block_on` fonksiyonu dönerken engellenir. Bu,
> `trpl::block_on` fonksiyonun tam noktasıdır: senkron ve async kod
> arasındaki nerede geçiş yapacağınızı seçmenizi sağlar.

Bu örnekte iki şeye dikkat edin. İlk olarak, mesaj hemen gelecektir. İkinci
olarak, burada bir future kullansa bile henüz eşzamanlılık yok. Listede her şey
sıra ile gerçekleşiyor tıpkı hiç future dahil olmasaydı.

İlk kısmı ele alalım, Kod Listesi 17-10'da gösterildiği gibi, bir dizi mesaj
gönderecek ve aralarında uyuyacağız.

<Listing number="17-10" caption="Sending and receiving multiple messages over async channel and sleeping with an=`await` between each message" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-10/src/main.rs:many-messages}}
```

</Listing>

Mesajları göndermenin yanı sıra, onları almamız gerekir. Bu durumda, kaç mesajın
geldiğini bildiğimiz için, elle bunu `rx.recv().await` dört kez çağırarak yapabiliriz.
Ancak gerçek dünyada genellikle _bilinmeyen_ bir sayıda mesajı bekleyeceğiz bu yüzden
daha fazla mesaj olmadığına karar verene kadar beklemeye devam etmemiz gerekecek.

Kod Listesi 16-10'da, senkronize bir kanaldan alınan tüm ögeleri işlemek için bir
`for` döngüsü kullandık. Rust henüz _asenkron olarak üretilmiş_ bir öge serisi ile
bir `for` döngüsü kullanmanın bir yolu yoktur bu yüzden daha önce görmediğimiz bir döngü
kullanmamız gerekir: `while let` koşullu döngüsü. Bu, Bölüm 6'deki [“if let ve
let...else ile Özlü Kontrol Akışı”][if-let]<!-- ignore --> bölümünde gördüğümüz `if let`
yapısının döngü sürümüdür. Belirttiği desen değeri eşleştikçe döngü çalışmaya
devam eder.

`rx.recv` çağrısı bir future üretir ki biz bekliyoruz. Runtime, hazır olana kadar
future'yi durdurur. Bir mesaj geldiğinde, future mesaj her geldiğinde `Some(message)` olarak
çözülür. Kanal kapandığında, _herhangi_ mesaj gelmiş olmasına bakmaksızın,
future bunun yerine daha fazla değer olmadığını ve bu yüzden pollamayı durdurmayı—yani
beklemeyi durdurmayı—belirtmek için `None` olarak çözülür.

`while let` döngüsü tüm bunları bir araya getirir. `rx.recv().await` çağırmanın
sonucu `Some(message)` ise mesaja erişiyoruz ve döngü gövdesinde kullanabiliriz,
tıpkı `if let` ile yapabileceğimiz gibi. Sonuç `None` ise döngü biter. Döngü
her seferinde bittiğinde, bir await noktasına tekrar çarpıp runtime'ın başka bir mesaj
gelene kadar onu tekrar durdurmalar.

Şimdi kod başarıyla tüm mesajları gönderiyor ve alıyor. Maalesef, hala birkaç
sorun var. Birincisi, mesajlar yarım saniye aralıklarla gelmiyor. Hepsi aynı anda,
programı başlattıktan 2 saniye (2.000 milisaniye) sonra geliyor. İkincisi, bu
program da asla çıkmaz! Bunun yerine, yeni mesajlar için sonsuza bekler.
Bunu <kbd>ctrl</kbd>-<kbd>C</kbd> ile kapatmanız gerekecek.

#### Bir Async Bloğu İçindeki Kod Doğrusal Çalışır (Code Within One Async Block Executes Linearly)

Tam gecikmeden sonra tüm bir anda mesajların neden geldiğini, her biri
arasında gecikmelerle gelmesi yerine, inceleyelim. Belirli bir async bloğu içinde,
koddaki `await` anahtar kelimelerinin göründüğü sıra da program çalıştığında yürütülen
sırada.

Kod Listesi 17-10'te yalnızca bir async blok var bu yüzden içindeki her şey
doğrusal çalışıyor. Hala eşzamanlılık yok. Tüm `tx.send` çağrıları gerçekleşiyor,
tüm `trpl::sleep` çağrıları ve onların ilgili await noktalarıyla karışık. O zaman
`while let` döngüsü `recv` çağrılarındaki `await` noktalarından herhangi birinden geçebilir.

İstediğimiz davranışı, yani her mesaj arasındaki uyku gecikmesi, elde etmek
için `tx` ve `rx` işlemlerini kendi async bloklarına koymamız gerekir,
Kod Listesi 17-11'de gösterildiği gibi. Sonra `trpl::join` kullanarak her birini
ayrı yürütebilir runtime, tıpkı Kod Listesi 17-8'deki gibi. Bir kez daha, `trpl::join`
çağrısının sonucunu bekliyoruz, bireysel future'leri değil. Eğer bireysel
future'leri sırada beklerdik, doğrusal bir akışta sonlandık—tam yapmak istemediğimiz şey.

<Listing number="17-11" caption="Separating `send` and `recv` into their own=`async` blocks and awaiting futures for those blocks" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-11/src/main.rs:futures}}
```

</Listing>

Kod Listesi 17-11'deki güncellenmiş kod ile, mesajlar 500 milisaniye
aralıklarla yazdırılıyor, hepsi 2 saniye sonra bir hepsi aynı anda olmak yerine.

#### Bir Async Bloğa Sahiplik Taşıma (Moving Ownership Into an Async Block)

Program hala asla çıkmaz çünkü `while let` döngüsünün `trpl::join` ile
etkileme biçimi:

- `trpl::join` tarafından dönen future, ona geçilen _iki_ future de bittikçe tamamlanır.
- `tx_fut` future, `vals` içinde son mesajı gönderdikten sonra uyuduğu bittiğinde tamamlanır.
- `rx_fut` future `while let` döngüsü bittiğinde bitmez.
- `while let` döngüsü, beklediği `rx.recv` `None` ürettiğinde bitecek.
- `rx.recv` beklemek yalnızca kanalın diğer tarafı kapandığında `None` döner.
- Kanal yalnızca `rx.close` çağırdığımızda veya gönderici tarafı olan `tx`
  bırakıldığında kapanır.
- `rx.close` hiçbir yerde çağırmıyoruz ve `tx`, `trpl::block_on`'e geçirilen
  en dıştaki async blok bitene kadar bırakılmayacak.
- Blok bitemez çünkü `trpl::join` tamamlanmasını engelliyor ki bu bizi
  bu listenin başına geri götürür.

Şu anda, mesajları gönderdiğimiz async bloğu yalnızca `tx`'yi _ödünç alıyor_
çünkü mesaj göndermek sahiplik gerektirmez ancak `tx`'yi o async bloğa _taşıyabilirsek_,
o blok bittiğinde bırakılır. Bölüm 13'deki [Referansları Yakalama veya Sahiplik Taşıma]
[capture-or-move]<!-- ignore --> bölümünde, kapanımlarla birlikte `move` anahtar kelimesini nasıl
kullanacağınızı öğrendiniz ve Bölüm 16'deki [İş Parçacıkları ile `move` Kapanımları Kullanma]
[move-threads]<!-- ignore --> bölümünde tartıştığımız gibi, iş parçacıkları ile çalışırken
genellikle verileri kapanımlara taşımanız gerekir. Aynı temel dinamikler async blokları
için de geçerlidir bu yüzden `move` anahtar kelimesi kapanımlarla çalıştığı gibi async
bloklarla da çalışır.

Kod Listesi 17-12'de, mesajları göndermek için kullanılan bloğu `async`'dan
`async move`'e değiştiriyoruz.

<Listing number="17-12" caption="A revision of code from Listing 17-11 that correctly shuts down when complete" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-12/src/main.rs:with-move}}
```

</Listing>

Kodun _bu_ sürümünü çalıştırdığımızda, son mesaj gönderildiğinde ve alındığında
zarifçe kapanır. Şimdi, birden fazla future'den veri göndermek için ne değiştirmemiz
gerekine bakalım.

#### `join!` Makrosu ile Bir Sayıda Future Katılma (Joining a Number of Futures with `join!` Macro)

Bu async kanalı da çoklu-üretici bir kanaldır bu yüzden birden fazla future'den
mesaj göndermek istiyorsak `tx` üzerinde `clone` çağırabiliriz, Kod Listesi 17-13'de
gösterildiği gibi.

<Listing number="17-13" caption="Using multiple producers with async blocks" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-13/src/main.rs:here}}
```

</Listing>

Önce, `tx`'yi klonlayarak ilk async bloğun dışında `tx1` oluşturuyoruz. `tx1`'i
o bloğa önce `tx` ile yaptığımız gibi taşıyoruz. Sonra, orijinal `tx`'i _yeni_ bir
async bloğa taşıyoruz ki burada biraz daha yavaş gecikme ile daha fazla mesaj gönderiyoruz.
Yeni async bloğu mesajları alma async bloğundan sonraya koymuş oluyoruz ancak ondan önce
de olabilir. Anahtar, future'ların beklenildiği sıra, oluşturuldukları sıra değil.

Mesaj gönderen her iki async bloğunun da `async move` blokları olması gerekir ki her
ikisinin bloklar bittiğinde hem `tx` hem `tx1` bırakılır. Aksi takdirde, başladığımız
aynı sonsuz döngüye geri sonlanırız.

Son olarak, `trpl::join`'den `trpl::join!`'a geçiyoruz ilave future'yi
yönetmek için: `join!` makrosu, derleme zamanında future sayısını bildiğiniz gelecek
gelecek gelecek future sayısını bekler. Bu bölümün ilerisinde bilinmeyen sayıda future'ların
bir koleksiyonunu beklemeyi tartışacağız.

Şimdi her iki gönderen future'den tüm mesajları görüyoruz ve çünkü gönderen
future'ler gönderdikten sonra biraz farklı gecikmeler kullanıyorlar, mesajlar da o farklı
aralıklarda alınıyor:

```text
received 'hi'
received 'more'
received 'from'
received 'the'
received 'messages'
received 'future'
received 'for'
received 'you'
```

Future'ler arasında veri göndermek için mesaj geçiri nasıl kullanılacağını, bir async
bloğu içindeki kodun nasıl sıralı çalıştığını, sahipliği bir async bloğa nasıl taşıyacağınızı
ve çoklu future'leri nasıl katılacağınızı inceledik. Şimdi, runtime'a başka bir göreve
geçebileceğini nasıl ve ne zaman söylemeniz gerektiğini tartışalım.

[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[join-handles]: ch16-01-threads.html#waiting-for-all-threads-to-finish
[message-passing-threads]: ch16-02-message-passing.html
[if-let]: ch06-03-if-let.html
[capture-or-move]: ch13-01-closures.html#capturing-references-or-moving-ownership
[move-threads]: ch16-01-threads.html#using-move-closures-with-threads