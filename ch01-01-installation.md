# Kurulum

İlk adım Rust'ı kurmaktır. Rust'ı `rustup` aracılığıyla indireceğiz, bu Rust sürümlerini ve ilgili araçları yönetmek için bir komut satırı aracıdır. İndirme için bir internet bağlantınız olmalıdır.

> Not: Bazı nedenlerden dolayı `rustup` kullanmayı tercih etmiyorsanız, daha fazla seçenek için
> [Other Rust Installation Methods page][otherinstall] sayfasına bakın.

Aşağıdaki adımlar Rust derleyicisinin en son stabil sürümünü kurar. Rust'ın kararlılık garantileri, kitaptaki tüm örneklerin yeni Rust sürümleriyle derlemeye devam etmesini sağlar. Rust genellikle hata mesajlarını ve uyarıları iyileştirdiği için sürümler arası çıktı biraz farklılık gösterebilir. Başka bir deyişle, bu adımları kullanarak kurduğunuz herhangi bir daha yeni, stabil Rust sürümü bu kitabın içeriğiyle beklenen şekilde çalışmalıdır.

> ### Komut Satırı Gösterimi
>
> Bu bölümde ve kitap boyunca, terminalde kullanılan bazı komutları göstereceğiz. Bir terminalde girmeniz gereken satırların hepsi `$` ile başlar. `$` karakterini yazmanıza gerek yok; bu her komutun başlangıcını göstermek için görüntülenen komut satırı istemidir. `$` ile başlamayan satırlar genellikle önceki komutun çıktısını gösterir. Ek olarak, PowerShell'e özgü örnekler `$` yerine `>` kullanacaktır.

### Linux veya macOS Üzerine `rustup` Kurma

Linux veya macOS kullanıyorsanız, bir terminal açın ve aşağıdaki komutu girin:

```console
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

Komut bir betik indirir ve `rustup` aracının kurulumunu başlatır, bu da Rust'ın en son stabil sürümünü kurar. Şifreniz istenebilir. Kurulum başarılı olursa, aşağıdaki satır görünecektir:

```text
Rust is installed now. Great!
```

Ayrıca bir _bağlayıcıya_ (linker) ihtiyacınız olacak, bu Rust'ın derlenmiş çıktılarını tek bir dosyada birleştirmek için kullandığı bir programdır. Muhtemelen zaten biriniz var. Bağlayıcı hataları alırsanız, genellikle bir bağlayıcı içeren bir C derleyicisi kurmalısınız. Bir C derleyicisi ayrıca bazı ortak Rust paketleri C koduna bağımlı olduğu ve bir C derleyicisine ihtiyaç duyduğu için de yararlıdır.

macOS üzerinde, bir C derleyicisini şunu çalıştırarak elde edebilirsiniz:

```console
$ xcode-select --install
```

Linux kullanıcıları genellikle dağıtımlarının dokümantasyonuna göre GCC veya Clang kurmalıdır. Örneğin, Ubuntu kullanıyorsanız, `build-essential` paketini kurabilirsiniz.

### Windows Üzerine `rustup` Kurma

Windows üzerinde, [https://www.rust-lang.org/tools/install][install]<!-- ignore
--> adresine gidin ve Rust'ı kurma talimatlarını izleyin. Kurulumun bir noktasında, Visual Studio kurmanız istenecektir. Bu bir bağlayıcı ve programları derlemek için gereken yerel kütüphaneleri sağlar. Bu adımla ilgili daha fazla yardıma ihtiyacınız varsa,
[https://rust-lang.github.io/rustup/installation/windows-msvc.html][msvc]<!--
ignore --> adresine bakın.

Kitabın geri kalanı hem _cmd.exe_ hem de PowerShell'de çalışan komutları kullanır. Belirli farklar varsa, hangisini kullanacağınızı açıklayacağız.

### Sorun Giderme

Rust'ın doğru şekilde kurulup kurulmadığını kontrol etmek için bir shell açın ve şu satırı girin:

```console
$ rustc --version
```

Yayınlanan en son stabil sürümün sürüm numarasını, commit hash'ini ve commit tarihini şu formatta görmelisiniz:

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

Bu bilgileri görüyorsanız, Rust'ı başarıyla kurdunuz! Bu bilgileri görmüyorsanız, Rust'ın `%PATH%` sistem değişkeninde olup olmadığını şu şekilde kontrol edin.

Windows CMD'de şunu kullanın:

```console
> echo %PATH%
```

PowerShell'de şunu kullanın:

```powershell
> echo $env:Path
```

Linux ve macOS'ta şunu kullanın:

```console
$ echo $PATH
```

Bunların hepsi doğru ve Rust hala çalışmıyorsa, yardım alabileceğiniz birçok yer var. [topluluk sayfasında][community] diğer Rustaceans'larla (kendi kendimize verdiğimiz komik bir takma ad) nasıl iletişim kuracağınızı öğrenin.

### Güncelleme ve Kaldırma

Rust bir kez `rustup` aracılığıyla kurulduktan sonra, yeni yayınlanan bir sürüme güncellemek kolaydır. Shell'den şu güncelleme betiğini çalıştırın:

```console
$ rustup update
```

Rust ve `rustup`'ı kaldırmak için shell'den şu kaldırma betiğini çalıştırın:

```console
$ rustup self uninstall
```

<!-- Old headings. Do not remove or links may break. -->
<a id="local-documentation"></a>

### Yerel Dokümantasyonu Okuma

Rust kurulumu ayrıca çevrimdışı okuyabilmeniz için dokümantasyonun yerel bir kopyasını içerir. Yerel dokümantasyonu tarayıcınızda açmak için `rustup doc` komutunu çalıştırın.

Standart kütüphane tarafından sağlanan bir tip veya fonksiyon olduğunda ne yaptığından veya nasıl kullanılacağından emin değilseniz, öğrenmek için uygulama programlama arayüzü (API) dokümantasyonunu kullanın!

<!-- Old headings. Do not remove or links may break. -->
<a id="text-editors-and-integrated-development-environments"></a>

### Metin Editörleri ve IDE'leri Kullanma

Bu kitap Rust kodu yazmak için hangi araçları kullandığınız hakkında hiçbir varsayım yapmaz. Neredeyse herhangi bir metin editörü işi görebilir! Ancak birçok metin editörü ve tümleşik geliştirme ortamı (IDE) Rust için yerleşik desteğe sahiptir. Rust web sitesindeki [araçlar sayfasında][tools] birçok editör ve IDE'ın oldukça güncel bir listesini her zaman bulabilirsiniz.

### Bu Kitapla Çevrimdışı Çalışma

Birkaç örnekte, standart kütüphanenin ötesinde Rust paketleri kullanacağız. Bu örnekler üzerinde çalışmak için, bir internet bağlantınız olması veya bu bağımlılıkları önceden indirmiş olmanız gerekecektir. Bağımlılıkları önceden indirmek için aşağıdaki komutları çalıştırabilirsiniz. (`cargo`'nun ne olduğunu ve bu komutların her birinin detaylı olarak ne yaptığını daha sonra açıklayacağız.)

```console
$ cargo new get-dependencies
$ cd get-dependencies
$ cargo add rand@0.8.5 trpl@0.2.0
```

Bu bu paketlerin indirmelerini önbelleğe alacaktır, böylece bunları daha sonra indirmeniz gerekmeyecektir. Bu komutu çalıştırdıktan sonra, `get-dependencies` klasörünü tutmanıza gerek yoktur. Bu komutu çalıştırdıysanız, kitabın geri kalanındaki tüm `cargo` komutlarıyla bu önbelleğe alınan sürümleri kullanmak yerine ağı kullanmayı denemek yerine `--offline` bayrağını kullanabilirsiniz.

[otherinstall]: https://forge.rust-lang.org/infra/other-installation-methods.html
[install]: https://www.rust-lang.org/tools/install
[msvc]: https://rust-lang.github.io/rustup/installation/windows-msvc.html
[community]: https://www.rust-lang.org/community
[tools]: https://www.rust-lang.org/tools