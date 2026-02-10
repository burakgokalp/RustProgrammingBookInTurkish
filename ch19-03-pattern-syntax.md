## Desen Sözdizimi (Pattern Syntax)

Bu bölümde, desenlerde geçerli olan tüm sözdizimi topluyoruz ve her birini neden ve ne zaman
kullanmak isteyebilirsiniz tartışıyoruz.

### Sabit Değerleri Eşleştirme (Matching Literals)

Bölüm 6'da gördüğünüz gibi, desenleri sabit değerlere karşı doğrudan eşleştirebilirsiniz.
Aşağıdaki kod bazı örnekler veriyor:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-01-literals/src/main.rs:here}}
```

Bu kod `one` yazdırır çünkü `x` içindeki değer `1`dir. Bu sözdizimi, kodunuzun belirli
somut bir değer aldığında bir eylem yapmasını istediğinizde kullanışlıdır.

### Adlandırılmış Değişkenleri Eşleştirme (Matching Named Variables)

Adlandırılmış değişkenler herhangi bir değere eşleşen yanılmaz desenlerdir ve bu kitapta
onları çok kez kullandık. Ancak, `match`, `if let` veya `while let` ifadelerinde
adlandırılmış değişkenler kullanırken bir komplikasyon vardır. Bu ifade türlerinin her
biri yeni bir kapsam başlattığı için, bu ifadeler içindeki desenin bir parçası olarak
tanımlanan değişkenler yapıların dışındaki aynı adlı değişkenleri gölgelendirir, tüm
değişkenlerde olduğu gibi. Kod Listesi 19-11'de, `Some(5)` değerine sahip bir
`x` değişkeni ve `10` değerine sahip bir `y` değişkeni tanımlıyoruz. Sonra
`x` değeri üzerinde bir `match` ifadesi oluşturuyoruz. Eşleşme kollarındaki
desenlere ve sondaki `println!`'e bakın ve bu kodu çalıştırmadan veya daha fazla okumadan
önce kodun ne yazacağını tahmin etmeye çalışın.

<Listing number="19-11" file-name="src/main.rs" caption="A `match` expression with an arm that introduces a new variable which shadows an existing variable `y`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-11/src/main.rs:here}}
```

</Listing>

`match` ifadesi çalıştığında ne olduğunu sırasıyla bakalım. Birinci eşleşme kolundaki
desen `x`'in tanımlı değerine uymaz, bu yüzden kod devam eder.

İkinci eşleşme kolundaki desen `Some` değeri içindeki herhangi bir değere eşleşecek
yeni bir `y` değişkeni getirir. Çünkü `match` ifadesi içinde yeni bir kapsam içindeyiz,
bu yeni bir `y` değişkenidir, başlangıçta `10` değeri ile tanımladığımız `y` değil.
Bu yeni `y` bağı `Some` içindeki herhangi bir değere eşleşir, bu da `x`'teki durumdur.
Bu yüzden bu yeni `y` `x`'teki `Some`'in iç değerine bağlanır. Bu değer `5`dir,
bu yüzden o kolun ifadesi çalışır ve `Matched, y = 5` yazdırır.

Eğer `x` `Some(5)` yerine `None` değeri olsaydı, ilk iki kolun desenleri
eşleşmezdi, bu yüzden değer alt çizgiye eşleşirdi. Alt çizgi kolunun deseninde `x`
değişkeni getirmediğimiz için, ifadedeki `x` hala gölgelenmemiş dış `x`'dir. Bu
varsayımsal durumda, `match` `Default case, x = None` yazdırırdı.

`match` ifadesi bittiğinde, onun kapsamı sona erer ve böylece iç `y`'nin kapsamı.
Son `println!` `at the end: x = Some(5), y = 10` üretir.

