<!-- Old headings. Do not remove or links may break. -->

<a id="the-match-control-flow-operator"></a>

## `match` Akış Kontrol Yapısı (The `match` Control Flow Construct)

Rust'ta bir değeri desenler serisiyle karşılaştırarak ve sonra hangi desen eşleşirse kod çalıştıran `match` adında aşırı güçlü bir akış kontrol yapısı vardır. Desenler değişmez değerlerden, değişken isimlerden, joker karakterlerden (wildcards) ve çok başka şeylerden oluşabilir; [Bölüm 19][ch19-00-patterns]<!-- ignore --> tüm desen türlerini ve ne yaptıklarını kapsar. `match`'in gücü desenlerin ifadeciliğinden ve derleyicinin tüm olası vakaları işlediğini onayladığı gerçeğinden gelir.

Bir `match` ifadesini, yanında değişik delikli boşlukları olan bir madeni sıralama makinesi gibi düşün: Paralar yanında çeşitli boyutlarda delikler ve her para ilk içine girdiği deliğe düşer. Aynı şekilde, değerler `match`'teki her desenden geçer ve ilk desende değerin "sığdığı" anında, değer yürütme sırasında kullanılacak ilişkili kod bloğuna düşer.

Paralardan konuşurken, onları `match` kullanarak bir örnek olarak kullanalım! Bilinmeyen bir ABD parasını alan ve benzer bir şekilde sayma makinesi gibi hangi para olduğunu belirleyip değerini sent olarak dönen bir fonksiyon yazabiliriz, Kod Listesi 6-3'te gösterildiği gibi.

<Listing number="6-3" caption="Varyantlarını desen olarak kullanan bir enum ve bir `match` ifadesi">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-03/src/main.rs:here}}
```

</Listing>

`value_in_cents` fonksiyonundaki `match`'i parçalayıp inceleyelim. İlk, `coin` değeri olan bir ifadeyi ardından gelen `match` anahtar kelimesini listeleriz. Bu `if` ile kullanılan koşul ifadeye çok benzer görünebilir ancak büyük bir fark var: `if` ile koşulun bir Boolean değere değerlendirmesi gerekir ancak burada herhangi bir tip olabilir. Bu örnekte `coin`'un tipi ilk satırda tanımladığımız `Coin` enum'idir.

Sonra `match` kolları vardır. Bir kolun iki parçası vardır: bir desen ve bazı kod. Buradaki ilk kolun `Coin::Penny` değeri olan bir deseni ve ardından deseni ve çalışacak kodu ayıran `=>` operatörü vardır. Bu durumda kod sadece `1` değeridir. Her kol birbirinden virgülle ayrılır.

`match` ifadesi çalıştığında, sırada değeri her kolun deseniyle karşılaştırır. Bir desen değere eşleşirse, o desenle ilişkili kod çalıştırılır. O desen değere eşleşmezse, yürütme sonraki kola devam eder, madeni sıralama makinesindeki çok benzer şekilde. İstediğimiz kadar çok kola sahip olabiliriz: Kod Listesi 6-3'te, `match`'imiz dört kolu vardır.

Her kolun ilişkili kod bir ifadedir ve eşleşen koldaki ifadenin değeri tüm `match` ifadesi için dönecek değerdir.

Match kolu kodu kısa ise genellikle kıvramalı parantez kullanmayız, bu Kod Listesi 6-3'teki gibidir ki her kolun sadece bir değer döner. Eğer bir match kolunda birden fazla kod satırı çalıştırmak isterseniz, kıvramalı parantezler kullanmalısınız ve koldan sonra gelen virgül isteğe bağlıdır. Örneğin, aşağıdaki kod yöntem `Coin::Penny` ile her çağırdığında "Şanslı penny!" yazdırır ancak bloğun son değeri olan `1`'i hala döner:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-08-match-arm-multiple-lines/src/main.rs:here}}
```

### Değerlere Bağlanan Desenler (Patterns That Bind to Values)

Match kollarının başka bir kullanışlı özelliği vardır ki eşleşen desenin değerlerin parçalarına bağlanabilirler. Bu, enum varyantlarından değerleri nasıl çıkarabileceğimizdir.

Bir örnek olarak, enum varyantlarından birini veri içerenini değiştirelim. 1999'den 2008'e kadar, Amerika Birleşik Devletleri bir tarafındaki 50 eyaletin her biri için farklı tasarılarla çeyrek basılmıştır. Başka paralar eyaleti tasarıları almamıştır, bu yüzden sadece çeyrekler bu ekstra değere sahiptir. `Quarter` varyantını içinde saklanan bir `UsState` değerini içermesi için `enum`'imize bu bilgiyi ekleyebiliriz ki bunu Kod Listesi 6-4'te yaptık.

