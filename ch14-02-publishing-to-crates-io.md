## Crates.io'ya Bir Crate Yayınlama (Publishing a Crate to Crates.io)

[crates.io](https://crates.io/)<!-- ignore --> üzerindeki paketleri projemizin bağımlılıkları olarak kullandık ama kendi paketlerinizi yayınlayarak diğer insanlarla kodunuzu paylaşabilirsiniz. [crates.io](https://crates.io/)<!-- ignore --> üzerindeki crate kayıttığı paketlerinizin kaynak kodunu dağıtırır, bu yüzden öncelikle açık kaynak kod sunar.

Rust ve Cargo, yayınladığınız paketi insanların bulması ve kullanması için kolaylaştıran özelliklere sahiptir. Bu özelliklerden bazılarından sonra konuşacağız ve sonra bir paketi nasıl yayınlacağınızı açıklayacağız.

### Faydalı Belgeleme Yorumları Yapma (Making Useful Documentation Comments)

Paketlerinizi doğru bir şekilde belgelemek, diğer kullanıcılara onları nasıl ve ne zaman kullanacaklarını bilmelerine yardımcı olacaktır, bu yüzden belge yazmak için zaman harcamaya değer. Bölüm 3'te, iki çizgi kullanarak Rust kodunu nasıl yorumlayacağımızı tartıştık. Rust ayrıca belgeleme için özel bir yorum türüne sahiptir ki bu, bir _belgeleme yorumu_ (documentation comment) olarak bilinir, bu HTML belgelemesi oluşturacaktır. HTML, crate'ınızın _nasıl kullanılacağına_ (use) (crate'ınızın _nasıl uygulandığına_ (implemented) karşı) ilgilen programcılar için hedeflenen genel API öğelerinin belgeleme yorumlarının içeriğini gösterir.

Belgeleme yorumları formatlama için Markdown notasyonunu destekleyen iki çizgi yerine üç çizgi, `///` kullanır. Belgeleme yorumlarını, belgeledikleri öğeden hemen önce koyun. Kod Listesi 14-1, `my_crate` adında bir crate'deki `add_one` fonksiyonu için belgeleme yorumlarını gösterir.

<Listing number="14-1" file-name="src/lib.rs" caption="Bir fonksiyon için bir belgeleme yorumu">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-01/src/lib.rs}}
```

</Listing>

Burada, `add_one` fonksiyonunun ne yaptığı bir açıklama veriyoruz, "Örnekler" başlığıyla bir bölüm başlatıyor ve sonra `add_one` fonksiyonunu nasıl kullanacağınızı gösteren kod sağlıyoruz. Bu belgeleme yorumundan HTML belgelemesi `cargo doc` çalıştırmakla oluşturabiliriz. Bu komut Rust ile dağıtılan `rustdoc` aracını çalıştırır ve oluşturulan HTML belgelemesini _target/doc_ dizinine koyar.

Kolaylık için, `cargo doc --open` çalıştırmak mevcut crate'ınızın belgelemesi için HTML oluşturacak (ve tüm crate bağımlılıklarının belgelemesi için) ve sonucu bir web tarayıcısında açacak. `add_one` fonksiyonuna gidin ve belgeleme yorumlarındaki metnin nasıl oluşturulduğunu göreceksiniz, Şekil 14-1'de gösterildiği gibi.

<img alt="`my_crate`'un `add_one` fonksiyonu için oluşturulmuş HTML belgesi" src="img/trpl14-01.png" class="center" />

<span class="caption">Şekil 14-1: `my_crate`'un `add_one` fonksiyonu için
HTML belgesi</span>

#### Yaygın Kullanılan Bölümler (Commonly Used Sections)

Kod Listesi 14-1'de "Örnekler" başlığıyla bir HTML bölümü oluşturmak için `# Examples` Markdown başlığını kullandık. İşte crate yazarlarının belgemelerinde yaygın olarak kullandığı bazı diğer bölümler:

