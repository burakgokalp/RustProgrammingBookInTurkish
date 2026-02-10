## Yapıları Kullanan Örnek Bir Program

Yapıları ne zaman kullanmak isteyebileceğimizi anlamak için, bir dikdörtgenin alanını hesaplayan bir program yazalım. Ayrı değişkenler kullanarak başlayalım ve sonra yapıları kullanana kadar programı yeniden düzenleyelim (refactor).

Cargo ile _rectangles_ adında yeni bir ikili proje oluşturalım ki piksellerde belirtilen dikdörtgenin genişliğini ve yüksekliğini alacak ve dikdörtgenin alanını hesaplayacak. Kod Listesi 5-8 projemizin _src/main.rs_'ında bunu yapmanın bir yolunu gösteren kısa bir program gösterir.

<Listing number="5-8" file-name="src/main.rs" caption="Ayrı genişlik ve yükseklik değişkenleri ile belirtilen dikdörtgenin alanını hesaplama">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:all}}
```

</Listing>

Şimdi, `cargo run` kullanarak bu programı çalıştıralım:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/output.txt}}
```

Bu kod, her boyut ile `area` fonksiyonunu çağırarak dikdörtgenin alanını hesaplamada başarır ancak bu kodu daha anlaşılır ve okunabilir yapmak için daha fazlasını yapabiliriz.

Bu koddaki sorun `area`'nin imzasında açıktır:

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:here}}
```

`area` fonksiyonunun tek bir dikdörtgenin alanını hesaplaması gerekiyor ancak yazdığımız fonksiyonun iki parametresi var ve programımızın hiçbir yerinde parametrelerin ilişkili olduğu açıkl değil. Genişlik ve yüksekliği gruplamak daha anlaşılır ve yönetilebilir olurdu. Bölüm 3'ün ["Tüple Tipi" (The Tuple Type)][the-tuple-type]<!-- ignore --> bölümünde yapabileceğimiz bir yolu zaten konuştuk: tüleler kullanarak.

### Tülelerle Yeniden Düzenleme (Refactoring with Tuples)

Kod Listesi 5-9 tüleler kullanan programımızın başka bir sürümünü gösterir.

<Listing number="5-9" file-name="src/main.rs" caption="Bir tüle ile dikdörtgenin genişliğini ve yüksekliğini belirleme">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-09/src/main.rs}}
```

</Listing>

Bir şekilde, bu program daha iyidir. Tüleler bize biraz yapı eklemeyi izin verir ve artık sadece bir argüman geçiyoruz. Ancak başka bir şekilde, bu sürüm daha az açıktır: Tüleler öğelerini adlandırmazlar, bu yüzden tüle'nin kısımlarına indekslememiz gerekiyor ki hesaplamamızı daha az belirgin yapıyor.

Alan hesaplaması için genişlik ve yükseklik karıştırılması önemli olmaz ancak eğer dikdörtgeni ekrana çizmek istersek, önemli olur! `width`'in tüle indeksi `0` olduğunu ve `height`'in tüle indeksi `1` olduğunu aklımızda tutmamız gerekecekti. Bu kodumuzu kullanan başkası için anlaması ve aklında tutması daha zor olurdu. Çünkü kodumuzda verimizin anlamını iletmedik, artık hataları tanıtmak daha kolay.

<!-- Old headings. Do not remove or links may break. -->

<a id="refactoring-with-structs-adding-more-meaning"></a>

### Yapılarla Yeniden Düzenleme (Refactoring with Structs)

Veriyi etiketleyerek yapılarla anlam ekliyoruz. Tüle kullandığımız şeyi, tümü için bir isim ve kısımlar için isimlerle birlikte bir yapıya dönüştürebiliriz, Kod Listesi 5-10'da gösterildiği gibi.

