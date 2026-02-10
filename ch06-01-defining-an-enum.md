## Bir Enum Tanımlama (Defining an Enum)

Yapılar ilgili alanları ve veriyi gruplamanın bir yolu verirler, örneğin `width` ve `height`'lu bir `Rectangle` gibi, enumler bir değerin olası değer kümesinden birisi olduğu yolda bir yol verirler. Örneğin, `Rectangle`'in `Circle` ve `Triangle` de içeren olası şekiller kümesinden birisi olduğunu söylemek isteyebiliriz. Bunu yapmak için, Rust bu olasılılıkları bir enum olarak kodlamamıza izin verir.

Enum'lerin bu durumda neden kullanışlı olduğunu ve yapılardan daha uygun olduğunu görmek için kodda ifade etmek isteyebileceğimiz bir duruma bakalım. IP adresleriyle çalışmamız gerektiğini söyleyelim. Şu an, IP adresleri için iki ana standart kullanılır: sürüm dört ve sürüm altı. Bunlar programımızın karşılaşacağı IP adresleri için olasılılıklar olduğu için tüm olası varyantları _numaralandırabiliriz_ ki bu numaralandırma adını aldığı yerdir.

Herhangi bir IP adresi ya sürüm dört veya sürüm altı adresi olabilir ancak aynı anda her ikisi de değil. IP adreslerinin bu özelliği enum veri yapısını uygun yapar çünkü bir enum değeri sadece varyantlarından biri olabilir. Sürüm dört ve sürüm altı adresleri hala temelde IP adresleridirler bu yüzden kod IP adresinin herhangi bir türü için geçerli olan durumları işlerken aynı tip olarak davranılmalıdır.

Bu kavramı kodda bir `IpAddrKind` numaralandırmasını tanımlayarak ve IP adresinin olası türleri olan `V4` ve `V6`'yi listeyerek ifade edebiliriz. Bunlar enum'in varyantlarıdır:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:def}}
```

`IpAddrKind` artık kodumuzun başka yerlerinde kullanabileceğimiz özel bir veri tipidir.

### Enum Değerleri

`IpAddrKind`'in iki varyantının her birinin örneklerini şu gibi oluşturabiliriz:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:instance}}
```

Enum'in varyantlarının tanımlayıcısı altında adlandırıldığını ve ikiyi ayırmak için çift nokta kullandığımızı fark edin. Bu kullanışlıdır çünkü artık `IpAddrKind::V4` ve `IpAddrKind::V6` değerlerinin ikisi de aynı tip: `IpAddrKind`. Örneğin, herhangi bir `IpAddrKind` alan bir fonksiyon tanımlayabiliriz:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn}}
```

Ve herhangi bir varyantla bu fonksiyonu çağırabiliriz:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn_call}}
```

Enum'leri kullanmak daha fazla avantajlara sahiptir. IP adresi tipimiz hakkında daha fazla düşünürsek, şu an IP adresi _verisini_ saklamanın bir yolu yoktur; sadece ne _tür_ olduğunu biliyoruz. Bölüm 5'te yapılar hakkında yeni öğrendiğiniz verilir, bu sorunu Kod Listesi 6-1'de gösterildiği gibi yapılarla çözmeye çalışabilirsiniz.

<Listing number="6-1" caption="IP adresinin verisini ve `IpAddrKind` varyantını bir `struct` kullanarak saklama">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-01/src/main.rs:here}}
```

</Listing>

Burada, iki alanı olan bir `IpAddr` yapısı tanımladık: `IpAddrKind` tipinde bir `kind` alanı (önceden tanımladığımız enum) ve `String` tipinde bir `address` alanı. Bu yapının iki örneği var. İlk olan `home`'dir ve `kind` olarak `IpAddrKind::V4` değeri ve `127.0.0.1` adres verisiyle ilişkili. İkinci örnek `loopback`'tir. Diğer `IpAddrKind` varyantını `kind` değeri olarak, `V6`'yi ve `::1` adresiyle ilişkiliye sahip. `kind` ve `address` değerlerini bir araya toplamak için bir yapı kullandık böylece varyant değerle ilişkilendir.

Ancak, sadece bir enum kullanarak aynı kavramı ifade etmek daha kısadır: Bir yapı içinde bir enum yerine, veriyi doğrudan her enum varyantına koyabiliriz. Bu `IpAddr` enum'in yeni tanımı hem `V4` hem de `V6` varyantlarının ilişkili `String` değerleri olacağını söyler:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-02-enum-with-data/src/main.rs:here}}
```

