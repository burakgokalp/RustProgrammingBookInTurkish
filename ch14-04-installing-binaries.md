<a id="installing-binaries-from-cratesio-with-cargo-install"></a>

## `cargo install` ile İkili Dosyaları Yükleme (Installing Binaries with `cargo install`)

`cargo install` komutu, ikili crate'leri yerel olarak yüklemenizi ve kullanmanızı sağlar. Bu sistem paketlerinin yerini almaya yönelik değildir; Rust geliştiricilerinin [crates.io](https://crates.io/)<!-- ignore --> üzerinde paylaştığı araçları yüklemek için uygun bir yol olmaya yöneliktir. Sadece ikili hedefleri olan paketleri yükleyebileceğinizi not edin. Bir _ikili hedef_ (binary target), crate'in bir _src/main.rs_ dosyası varsa veya ikili olarak belirtilen başka bir dosya varsa oluşturulan çalıştırılabilir programdır, kendi başına çalıştırılabilir olmayan ancak diğer programlar içinde dahil edilmeye uygun olan bir kütüphane hedefine karşıt. Genellikle, crate'ler README dosyasında crate'in bir kütüphane olup olmadığı, ikili hedefi olup olmadığı veya her ikisi hakkında bilgiye sahiptir.

`cargo install` ile yüklenen tüm ikili dosyalar, kurulum kökünün _bin_ klasöründe saklanır. Eğer Rust'ı _rustup.rs_ kullanarak yüklediyseniz ve herhangi bir özel yapılandırmaya sahip değilseniz, bu dizin *$HOME/.cargo/bin* olacaktır. `cargo install` ile yüklediğiniz programları çalıştırabilmek için bu dizinin `$PATH` içinde olduğundan emin olun.

Örneğin, Bölüm 12'de dosyaları aramak için `grep` aracının `ripgrep` adında bir Rust uygulaması olduğunu belirttik. `ripgrep` yüklemek için, şu komutu çalıştırabiliriz:

```console
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v14.1.1
  Downloaded 1 crate (213.6 KB) in 0.40s
  Installing ripgrep v14.1.1
--snip--
   Compiling grep v0.3.2
    Finished `release` profile [optimized + debuginfo] target(s) in 6.73s
  Installing ~/.cargo/bin/rg
   Installed package `ripgrep v14.1.1` (executable `rg`)
```

Çıktının sondan ikinci satırı, yüklenen ikili dosyanın konumunu ve adını gösterir ki bu `ripgrep` için `rg`'dir. Daha önce belirtildiği gibi, kurulum dizininiz `$PATH` içinde olduğu sürece, `rg --help` çalıştırabilir ve dosyaları aramak için daha hızlı, Rust'lı bir araç kullanmaya başlayabilirsiniz!