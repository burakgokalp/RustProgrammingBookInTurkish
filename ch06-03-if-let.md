## `if let` ve `let...else` ile Kısık Akış Kontrolü (Concise Control Flow with `if let` and `let...else`)

`if let` sözdizimi `if` ve `let`'i birbirine birleştirir ki bu bir desen eşleşen değerleri işlemek ancak geri kalanı yok saymak için daha az sözcüklü bir yol sağlar. Kod Listesi 6-6'teki `config_max` değişkenindeki `Option<u8>` değerini eşleştiren ancak sadece değerin `Some` varyantiyse kod çalıştırmak isteyen programı düşünün.

<Listing number="6-6" caption="Sadece değerin `Some` olduğu zaman kod çalıştırmasıyla ilgilenen bir `match`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-06/src/main.rs:here}}
```

</Listing>

Değer `Some` ise, `Some` varyantındaki değeri desen içindeki `max` değişkenine bağlayarak yazdırırız. `None` değeriyle hiçbir şey yapmak istemiyoruz. `match` ifadesini karşılamak için, sadece bir varyant işledikten sonra `_ => ()` eklemeliyiz ki bu eklemek sinir boilerplate kodu.

Bunun yerine, `if let` kullanarak daha kısa bir şekilde yazabiliriz. Aşağıdaki kod Kod Listesi 6-6'daki `match` ile aynı davranır:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-12-if-let/src/main.rs:here}}
```

`if let` sözdizimi bir deseni ve eşit işaretiyle ayrılmış bir ifade alır. `match` ile aynı şekilde çalışır ki ifade `match`'e verilir ve desen ilk koludur. Bu durumda, desen `Some(max)`'tır ve `max` `Some` içindeki değere bağlanır. Sonra `max`'i `if let` bloğunun gövdesinde karşılık gelen `match` kolda kullandığımız aynı şekilde kullanabiliriz. `if let` bloğundaki kod sadece desenle eşleşirse çalışır.

`if let` kullanmak daha az yazma, daha az girintileme ve daha az boilerplate kod anlamına gelir. Ancak, `match`'in zorladığı tamdızı kontrolü kaybedersiniz ki herhangi bir vakaı işlemeyi unuttuğunuzu onaylar. `match` ve `if let` arasında seçmek özel durumunuzda ne yaptığınıza ve tamdızı kontrolünü kaybetmek karşılığında kısık kazanmanın uygun bir takas olup olmadığına dayanır.

Başka bir deyimle, `if let`'i değerin bir desene eşleştiğinde kod çalıştıran ve sonra diğer tüm değerleri yok sayan bir `match` için sözdizmi şekeri (syntax sugar) olarak düşünebilirsiniz.

`if let` ile bir `else` de ekleyebiliriz. `else` ile giden kod bloğu, `if let` ve `else` ile eşdeğer olan `match` ifadesindeki `_` vakasıyla giden kod bloğu ile aynıdır. Kod Listesi 6-4'teki `Coin` enum tanımını hatırlayın ki `Quarter` varyantı ayrıca bir `UsState` değeri tutuyordu. Eğer görülen çeyrek olmayan tüm paraları saymak isteseydik ve ayrıca çeyreklerin durumunu açıklamak isteseydik, bir `match` ifadesiyle yapabilirdik, şöyle:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-13-count-and-announce-match/src/main.rs:here}}
```

Veya bir `if let` ve `else` ifadesi kullanabilirdik, şöyle:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-14-count-and-announce-if-let-else/src/main.rs:here}}
```

## `let...else` ile "Mutlu Yol"da Kalmak (Staying on the "Happy Path" with `let...else`)

Yaygın desen bir değerin var olduğu zaman bazı hesaplamalar yapmak ve aksi takdirde varsayılan bir değer dönmektir. `UsState` değerli paralar örneğimizle devam edelim, eğer çeyrekteki durumun ne kadar eski olduğuna bağlı komik bir şey söylemek isteseydik, `UsState` üzerinde bir yöntem tanıtabiliriz ki bir durumun yaşını kontrol eder, şöyle:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:state}}
```

Sonra, Kod Listesi 6-7'teki gibi `if let` kullanarak para türünü eşleştirebiliriz, koşulun gövdesi içinde bir `state` değişkeni tanıtırarak.

<Listing number="6-7" caption="Bir `if let` içine gömülü koşullar kullanarak bir durumun 1900'da var olup olmadığını kontrol etme">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:describe}}
```

</Listing>

Bu işi halleder ancak işi `if let` ifadesinin gövdesine itmiş ve yapılacak iş daha karmaşıksa, en üst düzey dalların tam nasıl ilişkilendiğini takip etmek zor olabilir. Ayrıca ifadelerin bir değer ürettiği gerçeğinden yararlanabiliriz ki ya `if let`'ten `state` üretmek için ya da erken dönmek için, Kod Listesi 6-8'deki gibi. (Bunu bir `match` ile de benzer yapabilirdiniz.)

<Listing number="6-8" caption="Bir değer üretmek veya erken dönmek için `if let` kullanma">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-08/src/main.rs:describe}}
```

</Listing>

Bu kendi yolunda takip etmesi biraz sinir bozucu! `if let`'in bir dalı bir değer üretir ve diğeri tamamen fonksiyondan döner.

Bu yaygın deseni daha güzel ifade etmek için Rust'ta `let...else` vardır. `let...else` sözdizimi sol tarafta bir desen ve sağ tarafta bir ifade alır, `if let` ile çok benzer ancak `if` dalı yoktur, sadece `else` dalı vardır. Eğer desen eşleşirse, desenin değerini dış kapsama bağlayacaktır. Eğer desen eşleşmezse, program `else` dala akacak ki bu fonksiyondan dönmelidir.

Kod Listesi 6-9'da, Kod Listesi 6-8'in `if let` yerine `let...else` kullanırken nasıl göründüğünü görebilirsiniz.

<Listing number="6-9" caption="Fonksiyon boyunca akışı netleştirmek için `let...else` kullanma">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-09/src/main.rs:describe}}
```

</Listing>

Fark edin ki bu yolla fonksiyonun ana gövdesinde "mutlu yol"da kalır ve iki dal için önemli derecede farklı akış kontrolü olmaz ki `if let` yaptığı gibi.

Eğer bir `match` kullanarak ifade etmek için fazla sözcüklü olan mantığı olan bir durumunuz varsa, `if let` ve `let...else`'in de Rust araç kutunuzda olduğunu hatırlayın.

## Özet

Şimdi enumler kullanarak numaralandırılmış değerler kümesinden biri olabilecek özel tipleri nasıl oluşturacağımızı kapsamladık. Standart kütüphanenin `Option<T>` tipinin tip sistemini kullanarak hataları önlemek için nasıl yardımcı olduğunu gösterdik. Enum değerleri içinde veriye sahip olduğunda, işlemeniz gereken vakalara bağlı olarak o değerleri çıkarmak ve kullanmak için `match` veya `if let` kullanabilirsiniz.

Rust programlarınız artık yapılar ve enumler kullanarak alanınızdaki kavramları ifade edebilir. API'nizde kullanılacak özel tipler oluşturmak tip güvenliğini sağlar: Derleyici, fonksiyonların sadece her fonksiyonun beklediği tiplerdeki değerleri aldığından emin olur.

Kullanıcılarınıza iyi organize edilmiş, kullanımı kolay ve tam olarak kullanıcılarınızın ihtiyaç duyacak şeyleri açığa sunan bir API sağlamak için, şimdi Rust'ın modüllerine dönelim.