# Tahmin Oyunu Programlamak

Rust'e birlikte uygulamalı bir proje üzerinde çalışarak atlayalım! Bu bölüm, gerçek bir programda nasıl kullanılacakları göstererek size birkaç ortak Rust kavramını tanıtır. `let`, `match`, yöntemler (methods), ilişkili fonksiyonlar, dış crate'lar ve daha fazlası öğreneceksiniz! Takip eden bölümlerde, bu fikirleri daha detaylı olarak inceleyeceğiz. Bu bölümde sadece temelleri pratik yapacaksınız.

Klasik bir başlayıcı programlama problemi uygulayacağız: bir tahmin oyunu. İşte nasıl çalıştığı: Program 1 ile 100 arasında rastgele bir tamsayı oluşturacak. Sonra oyuncudan bir tahmin girmesini isteyecektir. Bir tahmin girildikten sonra, program tahminin çok düşük mü yoksa çok yüksek mü olduğunu belirtecektir. Tahmin doğru ise, oyun kutlama mesajı yazdıracak ve çıkacaktır.

## Yeni Bir Proje Kurma

Yeni bir proje kurmak için, Bölüm 1'de oluşturduğunuz _projects_ dizinine gidin ve Cargo kullanarak şöyle yeni bir proje oluşturun:

```console
$ cargo new guessing_game
$ cd guessing_game
```

İlk komut, `cargo new`, projenin adını (`guessing_game`) ilk argüman olarak alır. İkinci komut yeni projenin dizinine değiştirir.

Oluşturulan _Cargo.toml_ dosyasına bakın:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial
rm -rf no-listing-01-cargo-new
cargo new no-listing-01-cargo-new --name guessing_game
cd no-listing-01-cargo-new
cargo run > output.txt 2>&1
cd ../../..
-->

<span class="filename">Dosya Adı: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/Cargo.toml}}
```

Bölüm 1'de gördüğünüz gibi, `cargo new` sizin için bir "Hello, world!" programı oluşturur. _src/main.rs_ dosyasını kontrol edin:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/src/main.rs}}
```

Şimdi bu "Hello, world!" programını derleyelim ve `cargo run` komutunu kullanarak tek adımda çalıştıralım:

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/output.txt}}
```

Bu oyunda yapacağımız gibi bir projeyi hızlıca yenileme işlemeniz gerektiğinde `run` komutu çok yararlı olur. Her yenilemeden önce hızlıca test etmenizi sağlar.

_src/main.rs_ dosyasını tekrar açın. Tüm kodu bu dosyaya yazacaksınız.

## Tahmini İşleme

Tahmin oyunu programının ilk kısmı kullanıcı girdisi isteyecek, o girdiyi işleyecek ve girdinin beklenen formda olup olmadığını kontrol edecek. Başlamak için, oyuncunun bir tahmin girmesine izin verelim. Kod Listesi 2-1'deki kodu _src/main.rs_'ye girin.

<Listing number="2-1" file-name="src/main.rs" caption="Kullanıcıdan bir tahmin alan ve yazdıran kod">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:all}}
```

</Listing>

Bu kod çok fazla bilgi içeriyor, bu yüzden satır satır inceleyelim. Kullanıcı girdisini almak ve sonucu çıktı olarak yazdırmak için, `io` giriş/çıkış kütüphanesini scope'a getirmemiz gerekiyor. `io` kütüphanesi standart kütüphaneden gelir, `std` olarak bilinir:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:io}}
```

Varsayılan olarak, Rust her programın scope'una getirdiği standart kütüphanede tanımlı bir öğe kümesine sahiptir. Bu kümeye _prelude_ denir ve [standart kütüphane dokümantasyonunda][prelude] içindeki her şeyi görebilirsiniz.

Kullanmak istediğiniz bir tip prelude içinde değilse, `use` ifadesiyle o tipi açıkça scope'a getirmeniz gerekir. `std::io` kütüphanesini kullanmak size kullanıcı girdisini kabul etme de dahil olmak üzere çok faydalı özellikler sağlar.

Bölüm 1'de gördüğünüz gibi, `main` fonksiyonu programın giriş noktasıdır:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:main}}
```

`fn` sözdizimi yeni bir fonksiyon bildirir; parantezler, `()`, parametresi olmadığını gösterir; ve küme parantezi, `{`, fonksiyonun gövdesinin başlangıcını başlatır.

Bölüm 1'de öğrendiğiniz gibi, `println!` bir dizeyi ekrana yazdıran bir macro'dur:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print}}
```

Bu kod oyunun ne olduğunu bildiren ve kullanıcıdan girdi isteyen bir istemi yazdırıyor.

### Değişkenlerle Değerler Saklama

Sonra, kullanıcı girdisini saklamak için bir _değişken_ oluşturacağız, şöyle:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:string}}
```

Şimdi program ilginç hale gelmeye başlıyor! Bu küçük satırda çok fazla şey oluyor. Değişken oluşturmak için `let` ifadesini kullanıyoruz. İşte başka bir örnek:

```rust,ignore
let apples = 5;
```

Bu satır `apples` adında yeni bir değişken oluşturur ve onu `5` değerine bağlar. Rust'ta, değişkenler varsayılan olarak değiştirilemezdir (immutable), yani bir kez değişkene bir değer verdiğinizde, değer değişmeyecek. Bu kavramı Bölüm 3'te ["Değişkenler ve Değiştirilebilirlik"][variables-and-mutability]<!-- ignore --> bölümünde detaylı olarak tartışacağız. Bir değişkeni değiştirilebilir yapmak için, değişken adının önüne `mut` ekleriz:

```rust,ignore
let apples = 5; // değiştirilemez
let mut bananas = 5; // değiştirilebilir
```