Veriyi doğrudan enum'in her varyantına ekliyoruz bu yüzden fazladan bir yapıya ihtiyaç yoktur. Burada, enum'lerin nasıl çalıştığının başka bir ayrıntısını görmek de daha koladır: Tanımladığımız her enum varyantının adı ayrıca enum'in bir örneğini oluşturan bir fonksiyon olur. Yani, `IpAddr::V4()` bir `String` argüman alan ve `IpAddr` tipinde bir örnek dönen bir fonksiyon çağırsıdır. Enum tanımlamasının sonucu olarak bu kurucu fonksiyonu otomatik olarak tanımlanmış oluruz.

Bir yapı yerine bir enum kullanmanın başka bir avantajı vardır: Her varyant farklı tiplerde ve miktarlarda ilişkili veriye sahip olabilir. Sürüm dört IP adresleri her zaman 0 ile 255 arasında değerleri olacak dört sayısal bileşene sahip olacak. Eğer `V4` adreslerini dört `u8` değeri olarak saklamak istesek ama hala `V6` adreslerini tek bir `String` değeri olarak ifade etmek istesek, bir yapıyla yapamazdık. Enumler bu durumu kolaylıkla ele alır:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-03-variants-with-different-data/src/main.rs:here}}
```

Sürüm dört ve sürüm altı IP adreslerini saklamak için farklı veri yapıları tanımlamanın birkaç farklı yolunu gösterdik. Ancak, görünenler gibi, IP adreslerini saklamak ve hangi tür olduklarını kodlamak o kadar yaygın ki [standart kütüphane kullanabileceğimiz bir tanımı vardır!][IpAddr]<!-- ignore --> Standart kütüphanenin `IpAddr`'yı nasıl tanımladığına bakalım. Tanımladığımız ve kullandığımız tam enum ve varyantları vardır ancak varyantların içindeki adres verisini iki farklı yapı içinde gömer ki her varyant için farklı tanımlanmışlar:

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

Bu kod bir enum varyantının içine herhangi bir türde veri koyabileceğinizi gösterir: dizeler, sayısal tipler veya yapılar, örneğin. Başka bir enum bile içerebilirsiniz! Ayrıca, standart kütüphane tipleri genellikle aklınıza gelebilecekten çok daha karmaşık değildirler.

Standart kütüphanenin `IpAddr` için bir tanım içerdiği halde, hala kendi tanımımızı oluşturabilir ve kullanabiliriz çünkü standart kütüphanenin tanımını kapsamımıza getirmedik. Bölüm 7'de tipleri kapsama getirme hakkında daha fazla konuşacağız.

Kod Listesi 6-2'deki başka bir enum örneğine bakalım: Bu birinin varyantlarında çeşitli türler gömülür.

<Listing number="6-2" caption="Varyantlarının her biri farklı miktarlarda ve türlerde değerler saklayan bir `Message` enum">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

</Listing>

Bu enum'in farklı tiplerde dört varyantı vardır:

- `Quit`: Hiçbir ilişkili verisi yok
- `Move`: Bir yapı yaptığı gibi adlandırılmış alanlara sahip
- `Write`: Tek bir `String` içerir
- `ChangeColor`: Üç `i32` değeri içerir

Kod Listesi 6-2'deki gibi varyantları olan bir enum tanımlamak, farklı yapı tanımlarının türlerini tanımlamaya benzer, ancak enum `struct` anahtar kelimesini kullanmaz ve tüm varyantlar `Message` tipi altında gruplanır. Aşağıdaki yapılar önceki enum varyantlarının sakladığı aynı veriyi tutabilir:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-04-structs-similar-to-message-enum/src/main.rs:here}}
```

Ancak her biri kendi tipine sahip farklı yapılar kullansaydık, Kod Listesi 6-2'de tanımlanan `Message` enum ile yaptığımız gibi bu tür mesajlardan herhangi birini alan bir fonksiyonu kolayca tanımlayamazdık ki bu tek bir tiptir.

