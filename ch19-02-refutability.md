## Yanılabilirlik: Bir Desenin Eşleşmeyi Başarısız Olabilir mi (Refutability: Whether a Pattern Might Fail to Match)

Desenler iki formda gelir: yanılabilir (refutable) ve yanılmaz (irrefutable). Geçilen her
olası değere eşleşecek desenler _yanılmazdır_. `let x = 5;` ifadesindeki `x` bunun bir
örneğidir çünkü `x` her şeye eşleşir ve bu yüzden eşleşmeyi başarısız olamaz. Bazı olası
değerler için eşleşmeyi başarısız olabilecek desenler _yanılabilirdir_. `if let Some(x) =
a_value` ifadesindeki `Some(x)` bunun bir örneğidir çünkü `a_value` değişkenindeki
değer `Some` yerine `None` ise, `Some(x)` deseni eşleşmeyecektir.

Fonksiyon parametreleri, `let` ifadeleri ve `for` döngüleri sadece yanılmaz desenleri
kabul edebilir çünkü değerler eşleşmediğinde program anlamlı bir şey yapamaz. `if let` ve
`while let` ifadeleri ve `let...else` ifadesi hem yanılabilir hem de yanılmaz desenleri kabul eder,
ancak derleyici yanılmaz desenlere karşı uyarır çünkü tanım gereği, olası başarısızlığı ele
almak için tasarlanmışlardır: Bir koşullunun işlevselliği, başarısızlığa veya başarıya bağlı
olarak farklı davranmasında yatar.

Genel olarak, yanılabilir ve yanılmaz desenler arasındaki ayrım konusunda endişelenmeyin;
ancak, yanılmazlık kavramına aşina olmanız gerekir böylece bir hata mesajında gördüğünüzde
yanıt verebilirsiniz. Bu durumlarda, kodun amaçlanan davranışına bağlı olarak ya deseni ya da
deseni kullandığınız yapıyı değiştirmeniz gerekir.

Rust'un yanılmaz bir desen gerektirdiği yerde yanılabilir bir desen kullanmaya çalıştığımızda
ve tam tersi olduğunda ne olduğunu bir örnekle bakalım. Kod Listesi 19-8 bir `let` ifadesi
gösterir ancak desen için `Some(x)`'i, bir yanılabilir deseni belirttik. Beklediğiniz gibi,
bu kod derlenmeyecektir.

<Listing number="19-8" caption="Attempting to use a refutable pattern with `let`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-08/src/main.rs:here}}
```

</Listing>

Eğer `some_option_value` bir `None` değeri ise, `Some(x)` desenine eşleşmeyecektir, bu
demektir ki desen yanılabilirdir. Ancak, `let` ifadesi sadece yanılmaz bir deseni kabul
edebilir çünkü bir `None` değeriyle kodun yapabileceği geçerli bir şey yoktur. Derleme zamanında,
Rust yanılmaz bir desen gerektiğinde yanılabilir bir desen kullanmaya çalıştığımızımızı şikayet
edecek:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-08/output.txt}}
```

Çünkü desen `Some(x)` ile geçerli her değeri ele alamadık (ve alamazdık!), Rust haklı bir şekilde
derleyici hatası üretir.

Eğer yanılabilir bir desen gerektiğinde yanılabilir bir desenimiz varsa, deseni kullanan kodu
değiştirerek düzeltebiliriz: `let` yerine `let...else` kullanabiliriz. Sonra, desen
eşleşmezse, küme parantez içindeki kod değeri ele alır. Kod Listesi 19-9, Kod Listesi 19-8'deki
kodu nasıl düzelteceğimizi gösterir.

<Listing number="19-9" caption="Using `let...else` and a block with refutable patterns instead of `let`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-09/src/main.rs:here}}
```

</Listing>

Koda bir kaçış verdiğiz! Bu kod tamamen geçerlidir, bu yüzden bir uyarı almadan yanılmaz bir
desen kullanamayız. Eğer `let...else`'e `x` gibi her zaman eşleşecek bir desen verirsek,
Kod Listesi 19-10'da gösterildiği gibi, derleyici bir uyarı verecek.

<Listing number="19-10" caption="Attempting to use an irrefutable pattern with `let...else`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-10/src/main.rs:here}}
```

</Listing>

Rust, `let...else`'i yanılmaz bir desenle kullanmak mantıklı değil diyerek şikayet eder:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-10/output.txt}}
```

Bu nedenle, match kolları yanılabilir desenler kullanmalıdır, son kol hariç ki o kalan tüm
değerlerle yanılabilir bir desenle eşleşmeli. Rust bize tek bir kolu olan bir `match` içinde
yanılmaz bir desen kullanmamıza izin verir ancak bu sözdizimi özellikle kullanışlı değildir
ve daha basit bir `let` ifadesiyle değiştirilebilir.

Artık desenleri nerede kullanacağınızı ve yanılabilir ve yanılmaz desenler arasındaki farkı
biliyorsunuz, desenler oluşturmak için kullanabileceğimiz tüm sözdizimini ele alalım.