> Not: `//` sözdizimi satırın sonuna kadar devam eden bir yorumu başlatır. Rust yorumlardaki her şeyi yok sayar. Yorumları Bölüm 3'te [Yorumlar][comments]<!-- ignore --> bölümünde daha detaylı olarak tartışacağız.

Tahmin oyunu programına dönersek, `let mut guess` ifadesinin `guess` adında değiştirilebilir bir değişken getirdiğini şimdi biliyorsunuz. Eşitlik işareti (`=`) Rust'a bir şeyi şimdi değişkene bağlamak istediğimizi söyler. Eşitlik işaretinin sağında `guess`'ün bağlandığı değer vardır, bu `String::new` fonksiyonunu çağırmanın sonucudur, bu fonksiyon `String`'nin yeni bir örneğini döndürür. [`String`][string]<!-- ignore --> standart kütüphane tarafından sağlanan bir dize tipidir, bu genişleyebilir, UTF-8 kodlanmış bir metin parçasıdır.

`::new` satırındaki `::` sözdizimi `new`'in `String` tipinin ilişkili fonksiyonu olduğunu gösterir. _İlişkili fonksiyon_ bir tip üzerinde uygulanan bir fonksiyondur, bu durumda `String`. Bu `new` fonksiyonu yeni, boş bir dize oluşturur. Çoğu tipte `new` fonksiyonu bulacaksınız çünkü bu bir nevi bir değeri oluşturan fonksiyon için yaygın bir addır.

Tam olarak, `let mut guess = String::new();` satırı şu anda yeni, boş bir `String` örneğine bağlanmış değiştirilebilir bir değişken oluşturdu. Oh be!

### Kullanıcı Girdisini Alma

`use std::io;` ile programın ilk satırında standart kütüphaneden giriş/çıkış işlevselliğini getirdiğimizi hatırlayın. Şimdi `io` modülünden `stdin` fonksiyonunu çağıracağız, bu bize kullanıcı girdisini işlememize izin verecek:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:read}}
```

Programın başında `use std::io;` ile `io` modülünü içe aktarmamış olsaydık, fonksiyonu `std::io::stdin` yazarak hala kullanabiliriz. `stdin` fonksiyonu [`std::io::Stdin`][iostdin]<!-- ignore --> örneğini döndürür, bu terminaliniz için standart girdiye bir tutamaç (handle) temsil eden bir tiptir.

Sonra, `.read_line(&mut guess)` satırı standart girdi tutamacında kullanıcıdan girdi almak için [`read_line`][read_line]<!-- ignore --> yöntemini çağırır. Ayrıca `&mut guess`'ü `read_line`'ye argüman olarak geçiriyoruz, ona girdiyi hangi dizeye saklayacağını söylüyoruz. `read_line`'in tam işi, kullanıcının standart girdiye yazdığı her şeyi almak ve onu bir dizeye eklemektir (içeriğini değiştirmeden), bu yüzden o dizeyi bir argüman olarak geçiriyoruz. Dize argümanı yöntemin dizenin içeriğini değiştirebilmesi için değiştirilebilir olmalıdır.

`&` işareti bu argümanın bir _referans_ olduğunu gösterir, bu size kodunuzun çok kısmının aynı veri parçasına birden fazla kereye belleğe kopyalamadan erişmesine izin veren bir yol sağlar. Referanslar karmaşık bir özellik ve Rust'ın büyük avantajlarından biri referansları ne kadar güvenli ve kolay kullanıldığıdır. Bu programı bitirmek için bu detayların çoğunu bilmenize gerek yok. Şimdilik bilmeniz gereken şudur: değişkenler gibi, referanslar varsayılan olarak değiştirilemezdir. Bu nedenle, onu değiştirilebilir yapmak için `&guess` yerine `&mut guess` yazmanız gerekir. (Bölüm 4 referansları daha kapsamlı olarak açıklayacaktır.)

<!-- Old headings. Do not remove or links may break. -->

<a id="handling-potential-failure-with-the-result-type"></a>

### `Result` ile Olası Hataları İşleme

Hala bu kod satırı üzerinde çalışıyoruz. Şimdi üçüncü metin satırını tartışıyoruz ancak not edin ki bu hala tek bir mantıksal kod satırının bir kısmıdır. Sonraki kısım bu yöntemdir:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:expect}}
```

Bu kodu şöyle yazabilirdik:

```rust,ignore
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

Ancak, tek uzun satır okumak zor, bu yüzden bölmek en iyidir. Uzun satırları bölmek için `.method_name()` sözdizimiyle bir yöntem çağırırken genellikle yeni satır ve diğer boşluklar eklemek akıllıcadır. Şimdi bu satırın ne yaptığını inceleyelim.

Daha önce belirttiğimiz gibi, `read_line` kullanıcıyı ne girerse onu ona geçtiğimiz dizeye koyar ama aynı zamanda bir `Result` değeri döndürür. [`Result`][result]<!-- ignore --> bir [_sayma_ (enumeration)[enums]<!-- ignore --> denir, çoğu kez _enum_ olarak adlandırılır, bu birden fazla olası durumdan birinde olabilen bir tiptir. Her olası durumu bir _varyant_ (variant) olarak adlandırırız.

Bölüm 6 [enum'ları][enums]<!-- ignore --> daha detaylı olarak kapsayacaktır. Bu `Result` tiplerinin amacı hata işleme bilgisini kodlamaktır.

`Result`'ün varyantları `Ok` ve `Err`'dir. `Ok` varyantı işlemin başarılı olduğunu ve başarıyla üretilmiş değeri içerir. `Err` varyantı işlemin başarısız olduğunu ve işlemin nasıl veya neden başarısız olduğu hakkında bilgi içerir.

Herhangi bir tipin değerleri gibi, `Result` tipinin değerleri de üzerinde tanımlı yöntemlere sahiptir. `Result`'ün bir örneği çağırabileceğiniz bir [`expect` yöntemi][expect]<!-- ignore --> vardır. `Result`'ün bu örneği bir `Err` değeri ise, `expect` programın çökmesine ve `expect`'e argüman olarak geçtiğiniz mesajı görüntülemesine neden olur. Eğer `read_line` yöntemi bir `Err` döndürürse, bu muhtemelen altta yatan işletim sisteminden gelen bir hatanın sonucu olur. Eğer `Result`'ün bu örneği bir `Ok` değeri ise, `expect` `Ok`'un tuttuğu geri dönüş değerini alacak ve sadece o değeri size geri döndürecek, böylece onu kullanabilirsiniz. Bu durumda, o değer kullanıcının girdisindeki bayt sayısıdır.

Eğer `expect` çağırmazsanız, program derlenecek ancak bir uyarı alacaksınız:

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-02-without-expect/output.txt}}
```

