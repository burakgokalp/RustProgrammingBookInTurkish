<!-- Old headings. Do not remove or links may break. -->

<a id="turning-our-single-threaded-server-into-a-multithreaded-server"></a>
<a id="from-single-threaded-to-multithreaded-server"></a>

## Tek İş Parçacıklıdan Çok İş Parçacıklıya (From a Single-Threaded to a Multithreaded Server)

Şu anda, sunucu her isteği sırayla işleyecek bu, ilk bağlantı işlenmeyi
bitirene kadar ikinci bir bağlantıyı işlemeyeceği anlamına gelir. Eğer sunucu
daha fazla istek alırsa, bu seri yürütme daha az verimli ve daha az optimal olacak.
Eğer sunucu işlenmesi uzun zaman alan bir istek alırsa, sonraki istekler, uzun
istek bitene kadar beklemek zorunda kalacak, yeni istekler hızlı işlenebilirse bile. Bunu düzeltmemiz
gerekiyor ancak önce sorunu eylemde inceleyelim.

<!-- Old headings. Do not remove or links may break. -->

<a id="simulating-a-slow-request-in-the-current-server-implementation"></a>

### Yavaş Bir İsteği Simüle Etme (Simulating a Slow Request)

Yavaş işlenen bir isteğin, geçerli sunucumuz uygulamasına diğer istekleri nasıl etkileyebileceğini
inceleyelim. Kod Listesi 21-10, _/sleep_ için yanıt vermeden önce sunucunun beş saniye uyumasına
neden olan bir yavaş yanıtı ile işlenmesini uygular.