<Listing number="6-4" caption="`Quarter` varyantının ayrıca bir `UsState` değeri tuttuğu bir `Coin` enum">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-04/src/main.rs:here}}
```

</Listing>

Bir arkadaşımızın tüm 50 eyalet çeyreklerini toplamaya çalıştığını düşünün. Paraları paralar türüne göre sıralarken, ayrıca her çeyrekle ilişkili eyaleti adını çağıralım böylece arkadaşımızın sahip olmadığı biriyse, koleksiyonlarına ekleyebilirler.

Bu kodun match ifadesinde, `Coin::Quarter` varyantının değerleriyle eşleşen desene `state` adında bir değişken ekliyoruz. Bir `Coin::Quarter` eşleştiğinde, `state` değişkeni o çeyreğin durum değerine bağlanacak. Sonra, o kolun kodunda `state`'i kullanabiliriz, şöyle:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-09-variable-in-pattern/src/main.rs:here}}
```

Eğer `value_in_cents(Coin::Quarter(UsState::Alaska))`'ni çağırsaydık, `coin` `Coin::Quarter(UsState::Alaska)` olacaktı. Bu değeri her match kolununla karşılaştırdığımızda hiçbiri eşleşmez `Coin::Quarter(state)`'e ulaşıncaya kadar. O noktada, `state` için bağlama `UsState::Alaska` değeri olacak. Sonra `println!` ifadesinde o bağlamayı kullanabiliriz, böylece `Quarter` için `Coin` enum varyantının iç durum değerini çıkarabiliriz.

<!-- Old headings. Do not remove or links may break. -->

<a id="matching-with-optiont"></a>

### `Option<T>` `match` Deseni (The `Option<T>` `match` Pattern)

Önceki bölümde, `Option<T>` kullanırken `Some` durumundan iç `T` değerini çıkarmak istemiştik; `Coin` enum ile yaptığımız gibi `Option<T>`'yi de `match` kullanarak işleyebiliriz ancak `match` ifadesinin çalışma şekli aynı kalır.

`Option<i32>` alan ve içinde bir değer varsa, ona 1 ekleyen bir fonksiyon yazmak istediğimizi söyleyelim. İçinde bir değer yoksa, fonksiyon `None` değeri dönmeli ve herhangi bir işlem yapmaya çalışmamalıdır.

Bu fonksiyon `match` sayesinde çok kolay yazılır ve Kod Listesi 6-5 gibi görünecektir.

<Listing number="6-5" caption="Bir `Option<i32>` üzerinde `match` ifadesi kullanan bir fonksiyon">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:here}}
```

</Listing>

`plus_one`'in ilk yürütmesini daha ayrıntılı inceleyelim. `plus_one(five)` çağırdığımızda, `plus_one`'un gövdesindeki `x` değişkeni `Some(5)` değerine sahip olacak. Sonra bunu her match koluyla karşılaştırırız:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

`Some(5)` değeri `None` deseniyle eşleşmez, bu yüzden sonraki kola devam ederiz:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:second_arm}}
```

`Some(5)` `Some(i)` ile eşleşir mi? Eşleşir! Aynı varyantımız var. `i`, `Some` içinde tutulan değere bağlanır bu yüzden `i` `5` değerini alır. Sonra match koldaki kod çalıştırılır böylece `i`'nin değerine 1 ekleriz ve `6` toplamımızla yeni bir `Some` değeri oluşturur.

Şimdi Kod Listesi 6-5'teki `plus_one`'in ikinci çağrısını düşünelim burada `x` `None`'dur. `match`'e gireriz ve ilk kolla karşılaştırırız:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

Eşleşir! Eklenecek bir değer yoktur, bu yüzden program durur ve `=>`'nin sağ tarafındaki `None` değerini döner. Çünkü ilk kol eşleşti, başka kollar karşılaştırılmaz.

`match` ve enum'leri birleştirmek çok durumlarda kullanışlıdır. Rust kodunda bu deseni çok sık göreceksiniz: Bir enum ile `match` yap, veriye bağlan ve sonra buna dayalı kod çalıştır. İlk başta biraz zor gibi ama onunla alıştığınızda onu tüm dillerde olduğunu dileyeceksiniz. Sürekli kullanıcı favorisidir.

### Match'ler Tamdır (Matches Are Exhaustive)

`match`'in konuşmamız gereken başka bir yönü vardır: Kolların desenleri tüm olasılıkları kapsamalıdır. `plus_one` fonksiyonunun bu sürümünü düşünün, bir hatası vardır ve derlenmez:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/src/main.rs:here}}
```

`None` durumunu işlemedik bu yüzden bu kod bir hataya neden olacak. Şanslıyız ki bu bir hatayı Rust yakalamayı nasıl bildiği. Eğer bu kodu derlemeye çalışırsak, şu hatayı alırız:

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/output.txt}}
```