- **Paniklar**: Bunlar, belgelemen fonksiyonunun panik yapabileceği senaryolardır. Programlarının panik yapmasını istemeyen fonksiyon çağıranları, bu durumlarda fonksiyonu çağırmadıklarından emin olmalıdırlar.
- **Hatalar**: Eğer fonksiyon bir `Result` dönerse, oluşabilecek hataların türlerini ve bu hataların hangi koşullarda dönebileceğini açıklamak, farklı şekillerde farklı hata türlerini ele almak için kod yazan çağıranları için yardımcı olabilir.
- **Güvenlik**: Eğer fonksiyon çağırmak `unsafe`'dir (Bölüm 20'de unsafety'yi tartışacağız), fonksiyonun neden unsafe olduğunu ve fonksiyonun çağıranların desteklemesini beklediği değişmezleri kapsayan bir bölüm olmalıdır.

Çoğu belgeleme yorumu tüm bu bölümlere ihtiyaç duymaz ama bu, kullanıcılarnın kodunuzun hangi yönlerini bilmekle ilgilendiklerini hatırlatmak için iyi bir kontrol listesidir.

#### Yorumlar Olarak Belgeleme Yorumları (Documentation Comments as Tests)

Belgeleme yorumlarınıza örnek kod blokları eklemek, kütüphanenizi nasıl kullanacağınızı göstermeye yardımcı olabilir ve ek bir bonusa sahiptir: `cargo test` çalıştırmak belgelemenizdeki kod örneklerini test olarak çalıştıracaktır! Örneklerle belgelerden daha iyisi yoktur. Ancak örneklerin çalışmamasından daha kötüsü yoktur çünkü belge yazıldığından beri kod değişti. Eğer Kod Listesi 14-1'deki `add_one` fonksiyonu için belgemesi ile `cargo test` çalıştırsak, test sonuçlarında şu gibi görünen bir bölüm göreceğiz:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-01/
cargo test
copy just relevant lines below
-->

```text
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
```

Şimdi, fonksiyonu veya örneği değiştirirsek ki örnekteki `assert_eq!` panik yapar ve tekrar `cargo test` çalıştırsak, belge testlerinin örnek ile kodun birbirleriyle senkronize olmadığını yakaladığını göreceğiz!

<a id="commenting-contained-items"></a>

#### İçeren Öğe Yorumları (Contained Item Comments)

`//!` belge yorumu tarzı, yorumları *izleyen* öğeler yerine yorumları *içeren* öğeye belgeleme ekler. Genellikle bu belgeleme yorumlarını crate kök dosyasında (_src/lib.rs_ kuralı) veya crate'i veya modülü bir bütün olarak belgelemek için bir modül içinde kullanırız.

Örneğin, `add_one` fonksiyonunu içeren `my_crate` crate'in amacını açıklayan belgeleme eklemek için, Kod Listesi 14-2'de gösterildiği gibi _src/lib.rs_ dosyasının başlangıcına `//!` ile başlayan belgeleme yorumları ekleriz.

<Listing number="14-2" file-name="src/lib.rs" caption="`my_crate` crate'in bir bütün olarak belgesi">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-02/src/lib.rs:here}}
```

</Listing>

`//!` ile başlayan son satırdan sonra herhangi bir kod olmadığına dikkat edin. `///` yerine `//!` ile yorumları başlattığımızdan, bu yorumu izleyen öğeyi değil, bu yorumu içeren öğeyi belgelediğimizizi fark edin. Bu durumda, bu öğe _src/lib.rs_ dosyasıdır ki crate köküdür. Bu yorumlar tüm crate'yi açıklar.

`cargo doc --open` çalıştırdığımızda, bu yorumlar crate'deki genel öğelerin listesi üzerinde `my_crate`'in belgesinin ön sayfasında görüntülenecek, Şekil 14-2'de gösterildiği gibi.

Öğeler içindeki belgeleme yorumları özellikle crate'leri ve modülleri açıklamak için yararlıdır. Onları kullanarak kullanıcıların crate'ın organizasyonunu anlamasına yardımcı olmak için konteynerin genel amacını açıklayın.

<img alt="Bir bütün olarak crate için bir yorum ile oluşturulmuş HTML belgesi" src="img/trpl14-02.png" class="center" />

<span class="caption">Şekil 14-2: Crate'i bir bütün olarak açıklayan yorum
ile oluşturulmuş `my_crate`'in oluşturulmuş belgesi</span>

<a id="exporting-a-convenient-public-api-with-pub-use"></a>

### Uygun Bir Genel API İhrac Etme (Exporting a Convenient Public API)

Genel API'nizin yapısı, bir crate yayınlarken büyük bir düşüncedir. Crate'ınızı kullanan kişiler, yapısından siz daha az tanıdık olabilir ve crate'iniz büyük bir modül hiyerarşisine sahipse kullanmak istedikleri parçaları bulmakta zorluk yaşayabilirler.

Bölüm 7'de, `pub` anahtar kelimesini kullanarak öğelerin genel yapmayı ve `use` anahtar kelimesini kullanarak öğeleri kapsama nasıl getireceğimizi ele aldık. Ancak, bir crate geliştirirken sizin için mantıklı gelen yapısı kullanıcılarınız için çok uygun olmayabilir. Çok seviye içeren bir hiyerarşide yapılarınızı düzenlemek isteyebilirsiniz ama sonra hiyerarşide derininde tanımladığınız bir türü kullanmak isteyen kişiler türün var olduğunu bulmakta zorluk yaşayabilirler. Ayrıca `my_crate::UsefulType;` yerine `my_crate::some_module::another_module::UsefulType;` yazmak zorunda kalmaktan da sinirlenebilirler.

İyi haber şu ki, yapısı başkaların başka bir kütüphaneden kullanmak için uygun _değilse_, iç organizasyonunuzu yeniden düzenlemeniz gerekmeyebilir: Bunun yerine, `pub use` kullanarak genel yapınızı özel yapınızdan farklı bir genel yapı yaparak öğeleri yeniden ihrac edebilirsiniz. *Yeniden ihrac etme* (Re-exporting) bir konumdaki genel bir öğeyi alır ve onu başka bir yerde genel yapar, sanki başka yerde tanımlanmış gibi.

Örneğin, `art` adında sanatsal konseptleri modellemek için bir kütüphane yaptığımızı söyleyin. Bu kütüphanenin içinde iki modül var: `PrimaryColor` ve `SecondaryColor` adında iki numaralandırma içeren bir `kinds` modülü ve `mix` adında bir fonksiyon içeren bir `utils` modülü, Kod Listesi 14-3'te gösterildiği gibi.

<Listing number="14-3" file-name="src/lib.rs" caption="Öğeleri `kinds` ve `utils` modüllerinde organize edilmiş bir `art` kütüphanesi">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-03/src/lib.rs:here}}
```

</Listing>

Şekil 14-3, `cargo doc` tarafından bu crate için oluşturulmuş belgenin ön sayfasının nasıl görüneceğini gösterir.

<img alt="`kinds` ve `utils` modüllerini listeleyen `art` crate'in oluşturulmuş belgesi" src="img/trpl14-03.png" class="center" />

<span class="caption">Şekil 14-3: `kinds` ve `utils` modüllerini listeleyen
`art` crate'in ön sayfası</span>

`PrimaryColor` ve `SecondaryColor` türlerinin ön sayfada listelenmediğine ve `mix` fonksiyonunun da listelenmediğine dikkat edin. Onları görmek için `kinds` ve `utils`'e tıklamız gerekiyor.

Bu kütüphaneye bağımlı olan başka bir crate, öğeleri `art`'ten kapsama getirmek için `use` ifadeleri gerekecektir ki bu, şu anda tanımlanmış olan modül yapısını belirtir. Kod Listesi 14-4, `art` crate'in `PrimaryColor` ve `mix` öğelerini kullanan bir crate örneği gösterir.

<Listing number="14-4" file-name="src/main.rs" caption="`art` crate'in öğelerini onun iç yapısını ihraç ederek kullanan bir crate">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-04/src/main.rs}}
```

</Listing>

Kod Listesi 14-4'teki kodun yazarı, `art` crate'ini kullanan, `PrimaryColor`'in `kinds` modülünde olduğunu ve `mix`'in `utils` modülünde olduğunu bulmak zorundaydı. `art` crate'inin modül yapısı, `art` crate'ini kullananlara göre onu üzerinde çalışan geliştiricilerle daha alakalıdır. İç yapısı, `art` crate'ini kullanmaya çalışanlar için kullanmak için herhangi bir faydalı bilgi içermiyor ve daha çok kafa karıştırmasına neden oluyor çünkü onu kullanan geliştiriciler nereye bakacaklarını bulmak zorunda kalıyorlar ve `use` ifadelerinde modül adlarını belirtmek zorundalar.

Genel API'den iç organizasyonu kaldırmak için, Kod Listesi 14-3'teki `art` crate kodunu Kod Listesi 14-5'te gösterildiği gibi üst seviyede öğeleri yeniden ihrac etmek için `pub use` ifadeleri ekleyerek değiştirebiliriz.

<Listing number="14-5" file-name="src/lib.rs" caption="Öğeleri yeniden ihrac etmek için `pub use` ifadeleri ekleme">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-05/src/lib.rs:here}}
```

</Listing>

Bu crate için `cargo doc` oluşturduğu API belgesi artık Şekil 14-4'te gösterildiği gibi ön sayfada yeniden ihracları listeleyecek ve bağlayacak ki bu, `PrimaryColor` ve `SecondaryColor` türlerini ve `mix` fonksiyonunu bulmayı kolaylaştıracaktır.

<img alt="Ön sayfada yeniden ihracları olan `art` crate'in oluşturulmuş belgesi" src="img/trpl14-04.png" class="center" />

<span class="caption">Şekil 14-4: Yeniden ihracları listeleyen `art` crate'in
ön sayfası</span>

`art` crate kullanıcıları hala Kod Listesi 14-3'teki iç yapısını görebilir ve Kod Listesi 14-4'te gösterildiği gibi kullanabilirler ya da Kod Listesi 14-5'teki daha uygun yapıyı kullanabilirler, Kod Listesi 14-6'da gösterildiği gibi.

<Listing number="14-6" file-name="src/main.rs" caption="`art` crate'nden yeniden ihrac edilmiş öğeleri kullanan bir program">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-06/src/main.rs:here}}
```

</Listing>

Çok iç içe geçmiş modüllerin olduğu durumlarda, üst seviyede `pub use` kullanarak türleri yeniden ihrac etmek, crate'i kullananların deneyimlerinde önemli bir fark yapabilir. `pub use`'un başka yaygın kullanımı, bağımlılığın tanımlarını mevcut crate'te yeniden ihrac etmek ve bu bağımlılığın tanımlarını crate'inizin genel API'sinin bir parçası yapmaktır.

Kullanışlı bir genel API yapısı oluşturmak bir bilimden daha çok bir sanattır ve kullanıcılarınız için en iyi çalışan API'yi bulmak için yineleyebilirsiniz. `pub use` seçmek, crate'inizi iç nasıl yapılandıracağınıza esneklik verir ve iç yapısını kullanıcılara sunduğunuz şeylerden ayırır. Yüklediğiniz crate'ların koduna bakın ve iç yapılarının genel API'lerinden farklı olup olmadığını görün.

### Bir Crates.io Hesabı Kurma (Setting Up a Crates.io Account)