Dış `x` ve `y`'nin değerlerini karşılaştıran bir `match` ifadesi oluşturmak için,
mevcut `y` değişkenini gölgelendiren yeni bir değişken getirmek yerine, bir eşleşme
gardi (match guard) koşullu kullanmamız gerekir. Eşleşme gardilerine sıradan
[“Eşleşme Gardileri ile Koşullu Eklemek”](#adding-conditionals-with-match-guards)<!--
ignore --> bölümünde bakacağız.

<!-- Old headings. Do not remove or links may break. -->
<a id="multiple-patterns"></a>

### Çoklu Desenleri Eşleştirme (Matching Multiple Patterns)

`match` ifadelerinde, desen `|` sözdizimini kullanarak çoklu desenleri eşleştirebilirsiniz,
bu desen _veya_ operatörüdür. Örneğin, aşağıdaki kodda, `x`'in değerini eşleşme
kollarına karşılaştırıyoruz, ilkinin bir _veya_ seçeneği var, bu demektir ki eğer
`x`'in değeri o koldaki değerlerden biriyle uyarsa, o kolun kodu çalışır:


```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-02-multiple-patterns/src/main.rs:here}}
```

Bu kod `one or two` yazdırır.

### Değer Aralıklarını `..=` ile Eşleştirme (Matching Ranges of Values with `..=`)

`..=` sözdizimi bizi dahil edici bir değer aralığına eşleşmemize izin verir. Aşağıdaki
kodda, bir desen verilen aralıktaki herhangi bir değere uyduğunda, o kol çalışır:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-03-ranges/src/main.rs:here}}
```

Eğer `x` `1`, `2`, `3`, `4` veya `5` ise, birinci kol uyar. Bu sözdizimi, aynı
fikri ifade etmek için `|` operatörünü kullanmaktan çoklu eşleşme değerleri için daha
uygundur; eğer `|` kullansaydik, `1 | 2 | 3 | 4 | 5` belirtmek zorunda kalırdık.
Bir aralık belirtmek çok daha kısadır, özellikle 1 ile 1.000 arasında herhangi bir
sayıya eşleşmek istiyorsak!

Derleyici derleme zamanında aralığın boş olmadığını kontrol eder ve çünkü Rust'ın bir
aralığın boş olup olmadığını söyleyebildiği tek türler `char` ve sayısal değerlerdir,
aralıklar sadece sayısal veya `char` değerleriyle izin verilir.

İşte `char` değerlerinin aralıklarını kullanan bir örnek:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-04-ranges-of-char/src/main.rs:here}}
```

Rust `'c'`'nin birinci desenin aralığı içinde olduğunu söyleyebilir ve `early ASCII
letter` yazdırır.

### Değerleri Bozmak için Yapılandırma (Destructuring to Break Apart Values)

Ayrıca, bu değerlerin farklı parçalarını kullanmak için yapılandırmak (destructure) struct'lar, enum'lar
ve demetleri (tuples) desenler kullanabiliriz. Her değer için sırasıyla bakalım.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-structs"></a>

#### Struct'lar (Structs)

Kod Listesi 19-12, `let` ifadesi ile bir desen kullanarak bozabileceğimiz iki alanı `x` ve `y`
olan bir `Point` struct'ını gösterir.

<Listing number="19-12" file-name="src/main.rs" caption="Destructuring a struct's fields into separate variables">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-12/src/main.rs}}
```

</Listing>

Bu kod `p` struct'ının `x` ve `y` alanlarının değerlerini eşleşen `a` ve `b` değişkenleri
oluşturur. Bu örnek, desendeki değişken adlarının struct'ın alan adlarıyla eşleşmesi gerekmediğini
gösterir. Ancak, hangi değişkenlerin hangi alanlardan geldiğini hatırlamayı kolaylaştırmak için
değişken adlarını alan adlarıyla eşleştirmek yaygındır. Bu yaygın kullanım nedeniyle ve
`let Point { x: x, y: y } = p;` yazmak çok fazla tekrar içerdiği için, Rust struct alanlarını
eşleştiren desenler için bir kısa yol vardır: Sadece struct alanının adını listelemeniz gerekir,
desenden oluşturulan değişkenler aynı adlara sahip olacaktır. Kod Listesi 19-13, Kod Listesi
19-12'deki kodla aynı şekilde davranır ancak `let` deseninde oluşturulan değişkenler `a` ve
`b` yerine `x` ve `y`'dir.

