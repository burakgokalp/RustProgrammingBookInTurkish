## Hello, World!

Rust'ı kurduğunuza göre, ilk Rust programınızı yazma zamanı geldi. Yeni bir dil öğrenirken ekrana `Hello, world!` metnini yazdıran küçük bir program yazmak gelenektir, bu yüzden biz de burada aynı şeyi yapacağız!

> Not: Bu kitap komut satırıyla temel bir aşinalığınız olduğunu varsayar. Rust kodlarınızın nerede yaşadığı veya düzenleme ve araç kullanımınız konusunda özel bir talebi yoktur, bu yüzden komut satırı yerine bir IDE kullanmayı tercih ederseniz, favori IDE'nizi kullanmakta özgürsünüz. Birçok IDE şimdi bir dereceye kadar Rust desteğine sahiptir; detaylar için IDE'nin dokümantasyonunu kontrol edin. Rust ekibi `rust-analyzer` aracılığıyla harika IDE desteği sağlamaya odaklanmıştır. Daha fazla detay için [Ek D][devtools]<!-- ignore -->'e bakın.

<!-- Old headings. Do not remove or links may break. -->
<a id="creating-a-project-directory"></a>

### Proje Dizini Kurulumu

Rust kodunuzu saklamak için bir dizin oluşturarak başlayacaksınız. Rust için kodunuzun nerede yaşadığı önemli değildir, ancak bu kitaptaki alıştırmalar ve projeler için ev dizininizde bir _projects_ (projeler) dizini oluşturmanızı ve tüm projelerinizi orada tutmanızı öneriyoruz.

Bir terminal açın ve _projects_ dizini ve _projects_ dizini içinde "Hello, world!" projesi için bir dizin oluşturmak için şu komutları girin.

Linux, macOS ve Windows üzerinde PowerShell için şunu girin:

```console
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

Windows CMD için şunu girin:

```cmd
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

<!-- Old headings. Do not remove or links may break. -->
<a id="writing-and-running-a-rust-program"></a>

### Rust Program Temelleri

Sonra, yeni bir kaynak dosyası oluşturun ve adını _main.rs_ koyun. Rust dosyaları her zaman _.rs_ uzantısıyla biter. Dosya adınızda birden fazla kelime kullanıyorsanız, kural onları ayırmak için alt çizgi kullanmaktır. Örneğin, _helloworld.rs_ yerine _hello_world.rs_ kullanın.

Şimdi az önce oluşturduğunuz _main.rs_ dosyasını açın ve Kod Listesi 1-1'deki kodu girin.

<Listing number="1-1" file-name="main.rs" caption="`Hello, world!` yazdıran bir program">

```rust
fn main() {
    println!("Hello, world!");
}
```

</Listing>

Dosyayı kaydedin ve _~/projects/hello_world_ dizinindeki terminal penceresine geri dönün. Linux veya macOS'ta dosyayı derlemek ve çalıştırmak için şu komutları girin:

```console
$ rustc main.rs
$ ./main
Hello, world!
```

Windows üzerinde `./main` yerine `.\main` komutunu girin:

```powershell
> rustc main.rs
> .\main
Hello, world!
```

İşletim sisteminiz ne olursa olsun, `Hello, world!` dizesi terminale yazdırılmalıdır. Bu çıktıyı görmüyorsanız, yardım almanın yolları için Kurulum bölümünün ["Sorun Giderme"][troubleshooting]<!-- ignore --> kısmına geri başvurun.

Eğer `Hello, world!` yazdırıldıysa, tebrikler! Resmi olarak bir Rust programı yazdınız. Bu sizi bir Rust programcısı yapar—hoş geldiniz!

<!-- Old headings. Do not remove or links may break. -->

<a id="anatomy-of-a-rust-program"></a>

### Bir Rust Programın Anatomisi

Bu "Hello, world!" programını detaylı olarak inceleyelim. İşte bulmacanın ilk parçası:

```rust
fn main() {

}
```

Bu satırlar `main` adında bir fonksiyon tanımlar. `main` fonksiyonu özeldir: Her çalıştırılabilir Rust programında çalışan ilk koddur. Burada, ilk satır parametresi olmayan ve hiçbir şey döndürmeyen `main` adında bir fonksiyon bildirir. Parametreler olsaydı, parantezlerin içine (`()`) giderlerdi.

Fonksiyon gövdesi `{}` ile sarılmıştır. Rust tüm fonksiyon gövdelerinin etrafında küme parantezi gerektirir. Açılış küme parantezini fonksiyon bildirimiyle aynı satıra koymak ve araya bir boşluk eklemek iyi bir stildir.

