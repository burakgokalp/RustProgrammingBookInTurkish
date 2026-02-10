## `Result` ile Kurtarılabilir Hatalar (Recoverable Errors with `Result`)

Çoğu hata programı tamamen durdurmak için yeterince ciddi değildir. Bazen bir fonksiyon başarısız olduğunda, bunu kolayca yorumlayabileceğiniz ve yanıtlayabileceğiniz bir sebepden dolayıdır. Örneğin, bir dosya açmayı çalışırsanız ve operasyon dosya yokluğu dolayı başarısız olursa, işelemi durdurmak yerine dosya oluşturmak isteyebilirsiniz.

Bölüm 2'deki ["`Result` ile Potansiyel Başarısızlığı Ele Alma"][handle_failure]<!-- ignore --> hatırlayın ki `Result` enum'ü iki değişken (variant) `Ok` ve `Err` ile aşağıdaki gibi tanımlanmıştır:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T` ve `E` genel tip parametrelerdir (generic type parameters): Bölüm 10'de genel tipleri (generics) daha ayrıntılı konuşacağız. Şu anlamanız gereken şudur ki `T`, başarı durumunda `Ok` değişkeni içinde dönecek değerin tipini temsil eder ve `E`, başarısızlık durumunda `Err` değişkeni içinde dönecek hatanın tipini temsil eder. Çünkü `Result` bu genel tip parametrelerine sahip, çok farklı durumlarda başarısızlık değeri ve hata değeri dönmek istediğimiz durumlar için `Result` tipini ve üzerinde tanımlanmış yöntemleri kullanabiliriz.

Fonksiyonu çağıralım ki bir `Result` değeri döner çünkü fonksiyon başarısız olabilir. Kod Listesi 9-3'te, bir dosya açmayı çalışıyoruz.

<Listing number="9-3" file-name="src/main.rs" caption="Bir dosya açma">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-03/src/main.rs}}
```

</Listing>

`File::open`'ın dönüş tipi bir `Result<T, E>`'dir. Genel parametre `T`, `File::open` uygulaması tarafından başarısızlık değeri tipi olan `std::fs::File` ile doldurulmuştur ki bu bir dosya kulumudur (file handle). Hata değerinde kullanılan `E` tipi `std::io::Error`'dir. Bu dönüş tipi, `File::open` çağrısı başarılıp okuyabileceğimiz veya yazabileceğimiz bir dosya kulumunu dönmiş olabileceği anlamına gelir. Fonksiyon çağrısı da başarısız olabilir: Örneğin, dosya yok olabilir veya dosyaya erişim iznimiz olmayabilir. `File::open` fonksiyonunun başarılıp başarısız olmadığını ve aynı zamanda bize ya dosya kulumunu ya da hata bilgisini vermesi gerekir. Bu bilgi tam olarak `Result` enum'inin ilettiği şeydir.

`File::open` başarılırsa durumda, `greeting_file_result` değişkenindeki değer `Ok` değişkeninin bir kopyası (instance) olacak ki bir dosya kulumunu içerir. Başarısız olduğu durumda, `greeting_file_result` içindeki değer `Err` değişkeninin bir kopyası olacak ki oluğan hatanın tür hakkında daha fazla bilgi içerir.

Kod Listesi 9-3'e eklememiz gerekiyor ki `File::open`'ın döndüğü değere dayalı farklı eylemler alalım. Kod Listesi 9-4, Bölüm 6'da konuştuğumuz `match` ifadesini kullanarak `Result`'ü ele almanın bir yolunu gösterir.