<Listing number="19-13" file-name="src/main.rs" caption="Destructuring struct fields using struct field shorthand">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-13/src/main.rs}}
```

</Listing>

Bu kod `p` değişkeninin `x` ve `y` alanlarına eşleşen `x` ve `y` değişkenlerini oluşturur.
Sonuç, `x` ve `y` değişkenlerinin `p` struct'ından değerleri içerdiği şeklindedir.

Ayrıca, tüm alanlar için değişkenler oluşturmak yerine, bir struct deseninin bir parçası olarak
sabit değerleri kullanarak da bozabiliriz. Bunu yapmak, diğer alanları bozmak için değişkenler
oluştururken bazı alanları belirli değerler için test etmemize izin verir.

Kod Listesi 19-14'te, `Point` değerlerini üç duruma ayıran bir `match` ifadesi var: doğrudan
`x` ekseninde bulunan noktalar (`y = 0` olduğunda doğru), `y` ekseninde (`x = 0`),
veya hiçbir ekseninde olmayan noktalar.

<Listing number="19-14" file-name="src/main.rs" caption="Destructuring and matching literal values in one pattern">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-14/src/main.rs:here}}
```

</Listing>

Birinci kol, `y` alanının değerinin `0` sabit değerine eşleşirse eşleşeceği belirterek
`x` ekseninde bulunan herhangi bir noktayla uyar. Desen yine bu kolun kodunda kullanabileceğimiz
bir `x` değişkeni oluşturur.

Benzer şekilde, ikinci kol `x` alanının değerinin `0` olduğunu belirterek `y` eksenindeki
herhangi bir noktayla uyar ve `y` alanının değeri için bir `y` değişkeni oluşturur. Üçüncü kol
herhangi bir sabit belirtmez, bu yüzden başka herhangi bir `Point`'la uyar ve hem `x` hem de
`y` alanları için değişkenler oluşturur.

Bu örnekte, `p` değeri `x`'in içinde `0` olması sayesinde ikinci kolla uyar, bu yüzden
bu kod `On the y axis at 7` yazdırır.

Bir `match` ifadesinin ilk eşleşen deseni bulduğu anda kolları kontrol etmeyi durdurduğunu
hatırlayın, bu yüzden `Point { x: 0, y: 0 }` `x` ekseninde ve `y` ekseninde olsa
da, bu kod sadece `On the x axis at 0` yazdırırdı.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-enums"></a>

#### Enum'lar (Enums)

Bu kitapta enum'ları bozduk (örneğin, Bölüm 6'daki Kod Listesi 6-5), ancak bir enum'u
bozan desenin enum içinde saklanan verinin tanımlandığı şekilde karşılı geldiğini henüz açıkça
tartışmadık. Bir örnek olarak, Kod Listesi 19-15'te, Kod Listesi 6-2'deki `Message`
enum'unu kullanıyoruz ve her bir iç değeri bozacak desenlerle bir `match` yazıyoruz.

<Listing number="19-15" file-name="src/main.rs" caption="Destructuring enum variants that hold different kinds of values">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-15/src/main.rs}}
```

</Listing>

Bu kod `Change color to red 0, green 160, and blue 255` yazdırır. `msg` değerini değiştirmeyi
deneyin ve diğer kollardan kodun çalıştığını görün.

Veri içermeyen enum varyantları için, `Message::Quit` gibi, değeri daha fazla bozamayız. Sadece
sabit `Message::Quit` değerine uyarız ve o desende hiçbir değişken yoktur.

Struct-benzeri enum varyantları için, `Message::Move` gibi, struct'lara uymak için belirttiğimiz
desene benzer bir desen kullanabiliriz. Varyant adından sonra, küme parantezleri yerleştiririz
ve sonra kodda kullanmak için parçaları ayıran alanları değişkenlerle listeleriz. Burada Kod
Listesi 19-13'te yaptığımız gibi kısa form kullanıyoruz.

Demet-benzeri enum varyantları için, bir öğeli içeren bir demeti tutan `Message::Write`
ve üç öğeli bir demeti tutan `Message::ChangeColor` gibi, desen demetlere uymak için belirttiğimiz
desene benzerdir. Desendeki değişken sayısı uymakta olduğumuz varyantın öğe sayısıyla
eşleşmelidir.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-nested-structs-and-enums"></a>

#### İç İçe Struct'lar ve Enum'lar (Nested Structs and Enums)

Şimdiye kadar, örneklerimiz bir derinlikte struct'lar veya enum'larla uyma vardı ancak eşleşme
içe ögelerde de çalışabilir! Örneğin, Kod Listesi 19-15'teki kodu `ChangeColor`
mesajındaki RGB ve HSV renkleri desteklemek için yeniden düzenleyebiliriz, Kod Listesi 19-16'da
gösterildiği gibi.

<Listing number="19-16" caption="Matching on nested enums">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-16/src/main.rs}}
```