<Listing number="5-10" file-name="src/main.rs" caption="Bir `Rectangle` yapısını tanımlama">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-10/src/main.rs}}
```

</Listing>

Burada, bir yapı tanımladık ve onu `Rectangle` adlandırdık. Kıvramalı parantezlerin içinde, `width` ve `height` olarak alanları tanımladık, ikisinin `u32` tipi var. Sonra, `main`'de, genişliği `30` ve yüksekliği `50` olan `Rectangle`'in belirli bir örneğini oluşturduk.

`area` fonksiyonumuz artık tek bir parametreyle tanımlandı ki bunu `rectangle` adlandırdık ve tipi bir `Rectangle` yapı örneğinin değişmez (immutable) başvurusudur. Bölüm 4'te bahsettiğimiz gibi, yapıdan sahiplik almak yerine ödünçlemek istiyoruz. Bu şekilde, `main` sahipliğini korur ve `rect1`'i kullanmaya devam edebilir ki bu fonksiyon imzasında `&` kullandığımız ve fonksiyonu nerede çağırdığımızda olan sebeptir.

`area` fonksiyonu `Rectangle` örneğinin `width` ve `height` alanlarına erişir (not: ödünçlenmiş bir yapı örneğinin alanlarına erişmek alan değerlerini taşımaz ki bu yüzden yapıların ödünçlerini sık sık görebilirsiniz). `area`'nın fonksiyon imzası artık tam olarak ne istediğimizi söylüyor: `Rectangle`'in alanını hesapla, `width` ve `height` alanlarını kullanarak. Bu, genişlik ve yüksekliğin birbirleriyle ilgili olduğunu ve değerlere tüle indeks değerleri olan `0` ve `1` yerine açıklayıcı isimler verdiğini iletir. Bu netlik için bir kazanımdır.

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-useful-functionality-with-derived-traits"></a>

### Türetilmiş Özelliklerle İşlevsellik Ekleme (Adding Functionality with Derived Traits)

Programımızda hata ayıklarken `Rectangle` örneğini yazabilmek kullanışlı olurdu ve tüm alanlarının değerlerini görebilirdik. Kod Listesi 5-11 önceki bölümlerde kullandığımız gibi [`println!` makrosunu][println]<!-- ignore --> kullanmayı deniyor. Bu, ancak çalışmayacak.

<Listing number="5-11" file-name="src/main.rs" caption="Bir `Rectangle` örneğini yazmaya çalışma">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/src/main.rs}}
```

</Listing>

Bu kodu derlediğimizde, şu temel mesajla bir hata alacağız:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:3}}
```

`println!` makrosu çok çeşitli biçimlendirmeler yapabilir ve varsayılan olarak, kıvramalı parantezler `println!`'ya `Display` olarak bilinen biçimlendirmeyi kullanmasını söyler: doğrudan son kullanıcı kullanımı için amaçlanan çıktı. Şu ana kadar gördüğümüz ilkel tipler varsayılan olarak `Display`'i uygular çünkü son kullanıcıya bir `1` veya başka bir ilkel tipi göstermenin sadece tek bir yolu vardır. Ancak yapılarla, `println!`'nın çıktıyı nasıl biçimlendirmesi daha az açıktır çünkü daha fazla görüntüleme olanakları vardır: Virgül istiyor musunuz veya istemiyor musunuz? Kıvramalı parantezler yazdırmak istiyor musunuz? Tüm alanlar gösterilmeli mi? Bu belirsizlik nedeniyle, Rust ne istediğimizi tahmin etmeye çalışmaz ve yapılar `println!` ile kullanmak için sağlanan bir `Display` uygulamasına sahip değildir ve `{}` yer tutucu ile kullanılmaz.

Hataları okumaya devam edersek, şu yararlı notu bulacağız:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:9:10}}
```

Deneyelim! `println!` makro çağrısı artık `println!("rect1 is {rect1:?}");` gibi görünecek. Kıvramalı parantezlerin içine `:?` belirleyiciyi koymak `println!`'ya `Debug` adında bir çıktı biçimi kullanmak istediğimizi söyler. `Debug` özelliği bize yapıyı geliştiriciler için kullanışlı olan bir yolda yazmamızı izin verir böylece kodumuzu hata ayıklarken değerini görebiliriz.

Bu değişiklikle kodu derleyelim. Çalıştırın! Yine bir hata alıyoruz:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:3}}
```

Ancak yine, derleyici bize yararlı bir not verir:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:9:10}}
```

Rust hata ayıklama bilgisini yazdırmak için işlevsellik _içerir_ ancak bu işlevselliği yapımıza açıkça seçmek zorundayız. Bunu yapmak için, Kod Listesi 5-12'de gösterildiği gibi yapı tanımından hemen önce `#[derive(Debug)]` dış niteliğini ekliyoruz.

<Listing number="5-12" file-name="src/main.rs" caption="Hata ayıklama biçimlendirmesini türetmek için `#[derive(Debug)]` niteliğini ekleme ve `Debug` biçimlendirmesini kullanarak bir `Rectangle` örneğini yazdırma">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/src/main.rs}}
```

</Listing>

Şimdi programı çalıştırdığımızda, hiçbir hata almayacağız ve şu çıktıyı göreceğiz:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/output.txt}}
```