<Listing number="9-4" file-name="src/main.rs" caption="`match` ifadesini kullanarak dönebilecek `Result` değişkenilerini ele alma">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-04/src/main.rs}}
```

</Listing>

Fark edin ki, `Option` enum gibi, `Result` enum ve değişkenileri prelude tarafından kapsama getirilmiştir, bu yüzden `match` kollarında `Ok` ve `Err` değişkenilerinden önce `Result::` belirtmemiz gerekmez.

Sonuç `Ok` olduğunda, bu kod `Ok` değişkeninin içindeki `file` değerini dönec ve sonra bu dosya kulumu değerini `greeting_file` değişkenine atayacağız. `match`'ten sonra, okuma veya yazma için dosya kulumunu kullanabiliriz.

`match`'in diğer kolu, `File::open`'dan bir `Err` değeri aldığımız durumu ele alır. Bu örnekte, `panic!` makrosunu çağırmayı seçtik. Eğer mevcut diçinimizde _hello.txt_ adında bir dosya yoksa ve bu kodu çalıştırırsak, `panic!` makrosundan şu çıktıyı göreceğiz:

```console
{{#include ../listings/ch09-error-handling/listing-09-04/output.txt}}
```

Her zamanki gibi, bu çıktı bize tam olarak neyin yanlış gittiğini söyler.

### Farklı Hataları Eşleştirme (Matching on Different Errors)

Kod Listesi 9-4, `File::open` başarısız olursa ne olursa `panic!` yapacak. Ancak, farklı başarısızlık sebelerinden farklı eylemler almak istiyoruz. Eğer `File::open`, dosya yokluğu dolayı başarısız olursa, dosya oluşturmak ve yeni dosyaya kulum dönmek istiyoruz. Eğer `File::open` başka herhangi bir sebepyle başarısız olursa — örneğin, çünkü dosyayı açmak için iznimiz yok — hala kodun Kod Listesi 9-4'te yaptığı gibi `panic!` yapmasını istiyoruz. Bunun için, içteki bir `match` ifadesi ekliyoruz ki bu Kod Listesi 9-5'te gösterilir.

<Listing number="9-5" file-name="src/main.rs" caption="Farklı hata türlerini farklı şekillerde ele alma">

<!-- ignore this test because otherwise it creates hello.txt which causes other
tests to fail lol -->

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-05/src/main.rs}}
```

</Listing>

`File::open`'ın `Err` değişkeni içinde döndürdüğü değerin tipi `io::Error`'dir ki bu standart kütüphane tarafından sağlanmış bir struct (yapı). Bu struct'in, çağırabileceğimiz bir `io::ErrorKind` değeri döndüren `kind` adında bir yöntemi vardır. `io::ErrorKind` enum'i standart kütüphane tarafından sağlanmıştır ve bir `io` operasyonundan kaynaklanabilecek farklı hata türlerini temsil eden değişkenileri vardır. Kullanmak istediğimiz değişken `ErrorKind::NotFound`'tir ki bu, açmaya çalıştığımız dosyanın henüz varolmadığını belirler. Bu yüzden, `greeting_file_result` üzerinde eşleştiriyoruz ancak `error.kind()` üzerinde de içteki bir match'e sahibiz.

İçteki match'te kontrol etmek istediğimiz koşul şudur ki `error.kind()` tarafından dönen değer `ErrorKind` enum'ının `NotFound` değişkeni midir. Eğerse, `File::create` ile dosya oluşturmayı çalışıyoruz. Ancak, `File::create` de başarısız olabilir, bu yüzden içteki `match` ifadesinde ikinci bir kola ihtiyacımız var. Dosya oluşturulamazsa, farklı bir hata mesajı yazdırılır. Dışteki match'in ikinci kolu aynı kalır, bu yüzden program kayıp dosya hatası dışındaki herhangi bir hatada panikler.

> #### `Result<T, E>` ile `match` Kullanmaya Alternatifler (Alternatives to Using `match` with `Result<T, E>`)
> Çok fazla `match` var! `match` ifadesi çok kullanışlıdır ancak çok daha ilkel (primitive). Bölüm 13'te, kapanışları (closures) hakkında öğreneceksiniz ki bunlar `Result<T, E>` üzerinde tanımlanmış çok sayıda yöntem ile kullanılır. Bu yöntemler kodunuzdaki `Result<T, E>` değerlerini ele alırken `match` kullanmaktan daha kısa olabilirler.
> Örneğin, burası Kod Listesi 9-5'te gösterilen aynı mantığı yazmanın başka bir yolu, bu sefer kapanışları ve `unwrap_or_else` yöntemini kullanarak:
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
> ```rust,ignore
> use std::fs::File;
> use std::io::ErrorKind;
> 
> fn main() {
>     let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
>         if error.kind() == ErrorKind::NotFound {
>             File::create("hello.txt").unwrap_or_else(|error| {
>                 panic!("Problem creating the file: {error:?}");
>             })
>         } else {
>             panic!("Problem opening the file: {error:?}");
>         }
>     });
> }
> ```
> Bu kod Kod Listesi 9-5 ile aynı davranışa sahip olsa da herhangi bir `match` ifadesi içermiyor ve okumak için daha temizdir. Bölüm 13'ü okuduktan sonra bu örneğe geri dönün ve standart kütüphane belgelerinde `unwrap_or_else` yöntemini araştırın. Bu yöntemlerin çoğu, hatlarla uğraştığınızda devasa, içiçe (nested) `match` ifadelerini temizleyebilir.
<!-- Old headings. Do not remove or links may break. -->

<a id="shortcuts-for-panic-on-error-unwrap-and-expect"></a>

#### Hatada Paniklemek için Kısayollar (Shortcuts for Panic on Error)

`match` kullanmak yeterince iyi çalışır ancak biraz uzun olabilir ve her zaman niyeti iyi iletişimlendirmez (communicate intent well). `Result<T, E>` tipi üzerinde tanımlanmış çok sayıda yardımcı yöntem vardır ki bunlar çeşitli, daha spesifik görevleri yapar. `unwrap` yöntemi, Kod Listesi 9-4'te yazdığımız `match` ifadesi gibi uygulanmış bir kısayol yöntemidir. Eğer `Result` değeri `Ok` değişkenidir, `unwrap` `Ok` içindeki değeri dönec. Eğer `Result` `Err` değişkenidir, `unwrap` bizim için `panic!` makrosunu çağıracak. İşte `unwrap`'in eylemde bir örneği:

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-04-unwrap/src/main.rs}}
```

</Listing>

Eğer bu kodu _hello.txt_ dosyası olmadan çalıştırırsak, `unwrap` yönteminin yaptığı `panic!` çağrısından hata mesajı göreceğiz:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-04-unwrap
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at src/main.rs:4:49:
called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

Benzer şekilde, `expect` yöntemi bize `panic!` hata mesajını de seçmemizi sağlar. `unwrap` yerine `expect` kullanmak ve iyi hata mesajları sağlamak niyetinizi iletmek ve bir panikın kaynağını aşağı takip etmeyi kolaylaştırabilir. `expect`'ın sözdizimi (syntax) şuna benzer:

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-05-expect/src/main.rs}}
```