</Listing>

`match` ifadesindeki birinci kolun deseni, `Color::Rgb` varyantını içeren bir
`Message::ChangeColor` enum varyantına uyar; sonra, desen üç iç `i32` değerine bağlanır.
İkinci kolun deseni de bir `Message::ChangeColor` enum varyantına uyar ancak iç enum
`Color::Hsv`'ye uyar. İki enum involved olmasına rağmen bu karmaşık koşulları tek bir
`match` ifadesinde belirtebiliriz.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-structs-and-tuples"></a>

#### Struct'lar ve Demetler (Structs and Tuples)

Desenleri daha karmaşık yollarla karıştırabilir, eşleştirebilir ve içe yerleştirebiliriz.
Aşağıdaki örnek, bir demet içinde struct'lar ve demetleri içe yerleştirdiğimiz ve tüm ilkel
değerleri bozdumuz karmaşık bir yapılandırma gösterir:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-05-destructuring-structs-and-tuples/src/main.rs:here}}
```

Bu kod bizi karmaşık türleri bileşen parçalarına ayırmamıza izin verir böylece ilgilenen
değerleri ayrı ayrı kullanabiliriz.

Desenlerle yapılandırma, bir struct'taki her bir alandan gelen değer gibi değer parçalarını birbirlerinden
ayrı ayrı kullanmak için uygun bir yoldur.

### Bir Desen İçinde Değerleri Yoksayma (Ignoring Values in a Pattern)

Bazen bir desende değerleri yoksaymanın kullanışlı olduğunu gördünüz, örneğin bir
`match`'in son kolunda, kalan tüm olası değerleri hesap eden ama aslında hiçbir şey yapmayan
bir her şeyi yakalayan almak için. Bir desende tüm değerleri veya değer parçalarını yoksamak için
bazı yollar var: `_` desenini kullanmak (bun gördünüz), başka bir desen içinde `_`
desenini kullanmak, alt çizgi ile başlayan bir ad kullanmak veya `..`'yi kullanarak bir
değerin kalan parçalarını yoksamak. Bu desenlerin her birini nasıl ve neden kullanacağımızı
keşfedelim.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-an-entire-value-with-_"></a>

#### `_` ile Tüm Bir Değeri Yoksayma (An Entire Value with `_`)

Alt çizgiyi herhangi bir değere uyacak ama değere bağlanmayacak joker desen olarak kullandık.
Bu özellikle bir `match` ifadesinin son kolu olarak kullanışlıdır ancak fonksiyon parametreleri de dahil
herhangi bir desende kullanabiliriz, Kod Listesi 19-17'da gösterildiği gibi.

<Listing number="19-17" file-name="src/main.rs" caption="Using `_` in a function signature">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-17/src/main.rs}}
```

</Listing>

Bu kod ikinci argüman olarak geçirilen `3` değerini tamamen yoksayar ve `This code only uses
the y parameter: 4` yazdırır.

Çoğu durumda, belirli bir fonksiyon parametresine ihtiyacınız olmadığında, imzayı değiştirirsiniz
böylece kullanılmayan parametre dahil olmaz. Bir fonksiyon parametresini yoksamak,
örneğin bir trait uygularken belirli bir tür imzası gerektiğinde ancak uygulamanızdaki
fonksiyon gövdesinin parametrelerden birine ihtiyacı olmadığında gibi durumlarda özellikle
kullanışlı olabilir. Sonra kullanılmayan fonksiyon parametreleri hakkında derleyici uyarısı alırsınız,
bir ad kullansaydık alacağınız gibi.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-parts-of-a-value-with-a-nested-_"></a>

