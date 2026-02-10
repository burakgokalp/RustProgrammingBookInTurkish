## Yapıları Tanımlama ve Örneklendirme (Defining and Instantiating Structs)

Yapılar, ["Tüple Tipi" (The Tuple Type)][tuples]<!-- ignore --> bölümünde konuşulan tiplelerle benzerdir çünkü ikisi de birden fazla ilgili değeri tutarlar. Tüleler gibi, bir yapının parçaları farklı tipler olabilir. Tülelerden farklı olarak, bir yapıda her veri parçasını adlandırırız böylece ne anlama geldiği açıktır. Bu isimleri eklemek, yapıların tülelerden daha esnek olduğunu gösterir: Bir örneğin değerlerini belirtmek veya erişmek için verinin sırasına güvenmek zorunda değilsiniz.

Bir yapı tanımlamak için, `struct` anahtar kelimesini giriyoruz ve tüm yapıyı adlandırıyoruz. Bir yapının adı, birlikte gruplanan veri parçalarının önemini tanımlamalıdır. Sonra, kıvramalı parantezlerin içinde, _alanlar_ (fields) adını verdiğimiz veri parçalarının isimlerini ve tiplerini tanımlıyoruz. Örneğin, Kod Listesi 5-1 bir kullanıcı hesabı hakkında bilgi tutan bir yapı gösterir.

<Listing number="5-1" file-name="src/main.rs" caption="Bir `User` yapı tanımı">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-01/src/main.rs:here}}
```

</Listing>

Bir yapı tanımladıktan sonra kullanmak için, her alan için somut değerler belirterek o yapının bir _örneğini_ (instance) oluşturuyoruz. Bir örnek oluşturarak, yapının adını belirtiyoruz ve sonra _`key: value`_ çiftlerini içeren kıvramalı parantezler ekliyoruz, burada anahtarlar alanların isimleridir ve değerler bu alanlarda saklamak istediğimiz veridir. Alanları yapıda bildirdiğimiz aynı sırada belirtmek zorunda değiliz. Başka bir deyimle, yapı tanımı tip için genel bir şablondur ve örnekler o şablonu belirli verilerle doldurur o tipin değerlerini oluşturur. Örneğin, Kod Listesi 5-2'de gösterildiği gibi belirli bir kullanıcı bildirebiliriz.

<Listing number="5-2" file-name="src/main.rs" caption="Bir `User` yapısının bir örneğini oluşturma">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-02/src/main.rs:here}}
```

</Listing>

Bir yapıdan belirli bir değer almak için, nokta notasyonu kullanıyoruz. Örneğin, bu kullanıcının e-posta adresine erişmek için, `user1.email` kullanıyoruz. Eğer örnek değiştirilebilirse (mutable), nokta notasyonu kullanarak ve belirli bir alana atayarak bir değeri değiştirebiliriz. Kod Listesi 5-3 değiştirilebilir bir `User` örneğinin `email` alanındaki değeri nasıl değiştireceğini gösterir.

<Listing number="5-3" file-name="src/main.rs" caption="Bir `User` örneğinin `email` alanındaki değeri değiştirme">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-03/src/main.rs:here}}
```

</Listing>

Tüm örneğin değiştirilebilir olması gerektiğini fark edin; Rust bize sadece belirli alanları değiştirilebilir olarak işaretlemeye izin vermez. Herhangi bir ifadeyle olduğu gibi, yapı örneğini fonksiyon gövdesinin son ifadesi olarak oluşturabiliriz ve o yeni örneği örtük olarak dönebiliriz.

Kod Listesi 5-4 verilen e-posta ve kullanıcı adıyla bir `User` örneği dönen bir `build_user` fonksiyonu gösterir. `active` alanı `true` değerini alır ve `sign_in_count` `1` değerini alır.

<Listing number="5-4" file-name="src/main.rs" caption="Bir e-posta ve kullanıcı adı alan ve bir `User` örneği dönen bir `build_user` fonksiyonu">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-04/src/main.rs:here}}
```