</Listing>

`expect`'i `unwrap` ile aynı şekilde kullanıyoruz: dosya kulumunu dönmek veya `panic!` makrosunu çağırmak için. `expect`'in `panic!` çağrısında kullanacağı hata mesajı, `unwrap`'ın kullandığı varsayılan `panic!` mesajı yerine `expect`'e geçtiğimiz parametre olacaktır. İşte nasıl göründüğü:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-05-expect
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at src/main.rs:5:10:
hello.txt should be included in this project: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

Üretim kalitesinde (production-quality) kodda, çoğu Rustacean `unwrap` yerine `expect` seçer ve operasyonun neden her zaman başarısız olması bekendiği hakkında daha fazla bağlam sağlar. Bu şekilde, eğer varsayımlarınızın her zaman yanlış olduğu kanıtlanırsa, hata ayıklamasında (debugging) kullanmak için daha fazla bilginiz olur.

### Hataları Yayma (Propagating Errors)

Bir fonksiyonun uygulaması başarısız olabilecek bir şeyi çağırdığında, fonksiyonun içinde hatayı ele almak yerine, hatayı çağıran koda dönebilirsiniz ki böylece ne yapacağına karar verebilir. Bu, hatayı _yayma_ (propagating) olarak bilinir ve çağıran koda daha fazla kontrol verir ki orada kodunuzun bağlamında sahip olduğunuzdan daha fazla bilgi veya mantık olabilir ki hatanın nasıl ele alınması gerektiğini belirler.

Örneğin, Kod Listesi 9-6, bir dosyadan kullanıcı adını okuyan bir fonksiyon gösterir. Eğer dosya yok veya okunamıyorsa, bu fonksiyon o hataları fonksiyonu çağıran koda dönec.