#### İçe `_` ile Bir Değerin Parçalarını Yoksayma (Parts of a Value with a Nested `_`)

Ayrıca, bir değerin sadece bir parçasını test etmek istediğimiz ancak ilgili kolda çalıştırmak
istediğimiz kodda kullanmadığımız diğer parçalar için alt çizgiyi başka bir desenin içinde de
kullanabiliriz. Kod Listesi 19-18 bir ayarın değerini yöneten kodu gösterir. İş gereksinimleri şudur:
kullanıcı mevcut bir ayarlama özelleştirmesini üzerine yazmaya izin verilmez ancak ayarlamayı
kaldırabilir ve şu an kaldırılmışsa ona bir değer verebilir.

<Listing number="19-18" caption="Using an underscore within patterns that match `Some` variants when we don't need to use the value inside `Some`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-18/src/main.rs:here}}
```

</Listing>

Bu kod `Can't overwrite an existing customized value` ve sonra `setting is Some(5)` yazdırır.
Birinci eşleşme kolunda, ne `Some` varyantlarının içindeki değerleri eşleştirmemize ne de
kullanmamıza ihtiyacımız yok ancak `setting_value` ve `new_setting_value`'nin `Some`
varyantları olduğun durum için test etmemiz gerekir. O durumda, `setting_value`'yı değiştirmeme
nedenini yazdırır ve değişmez.

Diğer tüm durumlarda (eğer `setting_value` veya `new_setting_value`'den biri `None` ise) ikinci
koldaki `_` deseniyle ifade edildiğimiz gibi, `new_setting_value`'nin `setting_value` olmasına izin
vermek istiyoruz.

Ayrıca bir desen içinde birden çok yeri alt çizgiler kullanarak belirli değerleri yoksayabiliriz.
Kod Listesi 19-19, beş öğeli bir demetteki ikinci ve dördüncü değerleri yoksayan bir örnek
gösterir.

<Listing number="19-19" caption="Ignoring multiple parts of a tuple">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-19/src/main.rs:here}}
```

</Listing>

Bu kod `Some numbers: 2, 8, 32` yazdırır ve `4` ve `16` değerleri yoksayılacak.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-an-unused-variable-by-starting-its-name-with-_"></a>

#### Değişkenin Adını `_` ile Başlatarak Kullanılmamış Bir Değişken Yoksayma (An Unused Variable by Starting Its Name with `_`)

Bir değişken oluşturursanız ancak onu hiçbir yerde kullanmazsanız, Rust genellikle bir uyarı
çıkarır çünkü kullanılmayan bir değişken bir bug olabilir. Ancak, bazen kullanamayacağınız bir
değişken oluşturabilmek kullanışlı olabilir, örneğin bir prototip oluşturduğunuzda veya bir projeye
yeni başladığınızda. Bu durumda, değişkenin adını alt çizgiyle başlatarak Rust'a kullanılmayan
değişken hakkında sizi uyarırmamasını söyleyebilirsiniz. Kod Listesi 19-20'de, iki kullanılmayan
değişken oluşturuyoruz ancak bu kodu derlediğimizde onlardan sadece biri hakkında bir uyarı
almalıyız.

<Listing number="19-20" file-name="src/main.rs" caption="Starting a variable name with an underscore to avoid getting unused variable warnings">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-20/src/main.rs}}
```

</Listing>

Burada, `y` değişkenini kullanmama hakkında bir uyarı alıyoruz ancak `_x`'i kullanmama hakkında
bir uyarı almıyoruz.

Sadece `_` kullanmak ve alt çizgi ile başlayan bir ad kullanmak arasında ince bir fark olduğunu
not edin. `_x` sözdizimi yine değeri değişkene bağlar oysa `_` hiçbir şekilde
bağlamaz. Bu ayrımın önem verdiği bir durumu göstermek için, Kod Listesi 19-21 bize bir hata
verecektir.

