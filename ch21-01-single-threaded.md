## Tek İş Parçacıklı (Single-Threaded) Bir Web Sunucusu Oluşturma

Önce tek iş parçacıklı (single-threaded) bir web sunucusunun çalışmasını sağlayarak başlayacağız.
Başlamadan önce, web sunucuları oluşturmakla ilgili protokollerin hızlı bir genel
bakışına bakalım. Bu protokollerin detayları bu kitabın kapsamının dışındadır ancak kısa bir genel
bakış, ihtiyacınız olan bilgiyi verecektir.

Web sunucularında yer alan iki ana protokol _Hypertext Transfer Protocol_ _(HTTP)_ ve
_Transmission Control Protocol_ _(TCP)_'dir. Her iki protokol de _isteğin-yanıt (request-response)_
protokolleridir, bu, bir _istemci (client)_ istekleri başlatır ve bir _sunucu (server)_
istekleri dinler ve istemciye bir yanıt sağlar. Bu isteklerin ve yanıtların içerikleri
protokoller tarafından tanımlanır.

TCP, bilginin bir sunucudan diğerine nasıl gittiğini detaylarıyla açıklayan daha düşük
seviyeli bir protokoldur ancak o bilginin ne olduğunu belirtmez. HTTP, isteklerin ve yanıtların
içeriğini tanımlayarak TCP üzerine kurulur. Teknik olarak HTTP'i diğer protokollerle kullanmak mümkündür ancak
büyük çoğunlukla durumda, HTTP verilerini TCP üzerinden gönderir. TCP ve HTTP istekleri ve yanıtlarının
bayt (raw bytes) ile çalışacağız.

### TCP Bağlantısını Dinleme (Listening to the TCP Connection)

Web sunucumuz bir TCP bağlantısını dinlemeye ihtiyaç duyuyor bu yüzden çalışacağımız ilk
parça bu. Standart kütüphane buna izin veren bir `std::net` modülü sunar. Normal şekilde yeni bir
proje yapalım:

```console
$ cargo new hello
     Created binary (application) `hello` project
$ cd hello
```

Şimdi _src/main.rs_ içine Kod Listesi 21-1'deki kodu girin. Bu kod, gelen TCP akışları (streams)
için yerel adres `127.0.0.1:7878`'de dinleyecek. Gelen bir akış aldığında,
`Connection established!` yazdıracak.

<Listing number="21-1" file-name="src/main.rs" caption="Listening for incoming streams and printing a message when we receive a stream">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-01/src/main.rs}}
```

</Listing>

`TcpListener` kullanarak, adres `127.0.0.1:7878`'de TCP bağlantılarını dinleyebiliriz.
Adreste, iki noktadan önceki bölüm bilgisayarınızı temsil eden bir IP adresidir (bu her bilgisayarda aynıdır ve
yazarların bilgisayarını özel olarak temsil etmez) ve `7878` porttur. Bu portu iki nedenden seçtik:
HTTP normal olarak bu portta kabul edilmez bu yüzden sunucumuz, makinenizde çalışıyor olabileceğiniz diğer herhangi
bir web sunucusuyla çakışma olasılığı düşüktür ve 7878, bir telefonda yazılan _rust_'tır.

Bu senaryoda `bind` fonksiyonu, yeni bir `TcpListener` örneği döneceği gibi
`new` fonksiyonu gibi çalışır. Fonksiyon `bind` deniyor çünkü ağda (networking), dinlemek için bir
porta bağlanmak "bir porta bağlanma" olarak bilinir.

`bind` fonksiyonu bir `Result<T, E>` döner ki bu, bağlanmanın başarısız olabileceğini
gösterir, örneğin, programımızın iki örneğini çalıştırırsak ve böylece aynı portu dinleyen iki programımız olur.
Sadece öğrenme amaçları için temel bir sunucu yazdığımız için, bu tür hataları ele almaya
endişemeyeceğiz; bunun yerine, hatalar olursa programı durdurmak için `unwrap` kullanacağız.

`TcpListener` üzerindeki `incoming` yöntemi bize bir akış dizisi (daha spesifik olarak `TcpStream` türünden
akışlar) veren bir yineleyici (iterator) döner. Tek bir _akış (stream)_, bir istemci ile sunucu
arasındaki açık bir bağlantıyı temsil eder. _Bağlantı (connection)_, istemcinin sunucuya bağlandığı, sunucunun bir
yanıt oluşturduğu ve sunucunun bağlantıyı kapattığı tam istek ve yanıt sürecinin adıdır. Bu nedenle,
`TcpStream`'den okuyacağız ki istemcinin ne gönderdiğini görelim ve sonra yanıtı akışa
yazıp veriyi istemciye geri gönderelim. Genel olarak, bu `for` döngüsü her bağlantıyı sırayla
işleyecek ve bizim ele almamız için bir akış dizisi üretecek.

Şimdilik, akışın ele almamız, akışın herhangi bir hatası varsa programı sonlandırmak için `unwrap`
çağırmaktan ibaret; eğer herhangi bir hata yoksa, program bir mesaj yazdırır. Bir sonraki kod listesindeki başarı
durumu için daha fazla işlevsellik ekleyeceğiz. Bir istemci sunucuya bağlandığında `incoming` yönteminden
hatalar alabileceğimiz neden, aslında bağlantıların üzerinde yineleme yapmıyoruz. Bunun yerine,
_bağlantı denemeleri (connection attempts)_ üzerinde yinelemeyiz. Bir dizi nedenden, birçoğu işletim sistemi
spesifik olan bağlantı başarısız olabilir. Örneğin, birçok işletim sistemi destekleyebileceği eşzamanlı açık bağlantı sayısına
bir sınıra sahiptir; bu sayı ötesinde yeni bağlantı denemeleri, bazı açık bağlantılar kapanana kadar bir
hata üretecektir.

Bu kodu çalıştırmayı deneyelim! Terminalde `cargo run` çağırın ve sonra bir web tarayıcısında
_127.0.0.1:7878_ yükleyin. Tarayıcı, sunucu şu anda herhangi bir veri geri göndermediği için "Connection reset"
gibi bir hata mesajı göstermeli. Ancak terminalinize baktığınızda, tarayıcının sunucuya bağlandığında yazdırılan
birkaç mesaj görmelisiniz!

```text
     Running `target/debug/hello`
Connection established!
Connection established!
Connection established!
```

Bazen tek bir tarayıcı isteği için birden fazla mesajın yazdırıldığını göreceksiniz; sebep,
tarayıcının sayfayı ve tarayıcı sekmesinde görünen diğer kaynaklar gibi _favicon.ico_ simgesi için bir
istek de yapmış olması olabilir.

Ayrıca, sunucu herhangi bir veri ile yanıt vermediği için tarayıcının sunucuya birden çok
bağlanmaya çalışıyor olması da mümkündür. `stream` kapsam dışına gittiğinde ve döngünün sonunda düşürüldüğünde
(drop), bağlantı `drop` uygulamasının bir parçası olarak kapanır. Tarayıcılar bazen kapanmış bağlantıları
yeniden denemeyle uğraşırlar çünkü sorun geçici olabilir.

Tarayıcılar ayrıca bazen herhangi bir istek göndermeden sunucuya birden çok bağlantı açar
böylece eğer *daha sonra* istekler gönderirlerse, bu istekler daha hızlı gerçekleşebilir. Bu oluştuğunda,
sunucumuz her bağlantıyı görecektir, o bağlantı üzerinden herhangi bir istek olup olmamasına bakmaksızın. Örneğin,
Chrome tabanlı tarayıcıların birçok versiyonu bunu yapar; bunu özel gezinme modu (private browsing mode) kullanarak veya
farklı bir tarayıcı kullanarak devre dışı bırakabilirsiniz.

Önemli faktör şudur ki biz başarıyla bir TCP bağlantısının tanıtıcısına (handle) ulaştık!

Programı belirli bir kod sürümünü çalıştırmayı bitirdiğinizde programı durdurmak için
<kbd>ctrl</kbd>-<kbd>C</kbd> tuşlarına bastığınızı hatırlayın. Sonra, en yeni kodu çalıştırdığınızdan emin olmak için
her kod değişiklik seti yaptıktan sonra programı `cargo run` komutunu çağırarak yeniden başlatın.

### İsteği Okuma (Reading the Request)

İsteği tarayıcıdan okuma işlevselliğini uygulayalım! İlk önce bir bağlantı almak ve sonra
bağlantı ile bazı eylem yapmak arasında endişeleri ayırmak için, bağlantıları işlemek için yeni bir fonksiyon
başlatacağız. Bu yeni `handle_connection` fonksiyonunda, TCP akışından veri okuyacağız ve yazdıracağız
böylece tarayıcıdan gönderilen veriyi görebilelim. Kodu Kod Listesi 21-2 gibi görünecek şekilde değiştirin.

<Listing number="21-2" file-name="src/main.rs" caption="Reading from `TcpStream` and printing data">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-02/src/main.rs}}
```

</Listing>

Akıştan okumak ve akışa yazmak için bize izin veren trait'leri ve türleri almak için
`std::io::BufReader` ve `std::io::prelude`'yi kapsama getiriyoruz. `main` fonksiyonundaki `for`
döngüsünde, bir bağlantı yaptığımızı söyleyen bir mesaj yazdırmak yerine, artık yeni `handle_connection`
fonksiyonunu çağırıyoruz ve ona `stream`'i geçiyoruz.

`handle_connection` fonksiyonunda, `stream` referansını saran yeni bir `BufReader` örneği oluşturuyoruz.
`BufReader`, bizim için `std::io::Read` trait yöntemlerine çağrıları yöneterek arabellekleme (buffering) ekler.

Tarayıcının sunucumuza gönderdiği isteğin satırlarını toplamak için `http_request` adında bir
değişken oluşturuyoruz. `Vec<_>` tür notasyonu ekleyerek bu satırları bir vektörde toplamak istediğimizi
gösteriyoruz.

`BufReader`, `lines` yöntemini sağlayan `std::io::BufRead` trait'ini uygular. `lines`
yöntemi, yeni satır baytı gördüğünde veri akışını bölen `Result<String, std::io::Error>` yineleyicisi
döner. Her `String`'i almak için, her `Result`'ı `map` ve `unwrap` yapıyoruz. `Result`,
veri geçerli UTF-8 değilse veya akıştan okuma sorunu varsa bir hata olabilir. Yine de, bir üretim programı
bu hataları daha nazikçe ele almalı ancak bizim için basitlik için hata durumunda programı durdurmayı seçiyoruz.

Tarayıcı, HTTP isteğinin sonunu satırda iki yeni satır karakteri göndererek işaretler bu yüzden
akıştan bir istek almak için, boş string olan bir satır alana kadar satırları alıyoruz. Satırları vektörde
topladığımızda, web tarayıcısının sunucumuza gönderdiği talimatları inceleyebilmemiz için güzel hata ayıklama (pretty debug)
formatı ile yazdırıyoruz.

Bu kodu deneyelim! Programı başlatın ve bir web tarayıcısında tekrar bir istek yapın.
Tarayıcıda hala bir hata sayfası alacaksınız ancak programımızın terminaldeki çıktısı artık buna
benzer görünecek:

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-02
cargo run
make a request to 127.0.0.1:7878
Can't automate because of output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello`
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
    "Accept-Language: en-US,en;q=0.5",
    "Accept-Encoding: gzip, deflate, br",
    "DNT: 1",
    "Connection: keep-alive",
    "Upgrade-Insecure-Requests: 1",
    "Sec-Fetch-Dest: document",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-User: ?1",
    "Cache-Control: max-age=0",
]
```

Tarayıcınıza bağlı olarak, biraz farklı çıktı alabilirsiniz. Şimdi istek verilerini
yazdırdığımıza göre, bir tarayıcı isteğinden neden birden çok bağlantı aldığımızı ilk satırındaki `GET`
sonraki yola (path) bakarak görebiliriz. Tekrarlanan bağlantıların hepsi _/_ istiyorsa, tarayıcın _/_'ı tekrar
çekmeye çalıştığını biliyoruz çünkü bizim programımızdan bir yanıt almıyor.

Bu istek verisini bölelim ki tarayıcının programımızdan ne istediğini anlayalım.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-closer-look-at-an-http-request"></a>
<a id="looking-closer-at-an-http-request"></a>

### HTTP İsteğine Daha Yakından Bakma (Looking More Closely at an HTTP Request)

HTTP metin tabanlı bir protokoldur ve bir istek bu formatı alır:

```text
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
```

İlk satır, istemcinin ne istediği hakkında bilgi tutan _isteğin satırı (request line)_dur.
İsteğin satırının ilk kısmı, kullanılan yöntemi gösterir, örneğin `GET` veya `POST`, bu istemcinin bu
isteyi nasıl yaptığı açıklar. İstemcimiz bir `GET` isteği kullandı bu, bilgi istiyor anlamına gelir.

İsteğin satırının sonraki kısmı _/_dur bu, istemcinin istediği _uniform resource identifier_ _(URI)_'nı gösterir:
Bir URI, neredeyse ancak tam değil, bir _uniform resource locator_ _(URL)_ ile aynıdır. URI'ler ve URL'ler arasındaki
fark, bu bölüm amaçlarımız için önemli değildir ancak HTTP spec'ı _URI_ terimini kullanır bu yüzden burada
_URI_ için _URL_ zihinsel değiştirebiliriz.

Son kısım, istemcinin kullandığı HTTP sürümüdür ve sonra isteğin satırı bir CRLF dizisinde biter.
(_CRLF_, _carriage return_ ve _line feed_ anlamına gelir ki bunlar daktilo günlerinden terimlerdir!) CRLF dizisi
ayıca `\r\n` olarak yazılabilir ki burada `\r` bir taşıma karakteri ve `\n` bir besleme (feed) karakteridir.
_CRLF dizisi_, isteğin satırını isteğin verinin kalanından ayırır. CRLF yazdırıldığında, yeni bir satırın başladığını
`\r\n` yerine gördüğümüzü not edin.

Şimdiye kadar programımızı çalıştırdığımızda aldığımız istek satırı verisine bakınca, `GET`'in
yöntem, _/_'in istek URI'si ve `HTTP/1.1` sürüm olduğunu görüyoruz.

İsteğin satırından sonra, `Host:`'den başlayan kalan satırlar başlıklar (headers)dir. `GET`
isteklerinin gövdesi yoktur.

Farklı bir tarayıcıdan veya farklı bir adres, örneğin _127.0.0.1:7878/test_ istek yaparak,
istek verisinin nasıl değiştiğini görmeyi deneyin.

Artık tarayıcının ne istediğini bildiğimize göre, bazı veri geri gönderelim!

### Yanıt Yazma (Writing a Response)

Bir istemci isteğine yanıt olarak veri göndermeyi uygulayacağız. Yanıtlar şu formatı alırlar:

```text
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

İlk satır, yanıtta kullanılan HTTP sürümünü, isteğin sonucunu özetleyen sayısal bir durum kodu ve
durum kodunun metin açıklamasını sağlayan bir neden ifadeyi içeren _durum satırı (status line)_dur. CRLF
dizisinden sonra herhangi bir başlık, başka bir CRLF dizisi ve yanıtın gövdesi gelir.

İşte HTTP sürümünü 1.1 kullanan, 200 durum koduna sahip, OK neden ifadeye sahip, başlıkları
ve gövdesi olmayan örnek bir yanıt:

```text
HTTP/1.1 200 OK\r\n\r\n
```

Durum kodu 200 standart başarılı yanıttır. Metin, küçük başarılı bir HTTP yanıtıdır. Bunu
akışa bir başarılı isteğe yanıtımız olarak yazalım! `handle_connection` fonksiyonundan, istek
verilerini yazdıran `println!`'i kaldırın ve Kod Listesi 21-3'teki kodla değiştirin.

<Listing number="21-3" file-name="src/main.rs" caption="Writing a tiny successful HTTP response to stream">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-03/src/main.rs:here}}
```

</Listing>

İlk yeni satır, başarılı mesajın verisini tutan `response` değişkenini tanımlar. Sonra, `response` üzerinde
`as_bytes` çağırarak string veriyi baytlara dönüştürüyoruz. `stream` üzerindeki `write_all` yöntemi bir `&[u8]` alır
ve bu baytları doğrudan bağlantı aşağı gönderir. `write_all` işlemi başarısız olabileceği için,
daha önceki gibi herhangi bir hata sonucunda `unwrap` kullanıyoruz. Yine de, gerçek bir uygulamada, buraya hata ele alırsınız.

Bu değişikliklerle, kodumuzu çalıştıralım ve bir istek yapalım. Artık terminale herhangi bir
veri yazdırmıyoruz bu yüzden Cargo'dan çıktıdan başka bir şey görmeyeceğiz. Bir web tarayıcısında
_127.0.0.1:7878_ yüklediğinizde, bir hata yerine boş bir sayfa almalısınız. Az önce bir HTTP
isteğini el ile (handcoded) alıp ve bir yanıt gönderdiniz!

### Gerçek HTML Döndürme (Returning Real HTML)

Boş bir sayfadan daha fazlasını döndürme işlevselliğini uygulayalım. Proje dizininizin kökünde,
_src_ dizininde değil yeni dosya _hello.html_ oluşturun. İstediğiniz herhangi bir HTML'i girebilirsiniz; Kod Listesi
21-4 bir olasılık gösterir.

<Listing number="21-4" file-name="hello.html" caption="A sample HTML file to return in a response">

```html
{{#include ../listings/ch21-web-server/listing-21-05/hello.html}}
```

</Listing>

Bu, bir başlığı ve bazı metinleri olan minimal bir HTML5 belgesidir. Bunu, bir istek alındığında sunucudan döndürmek
için, `handle_connection`'i Kod Listesi 21-5'te gösterildiği gibi değiştireceğiz ki HTML dosyasını okusun,
gövde olarak ekle ve gönderin.

<Listing number="21-5" file-name="src/main.rs" caption="Sending contents of *hello.html* as body of response">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-05/src/main.rs:here}}
```

</Listing>

Standart kütüphanenin dosya sistemi modülünü kapsama getirmek için `fs`'yi `use` ifadesine ekledik.
Bir dosyanın içeriklerini string'e okumak için kod, Bölüm 12-4'deki I/O projemiz için bir dosyanın
içeriklerini okuduğumuzda kullandığımıza benzer görünmeli.

Sonra, dosyanın içeriklerini başarılı yanıtın gövdesi olarak eklemek için `format!` kullanıyoruz. Geçerli bir HTTP
yanıtı sağlamak için, yanıt gövdesinin boyutuna ayarlanan `Content-Length` başlığı ekliyoruz — bu durumda,
`hello.html`'in boyutu.

Bu kodu `cargo run` ile çalıştırın ve tarayıcınızda _127.0.0.1:7878_ yükleyin; HTML'inizin
işlendiğini görmelisiniz!

Şu anda, `http_request`'daki istek verisini yok sayıp koşulsuz HTML dosyasının içeriklerini geri gönderiyoruz.
Bu, tarayıcınızda _127.0.0.1:7878/something-else_ istemeye çalışırsanız hala bu aynı HTML yanıtını
alacağınız anlamına gelir. Şu anda, sunucumuz çok sınırlıdır ve çoğu web sunucusunun yaptığı şeyleri yapmıyor.
İsteğe bağlı yanıtlarımızı özelleştirmek ve _/_ için iyi biçimlendirilmiş (well-formed) bir istek için
sadece HTML dosyasını geri göndermek istiyoruz.

### İsteği Doğrulama ve Seçici Olarak Yanıt Verme (Validating the Request and Selectively Responding)

Şu anda, web sunucumuz ne istemci istememi olursa olsun dosyadaki HTML'i döndürmektedir. Hadi
tarayıcın _/_ isteyip istemediğini kontrollemek için işlevsellik ekleyelim ve tarayıc başka bir şeyi
istemeye çalışırsa bir hata dönelim. Bunun için `handle_connection`'ı, Kod Listesi 21-6'de gösterildiği gibi
değiştirmemiz gerekiyor. Bu yeni kod, aldığımız istek içeriğini bir _/_ isteği gibi göründüğüne karşı kontrol eder
ve istekleri farklı ele almak için `if` ve `else` blokları ekler.

<Listing number="21-6" file-name="src/main.rs" caption="Handling requests to */* differently from other requests">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-06/src/main.rs:here}}
```

</Listing>

Biz sadece HTTP isteğinin ilk satırına bakacağız bu yüzden tüm isteği bir vektöre okumak yerine,
yineleyiciden ilk öğeyi almak için `next` çağırıyoruz. İlk `unwrap`, `Option`'la ilgilenir ve yineleyicinin
öğeleri yoksa programı durdurur. İkinci `unwrap`, `Result`'la ilgilenir ve Kod Listesi 21-2'deki `map`'a
eklediğimiz `unwrap` ile aynı etkiye sahiptir.

Sonra, `request_line`'ı kontrol ediyoruz ki _/_ yoluna bir GET isteğinin isteğin satırına eşit mi bakalım.
Eğer öyleyse, `if` bloğu HTML dosyamızın içeriklerini döner.

Eğer `request_line`, _/_ yoluna bir GET isteğine eşit DEĞİLSE, başka bir istek aldığımız anlamına gelir.
Birazdan `else` bloğuna, tüm diğer isteklere yanıt verecek kod ekleyeceğiz.

Şimdi bu kodu çalıştırın ve _127.0.0.1:7878_ isteyin; _hello.html_ içindeki HTML'i almalısınız.
Herhangi bir başka istek, örneğin _127.0.0.1:7878/something-else_ yaparsanız, Kod Listesi 21-1
ve Kod Listesi 21-2'deki kodu çalıştırdığınızda gördüğünüze benzer bir bağlantı hatası alacaksınız.

Şimdi Kod Listesi 21-7'deki kodu `else` bloğuna ekleyelim ki istek için içerik bulunamadığını (not found)
gösteren durum kodu 404 ile bir yanıt dönsün. Ayrıca son kullanıcıya göstermek için sayfayı işlecek bir HTML de
döndüreceğiz.

<Listing number="21-7" file-name="src/main.rs" caption="Responding with status code 404 and an error page if anything other than */* was requested">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-07/src/main.rs:here}}
```

</Listing>

Burada, yanıtımız 404 durum koduna ve `NOT FOUND` neden ifadeye sahip bir durum satırına sahiptir. Yanıtın
gövdesi, _404.html_ dosyasındaki HTML olacaktır. Hata sayfası için, _hello.html_'ın yanına _404.html_
dosyası oluşturmanız gerekecek; yine, istediğiniz herhangi bir HTML kullanabilirsiniz veya Kod Listesi 21-8'deki
örnek HTML'i kullanabilirsiniz.

<Listing number="21-8" file-name="404.html" caption="Sample content for page to send back with any 404 response">

```html
{{#include ../listings/ch21-web-server/listing-21-07/404.html}}
```

</Listing>

Bu değişikliklerle, sunucunuzu tekrar çalıştırın. _127.0.0.1:7878_ istemeyi _hello.html_'ın
içeriklerini dönmeli ve diğer herhangi bir istek, örneğin _127.0.0.1:7878/foo_, _404.html_ dosyasından
hata HTML'ini dönmeli.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-touch-of-refactoring"></a>

### Yeniden Düzenleme (Refactoring)

Şu anda, `if` ve `else` blokları çok tekrar sahibe: Her ikisi de dosyaları okuyor ve dosyaların
içeriklerini akışa yazıyor. Tek farklar durum satırı ve dosya adı. Kodu daha kısa yapalım o farkları ayrı
`if` ve `else` satırlarına çekebilir ki bunlar durum satırı ve dosya adı değerlerini değişkenlere atayacak;
sonra bu değişkenleri koşulsuz kodda dosyayı okumak ve yanıt yazmak için kullanabiliriz. Kod Listesi 21-9,
büyük `if` ve `else` bloklarını değiştirmeyi sonrasındaki ortaya çıkan kodu gösterir.

<Listing number="21-9" file-name="src/main.rs" caption="Refactoring `if` and `else` blocks to contain only code that differs between the two cases">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-09/src/main.rs:here}}
```

</Listing>

Şimdi `if` ve `else` blokları sadece durum satırı ve dosya adı için uygun değerleri bir demet (tuple) içinde
dönüyor; sonra bu iki değeri `let` ifadesindeki bir desen kullanarak `status_line` ve `filename`'a atamak için
yıkma (destructuring) kullanıyoruz, Bölüm 19'de tartışıldığı gibi.

Önceden çoğaltılan kod artık `if` ve `else` bloklarının dışında ve `status_line` ve `filename` değişkenlerini
kullanıyor. Bu, iki durum arasındaki farkı görmeyi kolaylaştırır ve dosya okuma ve yanıt yazma işini değiştirmek
istersen kodu güncellemek için sadece bir yerimiz olduğu anlamına gelir. Kod Listesi 21-9'deki kodun davranışı, Kod Listesi
21-7'deki ile aynı olacaktır.

Harika! Artık yaklaşık 40 satır Rust kodunda basit bir web sunucusumuz var ki içerik sayfası ile bir isteğe
yanıt verir ve tüm diğer isteklere 404 yanıtı ile yanıt verir.

Şu anda, sunucumuz tek bir iş parçacığında çalışıyor bu, bir seferde sadece bir isteğe hizmet edebilir
anlamına gelir. Bu, bazı yavaş istekleri simüle ederek nasıl bir sorun olabileceğine bakalım. Sonra, sunucumuzun
bir seferde birden çok isteği ele alabilmesi için düzelteceğiz.