## Komut Satırı Argümanlarını Kabul Etme (Accepting Command Line Arguments)

`cargo new` ile, her zamanki gibi, yeni bir proje oluşturalım. Projemizi sisteminizde zaten bulunabilecek `grep` aracından ayırtmak için `minigrep` adlandıralım:

```console
$ cargo new minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

İlk görev, `minigrep`'in iki komut satırı argümanını kabul etmesini sağlamaktır: bir dosya yolu ve aranacak bir dize. Yani, programımızı `cargo run`, iki tire (two hyphens) ile, sonraki argümanların `cargo`'ya değil programımıza olduğunu belirten, aranacak bir dize, ve aranacak bir dosyaya giden bir yol ile, şöyle çalıştırabilmek istiyoruz:

```console
$ cargo run -- searchstring example-filename.txt
```

Şu anda, `cargo new` tarafından oluşturulan program verdiğimiz argümanları işleyemez. [crates.io](https://crates.io/) üzerindeki bazı mevcut kütüphaneler, komut satırı argümanlarını kabul eden bir program yazmada yardımcı olabilir, ancak bu konsepti yeni öğrendiğiniz için, bu yeteneği kendimiz uygulayalım.

### Argüman Değerlerini Okuma (Reading the Argument Values)

`minigrep`'in ona geçtiğimiz komut satırı argümanlarının değerlerini okumasını sağlamak için, Rust'ın standart kütüphanesinde sağlanan `std::env::args` fonksiyonuna ihtiyaç duyacağız. Bu fonksiyon, `minigrep`'e geçilen komut satırı argümanlarının bir iteratörünü (iterator) döndürür. İteratörleri [Bölüm 13][ch13]<!-- ignore -->'te tamamen kaplayacağız. Şimdilik, iteratörler hakkında sadece iki detaya ihtiyaçınız var: İteratörler değerler serisi üretirler ve iteratör üzerinde `collect` yöntemini çağırarak onu, iteratörün ürettiği tüm elemanları içeren bir vektör gibi bir koleksiyona (collection) dönüştürebiliriz.

Kod Listesi 12-1'deki kod `minigrep` programınıza ona geçilen herhangi bir komut satırı argümanını okumasını ve sonra değerleri bir vektörde toplamasını sağlar.

<Listing number="12-1" file-name="src/main.rs" caption="Komut satırı argümanlarını bir vektörde toplama ve onları yazdırma">

```rust
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-01/src/main.rs}}
```

</Listing>

Önce, `std::env` modülünü `use` bildirimi ile kapsama getiriyoruz ki onun `args` fonksiyonunu kullanabilelim. `std::env::args` fonksiyonunun iki seviye modülde iç içe olduğunu unutmayın. [Bölüm 7][ch7-idiomatic-use]<!-- ignore -->'de tartıştığımız gibi, arzulanan fonksiyon birden fazla modülde iç içe olduğu durumlarda, fonksiyon yerine üst modülü kapsama getirmeyi seçtik. Bunu yaparak, `std::env`'den başka fonksiyonları kolayca kullanabiliriz. Ayrıca `use std::env::args` eklemek ve sadece `args` ile fonksiyon çağırmaktan daha az belirsizdir, çünkü `args` geçerli modülde tanımlanmış bir fonksiyonla karıştırılabilir.

> ### `args` Fonksiyonu ve Geçersiz Unicode (The `args` Function and Invalid Unicode)
>
> Not: `std::env::args`, herhangi bir argüman geçersiz Unicode içerirse panik atacaktır. Programınız geçersiz Unicode içeren argümanları kabul etmesi gerekiyorsa, bunun yerine `std::env::args_os` kullanın. Bu fonksiyon, `String` değerleri yerine `OsString` değerleri üreten bir iteratör döndürür. Basitlik için burada `std::env::args` kullanmayı seçtik, çünkü `OsString` değerleri platforma göre değişir ve `String` değerlerinden çalışmak daha karmaşıktır.

`main`'in ilk satırında, `env::args` çağırıyoruz ve hemen iteratörü iteratörün ürettiği tüm değerleri içeren bir vektöre dönüştürmek için `collect` kullanıyoruz. Birçok türde koleksiyon oluşturmak için `collect` fonksiyonunu kullanabiliriz, bu yüzden `args`'ın tipini açıkça notalıyoruz ki bir dize vektörü istediğimizi belirtelim. Rust'ta tipleri çok nadir notalamanıza rağmen, `collect` genellikle notlamanız gereken bir fonksiyondur, çünkü Rust istediğiniz koleksiyon türünü çıkarılamaz.

Son olarak, debug makrosunu kullanarak vektörü yazdırıyoruz. Önce hiç argüman olmadan ve sonra iki argümanla kodu çalıştırmayı deneyelim:

```console
{{#include ../listings/ch12-an-io-project/listing-12-01/output.txt}}
```

```console
{{#include ../listings/ch12-an-io-project/output-only-01-with-args/output.txt}}
```

Vektördeki ilk değerin `"target/debug/minigrep"` olduğuna dikkat edin ki bu ikilimizin adıdır. Bu, C'de argümanlar listesinin davranışıyla eşleşir ki programların çalıştırılmalarıyla adları kullanmalarına izin verir. Program adına erişmek genellikle kullanışlıdır, örneğin onu mesajlarda yazdırmak istersiniz veya programın davranışını hangi komut satırı takma adının kullanılarak çağırıldığına göre değiştirmek istersiniz. Ancak bu bölümün amaçları için, onu yoksayacağız ve sadece ihtiyaç duyduğumuz iki argümanı kaydedeceğiz.

### Argüman Değerlerini Değişkenlerde Kaydetme (Saving the Argument Values in Variables)

Program şu anda komut satırı argümanları olarak belirtilen değerlere erişebiliyor. Şimdi iki argümanın değerlerini değişkenlerde kaydetmemiz gerekiyor ki programın geri kalanında değerleri kullanabilelim. Bunu Kod Listesi 12-2'de yapıyoruz.

<Listing number="12-2" file-name="src/main.rs" caption="Sorgu argümanı ve dosya yolu argümanını tutan değişkenler oluşturma">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-02/src/main.rs}}
```

</Listing>

Vektörü yazdırdığımızda gördüğümüz gibi, programın adı vektördeki ilk değeri `args[0]`'de alır, bu yüzden argümanları indeks 1'den başlıyoruz. `minigrep`'in aldığı ilk argüman aradığımız dizedir, bu yüzden ilk argümana bir referansı `query` değişkenine koyuyoruz. İkinci argüman dosya yolu olacaktır, bu yüzden ikinci argümana bir referansı `file_path` değişkenine koyuyoruz.

Bu değişkenlerin değerlerini geçici olarak kodun beklendiğimiz gibi çalıştığını kanıtlamak için yazdırıyoruz. Bu programı tekrar `test` ve `sample.txt` argümanlarıyla çalıştıralım:

```console
{{#include ../listings/ch12-an-io-project/listing-12-02/output.txt}}
```

Harika, program çalışıyor! İhtiyaç duyduğumuz argümanların değerleri doğru değişkenlere kaydediliyor. Sonra, kullanıcının hiç argüman vermediği gibi belirli potansiyel hatlı durumlarla başa çıkmak için bazı hata eleme ekleyeceğiz; şimdilik, bu durumu yoksayacağız ve bunun yerine dosya okuma yeteneklerini ekleme üzerinde çalışacağız.

[ch13]: ch13-00-functional-features.html
[ch7-idiomatic-use]: ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#creating-idiomatic-use-paths