</Listing>

Fonksiyon parametrelerini yapı alanlarıyla aynı isimle adlandırmak mantıklıdır ancak `email` ve `username` alan isimlerini ve değişkenleri tekrarlamak biraz can sıkıcıdır. Eğer yapı daha fazla alana sahipse, her ismi tekrarlamak daha can sıkıcı olur. Şanslıyız ki, uygun bir kısaltma var!

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-field-init-shorthand-when-variables-and-fields-have-the-same-name"></a>

### Alan Başlatma Kısaltmasını Kullanma (Using Field Init Shorthand)

Çünkü Kod Listesi 5-4'te fonksiyon parametre adları ve yapı alan adları tam olarak aynıdır, `build_user`'ü _alan başlatma kısaltması_ (field init shorthand) sözdizimi kullanarak yeniden yazabiliriz böylece tam olarak aynı davranır ancak `username` ve `email` tekrarı yok, Kod Listesi 5-5'te gösterildiği gibi.

<Listing number="5-5" file-name="src/main.rs" caption="`username` ve `email` parametreleri yapı alanlarıyla aynı ismi olduğu için alan başlatma kısaltması kullanan bir `build_user` fonksiyonu">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-05/src/main.rs:here}}
```

</Listing>

Burada, `User` yapısının yeni bir örneğini oluşturuyoruz ki `email` adında bir alanı var. `email` alanının değerini `build_user` fonksiyonunun `email` parametresindeki değere ayarlamak istiyoruz. Çünkü `email` alanı ve `email` parametresi aynı isme sahip, sadece `email: email` yerine `email` yazmamız gerekiyor.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-instances-from-other-instances-with-struct-update-syntax"></a>

### Yapı Güncelleme Sözdizimi ile Örnekler Oluşturma (Creating Instances with Struct Update Syntax)

Genellikle bir yapının yeni bir örneğini oluşturmak kullanışlıdır ki aynı tipin başka bir örneğinden çoğu değerini içerir ancak bazılarını değiştirir. Bu yapı güncelleme sözdizimi kullanarak yapabilirsiniz.

İlk olarak, Kod Listesi 5-6'da güncelleme sözdizimi olmadan düzenli yolda `user2` içinde yeni bir `User` örneği nasıl oluşturulacağını gösteriyoruz. `email` için yeni bir değer ayarlıyoruz ancak aksi takdirde Kod Listesi 5-2'de oluşturduğumuz `user1`'den aynı değerleri kullanıyoruz.

<Listing number="5-6" file-name="src/main.rs" caption="`user1`'den tüm değerleri kullanarak ancak bir değer kullanmadan yeni bir `User` örneği oluşturma">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-06/src/main.rs:here}}
```

</Listing>

Yapı güncelleme sözdizimini kullanarak, Kod Listesi 5-7'de gösterildiği gibi daha az kodla aynı etkiyi başarabiliriz. `..` sözdizimi açıkça ayarlanmamış kalan alanların verilen örnekteki alanlarla aynı değere sahip olması gerektiğini belirtir.