<Listing number="21-10" file-name="src/main.rs" caption="Simulating a slow request by sleeping for five seconds">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-10/src/main.rs:here}}
```

</Listing>

`if`'den `match`'e geçtik çünkü şimdi üç durumumuz var. Bir dilimin (slice) üzerinde desen eşleştirmesi
için string gerçek (literal) değerlerine karşı açıkça eşlememiz gerekiyor; `match`
eşitlik yöntemi ve referanslama (dereferencing), `==` yöntemi yaptığı gibi
otomatik olarak yapmıyor.

İlk kol, Kod Listesi 21-9'deki `if` bloğu ile aynıdır. İkinci kol,
_/sleep_ isteğini eşler. Bu istek alındığında, sunucu başarılı HTML sayfasını
işlemeden önce beş saniye uyuyacak. Üçüncü kol, Kod Listesi 21-9'deki `else` bloğu ile aynıdır.

Sunucumuzun ne kadar ilkel olduğunu görebilirsiniz: Gerçek kütüphaneler birden çok
istek tanımmasını çok daha az ayrıntılı bir yolla ele alırlar!

Sunucuyu `cargo run` kullanarak başlatın. Sonra, iki tarayıcı penceresi açın: biri
_http://127.0.0.1:7878_ için, diğer bir _http://127.0.0.1:7878/sleep_ için.
Eğer birkaç kez _/_ URI'sine girerseniz, önceden olduğu gibi, hızlı bir şekilde yanıt verdiğini göreceksiniz.
Ancak _/sleep_ girip sonra _/_ yüklerseniz, _/_'nın tam beş saniye uyumasını bitmesini
beklediğini göreceksiniz.

İsteğin, bir yavaş istek ardına birikmesini önlemek için kullanabileceğiniz birden çok
teknik var, Bölüm 17'de yaptığımız gibi async kullanmak; bir uygulayacağımız
tekniç, iş parçacığı havuzu (thread pool).

### Bir İş Parçacığı Havuzu ile İş Hacmini Geliştirme (Improving Throughput with a Thread Pool)

_Bir iş parçacığı havuzu (thread pool)_, görev için hazır ve bekleyen oluşturulmuş iş
parçacıklarının bir grubudur. Program yeni bir görev aldığında, havuzdaki iş
parçacıklarından birini göreve atar ve bu iş parçacığı görevi işleyecektir. Havuzdaki
kalan iş parçacıkları, ilk iş parçacığı görevi işlenirken gelebilecek herhangi bir diğer görevi
ele almak için hazırdır. İlk iş parçacığı görevini işlenmeyi bitirdiğinde, boşta
bekleyen iş parçacıkları havuzuna döner ve yeni bir görevi ele almaya hazırdır. Bir iş parçacığı havuzu,
bağlantıları eşzamanlı (concurrently) işlemenize izin verir, sunucunuzun iş hacmini (throughput)
artırır.

Biz, DoS saldırılarından bizi korumak için havuzdaki iş parçacıklarının sayısını küçük bir
sayıyla sınırlayacağız; programımız her istek için olduğu gibi yeni bir iş parçacığı oluştursa, biri 10
milyon isteğimizi sunucumuza yapması, tüm sunucumuz kaynaklarını kullanmak ve isteklerin
işlenmesini durdurarak çok zarar verebilir.

Sınırsız iş parçacıklarını oluşturmak yerine, bu yüzden havuzda bekleyen sabit sayıda iş
parçacıklar olacak. Gelen istekler işlenmek için havuza gönderilir. Havuz, gelen
isteklerin bir kuyruğunu (queue) koruyacak. Havuzdaki her iş parçacığı, bu kuyruktan bir
isteği alacak, isteği işleyecek ve sonra kuyruktan başka bir istek isteyecektir. Bu tasarımla, _`N`_
adet kadar isteği eşzamanlı işleyebiliriz ki burada _`N`_, iş parçacıklarının sayısıdır. Eğer her
iş parçacığı uzun süren bir isteği yanıtlıyorsa, sonraki istekler yine kuyrukta birikebilir ancak o noktaya ulaşmadan önce
ele alabileceğimiz uzun süren isteklerin sayısını artırdık.

Bu teknik, bir web sunucusunun iş hacmini artırmak için birçok yoldan sadece biridir.
Diğer seçenekler şunları keşfedebilirsiniz: fork/join modeli, tek iş parçacıklı (single-threaded) async I/O
modeli ve çok iş parçacıklı (multithreaded) async I/O modeli. Bu konuyla
ilgileniyorsanız, diğer çözümler hakkında daha fazlasını okuyabilir ve onları uygulamayı
deneyebilirsiniz; Rust gibi düşük seviyeli bir dil ile, bu seçeneklerin tümü mümkündür.

Bir iş parçacığı havuzu uygulamaya başlamadan önce, havuzun nasıl görünmesi gerektiği
hakkında konuşalım. Kod tasarlamakaya çalışırken, önce müşteri arayüzünü (client interface) yazmak
tasarımınızı yönlendirebilir. Kodunuzun API'sini, onu çağırmak istediğiniz şekilde
yapılandırın (structured); sonra, fonksiyonalite o yapı içinde uygulayın,
her şeyi uygulamak ve sonra genel API tasarlamak yerine.

Bölüm 12'deki projemizde test-driven development kullandığımıza benzer olarak,
burada derleyici güdüreli development (compiler-driven development) kullanacağız. Çağırmak
istediğimiz fonksiyonlara kod yazacağız ve sonra derleyiciden hatalara bakarak kodun
çalışmasını sağlamak için sonraki ne değiştirmemiz gerektiğini belirleyeceğiz. Bunu yapmadan önce
ancak, başlangıç noktası olarak kullanmayacağımız tekniği inceleyelim.

<!-- Old headings. Do not remove or links may break. -->

<a id="code-structure-if-we-could-spawn-a-thread-for-each-request"></a>

#### Her İstek İçin Bir İş Parçacığı Oluşturma (Spawning a Thread for Each Request)

Önce, kodumuzun her bağlantı için yeni bir iş parçacığı oluştursa nasıl görünebileceğini
inceleyelim. Daha önce belirtildiği gibi, bu son planımız değil çünkü sınırsız sayıda
iş parçacıkların oluşturulması potansiyel sorunlara sahiptir, ancak bu çalışan çok iş
parçacıklı sunucusu elde etmek için bir başlangıç noktasıdır. Sonra, iyileştirme olarak bir iş
parçacığı havuzu ekleyeceğiz ve iki çözümü karşılaştırmak daha kolay olacaktır.

Kod Listesi 21-11, `main`'de her akış (stream) için `for` döngüsünde yeni bir iş
parçacığı oluşturmak için değişiklikleri gösterir.

<Listing number="21-11" file-name="src/main.rs" caption="Spawning a new thread for each stream">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-11/src/main.rs:here}}
```

</Listing>