<Listing number="19-21" caption="An unused variable starting with an underscore still binds the value, which might take ownership of the value.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-21/src/main.rs:here}}
```

</Listing>

Bir hata alacağız çünkü `s` değeri hala `_s`'ye taşınacak, bu bizden `s`'yi tekrar
kullanmamızı engeller. Ancak, tek başına alt çizgi kullanmak değere asla bağlanmaz. Kod Listesi
19-22 hiçbir hatasız derlenecektir çünkü `s` `_`'ye taşınmaz.

<Listing number="19-22" caption="Using an underscore does not bind the value.">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-22/src/main.rs:here}}
```

</Listing>

Bu kod harika çalışır çünkü `s`'yi hiçbir şeye bağlamayız; taşınmaz.

<a id="ignoring-remaining-parts-of-a-value-with-"></a>

#### `..` ile Bir Değerin Kalan Parçalarını Yoksayma (Remaining Parts of a Value with `..`)

Çok fazla parçası olan değerlerde, belirli parçaları kullanmak ve geri kalanı yoksamak için
`..` sözdizimini kullanabiliriz, her yoksayılan değer için alt çizgiler listelemek zorunluluğundan
kaçınarak. `..` deseni, desenin geri kalanında açıkça eşleşmediğimiz bir değerin herhangi
parçasını yoksayar. Kod Listesi 19-23'te, üç boyutlu alanda bir koordinat tutan bir `Point`
struct'ı var. `match` ifadesinde, sadece `x` koordinatıyla işlem yapmak istiyoruz ve `y` ve
`z` alanlarındaki değerleri yoksamak istiyoruz.

<Listing number="19-23" caption="Ignoring all fields of a `Point` except for `x` by using `..`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-23/src/main.rs:here}}
```

</Listing>

`x` değerini listeleriz ve sonra sadece `..` desenini dahil ederiz. Bu, `y: _` ve `z: _`
listelemekten daha hızlıdır, özellikle sadece bir veya iki alanın ilgili olduğu durumlarda çok fazla
alanı olan struct'larla çalışırken.

`..` sözdizimi ihtiyaç duyduğu kadar çok değere genişleyecek. Kod Listesi 19-24 bir demetle
`..`'yi nasıl kullanacağınızı gösterir.

<Listing number="19-24" file-name="src/main.rs" caption="Matching only the first and last values in a tuple and ignoring all other values">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-24/src/main.rs}}
```

</Listing>

Bu kodda, ilk ve son değerler `first` ve `last` ile eşleşir. `..` ortadaki her şeyi
eşleşir ve yoksayar.

Ancak, `..`'yi kullanmak belirsiz olmalıdır. Hangi değerlerin eşleşmek için amaçlandığını
ve hangisinin yoksayılması gerektiği belirsizse, Rust bize bir hata verecektir. Kod Listesi
19-25, `..`'yi belirsiz bir şekilde kullanmayı gösterir, bu yüzden derlenmez.

<Listing number="19-25" file-name="src/main.rs" caption="An attempt to use `..` in an ambiguous way">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-25/src/main.rs}}
```

</Listing>

Bu örneği derlediğimizde şu hatayı alacağız:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-25/output.txt}}
```

Rust için, demette kaç değeri yoksayacağını belirlemek için `second` ile bir değeri eşleştirmeden önce
ve sonra kaç tane daha değeri bundan sonra yoksayacağını belirlemek imkansızdır. Bu kod,
`2`'yi yoksamak, `second`'i `4`'e bağlamak ve sonra `8`, `16` ve `32`'yi yoksamak
istediğimizi; veya `2` ve `4`'ü yoksamak, `second`'i `8`'e bağlamak ve sonra `16`
ve `32`'yi yoksamak istediğimizi; vb. anlamına gelebilir. `second` değişken adı Rust için
özel bir şey demez, bu yüzden `..`'yi iki yeri bu gibi kullanmak belirsiz olduğu için bir
derleyici hatası alıyoruz.

<!-- Old headings. Do not remove or links may break. -->

<a id="extra-conditionals-with-match-guards"></a>

### Eşleşme Gardileri ile Koşullu Eklemek (Adding Conditionals with Match Guards)

Bir _eşleşme gardiyi (match guard)_ bir `match` kolunda desenden sonra belirtilen ek bir `if`
koşulludur ve o kolun seçilmesi için de uymalıdır. Eşleşme gardileri, tek başına bir desenin
izin verdiğinden daha karmaşık fikirleri ifade etmek için kullanışlıdır. Ancak, not edin ki onlar
sadece `match` ifadelerinde kullanılabilir, `if let` veya `while let` ifadelerinde değil.

Koşul desende oluşturulan değişkenleri kullanabilir. Kod Listesi 19-26, birinci kolunun
`Some(x)` deseni olan ve ayrıca `if x % 2 == 0` eşleşme gardiyi olan (bu sayı çiftse `true`
olacak) bir `match` gösterir.

<Listing number="19-26" caption="Adding a match guard to a pattern">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-26/src/main.rs:here}}
```

</Listing>

Bu örnek `The number 4 is even` yazdırır. `num` birinci koldaki desene karşılaştırıldığında
uayar çünkü `Some(4)` `Some(x)`'e uyar. Sonra, eşleşme gardiyi `x`'in 2'ye bölümünden
kalanın 0'a eşit olup olmadığını kontrol eder ve öyle olduğu için, birinci kol seçilir.

Eğer `num` `Some(5)` olsaydı, birinci koldaki eşleşme gardiyi `false` olurdu çünkü 5'in
2'ye bölümünden kalan 1'dir ve bu 0'a eşit değildir. Rust ikinci kola geçerdi ki o uyar
çünkü ikinci kolun eşleşme gardiyi yoktur ve bu yüzden herhangi bir `Some` varyantına uyar.

`if x % 2 == 0` koşullunu bir desen içinde ifade etmenin yolu yoktur, bu yüzden eşleşme
gardiyi bize bu mantığı ifade etme yeteneği verir. Bu ek ifade gücün dezavantajı,
eşleşme gardiyi ifadeleri involved olduğunda derleyici tamlılığı (exhaustiveness) kontrol etmeye
çalışmaz.

Kod Listesi 19-11'i tartışırken, desen-gölgelenme sorunumuzu çözmek için eşleşme gardileri
kullanabileceğimizi söyledik. `match` ifadesinin içindeki desende dış `match`'teki değişkeni
kullanmak yerine yeni bir değişken getirdiğimizi hatırlayın. Bu yeni değişken dış değişkenin
değeriyle karşılaştırmamamızı engelledi. Kod Listesi 19-27 bu sorunu çözmek için bir
eşleşme gardiyi nasıl kullanabileceğimizi gösterir.

<Listing number="19-27" file-name="src/main.rs" caption="Using a match guard to test for equality with an outer variable">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-27/src/main.rs}}
```

</Listing>

Bu kod şimdi `Default case, x = Some(5)` yazdıracaktır. İkinci eşleşme koldaki desen dış `y`'yi
gölgelendirecek yeni bir `y` değişkeni getirmez, bu demektir ki dış `y`'yi eşleşme gardisinde
kullanabiliriz. Deseni `Some(y)` olarak belirlemek yerine, dış `y`'yi gölgelendirecekti,
`Some(n)` belirtiyoruz. Bu `match`'in dışında `n` değişkeni olmadığı için hiçbir şeyi gölgelendirmeyen
yeni bir `n` değişkeni oluşturur.

`if n == y` eşleşme gardiyi bir desen değildir ve bu yüzden yeni değişkenler getirmez.
Bu `y` _is_ yeni bir `y`'yi gölgelendiren dış `y`'dir ve `n`'yi `y` ile karşılaştırarak
dış `y` ile aynı değere sahip bir değeri arayabiliriz.

Ayrıca bir eşleşme gardisinde _veya_ operatörü `|` kullanarak çoklu desenler belirtebilirsiniz;
eşleşme gardiyi koşullu tüm desenlere uygulanır. Kod Listesi 19-28, `|` kullanan bir
desenle bir eşleşme gardiyi birleştirmekteki önceliği gösterir. Bu örneğin önemli kısmı
`if y` eşleşme gardiyi `4`, `5`, _ve_ `6`'ya uygulanır, sadece `6`'ya uygulanıyor gibi
görünmesine rağmen.

<Listing number="19-28" caption="Combining multiple patterns with a match guard">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-28/src/main.rs:here}}
```

</Listing>

Eşleşme koşulu, o kolun sadece `x`'in değeri `4`, `5` veya `6`'ya eşit _ve_ `y` `true`
olduğunda uyacağını belirtir. Bu kod çalıştığında, birinci kolun deseni uyar çünkü `x`
`4`'tür ancak eşleşme gardiyi `if y` `false`'tur, bu yüzden birinci kol seçilmez. Kod ikinci
kola geçer ki uyar ve bu program `no` yazdırır. Sebep şudur ki `if` koşullu sadece son
değer `6`'ya değil, `|` operatörünü kullanarak belirtilen değer listesindeki tüm desene uygulanır.
Diğer bir deyişle, bir eşleşme gardiyinin bir desene ilişkili önceliği şöyle davranır:

```text
(4 | 5 | 6) if y => ...
```

bunun yerine:

```text
4 | 5 | (6 if y) => ...
```

Kodu çalıştırdıktan sonra, öncelik davranışı açıktır: Eğer eşleşme gardiyi `|` operatörünü
kullanarak belirtilen değer listesindeki sadece son değere uygulanırsa, kol uyardı ve program
`yes` yazdırırdı.

<!-- Old headings. Do not remove or links may break. -->

<a id="-bindings"></a>

### `@` Bağlamalarını Kullanma (Using `@` Bindings)

_at_ operatörü `@` bize bir desende desen eşleşmesi test ederken aynı anda o değeri tutan
bir değişken oluşturmamıza izin verir. Kod Listesi 19-29'te, bir `Message::Hello` `id`
alanının `3..=7` aralığında olduğunu test etmek istiyoruz. Ayrıca o değeri kolla ilişkili
kodda kullanmak için `id` değişkenine bağlamak istiyoruz.

<Listing number="19-29" caption="Using `@` to bind to a value in a pattern while also testing it">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-29/src/main.rs:here}}
```

</Listing>

Bu örnek `Found an id in range: 5` yazdıracaktır. `3..=7` aralığından önce `id @`
belirterek, aralıkla eşleşen herhangi bir değeri `id` adlı bir değişkende yakalıyoruz aynı
anda o değerin aralık desene uyduğunu test ediyoruz.

İkinci kolunda, desende sadece bir aralık belirlediğimizde, kolla ilişkili kodun `id`
alanının gerçek değerini içeren bir değişkeni yoktur. `id` alanının değeri 10, 11 veya 12
olabilirdi ancak bu desenle giden kod hangisi olduğunu bilmez. Desen kodu `id` alanından
değeri kullanamaz çünkü `id` değerini bir değişkende kaydetmedik.

Son kolda, aralıksız bir değişken belirlediğimizde, kolun kodunda kullanmak için değeri `id`
adlı bir değişkendedir. Sebep, struct alan kısa yol sözdizimini kullandık. Ancak ilk iki
kolda yaptığımız gibi bu kolda `id` alanındaki değere herhangi bir test uygulamadık: Herhangi bir
değer bu desene uyardı.

`@` kullanmak bize bir desende bir değeri test edip onu bir değişkende kaydetmemize izin verir.

## Özet (Summary)

Rust'un desenleri farklı türde verileri ayırt etmek için çok kullanışlıdır. `match` ifadelerinde
kullanıldığında, Rust desenlerinizin her olası değeri kapsadığını veya programınızın derlenmeyeceğini
sağlar. `let` ifadelerindeki ve fonksiyon parametrelerindeki desenler bu yapıları daha kullanışlı
yapar, değerleri daha küçük parçalara ayırmayı ve bu parçaları değişkenlere atamayı sağlar.
İhtiyacımıza uygun basit veya karmaşık desenler oluşturabiliriz.

Sıradan, kitabın sondan ikinci bölümünde, Rust'un çeşitli özelliklerinin bazı gelişmiş
yanlarına bakacağız.