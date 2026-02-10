## Hello, Cargo!

Cargo Rust'ın derleme sistemi ve paket yöneticisidir. Çoğu Rustacean bu aracı Rust projelerini yönetmek için kullanır, çünkü Cargo sizin için kodunuzu derlemek, kodunuzun bağımlı olduğu kütüphaneleri indirmek ve bu kütüphaneleri derlemek gibi birçok görevi halleder. (Kodunuzun ihtiyaç duyduğu kütüphanelere _bağımlılıklar_ (dependencies) diyoruz.)

Şu ana kadar yazdığımız gibi en basit Rust programlarının herhangi bir bağımlılığı yoktur. "Hello, world!" projesini Cargo ile kurmuş olsaydık, sadece kodunuzu derlemeyi hapseden Cargo'ın bir bölümünü kullanırdı. Daha karmaşık Rust programları yazdıkça, bağımlılıklar ekleyeceksiniz ve eğer bir projeyi Cargo kullanarak başlatırsanız, bağımlılık eklemek çok daha kolay olacaktır.

Rust projelerinin büyük çoğunluğu Cargo kullandığından, bu kitabın geri kalanı sizin de Cargo kullandığınızı varsayar. ["Kurulum"][installation]<!-- ignore --> bölümünde tartışılan resmi yükleyicileri kullandıysanız, Cargo Rust ile birlikte gelir. Rust'ı başka bir yolla kurduysanız, Cargo'nun kurulu olup olmadığını görmek için terminalinize şunu girerek kontrol edin:

```console
$ cargo --version
```

Bir sürüm numarası görürseniz, onunuz var! `command not found` gibi bir hata görürseniz, Cargo'yu ayrı olarak nasıl kuracağınızı belirlemek için kurulum yönteminiz için dokümantasyona bakın.

### Cargo ile Proje Oluşturma

Cargo kullanarak yeni bir proje oluşturalım ve orijinal "Hello, world!" projemizden nasıl farklılaştığına bakalım. _projects_ dizininize (veya kodunuzu saklamaya karar verdiğiniz herhangi bir yere) geri gidin. Sonra, herhangi bir işletim sisteminde şunu çalıştırın:

```console
$ cargo new hello_cargo
$ cd hello_cargo
```

İlk komut _hello_cargo_ adında yeni bir dizin ve proje oluşturur. Projemizi _hello_cargo_ adlandırdık ve Cargo dosyaları aynı adlı bir dizinde oluşturur.

_hello_cargo_ dizinine gidin ve dosyaları listeyin. Cargo'nun bizim için iki dosya ve bir dizin oluşturduğunu göreceksiniz: bir _Cargo.toml_ dosyası ve içinde _main.rs_ dosyası olan bir _src_ dizini.

Ayrıca _.gitignore_ dosyasıyla birlikte yeni bir Git deposunu da başlattı. `cargo new` komutunu varolan bir Git deposunun içinde çalıştırırsanız Git dosyaları oluşturulmaz; `cargo new --vcs=git` kullanarak bu davranışı geçersiz kılabilirsiniz.

> Not: Git yaygın bir sürüm kontrol sistemidir. `cargo new`'ı farklı bir sürüm kontrol sistemi kullanmak veya hiçbir sürüm kontrol sistemi kullanmamak için `--vcs` bayrağını kullanarak değiştirebilirsiniz. Mevcut seçenekleri görmek için `cargo new --help` çalıştırın.

Seçtiğiniz metin düzenleyicinizde _Cargo.toml_'yi açın. Kod Listesi 1-2'deki koda benzer görünmelidir.

<Listing number="1-2" file-name="Cargo.toml" caption="`cargo new` tarafından oluşturulan *Cargo.toml* içeriği">

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"

