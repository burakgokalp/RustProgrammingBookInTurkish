## Cargo Çalışma Alanları (Cargo Workspaces)

Bölüm 12'de, bir ikili crate ve bir kütüphane crate içeren bir paket oluşturduk. Projeniz geliştikçe, kütüphane crate'inin büyümeye devam ettiğini ve paketinizi birden çok kütüphane crate'e daha fazla bölmek istediğinizi görebilirsiniz. Cargo, aynı anda geliştirilen birden çok ilgili paketi yönetmeye yardımcı olabilecek _çalışma alanları_ (workspaces) adında bir özellik sunar.

### Bir Çalışma Alanı Oluşturma (Creating a Workspace)

Bir _çalışma alanı_ (workspace), aynı _Cargo.lock_ ve çıkış dizinini paylaşan paketler kümesidir. Bir çalışma alanı kullanarak bir proje yapalım—çalışma alanının yapısına odaklanabilmemiz için önemsiz kod kullanacağız. Bir çalışma alanını yapılandırmanın birden çok yolu vardır, bu yüzden sadece bir yaygın yolu göstereceğiz. İkili ve iki kütüphane içeren bir çalışma alanımız olacak. Ana işlevselliği sağlayacak olan ikili, iki kütüphaneye bağımlı olacak. Bir kütüphane `add_one` fonksiyonu, diğer kütüphane `add_two` fonksiyonu sunacak. Bu üç crate aynı çalışma alanının parçası olacak. Çalışma alanı için yeni bir dizin oluşturarak başlayacağız:

```console
$ mkdir add
$ cd add
```

Sonra, _add_ dizininde, tüm çalışma alanını yapılandıracak _Cargo.toml_ dosyasını oluşturacağız. Bu dosyanın bir `[package]` bölümü olmayacak. Bunun yerine, çalışma alanına üyeler eklememizi sağlayacak bir `[workspace]` bölümü ile başlayacak. Ayrıca çalışma alanımızda Cargo'nun çözümleme algoritmasının en son ve en harika sürümünü kullanmak için `resolver` değerini `"3"` olarak ayarladık:

<span class="filename">Dosya Adı: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-01-workspace/add/Cargo.toml}}
```

Sonra, _add_ dizininde `cargo new` çalıştırarak `adder` ikili crate'ini oluşturacağız:

```console
$ cargo new adder
     Created binary (application) `adder` package
      Adding `adder` as member of workspace at `file:///projects/add`
