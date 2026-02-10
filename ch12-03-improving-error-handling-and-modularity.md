## Modülerlik ve Hata Eleme İyileştirmek İçin Refactor Etme (Refactoring to Improve Modularity and Error Handling)

Programımızı iyileştirmek için, programın yapısı ve potansiyel hataları ele alma şekliyle ilgili dört sorunu düzelteceğiz. Önce, `main` fonksiyonumuz şu anda iki görevi gerçekleştiriyor: Argümanları çözüyor ve dosyalar okuyor. Programımız büyüdükçe, `main` fonksiyonunun ele aldığı ayrı görevlerin sayısı artacak. Bir fonksiyon sorumluluklar kazandıkça, akıl yürütmek daha zor olur, test etmek daha zor olur ve bir parçasını bozmadan değiştirmek daha zor olur. İşlevselliği ayırmak en iyisidir ki her fonksiyon sadece bir görevden sorumlu olsun.

Bu sorun aynı zamanda ikinci soruna da bağlanır: `query` ve `file_path` programımızın konfigürasyon değişkenleri olmasına rağmen, `contents` gibi değişkenler programın mantığını gerçekleştirmek için kullanılır. `main` ne kadar uzun olursa, kapsama getirmemiz gereken değişkenlerin sayısı o kadar artar; kapsamda ne kadar çok değişkenimiz olursa, her birinin amacını takip etmek o kadar zor olacaktır. Konfigürasyon değişkenlerini amaçlarını netleştirmek için bir yapıda gruplamak en iyisidir.

Üçüncü sorun, dosya okuma başarısız olduğunda bir hata mesajı yazdırmak için `expect` kullandığımız, ancak hata mesajının sadece `Should have been able to read the file` yazdığıdır. Dosya okuma birkaç şekilde başarısız olabilir: Örneğin, dosya eksik olabilir veya açmak için iznimiz olmayabilir. Şu anda, durumdan bağımsız olarak, her şey için aynı hata mesajını yazdırıyoruz ki bu kullanıcının ne yapması gerektiği hakkında bilgi vermez!

Dördüncü, bir hatayı ele almak için `expect` kullanıyoruz ve eğer kullanıcımız programı yeterince argüman belirtmeden çalıştırırsa, sorunu net açıklamayan Rust'tan bir `index out of bounds` hatası alacaklar. Tüm hata eleme kodunun tek bir yerde olması en iyidir ki gelecekteki bakımçılar, hata eleme mantığının değişmesi gerektiğinde koda danışmak için sadece tek bir yer olsun. Tüm hata eleme kodunun tek bir yerde olması ayrıca son kullanıcılarımız için anlamlı olacak mesajları yazdırdığımızdan emin olacaktır.

Bu dört sorunu projemizi yerefactor ederek ele alalım.

<a id="separation-of-concerns-for-binary-projects"></a>

### İkili Projelerde İlgileri Ayırma (Separating Concerns in Binary Projects)

Birden fazla görevin sorumluluğunu `main` fonksiyonuna atamanın organizasyon sorunu birçok ikili projede yaygındır. Sonuç olarak, birçok Rust programcısı, `main` fonksiyon büyümeye başladığında, bir ikili programın ayrı ilgilerini ayırmayı faydalı bulur. Bu sürecin şu adımları vardır:

- Programınızı bir _main.rs_ dosyası ve bir _lib.rs_ dosyası olarak ayırın ve programınızın mantığını _lib.rs_'e taşıyın.
- Komut satırı çözme mantığınız küçük olduğu sürece, `main` fonksiyonunda kalabilir.
- Komut satırı çözme mantığı karmaşıklaşmaya başladığında, `main` fonksiyonundan diğer fonksiyonlara veya türlere çıkarın.

Bu süreçten sonra `main` fonksiyonunda kalan sorumluluklar aşağıdakilerle sınırlı olmalıdır:

- Komut satırı çözme mantığını argüman değerleriyle çağırmak
- Herhangi bir başka konfigürasyon ayarlama
- _lib.rs_'de bir `run` fonksiyonu çağırmak
- `run` hata döndürürse hatayı ele almak

Bu desen ilgileri ayırmakla ilgilidir: _main.rs_ programın çalışmasını ele alır ve _lib.rs_ elindeki görevin tüm mantığını ele alır. `main` fonksiyonunu doğrudan test edemediğimiz için, bu yapı programın tüm mantığını test etmenizi `main` fonksiyonundan çıkararak sağlar. `main` fonksiyonunda kalan kod, doğruluğunu okuyarak doğrulamak için yeterince küçük olacaktır. Programımızı bu süreci izleyerek yeniden düzenleyelim.

#### Argüman Çözücü Çıkarma (Extracting the Argument Parser)

Argümanları çözme işlevselliğini `main`'in çağıracağı bir fonksiyona çıkaracağız. Kod Listesi 12-5, yeni bir `parse_config` fonksiyonunu çağıran `main` fonksiyonunun yeni başlangıcını gösterir ki bunu _src/main.rs_'de tanımlayacağız.

<Listing number="12-5" file-name="src/main.rs" caption="`main`'den bir `parse_config` fonksiyonu çıkarma">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-05/src/main.rs:here}}
```

</Listing>

Hala komut satırı argümanlarını bir vektörde topluyoruz, ancak `main` fonksiyonunda argüman değerini indeks 1'den `query` değişkenine ve indeks 2'den `file_path` değişkenine atamak yerine, tüm vektörü `parse_config` fonksiyonuna geçiyoruz. `parse_config` fonksiyonu daha sonra hangi argümanın hangi değişkene gittiğini belirleyen mantığı tutar ve değerleri `main`'e geri geçirir. Hala `main`'de `query` ve `file_path` değişkenlerini oluşturuyoruz, ancak `main`'in artık komut satırı argümanları ve değişkenler arasında nasıl karşılık geldiğini belirleme sorumluluğu yoktur.

Bu yeniden düzenleme küçük programımız için gereksiz görünebilir, ancak küçük, artımlı adımlarla yerefactor ediyoruz. Bu değişikliği yaptıktan sonra, argüman çözmenin hala çalıştığını doğrulamak için programı tekrar çalıştırın. İlerlemenizi sık sık kontrol etmek iyi bir fikirdir ki sorunlar oluştuğunda nedenini belirlemeye yardımcı olur.

#### Konfigürasyon Değerlerini Gruplama (Grouping Configuration Values)

`parse_config` fonksiyonunu daha fazla iyileştirmek için başka küçük bir adım atabiliriz. Şu anda, bir tuple döndürüyoruz, ancak sonra hemen o tuple'yu tekrar ayrı parçalarına ayırıyoruz. Bu, belki henüz doğru soyutlamaya sahip olmadığımızı gösteren bir işarettir.

İyileştirme için yer olduğunu gösteren başka bir gösterge, `parse_config`'un `config` kısmıdır ki bu, döndürdüğümüz iki değerin ilişkili olduğunu ve ikisinin de tek bir konfigürasyon değerinin parçası olduğunu ima eder. Şu anda bu anlamı veri yapısında sadece iki değeri bir tuple'da gruplayarak aktarmıyoruz; bunun yerine iki değeri tek bir struct'a koyacağız ve struct'ın her alanına anlamlı bir isim vereceğiz. Bunu yapmak, bu kodun gelecekteki bakımcılarının farklı değerlerin birbirleriyle nasıl ilişkili olduğunu ve amaçlarının ne olduğunu anlamasını kolaylaştırır.

Kod Listesi 12-6 `parse_config` fonksiyonuna yapılan iyileştirmeleri gösterir.

<Listing number="12-6" file-name="src/main.rs" caption="`parse_config`'u `Config` struct'ının bir örneğini döndürecek şekilde yerefactor etme">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-06/src/main.rs:here}}
```