Bölüm 16'de öğrendiğiniz gibi, `thread::spawn` yeni bir iş parçacığı oluşturacak ve sonra
yeni iş parçacığındaki closure içindeki kodu çalıştıracak. Bu kodu çalıştırıp tarayıcınızda
_/sleep_ yüklerseniz ve sonra iki daha fazla tarayıcı sekmesinde _/_ girerseniz, gerçekten
_/sleep_'in bitmesini beklemek zorunda olmadığını göreceksiniz. Ancak, daha önce belirttiğimiz gibi bu,
sonunda sistemi, herhangi bir sınır olmadan yeni iş parçacıkları oluşturduğunuz için boğulacaktır.

Ayrıca Bölüm 17'den hatırlayacağınız gibi, bu tam async ve await'in parladığı
sına benzer bir durum! Bunu aklınızda tutarak iş parçacığı havuzu inşa eder ve durumların
async ile nasıl farklı veya benzer görünebileceğini düşünün.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-a-similar-interface-for-a-finite-number-of-threads"></a>

#### Sınırlı Sayıda İş Parçacıkları Oluşturma (Creating a Finite Number of Threads)

İş parçacığı havuzumuzu, bizim kullandığımız API'ye benzer, tanıdık şekilde çalışmasını
istiyoruz ki iş parçacıklarından iş parçacığı havuzuna geçiş, API'mizi kullanan kodda
büyük değişiklikler gerektirmesin. Kod Listesi 21-12, `thread::spawn` yerine
kullanmak istediğimiz `ThreadPool` struct'i için hipotetik (hypothetical) bir arayüz gösterir.

