<!-- Old headings. Do not remove or links may break. -->

<a id="writing-error-messages-to-standard-error-instead-of-standard-output"></a>

## Hataları Standart Hataya Yönlendirme (Redirecting Errors to Standard Error)

Şu anda, tüm çıktımızı `println!` makrosunu kullanarak terminale yazıyoruz. Çoğu terminalde, iki türde çıktı vardır: genel bilgiler için standart çıktı (`stdout`) ve hata mesajları için standart hata (`stderr`). Bu ayrım, kullanıcılara bir programın başarılı çıktısını bir dosyaya yönlendirmeyi seçmeyi ancak hala hata mesajlarını ekrana yazdırmayı izin verir.

`println!` makrosu sadece standart çıktı yazdırmaya yetenekli, bu yüzden standart hataya yazdırmak için başka bir şey kullanmak zorundayız.

### Hataların Nereye Yazıldığını Kontrol Etme (Checking Where Errors Are Written)

Önce, `minigrep` tarafından yazılan içeriğin şu anda standart çıktıya yazıldığını gözlemeliyoruz ki standart hataya yazmak istediğimiz herhangi bir hata mesajını da içeriyor. Bunu, standart çıktı akımını bir dosyaya yönlendirirken bilerek bir hataya neden olarak yapacağız. Standart hata akımını yönlendirmeyeceğiz, böylece standart hataya gönderilen herhangi bir içerik hala ekranda görünecektir.

Komut satırı programlarının standart hata akımına hata mesajları göndermesi beklenir ki böylece standart çıktı akımını bir dosyaya yönlendirseler bile, hala ekranda hata mesajlarını görebiliriz. Programımız şu anda iyi davranışlı değil: Hata mesajı çıktısının bir dosyaya kaydedildiğini göreceksiniz!

Bu davranışı göstermek için, programı `>` ve bir dosya yolu, _output.txt_, ile çalıştıracağız ki standart çıktı akımını bununla yönlendirmek istiyoruz. Herhangi bir argüman geçmeyeceğiz ki bu bir hataya neden olmalı:

```console
$ cargo run > output.txt
```

`>` sözdizimi shell'e standart çıktının içeriklerini ekran yerine _output.txt_ dosyasına yazmasını söyler. Ekrana yazılmasını beklediğimiz hata mesajını görmedik, bu da bunun dosyada sonlanmış olacağı anlamına gelir. _output.txt_ dosyasının içerdiği budur:

```text
Problem parsing arguments: not enough arguments
```

Evet, hata mesajımız standart çıktıya yazılıyor. Bu türde hata mesajlarının standart hataya yazılması çok daha faydalı çünkü sadece başarılı bir koşturmadan gelen veri dosyada sonlansın. Bunu değiştireceğiz.

### Hataları Standart Hataya Yazdırma (Printing Errors to Standard Error)

Kod Listesi 12-24'teki kodu kullanarak hata mesajlarının nasıl yazıldığını değiştireceğiz. Bu bölümde daha önce yaptığımız yerefactor etme nedeniyle, hata mesajları yazan tüm kod tek bir fonksiyonda, `main`'de. Standart kütüphanesi standart hata akımına yazdıran `eprintln!` makrosunu sağlar, bu yüzden hataları yazdırmak için `println!` çağırdığımız iki yeri de `eprintln!` kullanmak üzere değiştirelim.

<Listing number="12-24" file-name="src/main.rs" caption="`eprintln!` kullanarak hata mesajlarını standart çıktı yerine standart hataya yazma">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-24/src/main.rs:here}}
```

</Listing>

Şimdi programı aynı şekilde, herhangi bir argüman olmadan ve standart çıktıyı `>` ile yönlendirerek tekrar çalıştıralım:

```console
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

Şimdi hatayı ekranda görüyoruz ve _output.txt_ hiçbir şey içermiyor ki bu komut satırı programlarından beklediğimiz davranış. 

Şimdi programı, hataya neden olmayan ancak hala standart çıktıyı bir dosyaya yönlendiren argümanlarla tekrar çalıştıralım, şu şekilde:

```console
$ cargo run -- to poem.txt > output.txt
```

Terminale herhangi bir çıktı görmeyeceğiz ve _output.txt_ sonuçlarımızı içerecek:

<span class="filename">Dosya Adı: output.txt</span>

```text
Are you nobody, too?
How dreary to be somebody!
```

Bu, artık başarılı çıktı için standart çıktı ve hata çıktısı için standart hatayı uygun bir şekilde kullandığımızı gösteriyor.

## Özet (Summary)

Bu bölüm şimdiye kadar öğrendiğiniz bazı önemli kavramları özetledi ve Rust'te ortak I/O işlemlerinin nasıl gerçekleştirildiğini kapladı. Komut satırı argümanları, dosyalar, ortam değişkenleri ve hataları yazdırmak için `eprintln!` makrosunu kullanarak, artık komut satırı uygulamalarını yazmaya hazırsınız. Önceki bölümlerdeki kavramlarla birleştiğinde, kodunuz iyi düzenlenmiş, veriyi uygun veri yapılarında etkili bir şekilde saklayacak, hataları güzel bir şekilde ele alacak ve iyi test edilecek.

Sonra, fonksiyonel diller tarafından etkilenmiş bazı Rust özelliklerini keşfedeceğiz: kapanışlar ve iteratörler.