</Listing>

`query` ve `file_path` adında alanlara sahip olacak şekilde tanımlanan `Config` adında bir struct ekledik. `parse_config`'un imzası şimdi bir `Config` değeri döndürdüğünü gösteriyor. `parse_config`'un gövdesinde, `args`'taki `String` değerlerine referans olan dize dilimleri döndürürken, şimdi `Config`'u sahip `String` değerlerini içerecek şekilde tanımlıyoruz. `main` içindeki `args` değişkeni argüman değerlerinin sahibidir ve sadece `parse_config` fonksiyonunun onları ödünç almasına izin verir ki bu, `Config` `args`'taki değerlerin sahipliğini almaya çalışırsa Rust'ın ödünç alma kurallarını ihlal edeceğimiz anlamına gelir.

`String` verisini yönetmek için birkaç yol olabilir; en kolay, ancak biraz verimsiz yol, değerlerdeki `clone` yöntemini çağırmaktır. Bu, `Config` örneğinin sahipliği için verinin tam kopyasını yapar ki dize verisine bir referans saklamaktan daha fazla zaman ve bellek alır. Ancak, veriyi klonlamak kodumuzu çok basit yapar çünkü referansların ömürlerini yönetmemize gerek yoktur; bu durumda, basitlik kazanmak için biraz performansızı feda etmek değerli bir dengedir.

> ### `clone` Kullanmanın Dengeleri (The Trade-Offs of Using `clone`)
>
> Birçok Rustacean arasında, çalışma zamanı maliyeti nedeniyle `clone` kullanarak mülkiyet problemlerini düzeltmekten kaçınma eğilimi vardır. [Bölüm 13][ch13]<!-- ignore -->'te, bu türde durumlarda daha verimli yöntemleri nasıl kullanacağınızı öğreneceksiniz. Ancak şimdilik, ilerlemeyi sürdürmek için birkaç dize kopyalamakta sorun yok çünkü bu kopyaları sadece bir kez yapacaksınız ve dosya yolunuz ve sorgu dizeniz çok küçük. Biraz verimsiz çalışan çalışan bir programa sahip olmak, ilk geçişte kodu hiper-optimizasyon yapmaya çalışmaktan daha iyidir. Rust ile daha deneyimli hale geldikçe, en verimli çözümle başlamak daha kolay olacak, ancak şimdilik `clone` çağırmak tamamen kabul edilebilir.

`main`'i güncelledik ki `parse_config` tarafından döndürülen `Config` örneğini `config` adında bir değişkene koyar, ve daha önce ayrı `query` ve `file_path` değişkenlerini kullanan kodu güncelledik ki artık `Config` struct'ı üzerindeki alanları kullanır.

Şimdi kodumuz daha net bir şekilde `query` ve `file_path`'in ilişkili olduğunu ve amaçlarının programın nasıl çalışacağını konfigüre etmek olduğunu aktarıyor. Bu değerleri kullanan herhangi bir kod, amaçlarına göre adlandırılan alanlarda `config` örneğinde onları bulacağını bilir.

#### `Config` İçin Bir Oluşturucu Oluşturma (Creating a Constructor for `Config`)

Şu ana kadar, komut satırı argümanlarını çözmekten sorumlu mantığı `main`'den çıkardık ve `parse_config` fonksiyonuna koyduk. Bu yaparak `query` ve `file_path` değerlerinin ilişkili olduğunu ve bu ilişkinin kodumuzda aktarılması gerektiğini görmemize yardımcı oldu. Sonra `query` ve `file_path`'in ilişkili amacını adlandırmak ve değerlerin isimlerini struct alan isimleri olarak `parse_config` fonksiyonundan döndürebilmek için bir `Config` struct'ı ekledik.