Rust `read_line`'den dönen `Result` değerini kullanmadığınızı uyarır, bu programın olası bir hatayı işlemediğini gösterir.

Uyarıyı bastırmanın doğru yolu aslında hata işleme kodu yazmaktır ancak bizim durumumuzda bir sorun olduğunda bu programın çökmesini istiyoruz, bu yüzden `expect` kullanabiliriz. Hatalardan kurtulmayı Bölüm 9'da[recover]<!-- ignore --> öğreneceksiniz.

### `println!` Yer Tutucularıyla Değerler Yazdırma

Küme parantezi kapatmanın yanı sıra, şu ana kadar kodta tartışmanız gereken sadece bir satır daha var:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print_guess}}
```

Bu satır şu anda kullanıcının girdisini içeren dizeyi yazdırır. `{}` küme parantezi kümesi bir yer tutucudur: `{}`'i yerinde bir değer tutan küçük yengeç penseleri düşünün. Bir değişkenin değerini yazdırırken, değişkenin adını küme parantezlerin içine koyabilirsiniz. Bir ifadenin sonucunu yazdırırken, format dizesine boş küme parantezleri koyun, sonra format dizesini boş küme parantez yer tutucusunda yazdırmak istediğiniz ifadelerin virgülle ayrılmış listesini izleyin. Bir değişkeni ve bir ifadenin sonucunu `println!`'e tek bir çağrıda yazdırmak şöyle görünürdü:

```rust
let x = 5;
let y = 10;

println!("x = {x} and y + 2 = {}", y + 2);
```

Bu kod `x = 5 and y + 2 = 12` yazdırırdı.

### İlk Kısmı Test Etmek

Tahmin oyununun ilk kısmını test edelim. `cargo run` kullanarak çalıştırın:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-01/
cargo clean
cargo run
input 6 -->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 6.44s
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

Bu noktada, oyunun ilk kısmi bitti: Klavyeden girdi alıyoruz ve sonra onu yazdırıyoruz.

## Gizli Sayı Oluşturma

Sonra, kullanıcının tahmin etmeye çalışacağı gizli bir sayı oluşturmamız gerekiyor. Gizli sayının her seferinde farklı olması gerekir ki oyun birden fazla kez oynamak eğlenceli olsun. 1 ile 100 arasında bir rastgele sayı kullanacağız ki oyun çok zor olmasın. Rust henüz standart kütüphanesinde rastgele sayı işlevselliğini içermiyor. Ancak, Rust ekibi bahsettiğimiz işlevselliği içeren bir [`rand` crate][randcrate] sağlar.

<!-- Old headings. Do not remove or links may break. -->
<a id="using-a-crate-to-get-more-functionality"></a>

### Crate ile İşlevsellik Artırma

Bir crate'in Rust kaynak kodu dosyalarının bir koleksiyon olduğunu hatırlayın. Şu ana kadar oluşturduğumuz proje bir ikili (binary) crate'dir, bu çalıştırılabilir bir programdır. `rand` crate'ı bir kütüphane crate'ıdır, bu başka programlarda kullanılmak üzere tasarlanan kodu içerir ve kendi başına çalıştırılamaz.

Cargo'nun dış crate'ların koordinasyonu Cargo'nun gerçekten parladığı yerdir. `rand` kullanan kod yazmadan önce, _Cargo.toml_ dosyasını `rand` crate'ını bir bağımlılık olarak içermesi için değiştirmemiz gerekiyor. Şimdi o dosyayı açın ve Cargo sizin için oluşturduğu `[dependencies]` bölüm başlığının altına, en altta şu satırı ekleyin. Bu örnekteki kod örneklerinin çalışması için `rand`'i tam olarak burada belirttiğimiz gibi, bu sürüm numarasıyla belirttiğinizden emin olun:

<!-- When updating # version of `rand` used, also update # version of
# `rand` used in these files so they all match:
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
* ch14-03-cargo-workspaces.md
-->

<span class="filename">Dosya Adı: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:8:}}
```

_Cargo.toml_ dosyasında, bir başlığı izleyen her şey o bölümün bir parçasıdır ve başka bir başlığı başlayana kadar devam eder. `[dependencies]` içinde, Cargo'ya projenizin hangi dış crate'lara bağımlı olduğunu ve o crate'ların hangi sürümlerine ihtiyaç duyduğunuzu söylersiniz. Bu durumda, anlamsal sürüm belirteci `0.8.5` ile `rand` crate'ını belirtiyoruz. Cargo [Anlamsal Sürümleme][semver]<!-- ignore --> (bazen _SemVer_ olarak adlandırılır) anlamına gelir, bu sürüm numaralarını yazmak için bir standarttır. `0.8.5` belirteci aslında `^0.8.5`'in kısaltmasıdır, bu en az 0.8.5 ama 0.9.0'ın altında olan herhangi bir sürüm anlamına gelir.