<Listing number="5-7" file-name="src/main.rs" caption="Bir `User` örneği için yeni `email` değeri ayarlamak ancak `user1`'den geri kalan değerleri kullanmak için yapı güncelleme sözdizimi kullanma">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-07/src/main.rs:here}}
```

</Listing>

Kod Listesi 5-7'deki kod da `user2`'de `email` için farklı bir değere sahip ancak `user1`'den `username`, `active` ve `sign_in_count` alanları için aynı değerlere sahip bir örnek oluşturur. `..user1`'nin sonda gelmesi gerekir ki kalan alanların `user1`'deki karşılık gelen alanlardan değerlerini alması gerektiğini belirtir ancak alanların tanımındaki sıraya bakılmaksızın istediğimiz kadar çok alan için değerleri belirtmeyi seçebiliriz.

Yapı güncelleme sözdiziminin atama gibi `=` kullandığını fark edin; bu veriyi taşıdığı içindir, ["Değişkenler ve Veri Taşıma ile Etkileşimi" (Variables and Data Interacting with Move)][move]<!-- ignore --> bölümünde gördüğümüz gibi. Bu örnekte, `user2`'yi oluşturduktan sonra `user1`'i daha fazla kullanamayız çünkü `user1`'nin `username` alanındaki `String` `user2`'ye taşındı. Eğer `user2`'ye hem `email` hem `username` için yeni `String` değerleri vermiş olsaydık ve böylece sadece `user1`'den `active` ve `sign_in_count` değerlerini kullanmış olsaydık, o zaman `user2`'yi oluşturduktan sonra `user1` hala geçerli olurdu. `active` ve `sign_in_count`'ün ikisi de `Copy` özelliğini uygulayan tiplerdir bu yüzden ["Sadece Yığın Verisi: Kopya" (Stack-Only Data: Copy)][copy]<!-- ignore --> bölümünde konuştuğumuz davranış uygulanır. Bu örnekte `user1.email`'i hala kullanabiliriz çünkü değeri `user1`'den çıkarılmadı.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-tuple-structs-without-named-fields-to-create-different-types"></a>

### Tüle Yapılarıyla Farklı Tipler Oluşturma (Creating Different Types with Tuple Structs)

Rust tülelere benzeyen yapıları da destekler, bunlara _tüle yapıları_ (tuple structs) denir. Tüle yapıları yapının adı ekstra anlam sağlar ancak alanlarıyla ilişkili isimleri yoktur; aksine, sadece alanların tipleri vardır. Tüle yapılar tüm tüle bir ad vermek ve tüle'i diğer tülelerden farklı bir tip yapmak istediğinizde ve alanları düzenli bir yapıda adlandırmak söz veya yinelenmiş olacağı zaman kullanışlıdır.

Bir tüle yapısı tanımlamak için, `struct` anahtar kelimesiyle başlayın ve tüledeki tiplerle takip eden yapı adı. Örneğin, burada `Color` ve `Point` adında iki tüle yapısı tanımlayıp kullanıyoruz:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-01-tuple-structs/src/main.rs}}
```

</Listing>

`black` ve `origin` değerlerinin farklı tipler olduğunu fark edin çünkü farklı tüle yapılarının örnekleri. Tanımladığınız her yapı kendi tipidir, yapı içindeki alanlar aynı tiplere sahip olsa bile. Örneğin, `Color` tipinde bir parametre alan bir fonksiyon bir `Point`'i argüman olarak alamaz, her iki tip de üç `i32` değerinden oluşsa bile. Aksi takdirde, tüle yapı örnekleri tülelere benzerdir çünkü onları bireysel parçalarına ayırabilirsiniz ve tek bir değere erişmek için `.` ardından dizin kullanabilirsiniz. Tülelerden farklı olarak, tüle yapıları onları ayırdığınızda yapının tipini adlandırmanızı gerektirir. Örneğin, `origin` noktasındaki değerleri `x`, `y` ve `z` adındaki değişkenlere ayırmak için `let Point(x, y, z) = origin;` yazardık.

<!-- Old headings. Do not remove or links may break. -->

<a id="unit-like-structs-without-any-fields"></a>

### Birim Benzeri Yapıları Tanımlama (Defining Unit-Like Structs)