Şimdi, `parse_config` fonksiyonunun amacı bir `Config` örneği oluşturmak olduğu için, `parse_config`'u düz bir fonksiyondan `Config` ile ilişkili `new` adında bir fonksiyona değiştirebiliriz. Bu değişikliği yapmak kodun daha idiyomatik olmasını sağlayacak. Standart kütüphanedeki türlerin, örneğin `String`'in örneklerini `String::new` çağırarak oluşturabiliriz. Benzer şekilde, `parse_config`'u `Config` ile ilişkili bir `new` fonksiyonuna değiştirerek, `Config::new` çağırarak `Config` örnekleri oluşturabileceğiz. Kod Listesi 12-7 yapmamız gereken değişiklikleri gösterir.

<Listing number="12-7" file-name="src/main.rs" caption="`parse_config`'u `Config::new`'e değiştirme">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-07/src/main.rs:here}}
```

</Listing>

`parse_config` çağırdığımız `main`'i güncelledik ki bunun yerine `Config::new` çağırsın. `parse_config`'un adını `new`'e değiştirdik ve `new` fonksiyonunu `Config` ile ilişkilendiren bir `impl` bloğu içine taşıdık. Bu kodun çalıştığını doğrulamak için tekrar derlemeyi deneyin.

### Hata Eleme Düzeltme (Fixing the Error Handling)

Şimdi hata eleme düzeltmesi üzerinde çalışacağız. `args` vektöründeki değerlere indeks 1 veya indeks 2'den erişme girişimi, vektör üçten az öğe içeriyorsa programın panik atacaktır unutmayın. Programı hiç argüman olmadan çalıştırmayı deneyin; şöyle görünecektir:

```console
{{#include ../listings/ch12-an-io-project/listing-12-07/output.txt}}
```

`index out of bounds: the len is 1 but the index is 1` satırı programcılar için tasarlanmış bir hata mesajıdır. Son kullanıcılarımızın ne yapmaları gerektiğini anlamalarına yardımcı olmaz. Şimdi bunu düzeltecek.

#### Hata Mesajını İyileştirme (Improving the Error Message)

Kod Listesi 12-8'de, indeks 1 ve indeks 2'ye erişmeden önce dilimin yeterince uzun olduğunu doğrulayacak bir kontrolü `new` fonksiyonuna ekliyoruz. Eğer dilim yeterince uzun değilse, program panik atar ve daha iyi bir hata mesajı görüntüler.

<Listing number="12-8" file-name="src/main.rs" caption="Argüman sayısı için bir kontrol ekleme">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-08/src/main.rs:here}}
```

</Listing>

Bu kod [Kod Listesi 9-13'de yazdığımız `Guess::new` fonksiyonuna][ch9-custom-types]<!-- ignore --> benzer, burada `value` argümanı geçerli değerlerin aralığı dışında olduğunda `panic!` çağırdık. Burada bir değer aralığı kontrol etmek yerine, `args`'ın uzunluğunun en az `3` olduğunu kontrol ediyoruz ve fonksiyonun geri kalanı bu koşulun karşılandığını varsayarak çalışabilir. Eğer `args`'ta üçten az öğe varsa, bu koşul `true` olacak ve programı hemen bitirmek için `panic!` makrosunu çağıracağız.

`new`'daki bu ekstra birkaç satır kodla, programı tekrar hiç argüman olmadan çalıştıralım ki hatanın şimdi nasıl göründüğünü görelim:

```console
{{#include ../listings/ch12-an-io-project/listing-12-08/output.txt}}
```

Bu çıktı daha iyi: Şimdi makul bir hata mesajımız var. Ancak, kullanıcılarımıza vermek istemediğimiz gereksiz bilgiler de var. Belki Kod Listesi 9-13'te kullandığımız teknik burada kullanmak için en iyi teknik değildir: Bir kullanım sorunu yerine bir programlama sorunu için `panic!` çağırmak daha uygundur, [Bölüm 9'da tartışıldığı gibi][ch9-error-guidelines]<!-- ignore -->. Bunun yerine, Bölüm 9'da öğrendiğiniz diğer tekniği kullanacağız - başarı veya hata belirten bir [`Result` döndürme][ch9-result]<!-- ignore -->.

<a id="returning-a-result-from-new-instead-of-calling-panic"></a>

#### `panic!` Çağırmak Yerine bir `Result` Döndürme (Returning a `Result` Instead of Calling `panic!`)

Bunun yerine başarılı durumda bir `Config` örneği ve hata durumunda sorunu açıklayan bir `Result` değeri döndürebiliriz. Ayrıca fonksiyon adını `new`'ten `build`'e değiştireceğiz çünkü birçok programcı `new` fonksiyonlarının asla başarısız olacağını bekler. `Config::build` `main` ile iletişim kurarken, bir sorun olduğunu belirten `Result` türünü kullanabiliriz. Sonra, `main`'i bir `Err` varyantını, `panic!` çağrısının neden olduğu `thread 'main'` ve `RUST_BACKTRACE` hakkında çevreleyen metin olmadan, kullanıcılarımız için daha pratik bir hataya dönüştürmek için değiştirebiliriz.

Kod Listesi 12-9, şu anda `Config::build` olarak adlandırdığımız fonksiyonun dönen değeri için yapmamız gereken değişiklikleri ve bir `Result` döndürmesi gereken fonksiyon gövdesini gösterir. `main`'i de güncelleyene kadar derlenmeyeceğini unutmayın ki bunu bir sonraki kod listesinde yapacağız.

<Listing number="12-9" file-name="src/main.rs" caption="`Config::build`'den bir `Result` döndürme">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-09/src/main.rs:here}}
```

</Listing>

`build` fonksiyonumuz başarılı durumda bir `Config` örneği ve hata durumunda bir dize değişmezi (string literal) içeren bir `Result` döndürür. Hata değerlerimiz her zaman `'static` ömrüne sahip dize değişmezleri olacaktır.

Fonksiyon gövdesinde iki değişiklik yaptık: Kullanıcı yeterince argüman geçmediğinde `panic!` çağırmak yerine, şimdi bir `Err` değeri döndürüyoruz, ve `Config` dönen değerini bir `Ok` içine sardık. Bu değişiklikler fonksiyonu yeni tip imzasına uyarlar.

`Config::build`'den bir `Err` değeri döndürmek, `main` fonksiyonunun `build` fonksiyonundan dönen `Result` değerini ele almasını ve hata durumunda işlemi daha temizce bitirmesini sağlar.

<a id="calling-confignew-and-handling-errors"></a>

#### `Config::build` Çağırma ve Hataları Ele Alma (Calling `Config::build` and Handling Errors)

Hata durumunu ele almak ve kullanıcı dostu bir mesaj yazdırmak için, Kod Listesi 12-10'da gösterildiği gibi, `Config::build` tarafından dönen `Result`'u ele almak için `main`'i güncellememiz gerekiyor. Ayrıca komut satırı aracını sıfırdan farklı bir hata koduyla bitirme sorumluluğunu `panic!`'dan alıp bunu elle uygulayacağız. Sıfırdan farklı bir çıkış durumu, programı çağıran işleme programın hata durumuyla çıktığını belirten bir gelenektir.

<Listing number="12-10" file-name="src/main.rs" caption="Bir `Config` oluşturma başarısız olursa bir hata koduyla çıkma">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-10/src/main.rs:here}}
```

</Listing>

Bu kod listesinde, henüz ayrıntılı olarak kaplamadığımız bir yöntem kullandık: `unwrap_or_else`, ki bu standart kütüphanede `Result<T, E>` üzerinde tanımlanmıştır. `unwrap_or_else` kullanmak bazı özel, `panic!` olmayan hata eleme tanımlamamızı sağlar. Eğer `Result` bir `Ok` değerse, bu yöntemin davranışı `unwrap` benzerdir: `Ok`'un sardığı iç değeri döndürür. Ancak, değer bir `Err` değerse, bu yöntem kapanıştaki kodu çağırır ki bu, tanımladığımız ve `unwrap_or_else`'e bir argüman olarak geçirdiğimiz anonim bir fonksiyondur. Kapanışları [Bölüm 13][ch13]<!-- ignore -->'te daha ayrıntılı olarak kaplayacağız. Şimdilik, `unwrap_or_else`'in `Err`'nin iç değerini, bu durumda Kod Listesi 12-9'da eklediğimiz statik dize `"not enough arguments"` olan, dikey borular arasında görünen `err` argümanındaki kapanışımıza geçireceğini bilmeniz yeterlidir. Kapanıştaki kod çalıştığında `err` değerini kullanabilir.

Standart kütüphaneden `process`'i kapsama getirmek için yeni bir `use` satırı ekledik. Hata durumunda çalışacak kapanıştaki kod sadece iki satırdır: `err` değerini yazdırıyoruz ve sonra `process::exit` çağırıyoruz. `process::exit` fonksiyonu programı hemen durdurur ve çıkış durum kodu olarak geçilen sayıyı döndürür. Bu, Kod Listesi 12-8'de kullandığımız `panic!` tabanlı eleme benzerdir, ancak artık tüm ekstra çıktıyı almıyoruz. Deneyelim:

```console
{{#include ../listings/ch12-an-io-project/listing-12-10/output.txt}}
```

Harika! Bu çıktı kullanıcılarımız için çok daha dostça.

<a id="extracting-logic-from-the-main-function"></a>

### `main`'den Mantık Çıkarma (Extracting Logic from `main`)

Şimdi konfigürasyon çözmesini yerefactor etmeyi bitirdiğimize göre, programın mantığına dönelim. ["İkili Projelerde İlgileri Ayırma"](#separation-of-concerns-for-binary-projects)<!-- ignore --> bölümünde belirttiğimiz gibi, şu anda `main` fonksiyonunda olan ve konfigürasyon ayarlaması veya hata elemeyle ilgili olmayan tüm mantığı tutacak `run` adında bir fonksiyon çıkaracağız. Bittiğimizde, `main` fonksiyon kısa ve incelemeyle doğrulamak için kolay olacak, ve tüm diğer mantık için testler yazabileceğiz.

Kod Listesi 12-11 bir `run` fonksiyonu çıkarma küçük, artımlı iyileştirmesini gösterir.

<Listing number="12-11" file-name="src/main.rs" caption="Program mantığının geri kalanını içeren bir `run` fonksiyonu çıkarma">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-11/src/main.rs:here}}
```

</Listing>

`run` fonksiyonu artık `main`'den gelen tüm kalan mantığı içeriyor, dosya okumadan başlayarak. `run` fonksiyonu `Config` örneğini bir argüman olarak alır.

<a id="returning-errors-from-the-run-function"></a>

#### `run`'dan Hatalar Döndürme (Returning Errors from `run`)

Kalan program mantığı `run` fonksiyonuna ayrıldığı için, hata eleme iyileştirebiliriz, tıpkı Kod Listesi 12-9'da `Config::build` ile yaptığımız gibi. Programı `expect` çağırarak panik atmasına izin vermek yerine, `run` fonksiyonu bir şeyler yanlış giderse bir `Result<T, E>` döndürecek. Bu, hataları ele alma mantığını `main` içinde kullanıcı dostu bir şekilde daha fazla konsolide etmemizi sağlar. Kod Listesi 12-12 `run`'un imzası ve gövdesinde yapmamız gereken değişiklikleri gösterir.

<Listing number="12-12" file-name="src/main.rs" caption="`Result` döndürmek için `run` fonksiyonunu değiştirme">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-12/src/main.rs:here}}
```

</Listing>

Burada üç önemli değişiklik yaptık. Önce, `run` fonksiyonunun dönen tipini `Result<(), Box<dyn Error>>`'e değiştirdik. Bu fonksiyon daha önce birim türünü, `()`, döndürdü ve bunu `Ok` durumunda dönen değer olarak tutuyoruz.

Hata türü için, trait nesnesi `Box<dyn Error>` kullandık (ve `std::error::Error`'ü üstteki bir `use` bildirimiyle kapsama getirdik). Trait nesnelerini [Bölüm 18][ch18]<!-- ignore -->'te kaplayacağız. Şimdilik, `Box<dyn Error>`'in fonksiyonun `Error` trait'ini uygulayan bir tür döndüreceğini, ancak dönen değerin belirli türün ne olduğunu belirtmek zorunda olmadığımızı bilmeniz yeterlidir. Bu, farklı hata durumlarında farklı türlerde olabilecek hata değerleri döndürmemiz için esneklik sağlar. `dyn` anahtar kelimesi _dynamic_ kısaltmasıdır.

İkinci, [Bölüm 9][ch9-question-mark]<!-- ignore -->'te konuştuğumuz gibi, `expect` çağrısını lehine `?` operatörünü kaldırdık. Bir hatada `panic!` yerine, `?` geçerli fonksiyondan hatayı döndürür ki çağıran kişi ele alsın.

Üçüncü, `run` fonksiyonu artık başarılı durumda bir `Ok` değeri döndürüyor. `run` fonksiyonunun başarılı türünü imzada `()` olarak deklare ettik ki bu, birim tür değerini `Ok` değerine sarmamız gerektiği anlamına gelir. Bu `Ok(())` sözdizimi ilk başta biraz garip görünebilir. Ancak `()`'yi bu şekilde kullanmak, sadece yan etkileri için `run` çağırdığımızı belirtmenin idiyomatik yoludur; ihtiyaç duyduğumuz bir değer döndürmez.

Bu kodu çalıştırdığınızda, derlenir ancak bir uyarı görüntülenir:

```console
{{#include ../listings/ch12-an-io-project/listing-12-12/output.txt}}
```

Rust bize kodumuzun `Result` değerini yoksaydığını ve `Result` değerinin bir hata oluştuğunu gösterebileceğini söylüyor. Ancak bir hata oluşup oluşmadığını kontrol etmiyoruz ve derleyici burada bazı hata eleme kodu olması gerektiğini bize hatırlatıyor! Şimdi bu sorunu düzeltecek.

#### `run`'dan Dönen Hataları `main`'de Ele Alma (Handling Errors Returned from `run` in `main`)

Hataları kontrol edip Kod Listesi 12-10'da `Config::build` ile kullandığımız tekniğe benzer bir teknikle ele alacağız, ancak küçük bir farkla:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-01-handling-errors-in-main/src/main.rs:here}}
```

`unwrap_or_else` yerine `if let` kullanıyoruz ki `run`'un bir `Err` değeri döndürdüğünü kontrol edelim ve döndürürse `process::exit(1)` çağıralım. `run` fonksiyonu `Config::build`'un `Config` örneğini döndürdüğü gibi `unwrap` yapmak istediğimiz bir değer döndürmez. `run` başarılı durumda `()` döndürdüğü için, sadece bir hatayı tespit etmeyi önemsiyoruz, bu yüzden sadece `()` olacak açılmış değeri döndüren `unwrap_or_else`'ye ihtiyacımız yok.

Her iki durumda da `if let` ve `unwrap_or_else` fonksiyonlarının gövdesi aynıdır: Hatayı yazdırıyoruz ve çıkıyoruz.

### Kodu Bir Kütüphane Crate'ine Ayırma (Splitting Code into a Library Crate)

Bizim `minigrep` projemiz şu ana kadar iyi görünüyor! Şimdi _src/main.rs_ dosyasını ayıracağız ve bazı kodu _src/lib.rs_ dosyasına koyacağız. Bu şekilde, kodu test edebilir ve daha az sorumluluğu olan bir _src/main.rs_ dosyasına sahip olabiliriz.

_src/main.rs_'de değil _src/lib.rs_'de metni aramadan sorumlu kodu tanımlayalım ki bu, bizim (veya `minigrep` kütüphanemizi kullanan başkalarının) `minigrep` ikilimizden daha fazla bağlamdan arama fonksiyonunu çağırmasını sağlar.

Önce, Kod Listesi 12-13'te gösterildiği gibi _src/lib.rs_'de `search` fonksiyonu imzasını, gövdesi `unimplemented!` makrosunu çağıran bir gövde ile, tanımlayalım. Uygulamayı doldurduğumuzda imzayı daha ayrıntılı olarak açıklayacağız.

<Listing number="12-13" file-name="src/lib.rs" caption="*src/lib.rs*'de `search` fonksiyonu tanımlama">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-13/src/lib.rs}}
```

</Listing>

`search` fonksiyon tanımında `pub` anahtar kelimesini kullandık ki bunu kütüphane crate'imizin genel API'sinin bir parçası olarak belirleyelim. Şimdi ikili crate'imizden kullanabileceğimiz ve test edebileceğimiz bir kütüphane crate'imiz var!

Şimdi _src/lib.rs_'de tanımlanan kodu _src/main.rs_'deki ikili crate'in kapsamına getirmemiz ve çağırmamız gerekiyor ki Kod Listesi 12-14'te gösterildiği gibi.

<Listing number="12-14" file-name="src/main.rs" caption="*src/main.rs*'de `minigrep` kütüphane crate'inin `search` fonksiyonunu kullanma">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-14/src/main.rs:here}}
```

