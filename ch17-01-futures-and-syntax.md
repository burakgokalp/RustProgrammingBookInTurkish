## Futures ve Async Sözdizimi (Futures and Async Syntax)

Rust'te asenkron programlamanın ana öğeleri _futures_ ve Rust'ın
`async` ve `await` anahtar kelimeleridir.

Bir _future_, şimdi hazır olmayabileceği ancak gelecekte belirli bir noktada
hazır olacağı bir değerdir. (Bu aynı kavram birçok dilde görülür, bazen
_task_ veya _promise_ gibi diğer isimlerle.) Rust, farklı asenkron işlemlerinin
farklı veri yapıları ile ancak ortak bir arayüz ile uygulanabilmesi için bir
`Future` özelliği sunar. Rust'te, futures, `Future` özelliğini uygulayan
türlerdir. Her future, yapılmış ilerleme ve "hazır"ın ne anlama geldiği
hakkında kendi bilgisini tutar.

Bloklara ve fonksiyonlara `async` anahtar kelimesini uygulayarak onların
durdurulabilir ve sürdürülebileceğini belirtebilirsiniz. Bir async bloğu veya fonksiyonu
içinde, bir future _await etmek_ için `await` anahtar kelimesini kullanabilirsiniz
(yani, onun hazır olmasını beklemek). Bir async bloğu veya fonksiyonu içinde herhangi bir
noktada bir future await ederseniz, o blok veya fonksiyonun durabileceği ve
sürdürülebileceği bir olası noktadır. Bir future ile hazır olup olmadığını görmek için kontrol
etme süreci _polling_ (pollama) olarak adlandırılır.

Bazı diğer diller, örneğin C# ve JavaScript, de async programlama için `async` ve
`await` anahtar kelimelerini kullanır. Bu dillerle tanıkdığınızsa, Rust'ın sözdizimi
nasıl ele aldığında birkaç önemli fark fark edebilirsiniz. Bu iyi bir sebep için, göreceğimiz!

Async Rust yazarken, çoğu zaman `async` ve `await` anahtar kelimelerini
kullanıyoruz. Rust, bunları `Future` özelliği kullanarak eşdeğer koda derler,
tıpkı `for` döngülerini `Iterator` özelliği kullanarak eşdeğer koda derlerdiği gibi.
Ancak Rust `Future` özelliği sunduğu için, ihtiyaç duyduğunuzda kendi veri türleriniz
için de uygulayabilirsiniz. Bu bölüm boyunca göreceğimiz çok fonksiyon, kendi
`Future` uygulamalarına sahip türler döner. Özelliğin tanımına bölümün sonunda
döneceğiz ve çalışma biçimi hakkında daha fazlasına dalacağız ancak bu ileri
gidebilmemiz için yeterli ayrıntıdır.

Bu biraz soyut gibi görünebilir bu yüzden ilk async programımızı yazalım: küçük
bir web scraper. Komut satırından iki URL geçeceğiz, her ikisini eşzamanlı olarak
alacağız ve her ikisinden önce biteninin sonucunu döneceğiz. Bu örnekte
biraz yeni sözdizimi olacak ancak merak etmeyin—giderken ihtiyaç duyduğunuz her şeyi
açıklayacağız.

## İlk Async Programımız (Our First Async Program)

Bu bölümün odagının async öğrenmekte, ekosistemin parçalarıyla uğraşmada
sürdürmek için `trpl` crate'ini oluşturduk (`trpl`, "The Rust Programming
Language" için kısaltmadır). İhtiyaç duyacağınız tüm türler, özellikler ve
fonksiyonları yeniden ihraç eder, temel olarak [`futures`][futures-crate]<!-- ignore -->
ve [`tokio`][tokio]<!-- ignore --> crate'lerinden. `futures` crate, async kodu için
Rust deneiminin resmi evidir ve aslında `Future` özelliğinin orijinal olarak
tasarlandığı yerdir. Tokio, bugün Rust'te en yaygın kullanılan async runtime'dir,
özellikle web uygulamalar için. Başka mükemmel runtime'lar var ve amaçlarınız için
daha uygun olabilirler. `trpl` kapa altında `tokio` crate'ni kullanıyoruz çünkü
iyice test edilmiş ve yaygın olarak kullanılıyor.

Bazı durumlarda, `trpl` ayrıca orijinal API'leri yeniden adlandırır veya sasar
böylece bu bölüme ilgili ayrıntılara odaklanmanızı sağlar. Crate'in ne yaptığı
anlamak istiyorsanız, [kaynak kodunu][crate-source] kontrol etmenizi teşvik ederiz.
Her yeniden ihraçin hangi crate'ten geldiğini görebileceksiniz ve crate'in ne
yapdiğini açıklayan çok yorum bıraktık.

`hello-async` adında yeni bir ikilik projesi oluşturun ve `trpl` crate'ni bir
bağımlılık olarak ekleyin:

```console
$ cargo new hello-async
$ cd hello-async
$ cargo add trpl
```

Şimdi `trpl` tarafından sağlanan çeşitli parçaları kullanarak ilk async programımızı
yazabiliriz. İki web sayfası alıp, her birinden `<title>` öğesini çekip hangi
sayfanın tüm süreci önce bitiyorsa onun başlığını yazdıran küçük bir komut satırı
aracı oluşturacağız.

### page_title Fonksiyonunu Tanımlama

Sayfa URL'si parametre olarak alan, ona bir isteği yapan ve `<title>`
öğesinin metnini dönen (bkz. Kod Listesi 17-1) bir fonksiyonla başlayalım.

<Listing number="17-1" file-name="src/main.rs" caption="Defining an async function to get=title element from an HTML page">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-01/src/main.rs:all}}
```

</Listing>

Önce, `page_title` adında bir fonksiyon tanımlayıp `async` anahtar
kelimesiyle işaretliyoruz. Sonra geçirilen ne URL ise `trpl::get` fonksiyonunu
kullanıyoruz ve yanıtı beklemek için `await` anahtar kelimesini ekliyoruz. `yanıtı`'nın
metnini almak için, onun `text` yöntemini çağırıyoruz ve bir kez daha `await`
anahtar kelimesiyle bekliyoruz. Bu iki adımda asenkron. `get` fonksiyonu için,
sunucunun yanıtıın ilk parçasını geri göndermesini beklemek zorundayız ki bu HTTP
başlıkları, çerezler vb. içerebilir ve yanıtı gövdesinden ayrı teslim edilebilir.
Özellikle gövde çok büyükse, tümü gelmesi biraz zaman alabilir. Tüm
yanıtıın gelmesini beklemek zorunda olduğumuzdan, `text` yöntemi de asenkrondur.

Bu iki future da açıkça await etmek zorundayız çünkü Rust'teki futures _lazy_
(tembel): Onlara `await` anahtar kelimesiyle sorana kadar hiçbir şey yapmazlar.
(Aslında, bir future kullanmazsanız Rust bir derleyici uyarısı gösterecek.) Bu,
Bölüm 13'te [Yineleyiciler ile Öğe Serisi İşleme][iterators-lazy]<!-- ignore -->
bölümünde yineleyiciler üzerine tartışmayı hatırlatabilir. Yineleyiciler, siz
onların `next` yöntemini çağırana kadar hiçbir şey yapmazlar—doğrudan veya `for`
döngüleri veya `next`'i kapakta kullanan `map` gibi yöntemler kullanarak.
Benzer şekilde, futures da siz açıkça sorana kadar hiçbir şey yapmazlar. Bu tembellik,
Rust'ın async kodun gerçekten ihtiyaç duyana kadar çalışmasını önlemesine izin verir.

> Not: Bu, Bölüm 16'de [spawn ile Yeni Bir İş Parçacığı Oluşturma]
> [thread-spawn]<!-- ignore --> bölümünde gördüğümüz `thread::spawn` ile
> gördüğümüz davranıştan farklıdır. Orada diğer bir iş parçacığına geçtiğimiz
> kapanğı hemen çalışmaya başladı. Ayrıca çok diğer dillerin async'e yaklaşımından da
> farklıdır. Ancak Rust'ın performans garantilerini sağlayabilmesi, yineleyicilerde
> olduğu gibi önemlidir.

`response_text`'ye sahip olduktan sonra, onu `Html::parse` kullanarak bir `Html`
türü örneği olarak ayrıştırabiliriz. Ham bir dizge yerine, artık HTML'yi daha zengin bir
veri yapısı olarak kullanabileceğimiz bir veri türümüz var. Özellikle, belirli bir CSS
seçicisinin ilk örneğini bulmak için `select_first` yöntemini kullanabiliriz. `"title"`
dizgesini geçirerek, belgede ilk `<title>` öğesini alacağız, eğer bir tan varsa. Eşleşen
herhangi bir öğe olmayabileceğinden `select_first` bir `Option<ElementRef>` döner. Son olarak,
madde varsa madde üzerinde bir şey yapmamayan yoksa hiçbir şey yapmamayı sağlayan
`Option::map` yöntemini kullanıyoruz. (Burada bir `match` ifadesi de
kullanabilirdik ancak `map` daha idyomatik.) `map`'e sağladığımız fonksiyon gövdesinde,
`title` üzerinde `inner_html` çağırarak içeriğini, bir `String` olan, alıyoruz. Hepsinden
bittiğinde, bir `Option<String>`'ye sahibiz.

Rust'ın `await` anahtar kelimesinin beklediğiniz ifadenin _sonuna_ geçtiğini,
önüne değil fark edin. Yani bu bir _son ek_ (postfix) anahtar kelimesidir.
Diğer dillerde `async` kullandıysanız alışkınınızdan farklı gelebilir ancak Rust'te bu,
yöntem zincirlerini çalışmayı çok daha hoş hale getirir. Sonuç olarak, `page_title`'nin
gövdesini, Kod Listesi 17-2'de gösterildiği gibi, `trpl::get` ve `text`
fonksiyon çağrılarını aralarına `await` ile zincirleyecek şekilde değiştirebiliriz.

<Listing number="17-2" file-name="src/main.rs" caption="Chaining with=`await` keyword">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-02/src/main.rs:chaining}}
```

</Listing>

Böylece, ilk async fonksiyonumuzı başarıyla yazdık! `main` içinde çağrmak için bazı
kod ekleden önce, yazdığımızı ve ne anlama geldiği hakkında biraz daha konuşalım.

Rust, `async` anahtar kelimesiyle işaretli bir _blok_ gördüğünde, onu `Future`
özelliğini uygulayan benzersiz anonim bir veri türüne derler. Rust, `async` ile
işaretli bir _fonksiyon_ gördüğünde, onu gövdesi bir async bloğu olan asenkron
olmayan bir fonksiyona derler. Bir async fonksiyonunun dönüş türü, derleyicinin o async
bloğu için oluşturduğu anonim veri türünün türüdür.

Bu nedenle, `async fn` yazmak, dönüş türü bir future dönen bir fonksiyon yazmaya
eşdeğerdir. Derleyici için, Kod Listesi 17-1'teki `async fn page_title` gibi bir
fonksiyon tanımı kabaca aşağıdaki asenkron olmayan fonksiyonuna eşdeğerdir:

```rust
# extern crate trpl; // mdbook test için gerekli
use std::future::Future;
use trpl::Html;

fn page_title(url: &str) -> impl Future<Output = Option<String>> {
    async move {
        let text = trpl::get(url).await.text().await;
        Html::parse(&text)
            .select_first("title")
            .map(|title| title.inner_html())
    }
}
```

Dönüştürülmüş sürümün her parçasından geçelim:

- Bölüm 10'te [Parametreler Olarak Özellikler][impl-trait]<!-- ignore -->
  bölümünde tartıştığımız `impl Trait` sözdizimini kullanıyor.
- Dönen değer, ilişkili bir `Output` türü ile `Future` özelliğini uygular.
  `Output` türünün `Option<String>` olduğuna dikkat edin ki bu, `page_title`'nin `async fn`
  sürümünün orijinal dönüş türü ile aynıdır.
- Orijinal fonksiyonun gövdesinde çağrılan tüm kod bir `async move`
  bloğu içine sarılmış. Blokaların ifadeler olduğunu hatırlayın. Bu tüm blok,
  fonksiyondan dönen ifadedir.
- Bu async bloğu, tam olarak tarif edildiği gibi, `Option<String>` türüne sahip bir değer
  üretir. Bu değer, dönüş türündeki `Output` türüyle eşleşir. Daha önce gördüğünüz
  diğer bloklar gibidir.
- Yeni fonksiyon gövdesi, `url` parametresini nasıl kullandığı nedeniyle bir `async move`
  bloğudur. (`async` ile `async move` arasındaki bölümün ileride çok daha
  fazla konuşacağız.)

Şimdi `main` içinde `page_title`'i çağabiliriz.

<!-- Old headings. Do not remove or links may break. -->

<a id ="determining-a-single-pages-title"></a>

### Bir Runtime ile Async Fonksiyon Yürütme (Executing an Async Function with a Runtime)

Başlamak için, Kod Listesi 17-3'te gösterildiği gibi tek bir sayfanın başlığını
alacağız. Maalesef, bu kod henüz derlenmez.

<Listing number="17-3" file-name="src/main.rs" caption="Calling=`page_title` function from=`main` with a user-supplied argument">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-03/src/main.rs:main}}
```

</Listing>

Bölüm 12'deki [Komut Satırı Argümanları Kabul Etme][cli-args]<!-- ignore -->
bölümünde kullanıdığımız aynı deseni izliyoruz. Sonra URL argümanını `page_title`'e
geçiyoruz ve sonucu bekliyoruz. Future tarafından üretilen değer bir `Option<String>`
olduğu için, sayfanın `<title>` olup olmadığına göre farklı mesajlar yazdırmak için bir
`match` ifadesi kullanıyoruz.

`await` anahtar kelimesini kullanabileceğimiz tek yer async fonksiyonlar veya bloklardadır
ve Rust bizi özel `main` fonksiyonunu `async` olarak işaretlemeye izin vermez.

```text
error[E0752]: `main` function is not allowed to be `async`
 --> src/main.rs:6:1
  |
6 | async fn main() {
  | ^^^^^^^^^^^^^^^ `main` function is not allowed to be `async`
```

`main`'in `async` olarak işaretlenememeği nedeni async kodunun bir _runtime'a_
ihitiyaç duymasıdır: Asenkron kodun yürütülmesinin ayrıntılarını yöneten bir Rust
crate'i. Bir programın `main` fonksiyonu bir runtime _başlatabilir_ ancak kendisi bir
runtime değildir. (Bunu neden böyle olduğuna dair biraz daha fazla göreceğiz.)
Async kodu yürüten her Rust programında, async kodu yürüten bir runtime
ayarladığı en az bir yer vardır.

Async'i destekleyen çoğu dil bir runtime içerir ancak Rust içermez. Bunun
yerine, her biri amaçına uygun farklı dengelemeleri olan çok farklı async runtime'lar vardır.
Örneğin, çok CPU çekirde ve çok RAM olan yüksek verimlilikli bir web sunucusu,
tek bir çekirge, az RAM ve yığın tahsisi yeteneği olmayan bir mikrodenetleyici ile çok
farklı ihtiyaçlara sahiptir. Bu runtime'ları sağlayan crate'ler ayrıca genellikle
dosya veya ağ I/O gibi ortak işlevselliğin async sürümlerini sunar.

Burada ve bu bölümün geri kalanı boyunca, `trpl` crate'inden `block_on`
fonksiyonunu kullanacağız ki bu bir future argüman olarak alır ve bu future
tamamlanana kadar şu an iş parçacığını engeller. Sahne arkasında, `block_on`
çağırmak, geçilen future'yi yürütmek için kullanılan `tokio` crate'ni kullanarak bir
runtime ayarlar (trpl crate'inin `block_on` davranışı diğer runtime crate'lerinin
`block_on` fonksiyonlarına benzerdir). Future tamamladıktan sonra, `block_on`,
future ürettiği ne değer döner.

`page_title` tarafından dönen future'yi doğrudan `block_on`'e geçebilir ve tamamlandıktan
sonra, Kod Listesi 17-3'te yapmayı denediğimiz gibi sonuçlan `Option<String>` ile match
yapabiliriz. Ancak bu bölümdeki çoğu örnekte (ve gerçek dünyadaki çoğu async kodu),
yalnızca bir async fonksiyon çağrısından daha fazlasını yapacağımız, bu yüzden bunun
yerine Kod Listesi 17-4'teki gibi bir `async` bloğu geçip açıkça `page_title`
çağrısının sonucunu bekleyeceğiz.

<Listing number="17-4" caption="Awaiting an async block with=`trpl::block_on`" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook test does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-04/src/main.rs:run}}
```

</Listing>

Bu kodu çalıştırdığımızda, başta beklediğimiz davranışı alıyoruz:

```console
$ cargo run -- "https://www.rrust-lang.org"
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/async_await 'https://www.rust-lang.org'
The title for https://www.rust-lang.org was
            Rust Programlama Dili
```

Vay—sonunda çalışan async kodumuz var! Ancak iki siteyi birbirlerine karşı
yarışmak için kod eklemeden önce, dikkatimizi kısaca futures'ın nasıl çalıştığına
geri dönelim.

Her _await noktası_—yani, kodun `await` anahtar kelimesini kullandığı her
nokta—kontrolün runtime'a geri teslim edildiği bir yeri gösterir. Bu işmesi için
Rust'in async bloğu dahilindeki durumu takip etmesi gerekir ki böylece runtime bazı
başka işi başlatabilecek ve hazır olduğunda ilkini tekrar ilerletmeye dönebilir.
Bu görünmez durum makinesi gibidir, her await noktasında güncel durumu kaydetmek için
böyle bir enum yazmışsınız gibi:

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-state-machine/src/lib.rs:enum}}
```

Her durum arasında geçiş için kodu elle yazmak sıkıcı ve hataya eğilimli olur,
özellikle daha sonra koda daha fazla işlevsellik ve daha fazla durum eklemek
ihtiyaç duyduğunuzda. Şanslı ki, Rust derleyicisi async kod için durum makinesi
veri yapılarını otomatik olarak oluşturur ve yönetir. Veri yapıları etrafındaki normal
ödünç ve sahiplik kuralları geçerli ve mutlulu ki, derleyici ayrıca bunları
kontrol eder bize kullanışlı hata mesajları sunar. Bölümün ileride bunlardan birkaçına
alışacağız.

Son olarak, bu durum makinesini yürüten bir şey olmalı ve bu şey bir runtime'dir.
(Bu nedenle runtimelere bakarken _yürütücüler_ (executors) bahsına
gelebilirsiniz: bir yürütücü, async kodunu yürütmekten sorumlu olan runtime'ın bir
parçasıdır.)

Şimdi derleyicinin Bölüm 17-3'te bizi neden `main`'in kendisini bir async
fonksiyon yapmaktan engellediğini görebilirsiniz. `main`, bir async fonksiyon olsaydı,
başka bir şeyin `main`'in döndüğü her ne ise için durum makinesi yönetmeliydi ancak
`main` programın başlangıç noktasıdır! Bunun yerine, `main` içinde `trpl::block_on`
fonksiyonunu çağırarak bir runtime ayarladık ve `async` bloğu tarafından dönen future'yi
bitene kadar yürüttük.

> Not: Bazı runtime'lar, bir async `main` fonksiyonu _yazabilemeniz_ için makrolar
> sağlar. Bu makrolar, `async fn main() { ... }`'i yaptıları `fn main`'a
> yeniden yazarlar ki bu bizim Bölüm 17-4'te elle yaptığımız şeyi yapar:
> bir future'yi tamamlanaya kadar yürüten bir fonksiyon çağırmak, tıpkı
> `trpl::block_on`'ün yaptığı gibi.

Şimdi bu parçaları bir araya getirelim ve eşzamanlı kodu nasıl yazabileceğimizi
görelim.

<!-- Old headings. Do not remove or links may break. -->

<a id="racing-our-two-urls-against-each-other"></a>

### İki URL'yi Birbirine Karşı Eşzamanlı Olarak Yarıştırma (Racing Two URLs Against Each Other Concurrently)

Kod Listesi 17-5'te, komut satırından geçirilen iki farklı URL ile `page_title`
çağırıyoruz ve hangi future önce biterse seçerek yarıştıyoruz.

<Listing number="17-5" caption="Calling=`page_title` for two URLs to see which returns first" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-05/src/main.rs:all}}
```

</Listing>

Kullanıcı tarafından sağlanan her URL için `page_title` çağırarak başlıyoruz. Sonuçan
future'ları `title_fut_1` ve `title_fut_2` olarak kaydediyoruz. Hatırlayın, bunlar henüz
hiçbir şey yapmazlar çünkü futures tembel ve henüz onları await etmedik. Sonra
future'ları `trpl::select`'e geçiriyoruz ki bu, geçilen future'lardan hangisinin önce
bittiğini belirten bir değer döner.

> Not: Kapa altında, `trpl::select` daha genel bir `select` fonksiyonu üzerinde
> inşa edilmiştir ki bu `futures` crate'te tanımlanmıştır. `futures` crate'inin
> `select` fonksiyonu `trpl::select` fonksiyonunun yapamayacağı çok şeyi yapabilir
> ancak şimdilik atlayabileceğimiz bazı ek karmaşıklığa da sahiptir.

Her iki future meşruca "kazanabilir" bu nedenle `Result` dönmek mantıklı değildir.
Bunun yerine `trpl::select`, daha önce görmediğimiz bir tür olan `trpl::Either` döner.
`Either` türü `Result`'e biraz benzerdir ki iki durum vardır. Ancak `Result`'ün aksine,
`Either` içinde başarı veya başarsızlık kavramı gömülüdür. Bunun yerine "bir veya diğeri"
belirtmek için `Left` ve `Right` kullanır:

```rust
enum Either<A, B> {
    Left(A),
    Right(B),
}
```

`select` fonksiyonu, ilk argüman kazanırsa o future'ın çıktısı ile `Left`,
diğer _o_ kazanırsa ikinci future argümanının çıktısı ile `Right` döner. Bu,
fonksiyon çağrılırken argümanların göründük sırayı eşleşir: ilk argüman ikinci
argümanın solundadır.

Ayrıca `page_title`'i, geçirilen aynı URL'yi dönecek şekilde güncelliyoruz. Bu
sayede, önce dönen sayfanın `<title>` yoksa hala anlamlı bir mesaj yazdırabiliriz. Bu
bilgiyle birlikte, `println!` çıktısını, hangi URL önce bittiğini ve varsa, o URL'deki
web sayfasının `<title>`'nin ne olduğunu belirtmek için güncelliyoruz.

Şimdi küçük çalışan bir web scraper oluşturdunuz! Birkaç URL seçin ve komut satırı
aracı çalıştırın. Bazı sitelerin diğerlerinden sürekli olarak daha hızlı olduğunu, diğer
durumlarda ise hızlı sitenin çalıştırmadan çalıştırmaya değiştiğini fark edebilirsiniz. Daha
önemli olarak, futurelarla çalışma temellerini öğrendiniz bu yüzden şimdi async ile ne
yapabileceğimize daha derin dalabiliriz.

[impl-trait]: ch10-02-traits.html#traits-as-parameters
[iterators-lazy]: ch13-02-iterators.html
[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[cli-args]: ch12-01-accepting-command-line-arguments.html

[crate-source]: https://github.com/rust-lang/book/tree/main/packages/trpl
[futures-crate]: https://crates.io/crates/futures
[tokio]: https://tokio.rs