Güzel! En güzel çıktı değil ancak bu örneğin tüm alanlarının değerlerini gösterir ki bu kesinlikle hata ayıklama sırasında yardımcı olur. Daha büyük yapılar olduğunda, biraz daha okunabilir çıktıya sahip olmak kullanışlıdır; bu durumlarda, `println!` dizesinde `{:?}` yerine `{:#?}` kullanabiliriz. Bu örnekte, `{:#?}` stilini kullanmak şu çıktıyı üretecek:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-02-pretty-debug/output.txt}}
```

`Debug` biçimini kullanarak bir değeri yazdırmanın başka bir yolu [`dbg!` makrosunu][dbg]<!-- ignore --> kullanmaktır ki ifadenin sahipliğini alır (kavş olarak `println!` bir başvuru alır), kodunuzdaki `dbg!` makro çağrısının dosya ve satır numarası ile birlikte ifadenin sonuç değerini yazdırır ve değerin sahipliğini döndürür.

> Not: `dbg!` makrosunu çağırmak standart hataya konsol akışına
> (`stderr`) yazar, `println!`'un standart çıktı konsol akışına
> (`stdout`) yazdığı karşıt. Bölüm 12'deki ["Hataları Standart Hataya Yönlendirme" (Redirecting Errors to Standard Error)<!--
> ignore --> bölümünde `stderr` ve `stdout` hakkında daha fazla konuşacağız.

İşte `width` alanına atanan değere ve ayrıca `rect1`'deki tüm yapının değerine ilgilendiğimiz bir örnek:

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/src/main.rs}}
```

`dbg!` makrosunu `30 * scale` ifadesinin etrafına koyabiliriz çünkü `dbg!` ifadesinin değerinin sahipliğini döndürür ve `width` alanı orada `dbg!` çağrısı olmasaydı aynı değeri alacak. `rect1`'nin `dbg!`'nin sahipliğini almasını istemiyoruz bu yüzden bir sonraki çağrıda `rect1`'e bir başvuru kullanıyoruz. Bu örneğin çıktısı şöyle görünecek:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/output.txt}}
```

Çıktının ilk kısmının _src/main.rs_'nın 10. satırından geldiğini görebiliriz orada `30 * scale` ifadesini hata ayıklıyoruz ve sonuç değeri `60`'tır (tamsayılar için `Debug` biçimlendirmesi sadece değerlerini yazmaktır). _src/main.rs_'nın 14. satırındaki `dbg!` çağrısı `&rect1` değerini çıktı ki bu `Rectangle` yapısıdır. Bu çıktı `Rectangle` tipinin güzel `Debug` biçimlendirmesini kullanır. Kodunuz ne yaptığını anlamaya çalışırken `dbg!` makrosu gerçekten yararlı olabilir!

`Debug` özelliğine ek olarak, Rust `derive` niteliği ile kullanmak için bize çok sayıda özellik sağladı ki bunlar bizim özel tiplerimize kullanışlı davranış ekleyebilir. Bu özellikler ve davranışları [Ek C'de][app-c]<!-- ignore --> listelenmiştir. Özel davranışla bu özellikleri nasıl uygulayacağımızı ve kendi özelliklerinizi nasıl oluşturacağınızı Bölüm 10'da kapsamlı bir şekilde konuşacağız. `derive`'den başka çok sayıda nitelik vardır; daha fazla bilgi için [Rust Referansındaki "Nitelikler" (Attributes)<!--
> ignore --> bölümüne bakın.

`area` fonksiyonumuz çok özgüdür: Sadece dikdörtgenlerin alanını hesaplar. Bu davranışı `Rectangle` yapımıza daha yakından bağlamak yardımcı olurdu çünkü başka hiçbir tip ile çalışmaz. Bu kodu `area` fonksiyonunu `Rectangle` tipimiz üzerinde tanımlanan bir `area` yöntemine (method) dönüştürerek nasıl yeniden düzenleyebileceğimize bakalım.

[the-tuple-type]: ch03-02-data-types.html#the-tuple-type
[app-c]: appendix-03-derivable-traits.md
[println]: ../std/macro.println.html
[dbg]: ../std/macro.dbg.html
[err]: ch12-06-writing-to-stderr-instead-of-stdout.html
[attributes]: ../reference/attributes.html