> Not: Rust projeleri genelinde standart bir stile bağlı kalmak istiyorsanız, kodunuzu belirli bir stilde biçimlendirmek için `rustfmt` adlı otomatik bir biçimlendirme aracını kullanabilirsiniz ( `rustfmt` hakkında daha fazla bilgi için [Ek D][devtools]<!-- ignore -->'de). Rust ekibi bu aracı standart Rust dağıtımına, `rustc` gibi, dahil etti, bu yüzden bilgisayarınızda zaten kurulu olmalıdır!

`main` fonksiyonunun gövdesi şu kodu tutar:

```rust
println!("Hello, world!");
```

Bu satır bu küçük programdaki tüm işi yapar: Ekrana metin yazar. Burada dikkat etmeniz gereken üç önemli detay var.

İlk olarak, `println!` bir Rust macro'su (makrosu) çağırır. Bunun yerine bir fonksiyonu çağırsaydı, `println` (şangsız işaret olmadan) olarak girilirdi. Rust macro'ları Rust sözdizimini genişletmenin bir yoludur ve bunları [Bölüm 20][ch20-macros]<!-- ignore -->'de daha detaylı olarak tartışacağız. Şimdilik sadece şunu bilmeniz gerekiyor: `!` kullanmak normal bir fonksiyon yerine bir macro çağırdığınızı anlamına gelir ve macro'lar her zaman fonksiyonlarla aynı kuralları takip etmez.

İkinci olarak, `"Hello, world!"` dizesini görüyorsunuz. Bu dizeyi bir argüman olarak `println!`'a geçiriyoruz ve dize ekrana yazdırılıyor.

Üçüncü olarak, satırı noktalı virgül (`;`) ile bitiriyoruz, bu ifadenin bittiğini ve bir sonrakinin başlamaya hazır olduğunu gösterir. Rust kodunun çoğu satırı noktalı virgülle biter.

<!-- Old headings. Do not remove or links may break. -->
<a id="compiling-and-running-are-separate-steps"></a>

### Derleme ve Çalıştırma

Az önce yeni oluşturulan bir programı çalıştırdınız, bu yüzden süreçteki her adımı inceleyelim.

Bir Rust programını çalıştırmadan önce, `rustc` komutunu girip kaynak dosyanızın adını geçirerek Rust derleyicisini kullanarak derlemeniz gerekir, şöyle:

```console
$ rustc main.rs
```

C veya C++ geçmişiniz varsa, bu işlemin `gcc` veya `clang`'a benzediğini fark edeceksiniz. Başarıyla derledikten sonra Rust bir ikili (binary) çalıştırılabilir dosya çıkarır.

Linux, macOS ve Windows üzerinde PowerShell'de, shell'inizde `ls` komutunu girerek çalıştırılabilir dosyayı görebilirsiniz:

```console
$ ls
main  main.rs
```

Linux ve macOS'ta iki dosya göreceksiniz. Windows üzerinde PowerShell ile CMD kullanarak göreceğiniz aynı üç dosyayı göreceksiniz. Windows üzerinde CMD kullanıyorsanız, şunu girersiniz:

```cmd
> dir /B %= the /B option says to only show the file names =%
main.exe
main.pdb
main.rs
```

Bu, _.rs_ uzantılı kaynak kodu dosyasını, çalıştırılabilir dosyayı (Windows üzerinde _main.exe_, ancak diğer tüm platformlarda _main_) ve Windows kullanırken, hata ayıklama bilgisi içeren _.pdb_ uzantılı bir dosyayı gösterir. Buradan, şunu girerek _main_ veya _main.exe_ dosyasını çalıştırırsınız:

```console
$ ./main # veya Windows üzerinde .\main
```

Eğer _main.rs_ dosyanız "Hello, world!" programınızsa, bu satır terminale `Hello, world!` yazdırır.

Ruby, Python veya JavaScript gibi dinamik bir dille daha aşina iseniz, bir programı derlemeyi ve çalıştırmayı ayrı adımlar olarak kullanmaya alışkın olmayabilirsiniz. Rust bir _önceden derlenen_ (ahead-of-time compiled) dildir, yani bir programı derleyip çalıştırılabilir dosyasını başkasına verebilirsiniz ve onlar Rust kurulu olmasa bile çalıştırabilirler. Birine _.rb_, _.py_ veya _.js_ dosyası verirseniz, onların Ruby, Python veya JavaScript uygulaması kurulu olması gerekir (sırasıyla). Ancak bu dillerde, programınızı derlemek ve çalıştırmak için sadece bir komuta ihtiyacınız vardır. Dil tasarımında her şey bir ödünleşmedir.

`rustc` ile sadece derlemek basit programlar için iyidir, ancak projeniz büyüdükçe tüm seçenekleri yönetmek ve kodunuzu paylaşmayı kolaylaştırmak isteyeceksiniz. Sonra, gerçek dünya Rust programları yazmanıza yardımcı olacak Cargo aracını tanıtacağız.

[troubleshooting]: ch01-01-installation.html#troubleshooting
[devtools]: appendix-04-useful-development-tools.html
[ch20-macros]: ch20-05-macros.html