Herhangi bir alanı olmayan yapıları da tanımlayabilirsiniz! Bunlara _birim benzeri yapılar_ (unit-like structs) denir çünkü ["Tüple Tipi" (The Tuple Type)][tuples]<!-- ignore --> bölümünde bahsettiğimiz `()` birim tipine benzer davranırlar. Birim benzeri yapılar, bir tipte bir özellik uygulamak istediğinizde ancak tip içinde saklamak istediğiniz herhangi bir veriniz olmadığında kullanışlı olabilir. Özellikleri Bölüm 10'da konuşacağız. `AlwaysEqual` adında bir birim yapısı bildirme ve örnekleme örneği burada:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-04-unit-like-structs/src/main.rs}}
```

</Listing>

`AlwaysEqual`'i tanımlamak için, `struct` anahtar kelimesini, istediğimiz ismi ve sonra bir noktalı virgül kullanıyoruz. Kıvramalı parantezler veya parantezler gerekmez! Sonra, `subject` değişkeninde `AlwaysEqual`'in bir örneğini benzer bir yolda alabiliriz: tanımladığımız ismi kullanarak, herhangi bir kıvramalı parantez veya parantez olmadan. Daha sonunda bu tip için davranış uygulayacağımızı düşünün ki `AlwaysEqual`'in her örneği herhangi bir başka tipin her örneğiyle her zaman eşit olacak, belki test amaçları için bilinen bir sonuçe sahip olmak için. Bu davranışı uygulamak için herhangi bir veriye ihtiyacımız olmayacak! Bölüm 10'da özellikleri nasıl tanımlayacağınızı ve herhangi bir tipde, birim benzeri yapılar dahil, uygulayacağınızı göreceksiniz.

> ### Yapı Verisinin Sahipliği
> Kod Listesi 5-1'deki `User` yapı tanımında, `&str` dize dilimi tipi yerine sahip olunan `String` tipini kullandık.
> Bu kasıtlı bir seçim çünkü bu yapının her örneğinin tüm verisine sahip olmasını ve o verinin tüm yapı geçerli olduğu sürece geçerli olmasını istiyoruz.
>
> Yapıların başka bir şeye sahip olan verilere başvurular saklaması da mümkündür, ancak bunu yapmak için _yaşam süreleri_ (lifetimes) kullanmak gerekir ki Bölüm 10'da konuşacağımız bir Rust özelliği. Yaşam süreleri bir yapı tarafından başvurulan verinin yapı geçerli olduğu sürece geçerli olmasını sağlar. Bir yapıya yaşam süresi belirtmeden bir başvuru saklamaya çalıştığınızı söyleyin, *src/main.rs* içindeki gibi; bu çalışmaz:
>
> <Listing file-name="src/main.rs">
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore,does_not_compile
> struct User {
>     active: bool,
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
> }
>
> fn main() {
>     let user1 = User {
>         active: true,
>         username: "someusername123",
>         email: "someone@example.com",
>         sign_in_count: 1,
>     };
> }
> ```
>
> </Listing>
>
> Derleyici yaşam süresi belirleyicilere ihtiyaç duyacağını şikayet edecek:
>
> ```console
> $ cargo run
>    Compiling structs v0.1.0 (file:///projects/structs)
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:3:15
>   |
> 3 |     username: &str,
>   |               ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 ~     username: &'a str,
>   |
>
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:4:12
>   |
> 4 |     email: &str,
>   |            ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 |     username: &str,
> 4 ~     email: &'a str,
>   |
>
> For more information about this error, try `rustc --explain E0106`.
> error: could not compile `structs` (bin "structs") due to 2 previous errors
> ```
>
> Bölüm 10'da, yapıları başvuruları nasıl saklayacağınızı bu hataları nasıl düzelteceğinizi konuşacağız ancak şu an için, `&str` gibi başvurular yerine `String` gibi sahip olunan tipler kullanarak bu gibi hataları düzelteceğiz.

<!-- manual-regeneration
for error above
after running update-rustc.sh:
pbcopy < listings/ch05-using-structs-to-structure-related-data/no-listing-02-reference-in-struct/output.txt
paste above
add `> ` before every line -->

[tuples]: ch03-02-data-types.html#the-tuple-type
[move]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move
[copy]: ch04-01-what-is-ownership.html#stack-only-data-copy