Rust her olası vakayı kapsamadığımızı ve hangi deseni unuttuğumuzu bile biliyor! Rust'ta match'ler _tamdır_ (exhaustive): Kodun geçerli olması için en son olasılıkayı tüketmeliyiz. Özellikle `Option<T>` durumunda, Rust'ın açıkça `None` durumunu işlemeyi unutmamızı önlediğinde, değerimiz olduğunda varsaymakten korur bu yüzden daha önce konuşulan milyar dolarlık hatasını imkansız kılar.

### Tümünü Yakalayan Desenler ve `_` Yer Tutucusu (Catch-All Patterns and `_` Placeholder)

Enum'leri kullanarak, bazı belirli değerler için özel eylemler alabiliriz ama tüm diğer değerler için varsayılan bir eylem alabiliriz. Bir oyun uyguladığınızı düşünün eğer bir zar atışında 3 atarsanız, oyuncunuz hareket etmez ama süslü yeni bir şapka alır. Eğer 7 atarsanız, oyuncunuz süslü şapkasını kaybeder. Tüm diğer değerler için, oyuncunuz oyun tahtasındaki yer sayısını o kadar hareket eder. İşte bu mantığı uygulayan bir `match`, zar atışının sonucu sabit kodlanmış rastgele bir değer yerine ve tüm diğer mantık gövdesi olmayan fonksiyonlarle temsil edilmiş çünkü onları uygulamak bu örneğin kapsamı dışında:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-15-binding-catchall/src/main.rs:here}}
```

İlk iki kolun desenleri `3` ve `7` değişmez değerlerdir. Herhangi bir başka olası değeri kaplayan son kolun deseni, `other` adında seçtiğimiz bir değişkendir. `other` kolu için çalışan kod bu değişkeni `move_player` fonksiyonuna geçerek kullanır.

Bu kod derler, `u8`'in sahip olabileceği tüm olası değerleri listelememiş olmamıza rağmen çünkü son desen belirli listelenmeyen tüm değerleri eşleşecek. Bu tümünü yakalayan desen `match`'in tam olması gereksinimini karşılar. Tümünü yakalayan kolu son koymamız gerektiğini fark edin çünkü desenler sırada değerlendirilir. Tümünü yakalayan kolu daha erken koysaydık, diğer kollar asla çalışmaz bu yüzden tümünü yakalayan bir koldan sonra kollar eklerseniz Rust bizi uyarabilir!

Rust'ta ayrıca tümünü yakalamak istiyorsak ama tümünü yakalayan desende değeri kullanmak istemediğimizde kullanabileceğimiz özel bir desen vardır: `_` herhangi bir değeri eşleşen ve o değere bağlanmayan özel bir desendir. Bu Rust'a değeri kullanmayacağımızı söyler bu yüzden Rust kullanılmayan bir değişken konusunda bizi uyarmaz.

Oyun kurallarını değiştirelim: Şimdi, 3 veya 7 dışında bir şey atarsanız, tekrar atmanız gerekiyor. Tümünü yakalayan değere ihtiyaçımız yok bu yüzden kodumuzu `other` adında bir değişken yerine `_` kullanmak için değiştirebiliriz:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-16-underscore-catchall/src/main.rs:here}}
```

Bu örnek de tamdızlık gereksinimini karşılar çünkü son kolda tüm diğer değerleri açıkça yok sayıyoruz; hiçbir şey unuttuk.

Son olarak, oyun kurallarını tekrar değiştirelim böylece 3 veya 7 dışında bir şey atarsanız turunuzda başka hiçbir şey olmaz. Bunu `_` koluyla giden kod olarak birim değeri (["Tüple Tipi" (The Tuple Type)][tuples]<!-- ignore --> bölümünde bahsettiğimiz boş tüple tipi) kullanarak ifade edebiliriz:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-17-underscore-unit/src/main.rs:here}}
```

Burada, Rust'a açıkça herhangi bir başka değeri kullanmayacağımızı ve daha erken bir kolda eşleşmeyen bir desenle kod çalıştırmak istemediğimizi söylüyoruz.

Desenler ve eşleştirme hakkında konuşacak daha fazla var ki bunları [Bölüm 19'da][ch19-00-patterns]<!-- ignore --> kapsamlıyacağız. Şu an için, `match` ifadesi biraz uzun sözcüklü olan durumlarda kullanışlı olabilen `if let` sözdizimine geçiyoruz.

[tuples]: ch03-02-data-types.html#the-tuple-type
[ch19-00-patterns]: ch19-00-patterns.html