Cargo bu sürümlerin 0.8.5 sürümüyle herkese uygun API'ları olduğunu düşünür ve bu belirtim bu bölümdeki kodla hala derleyecek en son patch sürümünü alacağınızı sağlar. 0.9.0 veya daha büyük herhangi bir sürümün takip eden örneklerin kullandığı API ile aynı olacağı garanti edilmez.

Şimdi, kodda herhangi bir değişiklik yapmadan, projeyi derleyelim, Kod Listesi 2-2'de gösterildiği gibi.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
rm Cargo.lock
cargo clean
cargo build -->

<Listing number="2-2" caption="`rand` crate'ını bir bağımlılık olarak ekledikten sonra `cargo build` çalıştırmanın çıktısı">

```console
$ cargo build
  Updating crates.io index
   Locking 15 packages to latest Rust 1.85.0 compatible versions
    Adding rand v0.8.5 (available: v0.9.0)
 Compiling proc-macro2 v1.0.93
 Compiling unicode-ident v1.0.17
 Compiling libc v0.2.170
 Compiling cfg-if v1.0.0
 Compiling byteorder v1.5.0
 Compiling getrandom v0.2.15
 Compiling rand_core v0.6.4
 Compiling quote v1.0.38
 Compiling syn v2.0.98
 Compiling zerocopy-derive v0.7.35
 Compiling zerocopy v0.7.35
 Compiling ppv-lite86 v0.2.20
 Compiling rand_chacha v0.3.1
 Compiling rand v0.8.5
 Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
  Finished `dev` profile [unoptimized + debuginfo] target(s) in 2.48s
```

</Listing>

Farklı sürüm numaraları (ama SemVer sayesinde her şeyin kodla uyumlu olacağı!) ve farklı satırlar (işletim sistemine bağlı olarak) ve satırlar farklı bir sırada olabilir görebilirsiniz.

Dış bir bağımlılık içerdiğimizde, Cargo o bağımlılığın ihtiyaç duyduğu her şeyin en son sürümlerini _kayıt defterinden_ (registry) getirir, bu [Crates.io][cratesio]'dan verinin bir kopyasıdır. Crates.io, Rust ekosistemindeki kişilerin başkalarının kullanması için açık kaynak Rust projelerini paylaştıkları yerdir.

Kayıt defterini güncelledikten sonra, Cargo `[dependencies]` bölümünü kontrol eder ve daha önce indirilmemiş listelenmiş herhangi bir crate'ı indirir. Bu durumda, biz sadece `rand`'ı bir bağımlılık olarak listemiş olsak bile, Cargo `rand`'ın çalışması için ihtiyaç duyduğu başka crate'ları da aldı. Crate'leri indirdikten sonra, Rust onları derler ve sonra projeyi mevcut bağımlılıklarla derler.

Herhangi bir değişiklik yapmadan hemen `cargo build`'i tekrar çalıştırırsanız, `Finished` satırının yanı sıra herhangi bir çıktı alamazsınız. Cargo bağımlılıkları zaten indirdiğini ve derlediğini ve _Cargo.toml_ dosyanızda onlar hakkında herhangi bir şeyi değiştirmediğinizi bilir. Cargo ayrıca kodunuzu herhangi bir şeyi değiştirmediğinizi bilir, bu yüzden onu da tekrar derlemez. Yapılacak bir şey olmadığından, sadece çıkar.

_src/main.rs_ dosyasını açarsanız, önemsiz bir değişiklik yaparsanız ve sonra onu kaydedip tekrar derlerseniz, sadece iki satırlık çıktı görürsünüz:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
touch src/main.rs
cargo build -->

```console
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
```

Bu satırlar Cargo'nun sadece _src/main.rs_ dosyasındaki önemsiz değişikliğe göre derlemeyi güncellediğini gösterir. Bağımlılıklarınız değişmedi, bu yüzden Cargo onlar için zaten indirdiği ve derlediği yeniden kullanabileceğini bilir.

<!-- Old headings. Do not remove or links may break. -->
<a id="ensuring-reproducible-builds-with-the-cargo-lock-file"></a>

#### Tekrarlanabilir Derlemeleri Sağlama

Cargo'nın sizin veya başkasının kodunuzu derlediği her seferinde aynı eseri tekrar oluşturmasını sağlayan bir mekanizması vardır: Cargo, açıkça başka bir şeye işaret edene kadar belirttiğiniz bağımlılıkların sadece o sürümlerini kullanır. Örneğin, bir hafta sonra `rand` crate'ının 0.8.6 sürümü çıksın ve bu sürüm önemli bir hata düzeltmesi içersin, ama ayrıca kodunuzu bozacak bir regreesyon içerir. Bunu işlemek için, Rust ilk kez `cargo build` çalıştırdığınızda _Cargo.lock_ dosyası oluşturur, bu yüzden şimdi _guessing_game_ dizinimde bu var.

Bir projeyi ilk kez derlediğinizde, Cargo kriterlere uyan bağımlılıkların tüm sürümlerini belirler ve onları _Cargo.lock_ dosyasına yazar. Gelecekte projenizi derlediğinizde, Cargo _Cargo.lock_ dosyasının var olduğunu görecek ve sürümleri tekrar belirlemek yerine orada belirtilmiş sürümleri kullanacak. Bu size otomatik olarak tekrarlanabilir bir derleme sağlar. Başka bir deyişle, projeniz açıkça güncellemediğiniz sürece _Cargo.lock_ dosyası sayesinde 0.8.5'te kalacak. Tekrarlanabilir derlemeler için önemli olduğu için _Cargo.lock_ dosyası genellikle projenizdeki kodun geri kalanıyla birlikte kaynak kontrolüne kontrol edilir.