Herhangi bir crate yayınlamadan önce, [crates.io](https://crates.io/)<!-- ignore --> üzerinde bir hesap oluşturmanız ve bir API token'i almanız gerekiyor. Bunu yapmak için, [crates.io](https://crates.io/)<!-- ignore --> ana sayfasını ziyaret edin ve bir GitHub hesabı üzerinden giriş yapın. (GitHub hesabı şu an için bir gereklilik ancak site gelecekte hesap oluşturmanın başka yollarını destekleyebilir.) Bir kez giriş yaptığınızda, [https://crates.io/me/](https://crates.io/me/)<!-- ignore --> adresinden hesap ayarlarınıza gidin ve API anahtarınızı alın. Sonra, şu gibi istendiğinde `cargo login` komutunu çalıştırın ve API anahtarınızı yapıştırın:

```console
$ cargo login
abcdefghijklmnopqrstuvwxyz012345
```

Bu komut Cargo'ya API tokeninizi bildirecek ve onu _~/.cargo/credentials.toml_'de yerel olarak saklayacak. Bu tokenin bir sır olduğuna dikkat edin: Başkasıyla paylaşmayın. Herhangi bir nedenle başkasıyla paylaşırsanız, onu iptal etmeli ve [crates.io](https://crates.io/)<!-- ignore --> üzerinde yeni bir token oluşturmalısınız.

### Yeni Bir Crate'a Metadata Ekleme (Adding Metadata to a New Crate)

Yayınlamak istediğiniz bir crate'iniz olduğunu söyleyin. Yayınlamadan önce, crate'inizin _Cargo.toml_ dosyasındaki `[package]` bölümüne bazı metadata eklemeniz gerekecektir.

Crate'iniz benzersiz bir ada ihtiyaç duyacak. Crate'i yerel üzerinde çalışırken istediğiniz adı verebilirsiniz. Ancak, [crates.io](https://crates.io/)<!-- ignore --> üzerindeki crate adları ilk gelen, ilk servis esasına göre ayrılır. Bir kez crate adı alındı mı, başkası o adla bir crate yayınlamayabilir. Bir crate yayınlamaya çalışmadan önce, kullanmak istediğiniz ad için arama yapın. Eğer ad kullanıldıysa, başka bir ad bulmanız ve yayınlama için _Cargo.toml_ dosyasındaki `[package]` bölümü altındaki `name` alanını yeni adı kullanmak için düzenlemeniz gerekecektir, şu gibi:

<span class="filename">Dosya Adı: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
```

Benzersiz bir ad seçmiş olsanız bile, bu noktada crate'i yayınlamak için `cargo publish` çalıştırsanız, bir uyarı ve sonra bir hata alırsınız:

```console
$ cargo publish
    Updating crates.io index
warning: manifest has no description, license, license-file, documentation, homepage or repository.
See https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata for more info.
--snip--
error: failed to publish to registry at https://crates.io

Caused by:
  remote server responded with an error (status 400 Bad Request): missing or empty metadata fields: description, license. Please see https://doc.rust-lang.org/cargo/reference/manifest.html for more information on configuring these fields
```

Bu bir hataya neden olur çünkü bazı kritik bilgiler eksik: Bir açıklama ve lisans gereklidir ki insanlar crate'inizin ne yaptığını ve hangi şartlarda kullanabileceklerini bilsinler. _Cargo.toml_'de, crate'iniz arama sonuçlarında görüneceği için bir cümle veya ikilik bir açıklama ekleyin. `license` alanı için, bir _lisans tanımlayıcı değeri_ (license identifier value) vermeniz gerekir. [Linux Foundation'ın Software Package Data Exchange (SPDX)][spdx] bu değer için kullanabileceğiniz tanımlayıcıları listeler. Örneğin, crate'inizi MIT Lisansı kullanarak lisansladığınızı belirtmek için, `MIT` tanımlayıcısını ekleyin:

<span class="filename">Dosya Adı: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
license = "MIT"
```

SPDX'de görünmeyen bir lisans kullanmak istiyorsanız, o lisansın metnini bir dosyaya koymanız, dosyayı projenize dahil etmeniz ve sonra `license` anahtarı yerine `license-file` kullanarak o dosyanın adını belirtmeniz gerekecektir.

Projeniz için hangi lisansın uygun olduğu rehberliği bu kitabın kapsamının dışındadır. Rust topluluğundaki birçok kişi projelerini Rust'ın kullandığı aynı yolla, yani `MIT OR Apache-2.0` çifte lisansı kullanarak lisanslarlar. Bu uygulama, `OR` ile ayrılmış birden fazla lisans tanımlayıcısını belirleyerek projeniz için birden fazla lisansınız olabileceğini gösterir.

Benzersiz bir ad, sürüm, açıklamanız ve bir lisans eklediğinizde, yayınlamaya hazır bir projenin _Cargo.toml_ dosyası şu gibi görünebilir:

<span class="filename">Dosya Adı: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2024"
description = "Bilgisayarın seçtiği sayıyı tahmin ettiğiniz eğlenceli bir oyun."
license = "MIT OR Apache-2.0"

[dependencies]
```

[Cargo'nun belgesi](https://doc.rust-lang.org/cargo/), başkaların crate'inizi daha kolay bulmasını ve kullanmasını sağlamak için belirtebileceğiniz diğer metadata'yı açıklar.

### Crates.io'ya Yayınlama (Publishing to Crates.io)

Şimdiye kadar bir hesap oluşturdunuz, API tokeninizi kaydettiniz, crate'iniz için bir ad seçtiniz ve gerekli metadata'yı belirttiniz, yayınlamaya hazırsınız! Bir crate yayınlamak [crates.io](https://crates.io/)<!-- ignore --> üzerinde başkaların kullanması için belirli bir sürümü yükler.

Dikkatli olun, çünkü bir yayınlama _kalıcıdır_. Sürüm hiçbir zaman üst yazılamaz ve kod belirli koşullar dışında silinemez. Crates.io'nun bir ana hedefi, kodun kalıcı bir arşivi olarak işlemesidir ki bu, [crates.io](https://crates.io/)<!-- ignore --> üzerindeki crate'lere bağımlı olan tüm projelerin yapıları çalışmaya devam edecektir. Sürüm silinmesine izin vermek bu hedefi karşılanmasını imkansız kılardı. Ancak, yayınlayabileceğiniz crate sürümlerinin sayısında bir limit yoktur.

`cargo publish` komutunu tekrar çalıştırın. Şimdi başarılı olmalı:

```console
$ cargo publish
    Updating crates.io index
   Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
    Packaged 6 files, 1.2KiB (895.0B compressed)
   Verifying guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.19s
   Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
    Uploaded guessing_game v0.1.0 to registry `crates-io`
note: waiting for `guessing_game v0.1.0` to be available at registry
`crates-io`.
You may press ctrl-c to skip waiting; the crate should be available shortly.
   Published guessing_game v0.1.0 at registry `crates-io`
```

Tebrikler! Kodunuzu şimdi Rust topluluğuyla paylaştınız ve herhangi bir kişi crate'inizi projelerinin bir bağımlılığı olarak kolayca ekleyebilir.

### Mevcut Bir Crate'in Yeni Sürümünü Yayınlama

Crate'inizde değişiklikler yaptınız ve yeni bir sürüm yayınlamaya hazır olduğunuzda, _Cargo.toml_ dosyanızda belirtilen `version` değerini değiştirin ve tekrar yayınlarkın. Uygun bir sonraki sürüm numarasını karar vermek için [Semantik Sürümleme kurallarını][semver] kullanın ve yaptığığınız değişiklik türlerine dayanarak. Sonra, yeni sürümü yüklemek için `cargo publish` çalıştırın.

<a id="removing-versions-from-cratesio-with-cargo-yank"></a>
<a id="deprecating-versions-from-cratesio-with-cargo-yank"></a>

### Crates.io'dan Sürümleri Kullanımdan Kaldırma (Deprecating Versions from Crates.io)

Bir crate'in önceki sürümlerini kaldıramasanız da gelecekteki projelerin onları yeni bir bağımlılık olarak eklemesini engelleyebilirsiniz. Bu, bir crate sürümünün bir nedenden ötürü bozuk olduğu veya başka bir durum olduğunda yararlıdır. Bu durumlarda Cargo, crate sürümünü yank etmeyi (yanking) destekler.

Bir sürümü yank etmek, yeni projelerin bu sürüme bağımlamasını önlerken bu sürüme zaten bağımlı olan mevcut projelerin devam etmesine izin verir. Temel olarak, bir yank, _Cargo.lock_'i olan tüm projelerin bozulmayacağını ve oluşturulan herhangi bir gelecekteki _Cargo.lock_ dosyasının yanked sürümü kullanmayacağını anlamına gelir.

Bir crate'in bir sürümünü yank etmek için, daha önce yayınladığınız crate'in dizininde, sürümü belirtmek için `cargo yank` çalıştırın. Örneğin, eğer `guessing_game` adında bir crate'in 1.0.1 sürümünü yayınladıysak ve onu yank etmek istiyorsak, `guessing_game` için proje dizininde şu şekilde çalıştırırdık:

```console
$ cargo yank --vers 1.0.1
    Updating crates.io index
        Yank guessing_game@1.0.1
```

Komuta `--undo` ekleyerek, bir yankı da geri alabilir ve projelerin bir sürüme tekrar bağımlamasına izin verebilirsiniz:

```console
$ cargo yank --vers 1.0.1 --undo
    Updating crates.io index
      Unyank guessing_game@1.0.1
```

Yank _herhangi bir kod silmez_. Örneğin, yanlışlıkla yüklenmiş sırları silemez. Bu olursa, bu sırları hemen sıfırlamanız gerekir.

[spdx]: https://spdx.org/licenses/
[semver]: https://semver.org/