Enum'ler ve yapılar arasında bir benzerlik daha vardır: Yapılarda `impl` kullanarak yöntemler tanımlayabildiğimiz gibi, enum'lerde de yöntemler tanımlayabiliriz. İşte `Message` enumimizde tanımlayabileceğimiz `call` adında bir yöntem:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-05-methods-on-enums/src/main.rs:here}}
```

Yöntemin gövdesi yöntemin çağrıldığı değeri almak için `self` kullanacaktır. Bu örnekte, `Message::Write(String::from("hello"))` değerine sahip bir `m` değişkeni oluşturduk ki bu `m.call()` çalıştığında `call` yönteminin gövdesinde `self` olacak şeydir.

Standart kütüphanede çok yaygın ve kullanışlı başka bir enum'e bakalım: `Option`.

<!-- Old headings. Do not remove or links may break. -->

<a id="the-option-enum-and-its-advantages-over-null-values"></a>

### `Option` Enum (The `Option` Enum)

Bu bölüm standart kütüphane tarafından tanımlanan başka bir enum olan `Option`'u bir vaka çalışması olarak inceler. `Option` tipi bir değerin bir şey olabileceğini veya hiçbir şey olabileceğini ifade eden çok yaygın bir senaryoyu kodlar.

Örneğin, boş olmayan bir listedeki ilk ögeyi isterseniz, bir değer alırsınız. Boş bir listedeki ilk ögeyi isterseniz, hiçbir şey alırsınız. Bu kavramı tip sistemi cinsinden ifade etmek, derleyicinin işlemelmeniz gereken tüm vakaları işleyip işlemediğinizi kontrol edebilmesini sağlar; bu işlevsellik diğer programlama dillerinde aşırı yaygın olan hataları önlemeye yardımcı olabilir.

Programlama dili tasarımı genellikle hangi özellikleri eklediğiniz cinsinden düşünülür ancak eklemediğiniz özellikler de önemlidirler. Rust'ın çok başka dilde olan null özelliği yoktur. _Null_ orada hiçbir değer olmadığı anlamına gelen bir değerdir. Null'lu dillerde değişkenler her zaman iki durumdan birinde olabilir: null veya not-null.

2009'deki "Null Başvuruları: Milyar Dolarlık Hata" sunuşunda null'un mucidi Tony Hoare şunu söyledi:

> Buna milyar dolarlık hatabiyorum diyoruz. O zamanda, bir nesne yönelimli dilinde
> başvurular için ilk kapsamlı tip sistemi tasarlıyordum. Hedefim tüm başvuru
> kullanımlarının mutlak güvenli olmasını sağlamak ve kontrolü derleyici tarafından otomatik olarak
> gerçekleştirmekti. Ancak null başvurusunu koyma cazasına karşı koyamadım çünkü
> uygulamak o kadar koladı. Bu sayılamaz hatalara, güvenlik açıklarına ve sistem
> çökmesine yol açtı ki son kırk yılda muhtemelen milyar dolarlık acı ve hasara neden oldu.

Null değerlerin sorunu şudur ki null olmayan bir değer gibi null bir değer kullanmaya çalışırsanız, bir çeşit hata alırsınız. Bu null veya not-null özelliği yaygın olduğu için, bu tür hatayı yapmak aşırı kolaydır.

Ancak, null'un ifade etmeye çalıştığı kavram hala kullanışlıdır: Null şu an için geçersiz veya yok olan bir değerdir.

Sorun gerçekten kavramla değil ancak belirli uygulamadır. Bu şeklide, Rust null'a sahip değildir ancak bir değerin var veya yok olduğunu kodlayan bir enum'e sahiptir. Bu enum `Option<T>`'dir ve standart kütüphane tarafından şöyle [tanımlanmıştır][option]<!-- ignore -->:

```rust
enum Option<T> {
    None,
    Some(T),
}
```

`Option<T>` enum'i o kadar kullanışlıdır ki prelude'a (ön söz) dahil edilir; onu kapsama açıkça getirmenize gerekmez. Varyantları da prelude'a dahildir: `Option::` öneki olmadan doğrudan `Some` ve `None` kullanabilirsiniz. `Option<T>` enum'i hala normal bir enum'dir ve `Some(T)` ve `None` hala `Option<T>` tipinin varyantlarıdır.

`<T>` sözdizimi henüz konuşmadığımız bir Rust özelliğidir. Bu genel bir tip parametresidir ve genel tipleri Bölüm 10'da daha ayrıntılı olarak kapsamlıyacağız. Şu an bilmeniz gereken şudur ki `<T>` `Option` enum'in `Some` varyantının herhangi bir tipde bir parça veri tutabileceğini ve `T`'nin yerine kullanılan her somut tipinin tüm `Option<T>` tipini farklı bir tip yaptığı anlamına gelir. İşte sayısal tipler ve karakter tiplerini tutmak için `Option` değerlerini kullanmanın bazı örnekleri:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-06-option-examples/src/main.rs:here}}
```

`some_number`'ın tipi `Option<i32>`'dir. `some_char`'ın tipi `Option<char>`'dir ki bu farklı bir tiptir. Rust bu tipleri çıkarabilir çünkü `Some` varyantının içinde bir değer belirttik. `absent_number` için, Rust bize tüm `Option` tipini açıklamamızı gerektirir: Derleyici sadece `None` değerine bakarak karşılık gelen `Some` varyantının hangi tipi tutacağını çıkaramaz. Burada, Rust'a `absent_number`'ın `Option<i32>` tipinde olmasını söyleiyoruz.

`Some` değerine sahip olduğumuzda, bir değerin var olduğunu ve değerin `Some` içinde tutulduğunu biliyoruz. `None` değerine sahip olduğumuzda, bir anlamda null ile aynı şey anlamına gelir: Geçerli bir değerimiz yok. O zaman, `Option<T>`'e sahip olmak null'dan daha iyi neden?

Kısacası, çünkü `Option<T>` ve `T` (burada `T` herhangi bir tip olabilir) farklı tiplerdir, derleyici bizi bir `Option<T>` değerini kesin geçerli bir değermiş gibi kullanmaya izin vermez. Örneğin, bu kod derlenmez çünkü bir `i8`'i bir `Option<i8>`'e eklemeye çalışıyor:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/src/main.rs:here}}
```

Bu kodu çalıştırsak, şu gibi bir hata mesajı alırız:

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/output.txt}}
```

Yoğun! Etkide, bu hata mesajı Rust'ın bir `i8`'i ve bir `Option<i8>`'i nasıl ekleyeceğini anlamadığını demektir çünkü farklı tiplerdirler. Rust'ta `i8` gibi bir tipde bir değere sahip olduğumuzda, derleyici her zaman geçerli bir değerimize sahip olduğumuzı sağlar. O değeri kullanmadan önce null için kontrol yapmadan güvenle ilerleyebiliriz. Sadece `Option<i8>`'ye (veya çalıştığımız değer herhangi ne tipde) sahip olduğumuzda değer olmayabilirliği konusunda endişelenmek zorunda kalırız ve derleyici o vakayı işlemeden önce değeri kullanmayı sağlar.

Başka bir deyimle, `Option<T>`'yi `T`'ye dönüştürmeniz gerekir ki onunla `T` işlemlerini yapabilirsiniz. Genel olarak, bu null ile en yaygın sorunlardan birini yakalamaya yardımcı olur: Bir şey aslında nullken null olmadığını varsaymak.

Null olmayan bir değeri yanlış varsayma riskini ortadan kaldırmak kodunuzda daha güvenli olmanıza yardımcı olur. Null olabilecek bir değere sahip olmak için, o değerin tipini `Option<T>` yaparak açıkça seçmelisiniz. Sonra, o değeri kullandığınızda, değer null olduğunda o vakayı açıkça işlemek zorundasınız. Bir değerin `Option<T>` olmayan bir tipi olduğu her yerde, değerin null olmadığını güvenli bir şekilde _varsayabilirsiniz_. Bu Rust için null'un yaygınlığını sınırlamak ve Rust kodunun güvenliliğini artırmak için bilinçli bir tasarım kararındır.

O zaman `Option<T>` tipinde bir değere sahip olduğunuzda `Some` varyantından `T` değerini nasıl alırsınız ki o değeri kullanabilirsiniz? `Option<T>` enum'i çeşitli durumlarda kullanışlı olan çok sayıda yöntemlere sahiptir; [belgelerinde][docs]<!-- ignore --> kontrol edebilirsiniz. `Option<T>` üzerinde yöntemlerle tanışmanız Rust yolculuğunuzda aşırı kullanışlı olacaktır.

Genel olarak, bir `Option<T>` değerini kullanmak için, her varyantını işleyen koda ihtiyaçınız olacak. Sadece `Some(T)` değerine sahip olduğunuzda çalışacak bazı koda ihtiyaçınız olacak ve bu kod iç `T`'yi kullanabilir. Sadece `None` değerine sahip olduğunuzda çalışacak bazı başka koda ihtiyaçınız olacak ve bu kodun bir `T` değeri mevcut değil. `match` ifadesi enum'lerle kullanıldığında tam bunu yapan bir akış kontrol yapısıdır: Enum'in hangi varyantına sahip olduğuna bağlı farklı kod çalıştırır ve o kod eşleşen değer içindeki veriyi kullanabilir.

[IpAddr]: ../std/net/enum.IpAddr.html
[option]: ../std/option/enum.Option.html
[docs]: ../std/option/enum.Option.html