#### Bir Crate'ı Yeni Bir Sürüm İçin Güncelleme

Bir crate'ı _gerçekten_ güncellemek istediğinizde, Cargo `update` komutunu sağlar, bu _Cargo.lock_ dosyasını yok sayacak ve _Cargo.toml_'daki belirtimelerinize uyan tüm en son sürümleri belirleyecek. Cargo sonra o sürümleri _Cargo.lock_ dosyasına yazacak. Aksi taktirde, varsayılan olarak Cargo sadece 0.8.5'ten büyük ve 0.9.0'ın altında olan sürümlere bakacaktır. Eğer `rand` crate'ı 0.8.6 ve 0.999.0 sürümlerinde iki yeni sürüm yayınlarsa, `cargo update` çalıştırdığınızda şunu görürsünüz:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
cargo update
assuming there is a new 0.8.x version of rand; otherwise use another update
as a guide to creating the hypothetical output shown here -->

```console
$ cargo update
    Updating crates.io index
     Locking 1 package to latest Rust 1.85.0 compatible version
    Updating rand v0.8.5 -> v0.8.6 (available: v0.999.0)
```

Cargo 0.999.0 sürümünü yok sayar. Bu noktada, _Cargo.lock_ dosyanızda `rand` crate'ının şu anda kullandığınız sürümünün 0.8.6 olduğunu belirten bir değişiklik fark edersiniz. `rand`'ın 0.999.0 sürümünü veya 0.999._x_ serisindeki herhangi bir sürümü kullanmak isterseniz, _Cargo.toml_ dosyasını şöyle görünecek şekilde güncellemelisiniz (takip eden örneklerin `rand` 0.8 kullandığınızı varsadığından bu değişikliği yapmayın):

```toml
[dependencies]
rand = "0.999.0"
```

Bir sonraki `cargo build` çalıştırdığınızda, Cargo mevcut crate kayd defterini güncelleyecek ve belirlediğiniz yeni sürüme göre `rand` gereksinimlerini tekrar değerlendirecek.

[Cargo][doccargo]<!-- ignore --> ve [onun ekosistemi][doccratesio]<!-- ignore --> hakkında söylecek çok daha fazla şey var, bunları Bölüm 14'te tartışacağız, ancak şimdilik bilmeniz gereken bu kadar. Cargo kütüphaneleri yeniden kullanmayı çok kolaylaştırıyor, bu yüzden Rustaceanlar çok sayıda paketten birleştirilen daha küçük projeler yazabiliyor.

### Rastgele Sayı Oluşturma

Tahmin etmek için bir sayı oluşturmakta `rand` kullanmaya başlayalım. Sonraki adım _src/main.rs_'yi güncellemek, Kod Listesi 2-3'te gösterildiği gibi.

<Listing number="2-3" file-name="src/main.rs" caption="Rastgele sayı oluşturmak için kod ekleme">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:all}}
```

</Listing>

Önce, `use rand::Rng;` satırını ekliyoruz. `Rng` trait'i rastgele sayı üreticilerinin uyguladığı yöntemleri tanımlar ve bu trait'in scope'da olması gerekir ki bu yöntemleri kullanabilelim. Bölüm 10 trait'leri detaylı olarak kapsayacaktır.

Sonra, ortaya iki satır ekliyoruz. İlk satırda, kullanacağımız belirli rastgele sayı üreticisini bize veren `rand::thread_rng` fonksiyonunu çağırıyoruz: birinde geçerli işletim iş parçası için yerel olan ve işletim sistemi tarafından tohumlanmış olan. Sonra, rastgele sayı üreticisinde `gen_range` yöntemini çağırıyoruz. Bu yöntem `use rand::Rng;` ifadesiyle scope'a getirdiğimiz `Rng` trait'i tarafından tanımlanmıştır. `gen_range` yöntemi bir aralık ifadesini argüman olarak alır ve aralık içinde rastgele bir sayı üretir. Burada kullandığımız aralık ifadesinin türü `start..=end` şeklindedir ve alt ve üst sınırlar dahildir, bu yüzden 1 ile 100 arasında bir sayı istemek için `1..=100` belirtmemiz gerekiyor.

> Not: Hangi trait'leri kullanacağınızı ve hangi yöntemleri ve fonksiyonları bir crate'den çağıracağınızı bilmeyeceksiniz, bu yüzden her crate'in kullanım talimatlarını içeren bir dokümantasyonu vardır. Cargo'nun başka bir güzel özelliği `cargo doc --open` komutunu çalıştırmanız, bu bağımlılıklarınız tarafından sağlanan dokümantasyonu yerel olarak oluşturacak ve tarayıcınızda açacaktır. Örneğin, `rand` crate'ındaki başka işlevsellikle ilgileniyorsanız, `cargo doc --open` çalıştırın ve soldaki kenar çubuğundaki `rand`'a tıklayın.

İkinci yeni satır gizli sayıyı yazdırır. Bu programı geliştirirken test etmemizi sağlamak için kullanışlıdır, ancak final sürümden sileceğiz. Program başlar başlar cevabı yazdırsa pek bir oyun olmaz!

Programı birkaç kez çalıştırmayı deneyin:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-03/
cargo run
4
cargo run
5
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4

$ cargo run
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
```

Farklı rastgele sayılar almalısınız ve hepsinin 1 ile 100 arasında sayılar olması gerekir. İyi iş!

## Tahmini Gizli Sayıyla Karşılaştırma

Şu ana kadar kullanıcı girdisi ve bir rastgele sayımız var, onları karşılaştırabiliriz. Bu adım Kod Listesi 2-4'te gösterilmiştir. Bu kodun henüz derlenmeyeceğini not edin, açıklayacağız.

<Listing number="2-4" file-name="src/main.rs" caption="İki sayıyı karşılaştırmanın olası geri dönüş değerlerini işleme">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-04/src/main.rs:here}}
```

</Listing>

Önce, standart kütüphaneden `std::cmp::Ordering` adında bir tipi scope'a getiren başka bir `use` ifadesi ekliyoruz. `Ordering` tipi başka bir enum'dir ve `Less`, `Greater` ve `Equal` varyantlarına sahiptir. Bunlar iki değeri karşılaştırdığınızda olası üç sonuçtur.

Sonra, en altta `Ordering` tipini kullanan beş yeni satır ekliyoruz. `cmp` yöntemi iki değeri karşılaştırır ve karşılaştırılabilen herhangi bir şeyde çağırılabilir. Karşılaştırmak istediğiniz her şeye bir referans alır: Burada, `guess`'ü `secret_number` ile karşılaştırıyor. Sonra, `use` ifadesiyle scope'a getirdiğimiz `Ordering` enum'ün bir varyantını döndürür. `guess` ve `secret_number` içindeki değerlerle `cmp` çağrısından dönen `Ordering` varyantına göre ne yapacağımızı belirlemek için bir [`match`][match]<!-- ignore --> ifadesi kullanıyoruz.

Bir `match` ifadesi _kollardan_ oluşur. Bir kol, eşleşmesi gereken bir _desen_ ve verilen `match` ifadesine o desen uyarsa çalıştırması gereken koddan oluşur. Rust verilen `match` ifadesine alır ve her kolun desenini sırayla kontrol eder. Desenler ve `match` yapısı güçlü Rust özellikleridir: Bunlar kodunuzun karşılaşabileceği çok fazla durumu ifade etmenizi sağlar ve hepsini işlemenizi sağlarlar. Bu özellikler sırasıyla Bölüm 6'da ve Bölüm 19'de detaylı olarak kapsayacaktır.

Burada kullandığımız `match` ifadesi ile bir örnek üzerinden gezelim. Kullanıcının 50 tahmin ettiğini ve bu seferde rastgele oluşturulan gizli sayının 38 olduğunu varsayalım.

Kod 50'yi 38 ile karşılaştırdığında, `cmp` yöntemi `Ordering::Greater` döndürecektir çünkü 50, 38'den büyüktür. `match` ifadesi `Ordering::Greater` değerini alır ve her kolun desenini kontrol etmeye başlar. İlk kolun desenine, `Ordering::Less` bakar ve `Ordering::Greater` değerinin `Ordering::Less` ile eşleşmediğini görür, bu yüzden o koldaki kodu yok sayar ve sonraki kola geçer. Sonraki kolun deseni `Ordering::Greater`'dir ve bu _eşleşir_ `Ordering::Greater` ile! O koldaki kod çalışacak ve ekrana `Too big!` yazacak. `match` ifadesi ilk başarılı eşleşmeden sonra biter, bu yüzden bu senaryoda son kola bakmaz.

Ancak, Kod Listesi 2-4'teki kod henüz derlenemiyor. Deneyelim:

<!--
The error numbers in this output should be that of the code **WITHOUT** the
anchor or snip comments
-->

```console
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-04/output.txt}}
```

Hatanın özü, _eşleşmeyen tiplerin_ (mismatched types) olduğunu söyler. Rust güçlü, statik bir tip sistemine sahiptir. Ancak, ayrıca tip çıkarımı (type inference) özelliğine sahiptir. `let mut guess = String::new()` yazdığımızda, Rust `guess`'ün bir `String` olması gerektiğini çıkarabildi ve bize tipi yazmamızı zorlamadı. `secret_number`, diğer yandan, bir sayı tipidir. Rust'ın birkaç sayı tipi 1 ile 100 arasında bir değere sahip olabilir: `i32`, 32-bit sayı; `u32`, işaretsiz 32-bit sayı; `i64`, 64-bit sayı; başkaları da vardır. Aksi belirtilmedikçe, Rust varsayılan olarak bir `i32`'ye varsayar, bu `secret_number`'ın tipidir, size başka yerde Rust'ın farklı bir sayısal tip çıkarmasına neden olacak tip bilgisi eklemediğiniz sürece. Hatanın nedeni Rust'ın bir dizeyi ve bir sayı tipini karşılaştıramamasıdır.

Sonuçta, programın girdi olarak okuduğu `String`'i sayısal olarak gizli sayıyla karşılaştırabilmek için bir sayı tipine dönüştürmek istiyoruz. Bunu `main` fonksiyonu gövdesine şu satırı ekleyerek yapıyoruz:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/src/main.rs:here}}
```

Satır şöyledir:

```rust,ignore
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```

`guess` adında bir değişken oluşturuyoruz. Ama bekleyin, program zaten `guess` adında bir değişkene sahip değil mi? Sahip, ancak Rust bizim yardımse bir önceki `guess` değerini yeni biriyle gölgelememize izin veriyor. _Gölgeleme_ (shadowing) bizi `guess_str` ve `guess` gibi iki benzersiz değişken yaratmaya zorlamak yerine `guess` değişken adını yeniden kullanmamızı sağlar. Bunu Bölüm 3'te [Gölgeleme][shadowing]<!-- ignore --> daha detaylı olarak kapsayacağız, ancak şimdilik bu özelliğin bir değeri bir tipten başka bir tipte dönüştürmek istediğinizde sık sık kullanıldığını bilin.