```

Bir çalışma alanı içinde `cargo new` çalıştırmak, çalışma alanı _Cargo.toml_'indeki `[workspace]` tanımındaki `members` anahtarına yeni oluşturulan paketi de otomatik olarak ekler, şu gibi:

```toml
{{#include ../listings/ch14-more-about-cargo/output-only-01-adder-crate/add/Cargo.toml}}
```

Bu noktada, `cargo build` çalıştırarak çalışma alanını derleyebiliriz. _add_ dizinindeki dosyalarınız şu gibi görünmelidir:

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

Çalışma alanının derlenmiş yapıtların yerleştirileceği üst seviyede bir _target_ dizini vardır; `adder` paketinin kendi _target_ dizini yoktur. _adder_ dizinin içinden bile `cargo build` çalıştırsak, derlenmiş yapıtlar yine de _add/adder/target_ yerine _add/target_'te sona ererdi. Cargo bir çalışma alanında _target_ dizinini şu şekilde yapılandırır çünkü çalışma alanındaki crate'ler birbirine bağımlı olmaya yöneliktir. Eğer her crate'in kendi _target_ dizini olsaydı, her crate kendi _target_ dizinine yapıtları yerleştirmek için çalışma alanındaki diğer crate'lerin her birini yeniden derlemek zorunda kalacaktı. Bir _target_ dizinini paylaşarak, crate'ler gereksiz yeniden derlenmeyi önleyebilirler.

### Çalışma Alanında İkinci Bir Paket Oluşturma (Creating the Second Package in the Workspace)

Sonra, çalışma alanında başka bir üye paket oluşturalım ve onu `add_one` diyelim. `add_one` adında yeni bir kütüphane crate oluşturun:

```console
$ cargo new add_one --lib
     Created library `add_one` package
      Adding `add_one` as member of workspace at `file:///projects/add`
```

Üst seviyeli _Cargo.toml_ artık `members` listesinde _add_one_ yolunu içerecektir:

<span class="filename">Dosya Adı: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/Cargo.toml}}
```

_add_ dizininiz şimdi bu dizinlere ve dosyalara sahip olmalıdır:

```text
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

_add_one/src/lib.rs_ dosyasında, bir `add_one` fonksiyonu ekleyelim:

<span class="filename">Dosya Adı: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/add_one/src/lib.rs}}
```

Şimdi ikili ile `adder` paketini, kütüphanemiz olan `add_one` paketine bağımlı hale getirebiliriz. Önce, _adder/Cargo.toml_'a `add_one`'a bir yol bağımlılığı eklememiz gerekecek.

<span class="filename">Dosya Adı: adder/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/adder/Cargo.toml:6:7}}
```

Cargo, çalışma alanındaki crate'lerin birbirine bağımlı olacağını varsaymaz, bu yüzden bağımlılık ilişkilerinde net olmalıyız.

Sonra, `adder` crate'indeki `add_one` fonksiyonunu (`add_one` crate'inden) kullanalım. _adder/src/main.rs_ dosyasını açın ve `main` fonksiyonunu `add_one` fonksiyonunu çağıracak şekilde değiştirin, Kod Listesi 14-7'deki gibi.

<Listing number="14-7" file-name="adder/src/main.rs" caption="`adder` crate'inden `add_one` kütüphane crate'ini kullanma">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-07/add/adder/src/main.rs}}
```

</Listing>

Üst seviyeli _add_ dizininde `cargo build` çalıştırarak çalışma alanını derleyelim!

```console
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.22s
```

_add_ dizininden ikili crate'i çalıştırmak için, `cargo run` ile paket adı kullanarak çalışma alanında hangi paketi çalıştırmak istediğimizi `-p` argümanı ile belirtebiliriz:

```console
$ cargo run -p adder
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

Bu, _adder/src/main.rs_ içindeki kodu çalıştırır ki bu `add_one` crate'ine bağımlıdır.

<a id="depending-on-an-external-package-in-a-workspace"></a>

### Bir Harici Pakete Bağımlılık (Depending on an External Package)

Çalışma alanının her crate'in dizininde bir _Cargo.lock_ yerine üst seviyede sadece bir _Cargo.lock_ dosyası olduğuna dikkat edin. Bu, tüm crate'lerin tüm bağımlılıkların aynı sürümünü kullandığını sağlar. Eğer _adder/Cargo.toml_ ve _add_one/Cargo.toml_ dosyalarına `rand` paketi eklersek, Cargo her ikisini de `rand`'in bir sürümüne çözecek ve bunu bir _Cargo.lock_'a kaydedecek. Çalışma alanındaki tüm crate'lerin aynı bağımlılıkları kullanmasını, crate'lerin her zaman birbirleriyle uyumlu olacağı anlamına gelir. Şimdi _add_one/Cargo.toml_ dosyasındaki `[dependencies]` bölümüne `rand` crate'ini ekleyelim ki `add_one` crate'inde `rand` crate'ini kullanabilelim:

<span class="filename">Dosya Adı: add_one/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add/add_one/Cargo.toml:6:7}}
```

Şimdi _add_one/src/lib.rs_ dosyasına `use rand;` ekleyebiliriz ve _add_ dizininde `cargo build` çalıştırarak tüm çalışma alanını derlemek, `rand` crate'ini getirecek ve derleyecek. Kapsama getirdiğimiz `rand`'a atıfta bulunmadığımız için bir uyarı alacağız:

```console
$ cargo build
    Updating crates.io index
  Downloaded rand v0.8.5
   --snip--
   Compiling rand v0.8.5
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
warning: unused import: `rand`
 --> add_one/src/lib.rs:1:5
  |
1 | use rand;
  |     ^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

warning: `add_one` (lib) generated 1 warning (run `cargo fix --lib -p add_one` to apply 1 suggestion)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.95s
```

Üst seviyeli _Cargo.lock_ artık `add_one`'un `rand`'e bağımlılığı hakkında bilgi içerir. Ancak, `rand` çalışma alanının bir yerinde kullanılsa bile, çalışma alanındaki diğer crate'lerde onu kullanamayız eğer onların _Cargo.toml_ dosyalarına da `rand` eklemesek. Örneğin, eğer _adder/src/main.rs_ dosyasına `use rand;` eklersek, bir hata alacağız:

```console
$ cargo build
  --snip--
   Compiling adder v0.1.0 (file:///projects/add/adder)
error[E0432]: unresolved import `rand`
 --> adder/src/main.rs:2:5
  |
2 | use rand;
  |     ^^^^ no external crate `rand`
```

Bunu düzeltmek için, `adder` paketi için _Cargo.toml_ dosyasını düzenleyin ve `rand`'in de bir bağımlılık olduğunu belirtin. `adder` paketini derlemek, _Cargo.lock_'daki `adder` için bağımlılık listesine `rand` ekleyecek ama `rand`'in ek kopyaları indirilmeyecek. Cargo, çalışma alanındaki `rand` paketini kullanan her paketteki her crate'in, uyumlu sürümlerini belirttikleri sürece, aynı sürümü kullanmasını sağlayacak ki bu bizi yerden tasarruf ettirir ve çalışma alanındaki crate'lerin birbirleriyle uyumlu olmasını sağlar.

Eğer çalışma alanındaki crate'ler aynı bağımlılığın uyumsuz sürümlerini belirtirse, Cargo her birini çözecek ama yine de mümkün olduğunca az sürüm çözmeye çalışacak.

### Bir Çalışma Alanına Test Ekleme (Adding a Test to a Workspace)

Başka bir iyileştirme için, `add_one` crate içinde `add_one::add_one` fonksiyonu için bir test ekleyelim:

<span class="filename">Dosya Adı: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add/add_one/src/lib.rs}}
```

Şimdi üst seviyeli _add_ dizininde `cargo test` çalıştırın. Bu gibi yapılandırılmış bir çalışma alanında `cargo test` çalıştırmak, çalışma alanındaki tüm crate'lerin testlerini çalıştıracaktır:

```console
$ cargo test
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.20s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/adder-3a47283c568d2b6a)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Çıktının ilk bölümü, `add_one` crate'indeki `it_works` testinin geçtiğini gösterir. Sonraki bölüm, `adder` crate'inde sıfır test bulunduğunu ve sonra son bölüm, `add_one` crate'inde sıfır belgeleme testinin bulunduğunu gösterir.

Üst seviyeli dizinden çalışma alanındaki belirli bir crate için `-p` bayrağı ve test etmek istediğimiz crate'in adını belirterek de test çalıştırabiliriz:

```console
$ cargo test -p add_one
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Bu çıktı, `cargo test`'in sadece `add_one` crate'inin testlerini çalıştırdığını ve `adder` crate testlerini çalıştırmadığını gösterir.

Eğer çalışma alanındaki crate'leri [crates.io](https://crates.io/)<!-- ignore --> üzerinde yayınlarsanız, çalışma alanındaki her crate'in ayrı olarak yayımlanması gerekecektir. `cargo test` gibi, çalışma alanımızda belirli bir crate'i `-p` bayrağı ve yayınlamak istediğimiz crate'in adını belirterek yayınlamak için kullanabiliriz.

Ekstra pratik için, bu çalışma alanına `add_two` crate'ini `add_one` crate'ine benzer bir şekilde ekleyin!

Projeniz büyüdükçe, bir çalışma alanı kullanmayı düşünün: Size tek büyük kod blob'u yerine daha küçük, daha kolay anlaşılan bileşenlerle çalışmanızı sağlar. Ayrıca, bir çalışma alanında crate'leri tutmak, crate'ler genellikle aynı zamanda değişiliyorsa crate'ler arasında koordinasyonu kolaylaştırabilir.