[dependencies]
```

</Listing>

Bu dosya [_TOML_][toml]<!-- ignore --> (_Tom's Obvious, Minimal Language_) formatındadır, bu Cargo'nun yapılandırma formatıdır.

İlk satır, `[package]`, aşağıdaki ifadelerin bir paketi yapılandırdığını gösteren bir bölüm başlığıdır. Bu dosyaya daha fazla bilgi ekledikçe, başka bölümler ekleyeceğiz.

Sonraki üç satır, programınızı derlemek için Cargo'nun ihtiyaç duyduğu yapılandırma bilgisini ayarlar: ad, sürüm ve kullanılacak Rust sürümü. `edition` anahtarından [Ek E][appendix-e]<!-- ignore -->'de konuşacağız.

Son satır, `[dependencies]`, projenizin herhangi bir bağımlılığını listelemek için bir bölümün başlangıcıdır. Rust'ta kod paketlerine _crate'lar_ denir. Bu proje için başka herhangi bir crate'a ihtiyacımız olmayacak, ancak Bölüm 2'deki ilk projede ihtiyacımız olacak, o yüzden o zaman bu bağımlılıklar bölümünü kullanacağız.

Şimdi _src/main.rs_'yi açın ve bakın:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargo sizin için bir "Hello, world!" programı oluşturdu, Kod Listesi 1-1'de yazdığımız gibi! Şu ana kadar, projemiz ve Cargo'nun oluşturduğu proje arasındaki farklar, Cargo'nun kodu _src_ dizinine yerleştirmesi ve üst dizinde bir _Cargo.toml_ yapılandırma dosyamız olmasıdır.

Cargo kaynak dosyalarınızın _src_ dizini içinde yaşamasını bekler. Üst seviye proje dizini sadece README dosyaları, lisans bilgisi, yapılandırma dosyaları ve kodunuzla ilgili olmayan her şey içindir. Cargo kullanmak projelerinizi düzenlemenize yardımcı olur. Her şey için bir yer var ve her şey yerinde.

Cargo kullanmayan bir proje başlattıysanız, "Hello, world!" projesinde yaptığımız gibi, onu Cargo kullanan bir projeye dönüştürebilirsiniz. Proje kodunu _src_ dizinine taşıyın ve uygun bir _Cargo.toml_ dosyası oluşturun. Bu _Cargo.toml_ dosyasını almanın kolay bir yolu `cargo init` çalıştırmaktır, bu sizin için otomatik olarak oluşturacaktır.

### Cargo Projesi Derleme ve Çalıştırma

Şimdi "Hello, world!" programını Cargo ile derlediğimizde ve çalıştırdığımızda neyin farklı olduğuna bakalım! _hello_cargo_ dizininizden, şu komutu girerek projenizi derleyin:

```console
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

Bu komut geçerli dizininize değil _target/debug/hello_cargo_ (Windows üzerinde _target\debug\hello_cargo.exe_) içinde bir çalıştırılabilir dosya oluşturur. Varsayılan derleme bir debug derlemesi olduğu için, Cargo ikili dosyayı _debug_ adlı bir dizine koyar. Çalıştırılabilir dosyayı şu komutla çalıştırabilirsiniz:

```console
$ ./target/debug/hello_cargo # veya Windows üzerinde .\target\debug\hello_cargo.exe
Hello, world!
```

Her şey yolunda giderse, `Hello, world!` terminale yazdırılmalıdır. İlk kez `cargo build` çalıştırmak ayrıca Cargo'nun üst seviyede yeni bir dosya oluşturmasına neden olur: _Cargo.lock_. Bu dosya projenizdeki bağımlılıkların tam sürümlerini takip eder. Bu projenin herhangi bir bağımlılığı yok, bu yüzden dosya biraz seyrek. Bu dosyayı manuel olarak değiştirmenize asla gerek yoktur; Cargo içeriğini sizin için yönetir.

Az önce `cargo build` ile bir proje derledik ve `./target/debug/hello_cargo` ile çalıştırdık, ancak `cargo run` kullanarak kodu derleyip ve ardından ortaya çıkan çalıştırılabilir dosyayı tek bir komutta da çalıştırabiliriz:

```console
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

`cargo run` kullanmak, `cargo build` çalıştırmayı ve sonra ikili dosyanın tüm yolunu kullanmayı hatırlamak zorunda kalmaktan daha uygundur, bu yüzden çoğu geliştirici `cargo run` kullanır.

Bu kez Cargo'nun `hello_cargo` derlediğini gösteren çıktı görmediğimize dikkat edin. Cargo dosyaların değişmediğini fark etti, bu yüzden yeniden derlemedi ama sadece ikili dosyayı çalıştırdı. Kaynak kodunuzu değiştirmiş olsaydınız, Cargo onu çalıştırmadan önce projeyi yeniden derleyecekti ve bu çıktıyı görürdünüz:

```console
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Cargo ayrıca `cargo check` adında bir komut sağlar. Bu komut kodunuzun derlendiğinden emin olmak için hızlıca kontrol eder ancak bir çalıştırılabilir dosya üretmez:

```console
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

Neden çalıştırılabilir bir dosya istemeyebilirsiniz? Genellikle, `cargo check` bir çalıştırılabilir dosya üretme adımını atladığı için `cargo build`'den çok daha hızlıdır. Kod yazarken sürekli çalışmanızı kontrol ediyorsanız, `cargo check` kullanmak projenizin hala derlendiğini bilmenin sürecini hızlandıracaktır! Bu nedenle, birçok Rustacean programlarını derlediğinden emin olmak için programlarını yazarken periyodik olarak `cargo check` çalıştırırlar. Sonra, çalıştırılabilir dosyayı kullanmaya hazır olduklarında `cargo build` çalıştırırlar.

Şu ana kadar Cargo hakkında öğrendiklerimizi özetleyelim:

- `cargo new` kullanarak bir proje oluşturabiliriz.
- `cargo build` kullanarak bir proje derleyebiliriz.
- `cargo run` kullanarak bir projeyi tek adımda derleyip çalıştırabiliriz.
- `cargo check` kullanarak hatalar için kontrol etmek amacıyla bir ikili dosya üretmeden bir proje derleyebiliriz.
- Derleme sonucunu kodumuzla aynı dizinde kaydetmek yerine, Cargo onu _target/debug_ dizinine saklar.

Cargo kullanmanın ek bir avantajı, hangi işletim sisteminde çalıştığınızdan bağımsız olarak komutların aynı olmasıdır. Bu nedenle, şu noktadan itibaren, Linux ve macOS ile Windows arasında özel talimatlar sağlamayacağız.

### Sürüm İçin Derleme

Projeniz sonunda sürüme hazır olduğunda, optimizasyonlarla derlemek için `cargo build --release` kullanabilirsiniz. Bu komut _target/debug_ yerine _target/release_ içinde bir çalıştırılabilir dosya oluşturacaktır. Optimizasyonlar Rust kodunuzun daha hızlı çalışmasını sağlar, ancak bunları açmak programınızı derlemek için geçen süreyi uzatır. Bu nedenle iki farklı profil var: hızlı ve sık sık yeniden derlemek istediğinizde geliştirme için biri, ve sürekli yeniden derlenmeyecek ve mümkün olduğunca hızlı çalışacak bir kullanıcıya vereceğiniz son programı derlemek için diğeri. Kodunuzun çalışma süresini benchmark yapıyorsanız, `cargo build --release` çalıştırdığınızdan ve _target/release_ içindeki çalıştırılabilir dosya ile benchmark yaptığınızdan emin olun.

<!-- Old headings. Do not remove or links may break. -->
<a id="cargo-as-convention"></a>

### Cargo'nun Sözleşmelerinden Yararlanma

Basit projelerle, Cargo sadece `rustc` kullanmak üzerinde çok fazla değer sağlamaz, ancak programlarınız daha karmaşık hale geldikçe değerini kanıtlayacaktır. Bir kez programlar birden fazla dosyaya büyüdüğünde veya bir bağımlılığa ihtiyaç duyduğunda, derlemeyi Cargo koordinasına bırakmak çok daha kolaydır.

`hello_cargo` projesi basit olmasına rağmen, artık Rust kariyerinizin geri kalanında kullanacağınız gerçek araçlamanın çoğunu kullanıyor. Aslında, herhangi bir varolan proje üzerinde çalışmak için, Git kullanarak kodu checkout etmek, o projenin dizinine gitmek ve derlemek için şu komutları kullanabilirsiniz:

```console
$ git clone example.org/someproject
$ cd someproject
$ cargo build
```

Cargo hakkında daha fazla bilgi için [dokümantasyonuna][cargo] bakın.

## Özet

Rust yolculuğunuzda harika bir başlangıç yapıyorsunuz! Bu bölümde şunları öğrendiniz:

- `rustup` kullanarak Rust'ın en son stabil sürümünü kurmak.
- Daha yeni bir Rust sürümüne güncellemek.
- Yerel olarak kurulu dokümantasyonu açmak.
- `rustc` doğrudan kullanarak "Hello, world!" programını yazmak ve çalıştırmak.
- Cargo'nun sözleşmelerini kullanarak yeni bir proje oluşturmak ve çalıştırmak.

Rust kodu okumaya ve yazmaya alışmak için daha önemli bir program oluşturma için harika bir zaman. Bu nedenle, Bölüm 2'de bir tahmin oyunu programı oluşturacağız. Ortak programlama kavramlarının Rust'ta nasıl çalıştığını öğrenerek başlamak isterseniz, Bölüm 3'e bakın ve sonra Bölüm 2'ye geri dönün.

[installation]: ch01-01-installation.html#installation
[toml]: https://toml.io
[appendix-e]: appendix-05-editions.html
[cargo]: https://doc.rust-lang.org/cargo/