</Listing>

Kütüphane crate'inden `search` fonksiyonunu ikili crate'in kapsamına getirmek için `use minigrep::search` satırı ekledik. Sonra, `run` fonksiyonunda, dosyanın içeriklerini yazdırmak yerine, `search` fonksiyonunu çağırıyoruz ve `config.query` değerini ve `contents`'i argüman olarak geçiyoruz. Sonra, `run` sorguyla eşleşen `search`'tan dönen her satırı yazdırmak için bir `for` döngü kullanır. Ayrıca `main` fonksiyonundaki sorguyu ve dosya yolunu görüntüleyen `println!` çağrılarını kaldırmanın da iyi bir zaman ki programımız sadece arama sonuçlarını yazdırsın (eğer hiç hata oluşmazsa).

Arama fonksiyonunun tüm sonuçları, herhangi bir yazdırma olmazdan önce döndürdüğü vektörde topladığına dikkat edin. Bu uygulama büyük dosyalarda ararken sonuçları görüntülemek yavaş olabilir, çünkü sonuçlar bulundukça yazdırılmaz; bunu Chapter 13'te iteratörler kullanarak düzeltmenin olası bir yolunu tartışacağız.

Off! Bu çok işti, ancak gelecekte başarı için hazırlık yaptık. Şimdi hataları ele almak çok daha kolay ve kodumuzu daha modüler yaptık. Bundan sonra neredeyse tüm işimiz _src/lib.rs_'de yapılacak.

Şimdi bu yeni modülerlikten, eski kodla zor ama yeni kodla kolay olacak bir şey yaparak faydalanalım: Testler yazalım!

[ch13]: ch13-00-functional-features.html
[ch9-custom-types]: ch09-03-to-panic-or-not-to-panic.html#creating-custom-types-for-validation
[ch9-error-guidelines]: ch09-03-to-panic-or-not-to-panic.html#guidelines-for-error-handling
[ch9-result]: ch09-02-recoverable-errors-with-result.html
[ch18]: ch18-00-oop.html
[ch9-question-mark]: ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator