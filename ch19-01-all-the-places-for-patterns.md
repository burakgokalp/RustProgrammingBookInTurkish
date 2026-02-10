## Desenlerin Kullanabileceği Tüm Yerler (All the Places Patterns Can Be Used)

Desenler Rust'ta çok yerde görünür ve onları hiç fark etmeden çok kullandınız! Bu
bölüm desenlerin geçerli olduğu tüm yerleri ele alır.

### `match` Kolları (match Arms)

Bölüm 6'da tartıştığımız gibi, `match` ifadelerinin kollarında desenleri kullanıyoruz.
Resmi olarak, `match` ifadeleri `match` anahtar kelimesi, eşleşilecek bir değer ve
değerin o kolun desenine uyarsa çalıştıralacak bir ifade içeren bir veya daha fazla
eşleşme kolundan oluşur, şöyle:

<!--
  Manually formatted rather than using Markdown intentionally: Markdown does not
  support italicizing code in body of a block like this!
-->

<pre><code>match <em>VALUE</em> {
    <em>PATTERN</em> => <em>EXPRESSION</em>,
    <em>PATTERN</em> => <em>EXPRESSION</em>,
    <em>PATTERN</em> => <em>EXPRESSION</em>,
}</code></pre>

Örneğin, Bölüm 6'dan Kod Listesi 6-5'teki `x` değişkenindeki bir `Option<i32>`
değerine eşleşen bu `match` ifadesi:

```rust,ignore
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

Bu `match` ifadesindeki desenler her okun solundaki `None` ve `Some(i)`.

`match` ifadeleri için bir gereklilik, onların `match` ifadesindeki değer için tüm
olasılıkları hesaba katmasıdır. Her olasıılığı kapsadığınızdan emin olmanın bir yolu,
son kol için bir her şeyi yakalayan desen bulundurmak: Örneğin, herhangi bir değere
eşleşen bir değişken adı hiçbir zaman başarısamaz ve böylece kalan her durumu kapsar.

`_` özel deseni her şeye eşleşir ancak hiçbir değişkene bağlanmaz, bu yüzden
son eşleşme kolunda sık kullanılır. `_` deseni, belirtilmemiş herhangi bir değeri yoksaymak
istediğinizde kullanışlı olabilir. `_` desenine bu bölümün sonunda
[“Desen İçinde Değerleri Yoksayma”][ignoring-values-in-a-pattern]<!-- ignore -->
bölümünde daha detaylı bakacağız.

### `let` İfadeleri (let Statements)

Bu bölümden önce, sadece `match` ve `if let` ile desenleri kullanmayı açıkça
tartışmıştık ancak gerçekte, desenleri başka yerlerde de kullandık, `let` ifadeleri
dahil. Örneğin, `let` ile bu basit değişken atamasını düşünün:

```rust
let x = 5;
```

Böyle bir `let` ifadesi her kullandığınızda desenler kullandınız, bunu fark
etmemiş olabilirseniz! Daha resmi olarak, bir `let` ifadesi şöyle görünür:

<!--
  Manually formatted rather than using Markdown intentionally: Markdown does not
  support italicizing code in body of a block like this!
-->

<pre>
<code>let <em>PATTERN</em> = <em>EXPRESSION</em>;</code>
</pre>

`let x = 5;` gibi PATTERN yuvasında bir değişken adı olan ifadelerde, değişken
adı desenin özel basit bir formudur. Rust ifadeden karşı desenle karşılaştırır ve
bulduğu her adı atar. Bu yüzden, `let x =5;` örneğinde, `x` "buraya eşleşen şeyi
`x` değişkenine ata" anlamında bir desendir. `x` adı tüm desen olduğu için, bu desen
etkili olarak "her şeyi `x` değişkenine ata, değerin ne olursa olsun" anlamına gelir.

`let`'in desen eşleşme yönünü daha net görmek için, Kod Listesi 19-1'e bakın, bu
bir demeti (tuple) bozmak için `let` ile bir desen kullanır.


<Listing number="19-1" caption="Using a pattern to destructure a tuple and create three variables at once">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-01/src/main.rs:here}}
```

</Listing>

Burada, bir demeti bir desenle eşleştiriyoruz. Rust `(1, 2, 3)` değerini `(x, y, z)`
deseniyle karşılaştırır ve değerin desenle eşleştiğini görür—yani her iki tarafta
öğe sayısının aynı olduğunu görür—bu yüzden Rust `1`'i `x`'e, `2`'yi `y`'ye ve `3`'ü
`z`'ye atar. Bu demet desenini içinde yuvarlanmış üç ayrı değişken deseni olarak
düşünebilirsiniz.

Desendeki öge sayısı demetteki öge sayısına uyamzsa, genel tip eşleşmeyecek ve
derleyici hatası alacağız. Örneğin, Kod Listesi 19-2 üç ögesi olan bir demeti iki
değişkene bozmaya çalışan bir girişimi gösterir ki çalışmaz.

<Listing number="19-2" caption="Incorrectly constructing a pattern whose variables don't match number of elements in tuple">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-02/src/main.rs:here}}
```

</Listing>

Bu kodu derlemeye çalışmak bu tip hatasını üretir:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-02/output.txt}}
```

Hatayı düzeltmek için, `_` veya `..` kullanarak demet içindeki bir veya daha fazla
değeri yoksayabilirsiniz, bunu [“Desen İçinde Değerleri Yoksayma”][ignoring-values-in-a-pattern]<!--
ignore --> bölümünde göreceksiniz. Eğer sorun desenin içinde çok fazla değişken
varsa, çözüm değişken sayısının demetteki öge sayısına eşit olması için değişkenleri
kaldırarak türleri eşleştirmektir.

### Koşullu `if let` İfadeleri (Conditional `if let` Expressions)

Bölüm 6'da, `if let` ifadelerini öncelikle sadece bir duruma eşleşen bir
`match`'in kısayolu olarak kullanmayı tartıştık. Seçene olarak, `if let` `if let`'teki
desen uyamzsa çalıştıralacak kod içeren birebir `else`'e sahip olabilir.

Kod Listesi 19-3, `if let`, `else if`, ve `else if let` ifadelerini karıştırmanın ve
eşleşmenin mümkün olduğunu gösterir. Bunu yapmak, desenlerle karşılaştırılacak yalnızca bir
değeri ifade edebileceğimiz bir `match` ifadesinden daha fazla esneklik sağlar. Ayrıca,
Rust bir `if let`, `else if`, ve `else if let` kollar serisindeki koşulların birbirleriyle
ilişkili olmasını gerektirmez.

Kod Listesi 19-3'teki kod, arka plan rengini belirlemek için birkaç koşul kontrolüne
dayanarak hangi rengi yapmanızı belirler. Bu örnek için, gerçek bir programın kullanıcı girdisinden
alabileceği kodlanmış değerlere sahip değişkenler oluşturduk.

<Listing number="19-3" file-name="src/main.rs" caption="Mixing `if let`, `else if`, `else if let`, and `else`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-03/src/main.rs}}
```

</Listing>

Eğer kullanıcı favori bir rengi belirtirse, o renk arka plan olarak kullanılır. Favori bir
rengi belirtilmezse ve bugün Salı ise, arka plan rengi yeşil olur. Aksi halde, eğer
kullanıcı yaşını bir dize olarak belirtirse ve onu başarılı bir sayı olarak ayrıştırabilirsek,
rengi ya mor ya da turuncu olur, sayının değerine göre. Eğer bu koşulların hiçbiri
uygulanmazsa, arka plan rengi mavidir.

Bu koşullu yapı, karmaşık gereksinimleri desteklememizi sağlar. Burada sahip olduğumuz
kodlanmış değerlerle, bu örnek `Using purple as the background color` yazdırır.

`if let`'in de, `match` kolları yapabildiği gibi mevcut değişkenleri gölgelendiren
yeni değişkenler getirebileceğini görebilirsiniz: `if let Ok(age) = age` satırı
`Ok` varyantının içindeki değeri tutan yeni bir `age` değişkenini getirir, mevcut
`age` değişkenini gölgelendirir. Bu demek ki `if age > 30` koşulunu bu bloğun içine
yerleştirmemiz gerek: Bu iki koşulu `if let Ok(age) = age && age > 30` şeklinde
birleştiremeyiz. 30 ile karşılaştırmak istediğimiz yeni `age`'yi, küme parantez ile
yeni kapsam başlayana kadar geçerli değildir.

`if let` ifadelerini kullanmanın dezavantajı, derleyicinin tamlılık (exhaustiveness)
kontrollememesidir, oysa `match` ifadeleriyle yapar. Son `else` bloğunu atlarırsak ve böylece
bazı durumlama ele almasayak, derleyici bize mümkün mantık hatasını uyarmazdı.

### `while let` Koşullu Döngüleri (while let Conditional Loops)

`if let` ile yapı olarak benzer, `while let` koşullu döngüsü, bir desen
eşleşmeye devam ettiği sürece bir `while` döngüsünün çalışmasını sağlar. Kod Listesi
19-4'te, iş parçacıkları arasındaki mesajları bekleyen bir `while let` döngüsünü
gösteriyoruz ancak bu durumda bir `Option` yerine bir `Result` kontrol ediyoruz.

<Listing number="19-4" caption="Using a `while let` loop to print values for as long as `rx.recv()` returns `Ok`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-04/src/main.rs:here}}
```

</Listing>

Bu örnek `1`, `2` ve sonra `3` yazdırır. `recv` yöntemi kanalın alıcı tarafındaki ilk
mesajı alır ve bir `Ok(value)` döner. Bölüm 16'da `recv`'yi ilk gördüğümüzde,
hatayı doğrudan açmıştık veya bir `for` döngüsü kullanarak bir yineleyici olarak onunla
etkileşmiştik. Ancak Kod Listesi 19-4'ün gösterdiği gibi, `while let`'i de kullanabiliriz
çünkü gönderi var olduğu sürece `recv` yöntemi her mesaj geldiğinde bir `Ok` döner ve
sonra gönderi tarafı ayrıldığında bir `Err` üretir.

### `for` Döngüleri (for Loops)

Bir `for` döngüsünde, `for` anahtar kelimesini doğrudan izleyen değer bir desendir.
Örneğin, `for x in y`'de `x` bir desendir. Kod Listesi 19-5, bir demeti bozmak veya
parçalara ayırmak için bir `for` döngüsünde bir desenin nasıl kullanılacağını
gösterir.


<Listing number="19-5" caption="Using a pattern in a `for` loop to destructure a tuple">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-05/src/main.rs:here}}
```

</Listing>

Kod Listesi 19-5'teki kod aşağıdakileri yazdırır:


```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-05/output.txt}}
```

`enumerate` yöntemini kullanarak bir yineleyici uyarlıyoruz böylece o bir değer ve o değer
için bir dizin (index) üretir, bir demete yerleştirilir. Üretilen ilk değer
`(0, 'a')` demetidir. Bu değer `(index, value)` deseniyle eşleştiğinde, dizin
`0` olacak ve değer `'a'` olacak, çıktının ilk satırını yazdırarak.


### Fonksiyon Parametreleri (Function Parameters)

Fonksiyon parametreleri de desen olabilir. Kod Listesi 19-6'teki, `i32` tipinde bir
`x` adında bir parametre alan `foo` adında bir fonksiyon bildiren kod, şimdiye kadar
tanıdık gelmeli.

<Listing number="19-6" caption="A function signature using patterns in parameters">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-06/src/main.rs:here}}
```

</Listing>

`x` parçası bir desendir! `let` ile yaptığımız gibi, bir fonksiyonun argümanlarında
bir desene demet eşleştirebiliriz. Kod Listesi 19-7, bir demeti içindeki değerleri onu bir
fonksiyona geçirirken böler.

<Listing number="19-7" file-name="src/main.rs" caption="A function with parameters that destructure a tuple">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-07/src/main.rs}}
```

</Listing>

Bu kod `Current location: (3, 5)` yazdırır. `&(3, 5)` değerleri `&(x, y)` deseniyle
eşleşir, böylece `x` `3` değeri ve `y` `5` değeri olur.

Kapamaların parametre listelerinde de fonksiyon parametre listelerinde olduğu gibi desenleri
kullanabiliriz çünkü kapamalar fonksiyonlara benzerdir, Bölüm 13'te tartıştığımız gibi.

Bu noktada, desenleri kullanmak için birkaç yol gördünüz ancak desenler, onları
kullandığımız her yerde aynı şekilde çalışmaz. Bazı yerlerde, desenler yanılmaz
(irrefutable) olmalıdır; başka koşullarda ise yanılabilir (refutable) olabilirler. Bu iki
kavramı sıradan tartışacağız.

[ignoring-values-in-a-pattern]: ch19-03-pattern-syntax.html#ignoring-values-in-a-pattern