<Listing number="21-12" file-name="src/main.rs" caption="Our ideal `ThreadPool` interface">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-12/src/lib.rs:here}}
```

</Listing>

`ThreadPool::new` kullanarak, bu durumda dört olan yapılandırılabilir sayıda iş parçacıklarıyla yeni bir
iş parçacığı havuzu oluşturuyoruz. Sonra, `for` döngüsünde, `pool.execute`'in,
bizim kapanma için bir closure aldığı ve onu havuzdaki bir iş parçacığına çalıştırması
`thread::spawn`'e benzer bir arayüze sahip. `pool.execute`'i implementememiz gerekiyor ki
closure alsın ve onu havuzdaki bir iş parçacığına çalıştırması için verir. Bu kod henüz
derlenmez ancak derleyicinin bizi nasıl düzelteceğini yönlendirmesi için deneyeceğiz.

<!-- Old headings. Do not remove or links may break. -->

<a id="building-the-threadpool-struct-using-compiler-driven-development"></a>

#### Derleyici Güdüreli Development ile `ThreadPool` Oluşturma (Building `ThreadPool` Using Compiler-Driven Development)

Kod Listesi 21-12'deki değişiklikleri _src/main.rs_'a yapın ve sonra `cargo check`'ten
derleyici hatalarını kullanarak gelişmemizi yönlendirelim. İşte ilk hatamız:

```console
{{#include ../listings/ch21-web-server/listing-21-12/output.txt}}
```

Harika! Bu hata bize bir `ThreadPool` türü veya modüle ihtiyacımız olduğunu söylüyor bu yüzden şimdi
bir tane oluşturacağız. `ThreadPool` uygulamamızın, web sunucumuzun yaptığı iş türünden
bağımsız olacak. Bu yüzden, web sunucusunu sunan iş için `ThreadPool` uygulamamızını
tutmak üzere `hello` crate'ini bir ikili (binary) crate'den bir kütüphane crate'ine değiştirelim.
Kütüphane crate'ine geçtikten sonra, iş parçacığı havuzu kütüphanesini kullanarak herhangi bir işi
bir iş parçacığı havuzu kullanmak isteyebiliriz, sadece web isteklerini sunmak için değil.

_src/lib.rs_ dosyası oluşturun ki şunları içerir ki şu an için en basit `ThreadPool`
struct tanımına sahibiz olabilir:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/lib.rs}}
```

</Listing>


Sonra, kütüphane crate'den `ThreadPool`'i kapsama getirmek için _src/main.rs_'in en üstüne
şunları ekleyerek düzenleyin:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/main.rs:here}}
```

</Listing>

Bu kod hala çalışmaz ancak bir sonraki hatayı ele almamız gerektiğini görmek için tekrar kontrol edelim:

```console
{{#include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/output.txt}}
```

Bu hata, sonra `new` adında bir ilişkili fonksiyon oluşturmamız gerektiğini gösterir.
Ayrıca `new`'in, `4`'ü bir argüman olarak kabul edebileceği ve bir `ThreadPool` örneği
döndürmesi gerektiğini biliyoruz. Hadi şu özelliklere sahip en basit `new` fonksiyonunu
uygulayalım:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/src/lib.rs}}
```

</Listing>

`size` parametresinin türü olarak `usize`'i seçtik çünkü negatif bir iş parçacığı sayısının
bir anlamı yoktur. Ayrıca bu `4`'ü, iş parçacıkları koleksiyonundaki eleman sayısı olarak kullanacağımızı
biliyoruz ki bu, Bölüm 3'teki [“Integer Types” (Tam Sayı Türleri)][integer-types]<!--
ignore --> bölümünde tartışıldığı üzere `usize` türü ne içindir.

Kodu tekrar kontrol edelim:

```console
{{#include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/output.txt}}
```

Şimdi, `ThreadPool` üzerinde bir `execute` yöntemimiz olmadığı için bir hata oluşuyor.

[“Creating a Finite Number of Threads” (Sınırlı Sayıda İş Parçacıkları Oluşturma)](#creating-a-finite-number-of-threads)<!--
ignore --> bölümünden hatırlayacağız gibi, iş parçacığı havuzumuzun `thread::spawn`'e benzer bir arayüze sahip
olmasına karar verdik. Ek olarak, closure verdiğimizi bir iş parçacığına verecek `execute`
fonksiyonunu uygulamalıyız ki aldığı bir iş parçacığındaki hazır iş parçacığına verir.
`ThreadPool` üzerindeki `execute` yöntemini tanımlamalıyız ki bir closure parametresi alacak.
[“Moving Captured Values Out of Closures” (Kapanmalardan Yakalanan Değerleri Taşıma)][moving-out-of-closures]<!--
ignore --> bölümünde, Bölüm 13'te kapana üç farklı trait'i parametre olarak alabileceğimizi
hatırlayabilirsiniz: `Fn`, `FnMut` ve `FnOnce`. Burada hangi tür kapanma kullanacağımıza
karar vermeliyiz. `thread::spawn`'in imzasındaki (signature) sınırları hakkında standart kütüphane belgeleri bize
şunları gösterir:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

`F` tür parametresi, burada ilgilendiğimiz türdür. `T` tür parametresi, dönüş değeri ile ilgilidir
ve biz bununla ilgilenmiyoruz. `spawn`'in parametresindeki `F` üzerinde `FnOnce` trait sınırı
kullandığını görebilirsiniz. Bu muhtemelen bizim de istediğimiz şeydir çünkü `execute`'te
alacağımız argümanı nihayetinde `spawn`'a geçeceğiz. `FnOnce`'in kullanmak istediğimiz trait olduğuna
daha da güvenli olabiliriz çünkü bir isteği çalıştıran iş parçacığı, sadece o isteğin kapanmasını
bir kez çalıştıracak ki bu, `FnOnce`'deki `Once` ile eşleşir.

`F` tür parametresi ayrıca `Send` trait sınırı ve `'static` yaşam süresi (lifetime bound) sahiptir,
bunlar durumumuzda yararlıdır: `Send`'e, kapanmayı bir iş parçacığından diğerine aktarmak ve
`'static`'e, iş parçacığının çalışmasını ne kadar alacağımızı bilmediğimiz için ihtiyaç duyuyoruz.
`ThreadPool` üzerinde `execute` yöntemini, bu sınırlarla `F` türünde genel bir parametre almak için
yaratelim:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-03-define-execute/src/lib.rs:here}}
```

</Listing>

`FnOnce`'den sonra `()` kullanmaya devam ediyoruz çünkü bu `FnOnce`, parametre almayan ve birim
türü `()` dönen bir kapanmayı temsil eder. Fonksiyon tanımlarına benzer olarak, dönüş türü imzadan
çıkarılabilir ancak parametremiz olmasa bile parantez `()`'e ihtiyaç duyarız.
Şu an, `execute` yönteminin en basit uygulamasıdır: Hiçbir şey yapmıyor ancak
sadece kodumuzu derlemeye çalışıyoruz. Tekrar kontrol edelim:

```console
{{#include ../listings/ch21-web-server/no-listing-03-define-execute/output.txt}}
```

Derleniyor! Ancak `cargo run` çalıştırmayı ve tarayıcınızda bir istek yapmayı denerseniz,
bölümün başında gördüğünüz hataları tarayıcıda göreceksiniz. Kütüphanemiz aslında
`execute`'e geçilen closure'ı çağırmıyor!

> Not: Haskell ve Rust gibi katı derleyicileri olan diller hakkında duyabileceğiniz bir söz,
> “Eğer kod derlenirse, çalışır.” Ancak bu söz evrensel olarak doğru değildir. Projemiz derleniyor ancak
> mutlak hiçbir şey yapmıyor! Gerçek, tam bir proje inşa ediyorsanız, bu, kodun derlenmesini _ve_
> istediğiniz davranışa sahip olmasını kontrol etmek için birim testlerini yazmaya başlamak için
> iyi bir zaman olabilir.

Düşünün: Eğer bir gelecek yerine bir kapanma çalıştırsaydık ne farklı olurdu?

#### `new`'deki İş Parçacığı Sayısını Doğrulama (Validating Number of Threads in `new`)

`new` ve `execute` parametreleriyle henüz bir şey yapmıyoruz. Fonksiyonların gövdelerini
istediğimiz davranışla uygulayalım. Başlamak için, `new`'i düşünelim. Daha önce
`size` parametresi için işaretsiz (unsigned) bir tür seçtik çünkü negatif sayıda iş parçacığı olan
bir havuz anlamsız olur. Ancak sıfır iş parçacıklı olan bir havuz da anlamsız olur, ancak sıfır
geçerli bir `usize`'tur. `ThreadPool` örneğini döndürmeden önce `size`'ın sıfırdan büyük olup
olmadığını kontrol etmek için kod ekleyeceğiz ve programın sıfır alırsa `assert!` makrosu
kullanarak paniklemesini sağlayacağız, Kod Listesi 21-13'te gösterildiği gibi.

<Listing number="21-13" file-name="src/lib.rs" caption="Implementing `ThreadPool::new` to panic if `size` is zero">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-13/src/lib.rs:here}}
```

</Listing>

Ayrıca `ThreadPool`'imiz için dokümantasyon ekledik. Fonksiyonumuzun panikleyebileceği
durumlarda hangi durumlarda çağırdığını çağıran bir bölüm ekleyerek Bölüm 14'te
tartışıldığı gibi iyi dokümantasyon pratiklerini izlediğimizi not edin. `cargo doc --open` çalıştırın ve
`ThreadPool` struct'ine tıklayarak `new` için oluşturulan belgelerin nasıl göründüğünü görün!

Burada yaptığımız gibi `assert!` makrosunu eklemek yerine, `new`'i Bölüm 12-9'deki
`Config::build` ile yaptığımız gibi bir `build` fonksiyonuna değiştirip bir `Result` döndürmeyi
deneyebilirsiniz. Ancak bu durumda, herhangi bir iş parçacığı olmayan bir iş parçacığı havuzu
oluşturmayı denemenin, kurtarılamaz bir hata olması gerektiğine karar verdik. Eğer hissediyorsanız,
şu imzaya sahip bir `build` adında fonksiyon yazmayı deneyebilirsiniz:

```rust,ignore
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
```

#### İş Parçacıklarını Depolamak İçin Alan Oluşturma (Creating Space to Store Threads)

Artık havuzda saklamak için geçerli bir sayıda iş parçacığı olduğunu bildiğimize göre,
bu iş parçacıklarını oluşturabilir ve `ThreadPool` struct'inde döndürmeden önce saklayabiliriz.
Ancak bir iş parçacığını nasıl "saklayacağız"? İş parçacığı oluşturmayı
başka bir bakalım. `thread::spawn` imzasına tekrar bakalım:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

`spawn` fonksiyonu, bir `JoinHandle<T>` döner ki burada `T`, kapanmanın döndüğü
türüdür. `JoinHandle`'ı de kullanmayı ve ne olacağına bakalım. Durumumuzda,
iş parçacığı havuzuna geçtiğimiz kapanmalar, bağlantıyı ele alacak ve herhangi bir şey dönmeyecek bu yüzden
`T` birim türü `()` olacaktır.

Kod Listesi 21-14, derlenir ancak henüz herhangi bir iş parçacığı oluşturmuyor.
`ThreadPool` tanımını, `thread::JoinHandle<()>` örneklerinin vektörünü tutacak şekilde değiştirdik,
`size` kapasitesiyle başlatılan ve bazı kod çalıştıran iş parçacıklarını oluşturan bir `for` döngüsü ve
onları tutan bir `ThreadPool` örneği dönerdik.

<Listing number="21-14" file-name="src/lib.rs" caption="Creating a vector for `ThreadPool` to hold threads">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-14/src/lib.rs:here}}
```

</Listing>

Kütüphane crate'indeki `std::thread`'i kapsama getirdik çünkü `ThreadPool`'taki
öğelerin türü olarak `thread::JoinHandle` kullanıyoruz.

Geçerli bir boyut alındığında, `ThreadPool`'umuz `size` öğesini tutabilen yeni bir vektör
oluşturur. `with_capacity` fonksiyonu, `Vec::new` ile aynı işi yapar ancak önemli bir farkla:
vektörde önceden alan ayırır (pre-allocates). Vektörde `size` öğesini saklamamız gerektiğini
bildiğimiz için, bunu önden yapmak, öğeler eklenirken kendini yeniden boyutlandıran
`Vec::new`'den biraz daha verimlidir.

`cargo check`'i tekrar çalıştırdığınızda, başarılı olması gerekir.

<!-- Old headings. Do not remove or links may break. -->
<a id="a-worker-struct-responsible-for-sending-code-from-the-threadpool-to-a-thread"></a>

#### Kodu `ThreadPool`'den Bir İş Parçacığına Gönderme (Sending Code from `ThreadPool` to a Thread)

Kod Listesi 21-14'teki `for` döngüsünde iş parçacıklarının oluşturulması hakkında bıraktığımız
bir yorum. Burada, aslında iş parçacıklarını nasıl oluşturduğumuzu inceleyeceğiz. Standart
kütüphane, iş parçacıkları oluşturmak için `thread::spawn` sağlar ve `thread::spawn`, iş
parçacığı oluşturulur oluşturulmaz oluşturulduğu andan kodu iş parçacığı tarafından çalıştırılmasını bekler.
Ancak bizim durumumuzda, iş parçacıklarını oluşturmak ve onları daha sonra göndereceğimiz
kodu için beklemek istiyoruz. Standart kütüphanenin iş parçacığı uygulaması bunu yapmanın
bir yolunu dahil etmez; biz manuel olarak uygulamalıyız.

Bu davranışı, `ThreadPool` ve iş parçacıkları arasında yeni bir veri yapısı (data structure) tanıtararak
uygulayacağız. Bu veri yapısını _Worker_ olarak adlandıracakğız ki bu, havuz uygulamalarında
(yaygınlar için "pooling implementations" - havuz uygulamaları) kullanılan ortak bir terimdir.
`Worker`, çalıştırılması gereken kodu alacak ve kendi iş parçacığında kodu çalıştıracaktır.

Bir restorandaki çalışan kişileri düşünün: İşçiler, müşterilerden siparişlerin gelmesini beklerler ve
sonra onların siparişlerini almak ve doldurmaktan sorumludurlar.

`ThreadPool`'te `JoinHandle<()>` örneklerinin vektörünü doğrudan saklamak yerine, `Worker`
struct örneklerini saklayacağız. Her `Worker`, tek bir `JoinHandle<()>` örneği tutacaktır.
Sonra, çalıştırılacak kodu alan ve zaten çalışan iş parçacığına gönderecek bir yöntemi,
`Worker` üzerinde tanımlayacağız. Ayrıca her `Worker`'e bir `id` vereceğiz ki havuzdaki
farklı `Worker` örnekleri arasında günlendirme veya hata ayıklama (debugging) yaparkla birbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirberbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirberbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirberbirbirbirbirbirbirbirbirbirbirbirbirbirberbirbirbirbirbirbirberbirbirbirbirbirbirbirbirbirberbirbirbirbirbirbirbirbirbirbirbirberbirbirberbirberbirbirberbirbirberbirbirbirberbirbirberbirberbirberbirberbirberbirberbirbirberbirbirberbirberbirberbirberbirberbirberbirberbirbirberbirberbirberbirberbirbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirbirberbirberbirberbirbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirbirberbirberbirberbirberbirberbirberberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirbirberbirberbirberbirberbirberbirberbirberbirberbirbirberbirberbirberbirberbirberbirbirbirbirberbirbirberbirberbirberbirbirbirberbirbirberbirbirbirbirberbirberbirbirberbirberbirberbirberbirberbirberbirberbirbirberbirbirberbirbirbirbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirbirberbirberbirberbirbirberbirberbirberbirberbirberbirberbirbirberbirberbirberbirberbirberbirberbirbirberbirberbirberbirberbirberbirberbirberbirberbirbirbirberbirbirberbirberbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirbirberbirbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirbirberbirberbirberbirberbirbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirberbirber