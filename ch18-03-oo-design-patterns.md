## Bir Nesne Yönelimli Tasarım Kalıbını Uygulamak (Implementing an Object-Oriented Design Pattern)

_Durum kalıbı_ (state pattern) bir nesne yönelimli tasarım kalıbıdır. Kalıbın
özü, bir değerin dahili olarak sahip olabileceği bir durumlar setini tanımlamaktır.
Durumlar bir _durum nesneleri_ setiyle temsil edilir ve değerin davranışı
durumuna göre değişir. Durum nesneleri setinden biri olan bir durum nesnesi içeren
bir alanı olan bir blog gönderisi struct'ı üzerinden çalışan bir örnekte çalışacağız:
"taslak," "incele" veya "yayınlan."

Durum nesneleri işlevselliği paylaşır: Rust'ta, tabii ki, nesneler ve miras
yerine struct'lar ve trait'ler kullanacağız. Her durum nesnesi kendi davranışından ve ne
zaman başka bir duruma değişmesi gerektiğini yönetmekten sorumludur. Bir durum
nesnesi tutan değer, durumların farklı davranışından veya durumlar arasında geçiş yapılacağı
zamanından hiçbir şey bilmez.

Durum kalıbını kullanmanın avantajı, programın iş gereksinimleri değiştiğinde,
durum tutan değerin kodunu veya değeri kullanan kodunu değiştirmeye gerekmemememizdir.
Sadece durum nesnelerinden birinin içindeki kodu, kurallarını değiştirmek veya belki
daha fazla durum nesnesi eklemek için güncellememiz gerekecek.

Önce, durum kalıbını daha geleneksel nesne yönelimli bir şekilde uygulayacağız.
Sonra, Rust'ta biraz daha doğal olan bir yaklaşımı kullanacağız. Durum kalıbını kullanarak
artımlı bir blog gönderisi iş akışı uygulamaya dalalım.

Son işlevselli şöyle görünecek:

1. Bir blog gönderisi boş bir taslak olarak başlar.
1. Taslak bittiğinde, gönderinin incelenmesi istenir.
1. Gönderi onaylandığında, yayınlanır.
1. Sadece yayınlanmış blog gönderileri yazdırmak için içerik döner böylece onaylanmamış
   gönderiler kazara yayınlanmaz.

Gönderi üzerinde denenen diğer herhangi bir değişiklik etkisi olmamalıdır. Örneğin,
incelemesi istenmeden önce bir taslak blog gönderisi onaylamaya çalışırsak, gönderi
yayınlanmamış bir taslak olarak kalmalıdır.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-traditional-object-oriented-attempt"></a>

### Geleneksel Nesne Yönelimli Tarzı Denemek (Attempting Traditional Object-Oriented Style)

Aynı sorunu çözmek için kodu yapılandırmak için sonsuz yollar var, her biri
farklı değiş tokuşlarla. Bu bölümün uygulaması daha çok geleneksel nesne yönelimli
tarzdır, bu Rust'ta yazılmak mümkündür ancak Rust'ın bazı güçlerinden
yararlanmaz. Sonra, hala nesne yönelimli tasarım kalıbını kullanan ama diğer dillerde
nesne yönelimli deneyimi olan programcılar için daha az tanıdık görünebilecek şekilde
yapılandırılmış farklı bir çözüm göstereceğiz. İki çözümü karşılaştırarak farklı
dillerde kodundan Rust kodunu tasarlamanın değiş tokuşlarını deneyimleyeceğiz.

Kod Listesi 18-11, bu iş akışını kod formunda gösterir: Bu, `blog` adında bir
kütüphane crate'inde uygulayacağımız API'nin bir örnek kullanımıdır. Bu henüz
derlemeyecek çünkü `blog` crate'ini uygulamadık.

<Listing number="18-11" file-name="src/main.rs" caption="Code that demonstrates desired behavior we want our `blog` crate to have">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:all}}
```

</Listing>

Kullanıcının `Post::new` ile yeni bir taslak blog gönderisi oluşturmasını izin vermek istiyoruz.
Blog gönderisine metin eklemesini izin vermek istiyoruz. Eğer onaylanmadan önce
gönderinin içeriğini almaya çalışırsak, gönderi hala bir taslak olduğu için hiçbir metin
almamalıyız. Gösterme amaçlı kodda `assert_eq!` ekledik. Bunun için mükemmel bir
birim testi, taslak blog gönderisinin `content` yönteminden boş bir dize döndürmesini
iddia etmek olurdu, ancak bu örnek için test yazmayacağız.

Sıradan, gönderinin incelenmesi için bir isteği etkinleştirmek istiyoruz ve içeriğin,
inceleme için beklerken boş bir dize dönmesini istiyoruz. Gönderi onay aldığında,
yayınlanmalıdır, bu da `content` çağırdığında gönderinin metni döneceği anlamına gelir.

Sadece crate'inden etkileştiğimiz tipin `Post` tipi olduğunu fark edin. Bu tip durum
kalıbını kullanacak ve bir gönderinin olabileceği çeşitli durumları—taslak, incele veya
yayınlan—temsil eden üç durum nesnesinden birini tutacak bir değer tutacak. Bir durumdan
başka birine geçiş, `Post` tipi içinde dahili olarak yönetilecek. Durumlar, `Post`
örneği üzerinde kütüphanemizin kullanıcıları tarafından çağılan yöntemlere yanıt olarak
değişecek ancak onlar durum değişikliklerini doğrudan yönetmek zorunda değil. Ayrıca,
kullanıcılar durumlarla hata yapamaz, örneğin gönderiyi incelemeden önce yayınlamaz.

<!-- Old headings. Do not remove or links may break. -->

<a id="defining-post-and-creating-a-new-instance-in-the-draft-state"></a>

#### `Post`'ü Tanımlamak ve Yeni Bir Örnek Oluşturmak (Defining `Post` and Creating a New Instance)

Kütüphanenin uygulamasına başlayalım! Bazı içeriği tutan bir genel `Post` struct'ı
gerektiğini biliyoruz, bu yüzden Kod Listesi 18-12'de gösterildiği gibi struct'ın
tanımını ve bir `Post` örneği oluşturmak için ilişkilmiş genel bir `new` fonksiyonuyla
başlayacağız. Ayrıca, bir `Post` için tüm durum nesnelerinin sahip olması gereken
davranışı tanımlayan özel bir `State` trait'ini yapacağız.

Sonra, `Post` durum nesnesini bir `Option<T>` içinde bir `Box<dyn State>` trait
nesnesi olarak `state` adında özel bir alanda tutacak. `Option<T>`'ın neden gerekli olduğunu
birazdan göreceksin.

<Listing number="18-12" file-name="src/lib.rs" caption="Definition of a `Post` struct and a `new` function that creates a new `Post` instance, a `State` trait, and a `Draft` struct">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-12/src/lib.rs}}
```

</Listing>

`State` trait'i, farklı gönderi durumları tarafından paylaşılan davranışı tanımlar. Durum
nesneleri `Draft`, `PendingReview` ve `Published`'dır ve tümü `State` trait'ini uygulayacak.
Şimdilik trait'in hiçbir yöntemi yok ve sadece `Draft` durumunu tanımlayarak başlayacağız çünkü
bu bir gönderinin başlamak istediğimiz durumdur.

Yeni bir `Post` oluşturduğumuzda, onun `state` alanını bir `Box` tutan bir `Some`
değere ayarlıyoruz. Bu `Box`, `Draft` struct'ının yeni bir örneğine işaret eder.
Bu, yeni bir `Post` örneği oluşturduğumuzda, onun her zaman bir taslak olarak başlamasını
sağlar. `Post`'un `state` alanı özel olduğu için, başka herhangi bir durumda `Post` oluşturmanın
bir yolu yoktur! `Post::new` fonksiyonunda, `content` alanını yeni, boş bir `String`'e
ayarlıyoruz.

#### Gönderi İçeriğinin Metnini Saklamak (Storing the Text of the Post Content)

Kod Listesi 18-11'de gördüğümüz gibi, `add_text` adında bir yöntem çağırmayı ve ona
blog gönderisinin metin içeriği olarak eklenecek bir `&str` geçirmek istiyoruz. Bunu bir
yöntem olarak uyguluyoruz, `content` alanını `pub` olarak dışa açmak yerine, bu yüzden
sonra `content` alanının verisinin nasıl okunacağını kontrol eden bir yöntem uygulayabiliriz.
`add_text` yöntemi oldukça basittir, bu yüzden Kod Listesi 18-13'te `impl Post` bloğuna
uygulamayı ekleyelim.

<Listing number="18-13" file-name="src/lib.rs" caption="Implementing `add_text` method to add text to a post's `content`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-13/src/lib.rs:here}}
```

</Listing>

`add_text` yöntemi, `self`'e değiştirilebilir bir referans alır çünkü
`add_text`'i çağırdığımız `Post` örneğini değiştiriyoruz. Sonra `content`'teki
`String` üzerinde `push_str` çağıyoruz ve `text` argümanını saklanan `content`'e eklemek
için geçiriyoruz. Bu davranış, gönderinin içinde olduğu durumdan bağımsızdır, bu yüzden
durum kalıbının bir parçası değildir. `add_text` yöntemi `state` alanıyla hiç
etkileşmez ama desteklemek istediğimiz davranışın bir parçasıdır.

<!-- Old headings. Do not remove or links may break. -->

<a id="ensuring-the-content-of-a-draft-post-is-empty"></a>

#### Taslak Gönderinin İçeriğinin Boş Olduğundan Emin Olmak (Ensuring That the Content of a Draft Post Is Empty)

`add_text` çağırdıktan ve gönderimize bazı içerik ekledikten sonra bile, hala
`content` yönteminin boş bir dize dilimi döndürmesini istiyoruz çünkü gönderi
hala taslak durumundadır, bu da Kod Listesi 18-11'deki ilk `assert_eq!` ile gösterilmiştir.
Şimdilik, bu gereksinimi karşılayacak en basit şeyi ile `content` yöntemini uygulayalım:
her zaman boş bir dize dilimi dönmek. Sonra, gönderinin durumunu değiştirebilme
yetenekliği uyguladıktan sonra bunu değiştireceğiz. Şimdiye kadar, gönderiler sadece
taslak durumunda olabilir, bu yüzden gönderi içeriği her zaman boş olmalıdır. Kod Listesi 18-14
bu yer tutucu uygulamasını gösterir.

<Listing number="18-14" file-name="src/lib.rs" caption="Adding a placeholder implementation for `content` method on `Post` that always returns an empty string slice">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-14/src/lib.rs:here}}
```

</Listing>

Bu eklenen `content` yöntemi ile, Kod Listesi 18-11'deki her şey ilk `assert_eq!`'e
kadar amaçlandığı gibi çalışır.

<!-- Old headings. Do not remove or links may break. -->

<a id="requesting-a-review-of-the-post-changes-its-state"></a>
<a id="requesting-a-review-changes-the-posts-state"></a>

#### Bir İncelemesi İstemek, Gönderinin Durumunu Değiştirir (Requesting a Review, Which Changes to Post's State)

Sıradan, gönderinin incelenmesi için işlevselli eklememiz gerekiyor, bu da gönderinin
durumunu `Draft`'ten `PendingReview`'e değiştirmelidir. Kod Listesi 18-15 bu kodu gösterir.

<Listing number="18-15" file-name="src/lib.rs" caption="Implementing `request_review` methods on `Post` and `State` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-15/src/lib.rs:here}}
```

</Listing>

`Post`'a `request_review` adında genel bir yöntem veriyoruz ki `self`'e değiştirilebilir
bir referans alacak. Sonra, `Post`'un şu anki durum üzerinde dahili bir `request_review`
yöntemini çağıyoruz ve bu ikinci `request_review` yöntemi şu anki durumu tüketir ve
yeni bir durum döner.

`request_review` yöntemini `State` trait'ine ekliyoruz; trait'i uygulayan tüm tipler
şimdi `request_review` yöntemini uygulamak zorundadır. Dikkat edin ki yöntemin ilk
parametresi olarak `self`, `&self` veya `&mut self` yerine, `self: Box<Self>` var.
Bu sözdizimi, yöntemin yalnızca bir `Box` tutan tip üzerinde çağıldığında geçerli olduğu
anlamına gelir. Bu sözdizimi `Box<Self>`'in sahipliğini alır, eski durumu geçersiz kılar
böylece `Post`'un durumu değeri yeni bir duruma dönüştürebilir.

Eski durumu tüketmek için, `request_review` yöntemi durum değerinin sahipliğini almak zorundadır.
İşte `Post`'un `state` alanındaki `Option`'ın işe geldiği yer: `state` alanından
`Some` değerini almak ve yerine `None` bırakmak için `take` yöntemini çağırıyoruz çünkü
Rust bize struct'larda yerleşmemiş alanlara izin vermez. Bu bize `state` değerini `Post`'tan
ödünce almamızı sağlar, onu ödünce almamızı sağlar. Sonra, gönderinin `state`
değerini bu işlemin sonucuna ayarlayacağız.

`state` değerinin sahipliğini almak için doğrudan `self.state = self.state.request_review();`
gibi kodla ayarlamak yerine, `state`'i geçici olarak `None`'a ayarlamız gerekiyor.
Bu, `Post`'un, onu yeni bir duruma dönüştürdükten sonra eski `state` değerini
kullanamamasını sağlar.

`Draft` üzerindeki `request_review` yöntemi, gönderi bir inceleme beklediğinde temsil eden yeni
`PendingReview` struct'ının yeni, kutulanmış bir örneğini döner. `PendingReview` struct'ı
da `request_review` yöntemini uygular ama hiçbir dönüşüm yapmaz. Bunun yerine, kendisini döner
çünkü zaten `PendingReview` durumundaki bir gönderide inceleme isteğinde bulunduğumuzda,
`PendingReview` durumunda kalmalıdır.

Şimdi durum kalıbının avantajlarını görmeye başlayabiliriz: `request_review` yöntemi
`Post` üzerinde, onun `state` değeri ne olursa olsun aynıdır. Her durum kendi kurallarından
sorumludur.

`Post` üzerindeki `content` yöntemini boş bir dize dilimi dönen olarak bırakacağız. Artık
hem `PendingReview` durumunda hem de `Draft` durumunda bir `Post`'a sahip olabiliriz ama
`PendingReview` durumunda aynı davranışı istiyoruz. Kod Listesi 18-11 şimdi ikinci
`assert_eq!` çağırısına kadar çalışır!

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-the-approve-method-that-changes-the-behavior-of-content"></a>
<a id="adding-approve-to-change-the-behavior-of-content"></a>

#### `content`'in Davranışını Değiştirmek için `approve` Eklemek (Adding `approve` to Change `content`'s Behavior)

`approve` yöntemi `request_review` yöntemine benzer olacak: O, Kod Listesi 18-16'da
gösterildiği gibi, durum onaylandığında şu anki durumun ne olması gerektiğini söylediği
değerine `state`'i ayarlayacak.

<Listing number="18-16" file-name="src/lib.rs" caption="Implementing `approve` method on `Post` and `State` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-16/src/lib.rs:here}}
```

</Listing>

`approve` yöntemini `State` trait'ine ekliyoruz ve `State`'i uygulayan yeni bir struct
ekliyoruz, `Published` durumu.

`PendingReview` üzerindeki `request_review`'in çalıştığına benzer şekilde, bir `Draft` üzerinde
`approve` yöntemini çağırsak, hiçbir etkisi olmayacak çünkü `approve` `self`'i döner.
`PendingReview` üzerinde `approve` çağırdığımızda, `Published` struct'ının yeni, kutulanmış bir
örneğini döner. `Published` struct'ı `State` trait'ini uygular ve hem `request_review`
yöntemi için hem de `approve` yöntemi için, kendisini döner çünkü bu durumlarda gönderi
`Published` durumunda kalmalıdır.

Şimdi `Post` üzerindeki `content` yöntemini güncellememiz gerekiyor. `content`'den
dönen değerin `Post`'un şu anki durumuna bağımlı olmasını istiyoruz, bu yüzden
Kod Listesi 18-17'de gösterildiği gibi `Post`'u kendi `state` üzerinde tanımlanan bir
`content` yöntemine temsilci yapacağız.

<Listing number="18-17" file-name="src/lib.rs" caption="Updating `content` method on `Post` to delegate to a `content` method on `State`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-17/src/lib.rs:here}}
```

</Listing>

Çünkü amaç, `State`'i uygulayan struct'larda tüm bu kuralları tutmaktır, `state`
içindeki değer üzerinde bir `content` yöntemi çağıyoruz ve gönderi örneğini (yani `self`) bir
argüman olarak geçiriyoruz. Sonra, `state` değerindeki `content` yöntemini kullanarak
dönen değeri döneriyoruz.

`Option` üzerinde `as_ref` yöntemini çağıyoruz çünkü `Option` içindeki değere referans
istemiyoruz, değerin sahipliğini değil. `state` bir `Option<Box<dyn State>>` olduğu için,
`as_ref` çağırdığımızda, bir `Option<&Box<dyn State>>` döner. `as_ref` çağırmazsak
bir hata alırdık çünkü fonksiyon parametresinin ödünçeli `&self`'inden `state`'i
hareket edemeyiz.

Sonra `unwrap` yöntemini çağıyoruz, bunun asla panik olmayacağını biliyoruz çünkü
`Post` üzerindeki yöntemlerin, bu yöntemler bittiğinde `state`'in her zaman bir `Some`
değeri tutacağından emin olduklarını biliyoruz. Bu, Bölüm 9'un [“Siz Rust Derleyicisinden
Daha Fazla Bilgiye Sahipseniz”][more-info-than-rustc]<!-- ignore --> bölümünde
konuştuğumuz, bir `None` değerinin asla mümkün olmadığını bildiğimiz ama derleyicinin
bunu anlayamadığı durumlardan biridir.

Bu noktada, `&Box<dyn State>` üzerinde `content` çağırdığımızda, referans çözme otomatik
olarak `&` ve `Box` üzerinde etkidecek böylece `content` yöntemi sonunda `State` trait'ini
uygalayan tip üzerinde çağılanacak. Bu demek ki `State` trait tanımına `content` eklememiz gerekiyor
ve bu da hangi durumda olduğumuza göre hangi içeriği döndürme mantığını koyacağımız
yerdir, bu da Kod Listesi 18-18'de gösterilmiştir.

<Listing number="18-18" file-name="src/lib.rs" caption="Adding `content` method to the `State` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-18/src/lib.rs:here}}
```

</Listing>

`content` yöntemi için boş bir dize dilimi dönen varsayılan bir uygulama ekliyoruz. Bu
demek ki `Draft` ve `PendingReview` struct'larında `content` uygulamamız gerekmüyor. `Published`
struct'ı `content` yöntemini geçersiz kılacak ve `post.content` içindeki değeri dönecek. Kolay
olmasına rağmen, `State` üzerindeki `content` yönteminin `Post`'un içeriğini belirlemesi,
`State`'in sorumluluğu ile `Post`'un sorumluluğu arasındaki çizgileri flulaştırıyor.

Bu yöntemde yaşam zamanı eklemelerinin gerekli olduğunu fark edin, Bölüm 10'da konuştuğumuz
gibi. Bir `post`'e referans bir argüman olarak alıyoruz ve o `post`'un bir parçasına
referans döneriyoruz, bu yüzden dönen referansın yaşam zamanı `post` argümanının yaşam
zamanıyla ilişkilidir.

Ve bittiğiz—Kod Listesi 18-11'deki her şey şimdi çalışır! Blog gönderisi iş akışının
kurallarıyla durum kalıbını uyguladık. Kurallarla ilişkili mantık durum nesnelerinde yaşıyor
değil, `Post` boyunca dağılmış.

> ### Neden Bir Enum Değil? (Why Not An Enum?)
> Farklı olası gönderi durumlarını varyantlar olarak içeren bir enum kullanmadığımızı
> merak etmiş olabilirsiniz. Bu kesinlikle mümkün bir çözümdür; bunu deneyin ve
> sonuçları karşılaştırın hangisini tercih ettiğinizi görmek için! Bir enum kullanmanın bir
> dezavantajı şudur ki enum değerini kontrol eden her yer bir `match` ifadesi veya benzer
> her olası varyantı ele almak için gerektir. Bu trait nesnesi çözümünden daha fazla
> tekrarlı olabilir.

<!-- Old headings. Do not remove or links may break. -->

<a id="trade-offs-of-the-state-pattern"></a>

#### Durum Kalıbını Değerlendirmek (Evaluating the State Pattern)

Rust'ın, bir gönderinin her durumunda sahip olması gereken farklı tür davranışı kapsüllemek için
nesne yönelimli durum kalıbını uygulama kabiliyetini gösterdik. `Post` üzerindeki yöntemler
çeşitli davranışlardan hiçbir şey bilmez. Kodumuzu düzenlediğimiz yol nedeniyle,
yayınlanmış bir gönderinin nasıl davranabileceğinin farklı yollarını bilmek için tek bir yere
bakmamız gerekiyor: `Published` struct'ında `State` trait'inin uygulaması.

Eğer durum kalıbını kullanmayan alternatif bir uygulama oluşturmasaydık, yerine `Post`
üzerindeki yöntemlerde veya hatta gönderinin durumunu kontrol eden `main` kodunda `match`
ifadeleri kullanabilirdik ve bu yerlerde davranışı değiştirirdik. Bu demek olurdı ki
yayınlanmış durumunda olan bir gönderinin tüm etkilerini anlamak için birkaç yere bakmak
zorunda kalırdık.

Durum kalıbıyla, `Post` yöntemleri ve `Post` kullandığımız yerlerde `match`
ifadelerine gerek yoktur ve yeni bir durum eklemek için, tek bir struct eklememiz ve tek bir
yerde o struct üzerinde trait yöntemlerini uygulamamız gerekecekti.

Durum kalıbını kullanan uygulama daha fazla işlevselli eklemek için kolaydır. Durum
kalıbını kullanan kodu koruma kolaylığını görmek için, bu önerilerin bazılarını deneyin:

- Gönderinin durumunu `PendingReview`'den tekrar `Draft`'e değiştiren bir `reject` yöntemi
  ekleyin.
- Durum `Published`'e değiştirilemeden önce `approve`'a iki çağrı gerektir.
- Kullanıcıların metin içeriğini sadece gönderi `Draft` durumundayken eklemesini izin verin.
  İpucu: durum nesnesini, içeriğin neyin değişebileceğinden sorumlu tutun ama
  `Post`'u değiştirmekten sorumlu tutmayın.

Durum kalıbının bir dezavantajı şudur ki, çünkü durumlar durumlar arasındaki geçişleri
uygularlar, bazı durumlar birbirine koplmıştır. `PendingReview` ile `Published` arasında
başka bir durum, örneğin `Scheduled`, eklerseniz, `PendingReview` içindeki kodu `Published`
yerine geçmek için `Scheduled`'e geçecek şekilde değiştirmemiz gerekecektir. `PendingReview`'ın yeni bir
durum ekleme ile değişmesinin gerek olmaması daha az iş olurdu ama bu başka bir tasarım
kalıbına geçmeyi gerektirirdi.

Başka bir dezavantaj, bazı mantığı tekrarladığımızdır. Bazı tekrarı ortadan kaldırmak için,
`State` trait'inde `request_review` ve `approve` yöntemleri için `self` dönen varsayılan
uygulamalar yapmayı deneyebiliriz. Ancak, bu çalışmaz: `State` bir trait nesnesi olarak
kullanıldığında, trait somut `self`'in tam olarak ne olacağını bilmez, bu yüzden dönüş
tipi derleme zamanında bilinmez. (Bu daha önce bahsedilen dyn uyumluluk kurallarından biridir.)

Başka tekrar, `Post` üzerindeki `request_review` ve `approve` yöntemlerinin benzer
uygulamalarını içerir. Her iki yöntem `Post`'un `state` alanı ile `Option::take` kullanır
ve `state` `Some` ise, sarılmış değerin aynı yönteminin uygulamasına temsilci yapar ve
`state` alanının yeni değerini sonucuna ayarlar. `Post` üzerinde bu kalıbı izleyen çok fazla
yöntemimiz olsaydı, tekrarı ortadan kaldırmak için bir makro tanımlamayı düşünebilirdik
(bakın Bölüm 20'deki [“Makrolar”][macros]<!-- ignore --> bölümü).

Durum kalıbını tam olarak nesne yönelimli diller için tanımlandığı gibi uygulayarak,
Rust'ın güçlerinden tam olarak faydalanmıyoruz. `blog` crate'ine yapabileceğimiz bazı
değişikliklere bakalım ki geçersiz durumları ve geçişleri derleme zamanı hatalarına
dönebilir.

### Durumları ve Davranışı Türler Olarak Kodlamak (Encoding States and Behavior as Types)

Durum kalıbını farklı bir değiş tokuşlar seti almak için nasıl yeniden düşünmemiz gerektiğini
göstereceğiz. Dış kodun onlardan hiçbir şey bilmemesini tamamen kapsüllemek yerine,
durumları farklı türlere kodlayacağız. Sonuç olarak, Rust'ın tip kontrol sistemi, sadece
yayınlanmış gönderilerin izin verildiği yerlerde taslak gönderiler kullanılmaya çalışılmayı bir
derleyici hatası vererek engelleyecektir.

Kod Listesi 18-11'deki `main`'in ilk kısmını düşünelim:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:here}}
```

</Listing>

Hala `Post::new` kullanarak taslak durumunda yeni gönderiler oluşturmayı ve gönderinin içeriğine
metin eklemeyi etkinleştiriyoruz. Ancak taslak bir gönderide boş bir dize dönen bir `content`
yöntemi bulundurmak yerine, taslak gönderilerin `content` yöntemi hiç olmayacak şekilde yapacağız.
Bu şekilde, taslak gönderinin içeriğini almaya çalışırsak, yöntem bulunmadığını söyleyen bir derleyici
hatası alacağız. Sonuç olarak, üretimde kazara taslak gönderi içeriği görüntelemeyiz çünkü bu
kod bile derlenmeyecek. Kod Listesi 18-19, bir `Post` struct'ının ve bir `DraftPost` struct'ının
tanımını, ve her biri üzerindeki yöntemleri gösterir.

<Listing number="18-19" file-name="src/lib.rs" caption="A `Post` with a `content` method and a `DraftPost` without a `content` method">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-19/src/lib.rs}}
```

</Listing>

Hem `Post` hem de `DraftPost` struct'ları blog gönderisi metnini saklayan özel bir `content`
alanına sahiptir. Struct'lar artık `state` alanına sahip değiller çünkü durumun
kodlamasını struct'ların türlerine taşıyoruz. `Post` struct'ı yayınlanmış bir gönderiyi
temsil eder ve içeriği dönen bir `content` yöntemi vardır.

Hala bir `Post::new` fonksiyonumuz var ama `Post` örneği dönmek yerine, bir
`DraftPost` örneği döner. `content` özel olduğu için ve `Post` dönen hiçbir fonksiyon yok,
şu anda bir `Post` örneği oluşturmak mümkün değildir.

`DraftPost` struct'ının bir `add_text` yöntemi vardır, böylece daha önce olduğu gibi `content`'e
metin ekleyebiliriz ama `DraftPost`'ın tanımlanmış bir `content` yöntemi olmadığına dikkat edin!
Şimdi program tüm gönderilerin taslak gönderiler olarak başlamasını sağlar ve taslak gönderilerinin
içeriğini görünteleme için mevcut değil. Bu kısıtlamalar etrafından geçme denemesi derleyici
hatasına yol açacak.

<!-- Old headings. Do not remove or links may break. -->

<a id="implementing-transitions-as-transformations-into-different-types"></a>

Peki, nasıl yayınlanmış bir gönderi elde edeceğiz? Bir taslak gönderinin yayınlamadan önce
incelenip onaylanması gerektiği kuralını uygulamak istiyoruz. Bekleyen inceleme durumundaki bir
gönderi hala hiçbir içeriği göstermemeli. Kod Listesi 18-20'de gösterildiği gibi bu
kısıtlamaları uygulamak için başka bir struct ekleyeceğiz, `PendingReviewPost`, `DraftPost` üzerinde
`request_review` yöntemini tanımlayarak `PendingReviewPost` dönen ve bir `Post` dönen `approve`
yöntemini `PendingReviewPost` üzerinde tanımlayarak.

<Listing number="18-20" file-name="src/lib.rs" caption="A `PendingReviewPost` that gets created by calling `request_review` on `DraftPost` and an `approve` method that turns a `PendingReviewPost` into a published `Post`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-20/src/lib.rs:here}}
```

</Listing>

`request_review` ve `approve` yöntemleri `self`'in sahipliğini alır, böylece
`DraftPost` ve `PendingReviewPost` örneklerini tüketir ve bunları sırasıyla bir `PendingReviewPost`
ve yayınlanmış bir `Post`'a dönüştürür. Bu şekilde, üzerlerinde `request_review` çağırdıktan sonra
asılı kalan `DraftPost` örnekleri olmayacak, ve böyle devam. `PendingReviewPost` struct'ının
üzerinde tanımlanmış bir `content` yöntemi yoktur, bu yüzden onun içeriğini okumaya çalışırmak
derleyici hatasına yol açar, `DraftPost` gibi. İçeriği okuyabilecek bir yöntem tanımlı yayınlanmış
bir `Post` örneğini elde etmenin tek yolu bir `PendingReviewPost` üzerinde `approve` yöntemini
çağırmak ve bir `PendingReviewPost` elde etmenin tek yolu bir `DraftPost` üzerinde
`request_review` yöntemini çağırmaktır, blog gönderisi iş akışını artık tip sistemine
kodladık.

Ancak `main` üzerinde bazı küçük değişiklikler de yapmamız gerekiyor. `request_review` ve
`approve` yöntemleri çağıldıkları yapılandırmak yerine yeni örnekler döner, bu yüzden dönen
örnekleri kaydetmek için daha fazla `let post =` gölgeli atamaları eklememiz gerekiyor.
Ayrıca taslak ve bekleyen inceleme gönderilerinin içeriklerinin boş dizeler olduğu iddiaları da
olamaz ve buna da ihtiyacımız yok: Bu durumlardaki gönderilerin içeriğini kullanmaya çalışan kodu
derleyemeyiz artık. `main`'deki güncellenmiş kod Kod Listesi 18-21'de gösterilir.

<Listing number="18-21" file-name="src/main.rs" caption="Modifications to `main` to use the new implementation of blog post workflow">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-21/src/main.rs}}
```

</Listing>

`post`'i yeniden atamak için `main` üzerinde yapmamız gereken değişiklikler, bu uygulamanın
artık tamamen nesne yönelimli durum kalıbını takip etmediği anlamına gelir: Durumlar arasındaki
dönüşümler artık `Post` uygulaması içinde tamamen kapsüllenmemiştir. Ancak kazancımız, tip sistemi ve
derleme zamanında oluşan tip kontrollemesi sayesinde geçersiz durumların artık mümkün olmamasıdır!
Bu, üretimde yayınlamamış bir gönderinin içeriğinin görüntelenmesi gibi belirli hataların,
üretim gitmeden önce keşfedileceğini sağlar.

Kod Listesi 18-21'den sonraki `blog` crate'i üzerinde bu bölümün başında önerilen
görevleri deneyin ve bu kod sürümünün tasarımının ne düşündüğünüzü görün. Bazı görevlerin bu
tasarımda zaten tamamlanmış olabileceğine dikkat edin.

Rust'ın nesne yönelimli tasarım kalıplarını uygulama kabiliyeti olsa bile, durumunu tip
sistemine kodlamak gibi başka kalıplar da Rust'ta mevcuttur. Bu kalıpların farklı değiş tokuşları
vardır. Nesne yönelimli kalıplarla çok aşina olabilirsiniz ancak, Rust'ın özelliklerinden
yararlanmak için problemi yeniden düşünmek, derleme zamanında bazı hataları önleme gibi faydalar
sağlayabilir. Nesne yönelimli kalıplar, sahiplik gibi bazı özellikler nedeniyle, Rust'ta her
zaman en iyi çözüm olmayabilir.

## Özet (Summary)

Bu bölümü okuduktan sonra Rust'ın nesne yönelimli bir dil olduğunu düşünüp düşünmediğiniz,
trait nesnelerini kullanarak Rust'ta bazı nesne yönelimli özellikler alabileceğinizi şimdi
biliyorsunuz. Dinamik dispatch, kodunuza çalışma zamanı performansına karşılık biraz
esneklik sağlayabilir. Bu esnekliği, kodunuzun bakımı kolaylaştıracak nesne yönelimli
kalıpları uygulamak için kullanabilirsiniz. Rust'ta ayrıca sahiplik gibi, nesne yönelimli dillerde
olmayan başka özellikler de vardır. Bir nesne yönelimli kalıp, Rust'ın güçlerinden yararlanmanın en
iyi yolu her zaman olmayabilir ama mevcut bir seçenektir.

Sıradan, desenlere (patterns) bakacağız, bu da çok fazla esneklik sağlayan Rust'ın başka bir
özelliğidir. Kitap boyunca kısaca onlara baktık ancak tam kapasitelerini henüz görmedik.
Gidelim!

[more-info-than-rustc]: ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler
[macros]: ch20-05-macros.html#macros