Bu yeni değişkeni `guess.trim().parse()` ifadesine bağlıyoruz. İfadeki `guess` orijinal `guess` değişkenine, girdi olarak dizeyi içeren, refer eder. `String` örneği üzerindeki `trim` yöntemi başlangıçtaki ve sonundaki herhangi bir boşluğu ortadan kaldıracaktır, bunu dizeyi bir `u32`'ye dönüştürmeden önce yapmamız gerekiyor, `u32` sadece sayısal veri içerebilir. Kullanıcı <kbd>enter</kbd> tuşuna basmalıdır ki `read_line`'i tatmin etsin ve tahminini girsin, bu dizeye yeni satır karakteri ekler. Örneğin, kullanıcı <kbd>5</kbd> yazıp <kbd>enter</kbd> tuşuna basarsa, `guess` şöyle görünür: `5\n`. `\n` "yeni satır"ı temsil eder. (Windows üzerinde, <kbd>enter</kbd> tuşuna basmak bir satır başı ve yeni satır, `\r\n` sonucunu verir.) `trim` yöntemi `\n` veya `\r\n`'i ortadan kaldırır, sonuç olarak sadece `5` kalır.

Dizeler üzerindeki [`parse` yöntemi][parse]<!-- ignore --> bir dizeyi başka bir tipte dönüştürür. Burada, onu bir dizeden sayıya dönüştürmek için kullanıyoruz. Rust'a tam olarak istediğimiz sayı tipini `let guess: u32` kullanarak söylememiz gerekiyor. `guess`'den sonra gelen iki nokta (`:`), Rust'a değişkenin tipini not edeceğimizi söyler. Rust birkaç yerleşik sayı tipine sahiptir; burada görülen `u32` işaretsiz, 32-bit tam sayıdır. Küçük pozitif bir sayı için iyi bir varsayılan seçenektir. Başka sayı tiplerini Bölüm 3'te [Tamsayılar][integers]<!-- ignore --> öğreneceksiniz.

Ek olarak, bu örnek programdaki `u32` notasyonu ve `secret_number` ile karşılaştırma, Rust'ın `secret_number`'ün de bir `u32` olması gerektiğini çıkaracağını sağlar. Bu yüzden, şimdi karşılaştırma aynı tipin iki değeri arasında olacak!

`parse` yöntemi sadece sayılara mantıksal olarak dönüştürilebilen karakterlerde çalışacaktır bu yüzden kolayca hatalara neden olabilir. Örneğin, dize `A👍%` içerirse, onu sayıya dönüştürmenin bir yolu olmaz. Başarısız olabileceği için, `parse` yöntemi, daha önce `read_line` yönteminin yaptığı gibi (["`Result` ile Olası Hataları İşleme"](#handling-potential-failure-with-result)<!-- ignore --> bölümünde tartışılan) bir `Result` tipi döndürür. Biz bu `Result`'ü yine `expect` yöntemini kullanarak aynı şekilde işleyeceğiz. Eğer `parse` dizeden sayı oluşturamadığı için bir `Err` `Result` varyantını döndürürse, `expect` çağrısı oyunu çökertirecek ve ona verdiğimiz mesajı görüntüleyecek. Eğer `parse` dizeyi başarıyla sayıya dönüştürebilirse, `Result`'ün `Ok` varyantını döndürecektir ve `expect` `Ok`'un tuttuğu bizim istediğimiz sayıyı döndürecektir.

Şimdi programı çalıştıralım:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/
touch src/main.rs
cargo run
  76
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.26s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

Güzel! Tahminin öncesinde boşluklar eklense bile, program kullanıcının 76 tahmin ettiğini hala fark etti. Programı birkaç kez çalıştırıp farklı türdeki girdilerle farklı davranışı doğrulayın: Sayıyı doğru tahmin edin, çok yüksek bir sayı tahmin edin ve çok düşük bir sayı tahmin edin.

Oyunun çoğu şu ana kadar çalışıyor, ancak kullanıcı sadece bir tahmin yapabiliyor. Bir döngü ekleyerek bunu değiştirelim!

## Döngü ile Çoklu Tahminlere İzin Verme

`loop` anahtar kelimesi sonsuz bir döngü oluşturur. Kullanıcılara sayı tahmin etmede daha fazla şans vermek için bir döngü ekleyeceğiz:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-04-looping/src/main.rs:here}}
```

Görebileceğiniz gibi, tahmin girdisi isteminden itibaren her şeyi döngünün içine taşıdık. Döngünün içindeki satırları her biri için dört boşluk fazladan girmediğinizden emin olun ve programı tekrar çalıştırın. Program artık sonsuza kadar başka bir tahmin isteyecek, bu aslında yeni bir problem getirecektir. Kullanıcının çıkabildiği gibi görünmüyor!

Kullanıcı her zaman <kbd>ctrl</kbd>-<kbd>C</kbd> klavye kısayoluyla programı kesintiye zorlayabilir. Ancak [`"Tahmini Gizli Sayıyla Karşılaştırma"`](#comparing-the-guess-to-the-secret-number)<!-- ignore --> bölümündeki `parse` tartışmasında bahsettiğimiz gibi bu kaçamaz yaratığından çıkmak için başka bir yol var: Eğer kullanıcı sayı olmayan bir cevap girerse, program çökecek. Bunu kullanıcının çıkmesine izin vermek için kullanabiliriz, burada gösterildiği gibi:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-04-looping/
touch src/main.rs
cargo run
(too small guess)
(too big guess)
(correct guess)
quit
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.23s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit

thread 'main' panicked at src/main.rs:28:47:
Please type a number!: ParseIntError { kind: InvalidDigit }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

`quit` yazmak oyunu çıkaracaktır, ancak fark edeceğiniz gibi, başka herhangi bir sayı olmayan girdi de öyle yapacaktır. Bu en azından söylendiğimiz kadar altoptimaldir; oyun doğru sayı tahmin edildiğinde de durması gerekiyor.

### Doğru Tahminden Sonra Çıkma

Kullanıcının kazandığında oyunu çıkması için programlamayı, bir `break` ifadesi ekleyerek yapalım:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-05-quitting/src/main.rs:here}}
```

`You win!`'den sonra `break` satırını eklemek, kullanıcının gizli sayıyı doğru tahmin ettiğinde programın döngüden çıkmasına neden olur. Döngüden çıkmak ayrıca programdan çıkmak demektir, çünkü döngü `main`'ın son kısmıdır.

### Geçersiz Girdiyi İşleme

Oyunun davranışını daha da geliştirmek için, kullanıcının sayı olmayan bir şey girdiğinde programı çökertmek yerine, sayı olmayan bir şeyi yok sayarak oyuncunun tahmin etmeye devam etmesini sağlayalım. Bunu Kod Listesi 2-5'te gösterildiği gibi, `guess`'ün bir `String`'den `u32`'ye dönüştürdüğü satırı değiştirerek yapabiliriz.

<Listing number="2-5" file-name="src/main.rs" caption="Sayı olmayan bir tahmini yok sayma ve programı çökertmek yerine başka bir tahmin isteme">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:here}}
```

</Listing>

Hata üzerinde çökmekten, hatayı işlemeye geçmek için bir `expect` çağrısından bir `match` ifadesine geçiyoruz. `parse`'in bir `Result` tipi döndürdüğünü ve `Result`'ün `Ok` ve `Err` varyantlarına sahip olan bir enum olduğunu hatırlayın. Burada, `cmp` yönteminin `Ordering` sonucunu yaptığımız gibi bir `match` ifadesi kullanıyoruz.

Eğer `parse` dizeyi başarıyla sayıya dönüştürebilirse, ortaya çıkan sayıyı içeren bir `Ok` değeri döndürecektir. O `Ok` değeri ilk kolun desenine uyar ve `match` ifadesi sadece `parse`'nin ürettiği ve `Ok` değerinin içine koyduğu `num` değerini döndürecektir. O sayı, oluşturduğumuz yeni `guess` değişkeninde istediğimiz yerde bitecek.

Eğer `parse` dizeyi sayıya dönüştüremiyorsa, hata hakkında daha fazla bilgi içeren bir `Err` değeri döndürecektir. `Err` değeri `Ok(num)` desenini ilk `match` kolunda eşleşmez ama ikinci koldaki `Err(_)` desenine eşleşir. Alt tire, `_`, her şeyi yakalayan bir değerdir; bu örnekte, içerlerinde hangi bilgi olursa olsun tüm `Err` değerlerini eşleşmek istediğimizi söylüyoruz. Bu yüzden, program ikinci kolun kodunu, `continue`'i çalıştıracaktır, bu programa döngünün sonraki yenilemesine gitmesini ve başka bir tahmin istemesini söyler. Bu yüzden, etkili olarak program `parse`'in karşılaşabileceği tüm hataları yok sayar!

Şimdi programdaki her şeyin beklediği gibi çalışması gerekir. Deneyelim:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-05/
cargo run
(too small guess)
(too big guess)
foo
(correct guess)
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

Harika! Küçük bir son incelemeye tahmin oyununu bitireceğiz. Programın hala gizli sayıyı yazdırdığını hatırlayın. Test etmek için iyi çalıştı ancak oyunu bozuyor. Gizli sayıyı yazdıran `println!`'yi silelim. Kod Listesi 2-6 final kodu gösterir.

<Listing number="2-6" file-name="src/main.rs" caption="Tam tahmin oyunu kodu">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-06/src/main.rs}}
```

</Listing>

Bu noktada, tahmin oyununu başarıyla oluşturdunuz. Tebrikler!

## Özet

Bu proje size çok fazla yeni Rust kavramını tanıtan uygulamalı bir yoldu: `let`, `match`, fonksiyonlar, dış crate'ların kullanımı ve daha fazlası. Takip eden birkaç bölümde bu kavramları daha detaylı olarak öğreneceksiniz. Bölüm 3, çoğu programlama dilinin sahip olduğu kavramları kapsar: değişkenler, veri tipleri ve fonksiyonlar ve bunları Rust'ta nasıl kullanacağınızı gösterir. Bölüm 4 sahipliği inceleyerek, Rust'ı başka dillerden farklı yapan bir özellik. Bölüm 5 yapıları (structs) ve yöntem sözdizimini tartışır ve Bölüm 6 enum'ların nasıl çalıştığını açıklayacak.

[prelude]: ../std/prelude/index.html
[variables-and-mutability]: ch03-01-variables-and-mutability.html#variables-and-mutability
[comments]: ch03-04-comments.html
[string]: ../std/string/struct.String.html
[iostdin]: ../std/io/struct.Stdin.html
[read_line]: ../std/io/struct.Stdin.html#method.read_line
[result]: ../std/result/enum.Result.html
[enums]: ch06-00-enums.html
[expect]: ../std/result/enum.Result.html#method.expect
[recover]: ch09-02-recoverable-errors-with-result.html
[randcrate]: https://crates.io/crates/rand
[semver]: http://semver.org
[cratesio]: https://crates.io/
[doccargo]: https://doc.rust-lang.org/cargo/
[doccratesio]: https://doc.rust-lang.org/cargo/reference/publishing.html
[match]: ch06-02-match.html
[shadowing]: ch03-01-variables-and-mutability.html#shadowing
[parse]: ../std/primitive.str.html#method.parse
[integers]: ch03-02-data-types.html#integer-types