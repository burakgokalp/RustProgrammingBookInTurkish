## Paketler ve Kafalar (Packages and Crates)

Modül sisteminin ilk parçaları paketler ve kafaları olacaktır.

Bir _kafa (crate)_, Rust derleyicinin bir anda düşündüğü en az kod miktarıdır. `cargo` yerine `rustc` kullanıp tek bir kaynak kod dosyası geçirseniz (Bölüm 1'de [“Rust Program Temelleri”][basics]<!-- ignore --> bölümünün tümü geriye yaptığımız gibi), derleyici o dosyayı bir kafa olarak düşünür. Kafalar modülleri içerebilir ve modüller kafa ile derlenmiş başka dosyalarda tanımlanabilir, gelecek bölümlerde göreceğimiz gibi.

Bir kafa iki formdan biri olabilir: bir ikili kafa veya bir kütüphane kafa.

_İkili kafalar_ (Binary crates_) yürütülebilir (compile edilebilir) bir programlardır ki çalıştırabilirsiniz, örneğin bir komut satırı programı veya bir sunucu gibi. Her birinin `main` adında bir fonksiyonu olmalıdır ki yürütülebilir ne olduğuna tanımlar. Şu ana kadar oluşturduğumuz tüm kafalar ikili kafaları olmuştur.

_Kütüphane kafaları_ (Library crates_) `main` fonksiyonuna sahip değildir ve bir yürütülebilire derlenmezler. Bunun yerine, birden fazla projeyle paylaşılmak için amaçlanmış işlevselliği tanımlarlar. Örneğin, Bölüm 2'de kullandığımız `rand` kafası rastgele sayılar üreten işlevselliği sağlar. Rastaceanlar çoğu zaman "kafa" dediklerinde, kütüphane kafası demek isterler ve "kafa" terimini genel "kütüphane" programlama kavramıyla değişebilir şekilde kullanırlar.

_Kafa kökü (crate root)_, Rust derleyicisinin başladığı ve kafanızın kök modülünü yaptığı bir kaynak dosyasıdır (modülleri [“Kapsam ve Gizlilik Modüllerle” (Control Scope and Privacy with Modules)][modules]<!-- ignore --> bölümünde derinlikte açıklayacağız).

Bir _paket (package)_, bir dizi işlevselliği sağlayan bir veya daha fazla kafanın bir demetidir. Bir paket bu kafaları nasıl inşa edeceğini açıklayan bir _Cargo.toml_ dosyası içerir. Cargo aslında kodunuzu inşa etmek için kullandığınız komut satırı aracını içeren bir pakettir. Cargo paketi ayrıca ikili kafanın bağımlı olduğu bir kütüphane kafası içerir. Başka projeler Cargo komut satırı aracıyla aynı mantığı kullanmak için Cargo kütüphane kafasına bağımlı olabilirler.

Bir paket istediğiniz kadar çok ikili kafa içerebilir ancak çoğu zaman sadece bir kütüphane kafa. Bir paket kütüphane veya ikili kafa olsun en az bir kafa içermelidir.

Bir paket oluşturduğumuzda ne olduğunu giderek bakalım. İlk, `cargo new my-project` komutunu gireriz:

```console
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```

`cargo new my-project` çalıştırdıktan sonra, Cargo'nın ne oluşturduğunu görmek için `ls` kullanırız. _my-project_ klasöründe bize bir paket veren bir _Cargo.toml_ dosyası vardır. Ayrıca _main.rs_ içeren bir _src_ klasörü vardır. Metin düzenleyicinizde _Cargo.toml_'i açın ve _src/main.rs_'den bahsedilmediğini fark edin. Cargo _src/main.rs_'nin paketle aynı adı olan bir ikili kafanın kafa kökü olduğunu bir konvansiyonu takip eder. Aynı şekilde, Cargo paket klasörü _src/lib.rs_ içerirse, paket paketle aynı adı olan bir kütüphane kafa içerir ve _src/lib.rs_ onun kafa köküdür. Cargo kütüphane veya ikili inşa etmek için kafa kök dosyalarını `rustc`'e geçirir.

Burada, sadece _src/main.rs_ içeren bir paketimiz var ki bu sadece `my-project` adında bir ikili kafa içerdiğini anlatır. Bir paket _src/main.rs_ ve _src/lib.rs_ içerirse, iki kafası vardır: bir ikili ve bir kütüphane, ikisi de paketle aynı ada sahip. Bir paket dosyaları _src/bin_ klasörüne koyarak çoklu ikili kafaya sahip olabilir: Her dosya ayrı bir ikili kafa olacaktır.

[basics]: ch01-02-hello-world.html#rust-program-basics
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number