<Listing number="9-6" file-name="src/main.rs" caption="`match` kullanarak çağıran koda hatalar döndüren bir fonksiyon">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-06/src/main.rs:here}}
```

</Listing>

Bu fonksiyon çok daha kısa bir şekilde yazılabilir ancak hata ele almayı keşfetmek için önce çok sayısını manuel olarak yapıyoruz; sonunda, daha kısa yolu göstereceğiz. Önce fonksiyonun dönüş tipine bakalım: `Result<String, io::Error>`. Bu anlamına gelir ki bu fonksiyonun bir `Result<T, E>` tipinde değer döndürdüğü, burada genel parametre `T` somut tip `String` ile doldurulmuştur ve genel tip `E` somut tip `io::Error` ile doldurulmuştur.

Eğer bu fonksiyon hiçbir problem olmadan başarılırsa, bu fonksiyonu çağıran kod bu fonksiyonun okuduğu `username`'i tutan bir `Ok` değeri alacak. Eğer bu fonksiyon herhangi bir problemle karşılaşırsa, çağıran kod ne problemlerin olduğu hakkında daha fazla bilgi tutan `io::Error`'in bir kopyası olan bir `Err` değeri alacak. Bu fonksiyonun dönüş tipi olarak `io::Error`'i seçtik çünkü bu fonksiyonun gövdesinde (body) çağırdığımız ve başarısız olabilecek her iki operasyondan dönen hata değeri tipi tam olarak bu: `File::open` fonksiyonu ve `read_to_string` yöntemi.

Fonksiyonun gövdesi önce `File::open` fonksiyonunu çağırak başlar. Sonra, Kod Listesi 9-4'teki `match`'e benzer bir `match` ile `Result` değerini ele alır. Eğer `File::open` başarılırsa, `file` desen değişkenindeki dosya kulumu `username_file` değişkenindeki değer olur ve fonksiyon devam eder. `Err` durumunda, `panic!` çağırmak yerine, fonksiyonun tamamen dışında erken çıkmak için `return` anahtar kelimesini kullanır ve `File::open`'dan gelen hata değerini, şimdi `e` desen değişkeninde, fonksiyonun hata değeri olarak çağıran koda geri döner.

Bu yüzden, eğer `username_file`'de bir dosya kulumumuz varsa, fonksiyon sonra `username` değişkeninde yeni bir `String` oluşturur ve `username_file`'deki dosya kulumunda `read_to_string` yöntemini çağırakır ki dosyanın içeriğini `username`'a okur. `read_to_string` yöntemi de bir `Result` döner çünkü başarısız olabilir, `File::open` başarısız olmasa ragmen. Bu yüzden, o `Result`'ü ele almak için başka bir `match`'e ihtiyacımız var: Eğer `read_to_string` başarılırsa, o zaman fonksiyonumuz başarılıştır ve dosyadan okuduğumuz ve şimdi `username`'da olan `username`'i bir `Ok` içinde sararak döneriz. Eğer `read_to_string` başarısız olursa, `File::open`'ın dönüş değerini ele alan `match`'te yaptığımız şekilde hata değerini döneriz. Ancak, `return` ifadesini açıkça söylememiz gerekmez çünkü bu fonksiyondaki son ifadedir.

Bu kodu çağıran kod sonra bir `Ok` değeri almayı ele alacak ki `username` içerir veya `io::Error` içeren bir `Err` değeri. Bu değerlerle ne yapılacağına karar vermek çağıran koda aittir. Eğer çağıran kod bir `Err` değeri alırsa, `panic!` çağırak programı çökertebilir, varsayılan bir kullanıcı adı kullanabilir veya dosya dışındaki biryerden kullanıcı adını arayabilir, örneğin. Çağıran kodun gerçekte ne yapmaya çalıştığı hakkında yeterince bilgiye sahip değiliz, bu yüzden başarı veya hata bilgisini yukarı (upward) yayıyoruz ki uygun şekilde ele alabilmesi için.

Hataları yayma bu modeli Rust'ta çok yaygındır ki Rust bunu kolaylaştırmak için soru işareti operatörü `?` sağlar.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-shortcut-for-propagating-errors-the--operator"></a>

#### `?` Operatörü Kısayolu (The `?` Operator Shortcut)

Kod Listesi 9-7, Kod Listesi 9-6'da olduğu ile aynı işlevselliğe sahip olan `read_username_from_file` uygulamasını gösterir ancak bu uygulama `?` operatörünü kullanır.

<Listing number="9-7" file-name="src/main.rs" caption="`?` operatörünü kullanarak çağıran koda hatalar döndüren bir fonksiyon">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-07/src/main.rs:here}}
```

</Listing>

`?` işareti `Result` değerinden sonra yerleştirilmiş neredeyse aynı şekilde çalışır
Kod Listesi 9-6'te `Result` değerlerini ele almak için tanımladığımız `match` ifadeleri gibi. Eğer `Result` değeri bir `Ok` değişkenidir, `Ok` içindeki değer bu ifadeden dönec ve program devam eder. Eğer değer bir `Err`'dir, `Err` fonksiyondan geri dönecek sanki biz `return` anahtar kelimesini kullanmışız gibi ki hata değeri çağıran koda yayılır (propagated) olur.

Kod Listesi 9-6'daki `match` ifadesi ve `?` operatörünün yaptığı arasında bir fark var: `?` operatörü onlardan hata değerleri, standart kütüphanenin `From` trait'inde tanımlanmış `from` fonksiyonundan geçer. `From` trait'i, değerleri bir tipten başka tipine dönüştürmek için kullanılır. `?` operatörü `from` fonksiyonunu çağırdığında, alınan hata tipi, mevcut fonksiyonun dönüş tipinde tanımlanmış hata tipine dönüştürülür. Bu, bir fonksiyonun başarısız olabilecek tüm yolları temsil etmek için bir hata tipi döndürdüğünde kullanışlıdır, kısımlar çok farklı sebepyle başarısız olabilir.

Örneğin, Kod Listesi 9-7'deki `read_username_from_file` fonksiyonunu, tanımladığımız `OurError` adında özel bir hata tipi döndürmesi için değiştirebiliriz. Ayrıca `OurError` için `impl From<io::Error>` tanımlarsak, `io::Error`'dan bir `OurError` kopyası oluşturabiliriz, o zaman `read_username_from_file` gövdesindeki `?` operatörü çağrıları `from` çağıracak ve hata tiplerini fonksiyona daha fazla kod eklemeksizin dönüştürecek.

Kod Listesi 9-7 bağlamında, `File::open` çağrısının sonundaki `?`, `username_file` değişkenine `Ok` içindeki değeri dönecek. Bir hata oluşursa, `?` operatörü fonksiyonun tamamından erken çıkacak ve çağıran koda herhangi bir `Err` değerini verecek. Aynı şey `read_to_string` çağrısının sonundaki `?` için de geçerlidir.

`?` operatörü çok fazla boilerplate kodunu ortadan kaldırır ve bu fonksiyonun uygulamasını daha basit hale getirir. Bu kodu, `?`'den hemen sonra yöntem çağrılarını zincirleyerek (chaining) daha da kısaltabiliriz, Kod Listesi 9-8'de gösterildiği gibi.

<Listing number="9-8" file-name="src/main.rs" caption="`?` operatöründen sonra yöntem çağrılarını zincirleme">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-08/src/main.rs:here}}
```

</Listing>

`username` içindeki yeni `String`'in oluşturmasını fonksiyonun başına taşıdık; bu kısım değişmedi. `username_file` değişkeni oluşturmak yerine, `File::open("hello.txt")?` sonucuna doğrudan `read_to_string` çağrısını zincirledik. Hala `read_to_string` çağrısının sonunda bir `?`'imiz var ve `File::open` ve `read_to_string` ikisi de başarılırsa hatalar dönmek yerine `username` içeren bir `Ok` değeri hala dönmekteyiz. İşlevsellik tekrar Kod Listesi 9-6 ve Kod Listesi 9-7'deki ile aynıdır; bu sadece yazmanın farklı, daha ergonomik (ergonomic) bir yolu.

Kod Listesi 9-9, `fs::read_to_string` kullanarak bunu daha da kısa yapmanın bir yolunu gösterir.

<Listing number="9-9" file-name="src/main.rs" caption="Dosyayı açıp sonra okumak yerine `fs::read_to_string` kullanma">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-09/src/main.rs:here}}
```

</Listing>

Bir dosyayı bir string'e okumak oldukça yaygın bir operasyondur, bu yüzden standart kütühane dosyayı açan, yeni bir `String` oluşturan, dosyanın içeriğini okuyan, içeriğini o `String`'in içine koyan ve dönen kullanışlı `fs::read_to_string` fonksiyonunu sağlar. Tabii ki, `fs::read_to_string` kullanmak bize tüm hata ele almayı açıklama fırsatı vermez, bu yüzden önce daha uzun yolu yaptık.

<!-- Old headings. Do not remove or links may break. -->

<a id="where-the--operator-can-be-used"></a>

#### `?` Operatörünü Nerede Kullanmalı (Where to Use the `?` Operator)

`?` operatörü sadece, üzerinde kullanılan değerin tipiyle uyumlu (compatible) dönüş tipine sahip fonksiyonlarda kullanılabilir. Bu, `?` operatörünün, Kod Listesi 9-6'da tanımladığımız `match` ifadesiyle aynı şekilde fonksiyonun dışına erken bir değer dönüşü yapmak için tanımlanmış olması olmasındandır. Kod Listesi 9-6'da, `match` bir `Result` değeri kullanıyordu ve erken dönüş kolu bir `Err(e)` değeri dönüyordu. Fonksiyonun dönüş tipi bir `Result` olmak zorundadır ki bu dönüşle uyumlu olsun.

Kod Listesi 9-10'te, `main` fonksiyonunda, üzerinde kullandığımız değerin tipiyle uyumsuz bir dönüş tipine sahip bir `main` fonksiyonunda `?` operatörünü kullanırsak hangi hatayı alacağımıza bakalım.

<Listing number="9-10" file-name="src/main.rs" caption="`()` döndüren bir `main` fonksiyonunda `?` kullanmayı deneme derlenmez.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-10/src/main.rs}}
```

</Listing>

Bu kod bir dosya açar ki başarısız olabilir. `?` operatörü `File::open` tarafından dönen `Result` değerini takip eder ancak bu `main` fonksiyonunun dönüş tipi `Result` değil `()`'dir. Bu kodu derlediğimizde şu hata mesajını alacağız:

```console
{{#include ../listings/ch09-error-handling/listing-09-10/output.txt}}
```

Bu hata, `?` operatörünü sadece `Result`, `Option` veya `FromResidual` trait'ini uygulayan başka bir tip döndüren bir fonksiyonda kullanmamıza izin verildiğini belirler.

Hatayı düzeltmek için iki seçeneğiniz var. Bir seçenek, `?` operatörünü kullandığınız değerin tipiyle uyumlu olması için fonksiyonun dönüş tipini değiştirmektir (bunu engelleyen herhangi bir kısıtlamanız olduğu sürece). Başka seçenek, `Result<T, E>`'ü uygun herhangi bir şekilde ele almak için bir `match` veya `Result<T, E>` yöntemlerinden birini kullanmaktır.

Hata mesajı da belirtti ki `?` `Option<T>` değerleriyle de kullanılabilir. `Result` üzerinde `?` kullanımı gibi, bir `Option` döndüren bir fonksiyonda sadece `Option` üzerinde `?` kullanabilirsiniz. Bir `Option<T>` üzerinde çağıldığında `?` operatörünün davranışı, bir `Result<T, E>` üzerinde çağıldığındakiyle benzerdir: Eğer değer `None` ise, o noktada fonksiyondan erken `None` dönecektir. Eğer değer `Some` ise, `Some` içindeki değer ifadenin sonuç değeri olacaktır ve fonksiyon devam eder. Kod Listesi 9-11'de, verilen metnin ilk satırının son karakterini bulan bir fonksiyonun örneği vardır.

<Listing number="9-11" caption="Bir `Option<T>` değeri üzerinde `?` operatörünü kullanma">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-11/src/main.rs:here}}
```

</Listing>

Bu fonksiyon `Option<char>` döner çünkü orada bir karakter olabilir ama orada olmayabilir. Bu kod `text` string dilimini (string slice) argümanı alır ve onu `lines` yöntemini çağırar ki bu, stringteki satırlar üzerinde bir iterator döner. Çünkü bu fonksiyon ilk satırı incelemek istiyor, iterator üzerinde `next` çağırak iteratordan ilk değeri alır. Eğer `text` boş bir string ise, bu `next` çağrısı `None` dönecek ki bu durumda `last_char_of_first_line`'den çıkmak ve `None` dönmek için `?` kullanırız. Eğer `text` boş bir string değilse, `next` `text` içindeki ilk satırın string dilimini içeren bir `Some` değeri dönecektir.

`?` string dilimini çıkarır ve o string dilimi üzerinde `chars` çağırak karakterlerinin iteratorini alabiliriz. İlk satırdaki son karaktere ilgi duyuyoruz, bu yüzden iteratordaki son öğeyi dönmek için `last` çağırır. Bu bir `Option`'tür çünkü ilk satırın boş bir string olması mümkündür; örneğin, eğer `text` boş bir satır ile başlar ancak diğer satırlarda karakterler varsa, `"\nhi"` gibi. Ancak, eğer ilk satırda bir son karakter varsa, `Some` değişkeni içinde dönecektir. Ortadaki `?` operatörü bize bu mantığı ifade etmenin kısa bir yol sağlar ki böylece fonksiyonu tek bir satırda uygulamamızı sağlar. Eğer `Option` üzerinde `?` operatörünü kullanamazsaydık, bu mantığı daha fazla yöntem çağrıları veya bir `match` ifadesi kullanarak uygulamamız gerekirdi.

Fark edin ki bir `Result` döndüren bir fonksiyonda `Result` üzerinde `?` operatörünü kullanabilirsiniz ve bir `Option` döndüren bir fonksiyonda `Option` üzerinde `?` operatörünü kullanabilirsiniz ancak ikisini karıştırıp (mix and match) yapamazsınız. `?` operatörü `Result`'u otomatik olarak `Option`'e veya tam tersine dönüştürmez (vice versa); bu durumlarda, `Result` üzerindeki `ok` yöntemini veya `Option` üzerindeki `ok_or` yöntemini gibi yöntemleri kullanarak dönüşümü açıkça yapabilirsiniz.

Şu ana kadar, kullandığımız tüm `main` fonksiyonları `()` döner. `main` fonksiyonu özel çünkü yürütülebilir bir programın giriş ve çıkış noktasıdır ve programın bekendiği gibi davranması için dönüş tipi üzerinde kısıtlamalar vardır.

Şanslı olarak, `main` da bir `Result<(), E>` dönebilir. Kod Listesi 9-12, Kod Listesi 9-10'den koda sahiptir ancak `main`'ın dönüş tipini `Result<(), Box<dyn Error>>` olarak değiştirdik ve sonuna bir `Ok(())` dönüş değeri ekledik. Bu kod şimdi derlenecek.

<Listing number="9-12" file-name="src/main.rs" caption="`main`'ı `Result<(), E>` dönmeye değiştirmek `Result` değerleri üzerinde `?` operatörünün kullanımına izin verir.">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-12/src/main.rs}}
```

</Listing>

`Box<dyn Error>` tipi bir trait nesnesidir (trait object) ki Bölüm 18'deki ["Ortak Davranışı Soyulamak için Trait Nesnelerini Kullanma" (Using Trait Objects to Abstract over Shared Behavior)][trait-objects]<!-- ignore --> bölümünde konuşacağız. Şu an, `Box<dyn Error>`'ı "herhangi türde hata" olarak okuyabilirsiniz. `Box<dyn Error>` hata tipine sahip bir `main` fonksiyonunda bir `Result` değeri üzerinde `?` kullanım izin verilir çünkü bu herhangi bir `Err` değerinin erken döndürülmesine izin verir. Bu `main` fonksiyonunun gövdesi yalnızca `std::io::Error` tipinde hatalar dönecek olsa bile, `Box<dyn Error>` belirterekleyerek, `main` gövdesine diğer hatalar dönen daha fazla kod eklenirse bile bu imzanın doğru kalmasını sağlayacaktır.

Bir `main` fonksiyonu bir `Result<(), E>` döndürdüğünde, yürütülebilir `main` `Ok(())` dönerse `0` değeriyle çıkacak ve `main` bir `Err` değeri dönerse sıfırdan farklı bir değerle çıkacak. C'de yazılmış yürütülebilirler (executables) çıktıklarında tamsayılar dönerler: Başarılı çıkan programlar tamsayı `0` döner ve hatalayan programlar `0`'dan farklı bir tamsayı döner. Rust de bu gelenekle uyumlu olmak için yürütülebilirliklerden tamsayılar döner.

`main` fonksiyonu [`std::process::Termination` trait'ini][termination]<!-- ignore --> uygulayan herhangi bir tipte dönebilir ki içinde bir `ExitCode` döndüren bir `report` fonksiyonu içerir. Kendi tipleriniz için `Termination` trait'ini uygulamak hakkında daha fazla bilgi için standart kütühane belgelerine başvurun.

Şimdi `panic!` çağırmayı veya `Result` dönmeyi tartıştığımıza, hangi durumda hangisinin uygun olduğu karar vermeye konumu dönelim.

[handle_failure]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result
[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[termination]: ../std/process/trait.Termination.html