<!-- Old headings. Do not remove or links may break. -->

<a id="digging-into-the-traits-for-async"></a>

## Async İçin Özelliklere Daha Yakın Bir Bakış (A Closer Look at Traits for Async)

Bölüm boyunca, çeşitli yollarla `Future`, `Stream`, ve `StreamExt`
özelliklerini kullandık. Ancak şimdiye kadar, onların nasıl çalıştığını veya
birbirleri nasıl uyum sağladıkları hakkında çok fazla ayrıntıya girmekten kaçındık,
ki bu çoğu zaman günlük Rust işleriniz için yeterlidir. Ancak bazen, bu
özelliklerin daha fazla ayrıntını anlamanız gereken durumlarla karşılaşacaksınız,
bununla birlikte `Pin` türü ve `Unpin` özelliği. Bu bölümde, o senaryolar için
yardımcı olacak kadarın dalacağız, yine de diğer belgelendirme için
_gerçekten_ derin dalıştan kaçınarak.

<!-- Old headings. Do not remove or links may break. -->

<a id="future"></a>

### `Future` Özelliği (The `Future` Trait)

`Future` özelliğinin nasıl çalıştığına daha yakından bakalım. İşte
Rust'in bunu tanımladığı:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

Bu özellik tanımlaması çok yeni tür ve daha önce görmediğimiz bazı sözdizimleri
içeriyor bu yüzden tanımlaması parça parça geçelim.

İlk olarak, `Future`'nin ilişkili `Output` türü, future'nin neye çözüldüğünü söyler.
Bu, `Yineleyici` özelliği için `Item` ilişkili türüne benzerdir.
İkinci olarak, `Future`, özel bir `Pin` referans alan ve değiştirilebilir bir `Context`
türüne referans alan `poll` yöntemine sahiptir ve bir `Poll<Self::Output>` döner.
Birazdan `Pin` ve `Context` hakkında daha fazla konuşacağız. Şimdilik, yöntemin döndiği
olan `Poll` türüne odaklanalım:

```rust
pub enum Poll<T> {
    Ready(T),
    Pending,
}
```

Bu `Poll` türü bir `Option`'a benzerdir. Değeri olan bir varyanta sahip, `Ready(T)`,
ve olmayan bir varyanta sahip, `Pending`. Ancak `Poll`, `Option`'dan oldukça
farklı bir şeyi temsil eder! `Pending` varyantı, future'nin hala yapması gereken işi
olduğunu ve çağıranının daha sonra tekrar kontrol etmesi gerektiğini belirtir. `Ready`
varyantı, `Future`'in işini bitirdiğini ve `T` değerinin kullanılabilir olduğunu
belirtir.

> Not: `poll`'ü doğrudan çağırmak nadir gerekir ancak yapmanız gerekirse,
> şunu aklınızda tutun ki çoğu future ile, çağıran, future `Ready` döndikten sonra
> `poll`'ü tekrar çağırmamalıdır. Çoğu future hazır olduktan sonra tekrar polanırsa
> panikler. Tekrar polanması güvenli olan future'lar bunu belgilerinde açıkça söyleyecektir.
> Bu, `Yineleyici::next`'in nasıl davrandığına benzerdir.

`await` kullanan kodu gördüğünüzde, Rust bunu kapa altında `poll`
çağrılan koda derler. Kod Listesi 17-4'e geri bakarsanız, tek bir URL'nin sayfa
başlığını çözüldiğinde yazdırdık, Rust bunu şunun gibi bir şeye (tam olarak değil)
derler:

```rust,ignore
match page_title(url).poll() {
    Ready(page_title) => match page_title {
        Some(title) => println!("The title for {url} was {title}"),
        None => println!("{url} had no title"),
    }
    Pending => {
        // Ama buraya ne gelir?
    }
}
```

Future hala `Pending` olduğunda ne yapmalıyız? Tekrar ve tekrar ve tekrar denemek,
çalışmaktan başka bir yol lazım ki future nihayet hazır olana kadar. Başka bir deyişle,
bir döngüye ihtiyacımız var:

```rust,ignore
let mut page_title_fut = page_title(url);
loop {
    match page_title_fut.poll() {
        Ready(value) => match page_title {
            Some(title) => println!("The title for {url} was {title}"),
            None => println!("{url} had no title"),
        }
        Pending => {
            // devam et
        }
    }
}
```

Ancak Rust bunu tam o koda derlerse, her `await` engelleyecek olurdu—tam
istediğimizin tersi! Bunun yerine, Rust, o döngünün gelecekte başka bir şeye kontrolü
vereceğini sağlar ki bu future üzerindeki işi durdurabilir sonra diğer future'lerle
çalışabilir ve bu olanı daha sonra tekrar kontrol edebilir. Gördüğümüz gibi, bu
bir şey bir async runtime'dır ve bu zamanlama ve koordinasyon işi ana işlerinden biridir.

[“İki Görev Arasında Mesaj Geçirme ile Veri Gönderme”][message-passing]<!--
ignore --> bölümünde, `rx.recv` üzerinde beklediğimizi açıkladık. `recv` çağrısı bir
future döner ve future'yi beklemek bunu pollar. Runtime'ın, ya `Some(message)` ya
da kanal kapanrsa `None` ile future hazır olana kadar durdurduğunu not ettik.
`Future` özelliğine ve özellikle `Future::poll`'a daha derin anlayışımızla, bunun nasıl
çalıştığını görebiliriz. Runtime, `Poll::Pending` döndüğünde future'nin hazır olmadığını
bilir. Tersine, runtime, `Poll::Ready(Some(message))` veya `Poll::Ready(None)` döndüğünde
future'in _hazır_ olduğunu ve onu ilerletirir.

Bir runtime'ın bunu nasıl yaptığı tam ayrıntılar bu kitabın kapsamı dışındadır
ancak anahtar, future'ların temel mekaniklerini görmektir: bir runtime, sorumlu olduğu
her future'yi _pollar_ ve henüz hazır olmadığında future'yi uyumak için geri koyar.

<!-- Old headings. Do not remove or links may break. -->

<a id="pinning-and-the-pin-and-unpin-traits"></a>
<a id="the-pin-and-unpin-traits"></a>

### `Pin` Türü ve `Unpin` Özelliği (The `Pin` Type and `Unpin` Trait)

Kod Listesi 17-13'e geri dönerek, `trpl::join!` makrosunu üç future'yi
beklemek için kullandık. Ancak, çalışma zamanına kadar bilinmeyen bir miktar future içeren bir
vektör gibi bir koleksiyona sahip olmak yaygındır. Kod Listesi 17-13'ü, Kod Listesi 17-23'teki
koda değiştirelim ki üç future'yi bir vektör içine koyar ve bunun yerine
`trpl::join_all` fonksiyonunu çağırar ki henüz derlenmez.

<Listing number="17-23" caption="Awaiting futures in a collection" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-23/src/main.rs:here}}
```

</Listing>

Her future'yi bir `Box` içinde koyarak onları _özellik nesnelerine_ dönüştürüyoruz,
tıpkı Bölüm 12'de [“`run`'tan Hatalar Dönme”] bölümünde yaptığımız gibi.
(Bölüm 18'de özellik nesneleri ayrıntılı olarak kaplayacağız.) Özellik nesnelerini
kullanmak, bu türler tarafından üretilen anonim future'leri aynı tür olarak davranmamızı sağlar çünkü
onların hepsi `Future` özelliğini uygular.

Bu şaşırtıcı olabilir. Sonuçta, async bloklarının hiçbiri bir şey dönmüyor,
bu yüzden her biri `Future<Output = ()>` üretir. Ancak `Future` bir özelliktir hatırlayın
ve derleyici her async bloğu için benzersiz bir enum oluşturur, aynı çıktı türlerine
sahip olsalar bile. Tıpkı iki farklı elle yazılmış yapıyı bir `Vec` içine
koyamayacağınız gibi, derleyici tarafından oluşturulan enumları karıştıramazsınız.

Sonra future'leri koleksiyonunu `trpl::join_all` fonksiyonuna geçip sonucu
bekliyoruz. Ancak bu derlenmez; hata mesajlarının ilgili kısmı burada:

```text
error[E0277]: `dyn Future<Output = ()>` cannot be unpinned
  --> src/main.rs:48:33
   |
48 |         trpl::join_all(futures).await;
   |                                 ^^^^^^^ trait `Unpin` is not implemented for `dyn Future<Output = ()>`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<dyn Future<Output = ()>>` to implement `Future`
note: required by a bound in `futures_util::future::join_all::JoinAll`
  --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`
```

Bu hata mesajındaki not bize `pin!` makrosunu kullanmamızı ve değerleri
_pinlememiz_ gerektiğini söylüyor ki bu değerleri bellekte hareket etmeyeceğini garanti eden
bir `Pin` türüne koymak demektir. Hata mesajı pinning'in gerekli olduğunu söylüyor çünkü
`dyn Future<Output = ()>` `Unpin` özelliğini uygulaması gerekiyor ve şu an bu yapmıyor.

`trpl::join_all` fonksiyonu `JoinAll` adında bir yapı döner. Bu yapı, `F` adlı bir
tür üzerinde generic ve `Future` özelliğini uygulamaya kısıtlanmış. `await` ile doğrudan bir
future beklemek future'yi dolaylı olarak pinler. Bu yüzden future'leri beklemek istediğimiz her yerde
`pin!` kullanmamıza gerekmiyor.

Ancak burada doğrudan bir future beklemiyoruz. Bunun yerine, `trpl::join_all`
fonksiyonuna future'lerin bir koleksiyonunu geçirerek yeni bir future olan JoinAll oluşturuyoruz.
`join_all` için imza, koleksiyondaki ögelerin türlerinin hepsi `Future` özelliğini
uyulamasını gerektirir ve `Box<T>`, sardığı `T` `Unpin` özelliğini uygulayan bir
future ise yalnızca `Future` uygular.

Bu, içmek için çok şey! Gerçekten anlamak için `Future` özelliğinin aslında nasıl
çalıştığına, özellikle pinning etrafında biraz daha dalalım. `Future` özelliğinin
tanımlamasına tekrar bakalım:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    // Gerekli yöntem
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

`cx` parametresi ve onun `Context` türü, bir runtime'ın hala tembel iken verilen herhangi bir
future'yi ne zaman kontrol edeceğini bilmesinin anahtarıdır. Yine de, bunun nasıl çalıştığı
ayrıntları bu bölümün kapsamı dışındadır ve genellikle özel bir `Future` uygulaması yazarken
bunun hakkında düşünmeniz gerekecektir. Bunun yerine, `self` için türe odaklanacağız çünkü bu,
`self`'in bir tür eklemesinin olduğu bir yöntem gördüğümüz ilk kez. `self` için bir tür
eklemesi, diğer fonksiyon parametreleri için tür eklemeleri gibi çalışır ancak iki temel
farkla:

- Rust'a `self`'in yöntemin çağrılabilmesi için hangi tür olması gerektiğini söyler.
- Yalnızca herhangi bir tür olamaz. Yöntemin uygulandığı türe, o türe referans veya akıllı gösterge
  veya o türe referansı saran bir `Pin` ile kısıtlanmıştır.

[Bölüm 18][ch-18]<!-- ignore -->'de bu sözdizimde daha fazlasını göreceğiz. Şimdilik,
bir future'nin `Pending` mı yoksa `Ready(Output)` mı olduğunu kontrol etmek için pollamak istersek,
türe için `Pin`-sarılmış değiştirilebilir bir referansa ihtiyacımız olduğunu bilmek yeterlidir.

`Pin`, `&`, `&mut`, `Box`, ve `Rc` gibi gösterge benzeri türler için bir
sarıcıdır. (Teknik olarak `Pin`, `Deref` veya `DerefMut` özelliklerini uygulayan
türler ile çalışır ancak bu pratikte yalnızca referanslar ve akıllı göstergeler ile çalışmakla
eşdeğerdir.) `Pin` kendisi bir gösterge değildir ve `Rc` ve `Arc`'ın referans sayımı ile
yapdığı gibi kendi davranışına sahip değildir; saf olarak derleyicinin gösterge kullanımı üzerinde
kısıtlamları uygulamak için kullandığı bir araçtır.

`await`'ın `poll` çağrıları cinsinden uygulandığını hatırlamak, bunun `Pin`'den
değil `Unpin` cinsinden açıklayan, gördüğümüz hata mesajını açıklamaya başlar ancak
bu, `Pin` ve `Unpin`'in birbirileriyle tam olarak nasıl ilişkili olduğu ve neden `Future`,
`poll` çağırmak için `self`'in bir `Pin` türünde olmasını gerektiriyor?

Bu bölümün öncesinden hatırlayın ki bir future'deki await noktaları serisi bir durum
makinesine derlenir ve derleyici durum makinesinin, ödünçleme ve sahiplik dahil Rust'ın tüm
normal kurallarına uyduğundan emin olmasını sağlar. Bunu çalışmak için Rust, bir await noktası ile
sonraki await noktası veya async bloğun sonu arasında hangi verinin gerekli olduğunu bakar.
Sonra derlenmiş durum makinesinde karşılık bir varyant oluşturur. Her varyant, kaynak kodunun o
bölümünde kullanılacak verilere erişim gerektiği alır, o verinin sahipliğini alarak veya
ona değiştirilebilir veya değiştirilemez referans alarak.

Şimdiye kadar, iyi: vermediğimiz bir async bloğunda sahiplik veya referanslar hakkında bir
şeyi yanlış yaparsak, ödünç denetleyicisi bize söyleyecek. Ancak o bloğa karşılık gelen
future'yi etrafında hareket etmek istiyorsak—`join_all`'e geçmek için onu bir `Vec` içine
gömmek gibi—işler daha pürüzlü hale gelir.

Bir future'yi hareket ettirdiğimizde—ister onu bir veri yapısına `join_all` ile
yineleyici olarak kullanmak için itermeye istiyorsak ister bir fonksiyondan döndürmek için—aslında bu,
Rust'in bizim için oluşturduğu durum makinesini hareket ettirmek demektir. Ve Rust'teki çoğu diğer türlerin
aksine, async bloklar için Rust tarafından oluşturulan future'lar, Şekil 17-4'te basitleştirilmiş
gösterimde olduğu gibi, herhangi bir varyantın alanlarında kendilerine referanslarla bitebilir.

<figure>

<img alt="Tek sütunlu, üç sırlı bir tablo bir future'yi temsil eden fut1, ilk iki sırlarda 0 ve 1 değerlerini var ve üçüncü sırdan ikinci sıraya işaret eden bir ok, future içinde dahili bir referansı temsil eder." src="img/trpl17-04.svg" class="center" />

<figcaption>Şekil 17-4: Öz referanslı veri türü (A self-referential data type)</figcaption>

</figure>

Ancak varsayılan olarak, kendisine bir referansı olan herhangi bir nesne hareket etmek güvensizdir çünkü
referanslar her zaman işaret ettikleri şeyin gerçek bellek adresine işaret eder (bkz.
Şekil 17-5). Veri yapısını kendisini hareket ettirirseniz, bu dahili referanslar eski konuma işaret eden
halde kalırlar. Ancak o bellek konumu artık geçersiz. Bir şey için, veri yapısına
değişiklik yaptığınızda değeri güncellenmez. Başka bir şeyde—daha önemli—bilgisayar artık o
belleği başka amaçlar için yeniden kullanmakta özgür! Sonradan tamamen ilişkisiz olmayan veriyi okuyabilirsiniz.

<figure>

<img alt="İki tablo, fut1 ve fut2 adında iki future'yi temsil ediyor, her biri bir sütun ve üç sıra var, bir future'in fut1'den fut2'ye taşınmasının sonucunu temsil eder. İlki, fut1, gri yapılmıştır, her indiste soru işareti vardır, bilinmeyen belleği temsil eder. İkinci, fut2, ilk ve ikinci sıralarda 0 ve 1 vardır ve onun üçüncü sırasından fut1'in ikinci sırasına işaret eden bir ok vardır, bu future taşınmadan önceki bellek konumundaki eski bir konumu referans eden bir göstergeyi temsil eder." src="img/trpl17-05.svg" class="center" />

<figcaption>Şekil 17-5: Öz referanslı veri türünü taşımanın güvensiz sonucu</figcaption>

</figure>

Teorik olarak, Rust derleyicisi bir nesne taşındığında her referansı güncellemeyi çalışabilir
ancak bu çok performans genel katı ekleyebilir, özellikle güncellenmesi gereken tüm bir referans ağı varsa.
Bunun yerine, söz konusu veri yapısının bellekte _hareket etmemesinden_ emin olabilseydik, hiçbir
referans güncellemeyecektik. Bu tam olarak Rust'ın ödünç denetleyicisinin olduğu şeydir:
güvenli kodda, üzerinde aktif bir referansı olan herhangi bir öğeyi hareket etmekten sizi
engeller.

`Pin` bu tam guarantee'i sağlamak için bunun üzerine inşa edilmiştir. Değer bir değeri, o değere
işaret eden bir göstergeyi `Pin` içine sararak _pinlersek_, artık hareket edemez. Böylece,
`Pin<Box<SomeType>>`'niz varsa, aslında `SomeType` değerini pimlersiniz, `Box` göstergesini değil.
Şekil 17-6 bu süreci gösterir.

<figure>

<img alt="Yan yana sıralan üç kutu. İlk “Pin” etiketli, ikinci “b1”, ve üçüncü “pinned”. “pinned” içinde “fut” etiketli tek sütunlu bir tablo vardır; bu veri yapısının her parçası için hücreleri olan bir future'yi temsil eder. İlk hücresinde “0” değeri vardır, ikinci hücresinde dışına çıkan ve dördüncü ve son hücresine işaret eden bir ok vardır ki içinde “1” değeri vardır ve üçüncü hücresinde kesik çizgiler ve üç nokta vardır bu da veri yapısının başka paraları olabileceğini belirtir. Hepsi bir arada “fut” tablosu öz referanslı olan bir future'yi temsil eder. “Pin” etiketli kutudan bir ok çıkar, “b1” etiketli kutudan geçer ve “pinned” kutusunun içinde “fut” tablosundadaki biter." src="img/trpl17-06.svg" class="center" />

<figcaption>Şekil 17-6: Öz referanslı future türüne işaret eden bir `Box` pinlemesi</figcaption>

</figure>

Aslında, `Box` göstergesi hala özgürce etrafında hareket edebilir. Hatırlayın: asıl önemsediğimiz,
son olarak referans edilen verinin yerinde kalmasını sağlamaktır. Eğer bir gösterge etrafında
hareket eder, _ancak işaret ettiği veri_ aynı yerde ise, Şekil 17-7'deki gibi, olası
bir problem yok. (Bağımsız bir egzersiz olarak, `std::pin` modülü gibi türlerin belgelerine
ve bir `Pin` ile bir `Box` sarmayla bunu nasıl yapacağınızı anlamaya çalışın.) Anahtar şudur ki
öz referanslı tür kendisi hareket edemez çünkü hala pimlidir.

<figure>

<img alt="Kabaca üç sütun düzenlenmiş dört kutu, önceki diyagramla aynı ancak ikinci sütunda bir değişiklik ile. Şimdi ikinci sütunda iki kutu vardır, “b1” ve “b2” etiketli, “b1” gri yapılmıştır ve “Pin”'den gelen ok “b1” yerine “b2”'ye geçer bu da göstergenin “b1”'den “b2”'ye hareket ettiğini gösterir ancak “pinned” içindeki veri hareket etmemiştir." src="img/trpl17-07.svg" class="center" />

<figcaption>Şekil 17-7: Öz referanslı future türüne işaret eden bir `Box`'ün hareket ettirilmesi</figcaption>

</figure>

Ancak çoğu tür, bir `Pin` göstergesinin arkasında bile olsalar, etrafında hareket etmek
tamamen güvenlidir. Yalnızca öğeler dahili referanslara sahip olduğunda pinning hakkında düşünmemiz
gerekecektir. Sayılar ve Booleanlar gibi ilkel değerler güvenlidir çünkü
açıkça hiçbir dahili referansları yoktur.

Rust'te normalde çalıştığınız çoğu tür de öyledir. Örneğin bir `Vec`'i etrafında
hareket ettirebilirsiniz, endişe almadan. Şimdiye kadar gördüklerimize göre, eğer
`Pin<Vec<String>>`'niz varsa, `Vec<String>` üzerindeki hiçbir referans yoksa bile hareket etmek
her zaman güvenliyken, `Pin` tarafından sağlanan güvel ancak kısıtlı API'ler üzerinden her şeyi yapmak
zorunda kalırsınız. Bu gibi durumlarda öğeleri etrafında hareket etmenin iyi olduğunu derleyiciye
söylemeniz için bir yola ihtiyacımız var—ve işte `Unpin` buraya devreye girer.

`Unpin`, Bölüm 16'da gördüğümüz `Send` ve `Sync` özelliklerine benzer bir işareti
özelliğidir ve böylece kendi işlevselliğine sahip değildir. İşareti özellikler yalnızca
belirli bir bağlamda belirli bir özelliği uygulayan bir türü kullanmanın güvenli olduğunu
derleyiciye söylemek için vardır. `Unpin`, derleyiciye belirli bir türün söz konusu değerin
güvenli bir şekilde hareket etmesi hakkında herhangi bir garanti tutması gerektmediğini söyler.

<!--
  Sondraki bloktaki satır içi `<code>`, içinde satır içi `<em>`'ye izin vermek
  içindir, NoStarch'ın stil açısından eşleşen ve buradaki metinde
  vurgulamak için onun normal bir türeyle farklı bir şey olduğunu göstermek için
-->

`Send` ve `Sync` ile olduğu gibi, derleyici `Unpin`'i, güvenli olduğunu
kanıtlayabildiği her tür için otomatik olarak uygular. Tekil bir durum, yine `Send` ve
`Sync`'a benzer, `Unpin`'in bir tür için _uygulanmadığı_ durumdur. Bunun gösterimi
<code>impl !Unpin for <em>SomeType</em></code> şeklindedir ki burada
<code><em>SomeType</em></code> bir `Pin`'de kullanıldığında o türün o garantiyi
tutmak zorunda olduğu bir türün adıdır.

Başka bir deyişle, `Pin` ve `Unpin` arasındaki ilişki hakkında aklınızda tutmanız gereken
iki şey vardır. İlk olarak, `Unpin` "normal" durumdur ve `!Unpin` özel durumdur.
İkinci olarak, bir türün `Unpin`'mi yoksa `!Unpin` olduğu _yalnızca_
<code>Pin<&mut <em>SomeType</em>></code> gibi o tür için pimlenmiş bir
gösterge kullandığınızda önemlidir.

Bunu somutlaştırmak için bir `String` düşünün: uzunluğu ve onu oluşturan Unicode
karakterleri vardır. Şekil 17-8'de görüldüğü gibi bir `String`'i `Pin` içinde sarabiliriz.
Ancak `String`, `Unpin`'i otomatik olarak uygular ki Rust'teki çoğu diğer tür de öyledir.

<figure>

<img alt="Sol tarafta “Pin” etiketli bir kutu, ondan sağ tarafa “String” etiketli kutuya giden bir ok vardır. “String” kutusu içinde 5usize verisi vardır ki bu dizinin uzunluğunu temsil eder ve “h”, “e”, “l”, “l” ve “o” harfleri bu String örneğinde saklanan “hello” dizisinin karakterlerini temsil eder. “String” kutusu ve onun etiketi çevreli kesikli bir dikdörtgen vardır ancak “Pin” kutusu yok." src="img/trpl17-08.svg" class="center" />

<figcaption>Şekil 17-8: Bir `String` pinlemesi; kesik çizgi, `String`'in `Unpin` özelliğini uyguladığını ve böylece pimlenmediğini gösterir</figcaption>

</figure>

Sonuç olarak, `String` bunun yerine `!Unpin` uygulasa yasadışı olan şeyleri
yapabiliriz, örneğin Şekil 17-9'deki gibi bellekte aynı tam konumda bir string'i başka bir string ile
değiştirmek. Bu `Pin` sözleşmesini ihlal etmez çünkü `String` etrafında hareket etmeyi
güvensiz kılan herhangi bir dahili referansı yoktur. Bu tam olarak onun `!Unpin` yerine
`Unpin` uyguladığıdır.

<figure>

<img alt="Önceki örnekteki aynı “hello” dizi verisi, şimdi “s1” etiketli ve gri yapılmıştır. Önceki örnekteki “Pin” kutusu şimdi farklı bir String örneğine işaret eder ki “s2” etiketli, geçerlidir, 7usize uzunluğu vardır ve “goodbye” dizisinin karakterlerini içerir. s2 de Unpin özelliği uyguladığı için çevreli kesikli bir dikdörtgen ile çevrilmiştir." src="img/trpl17-09.svg" class="center" />

<figcaption>Şekil 17-9: Bellekte tamamen farklı bir `String` ile bir `String`'in değiştirilmesi</figcaption>

</figure>

Şimdi Kod Listesi 17-23'teki `join_all` çağrısı için bildirilen hataları anlamak için yeterince
biliyoruz. Orijinal olarak async bloklar tarafından üretilen future'leri bir `Vec<Box<dyn
Future<Output = ()>>>` içine taşımaya çalıştık ancak gördüğümüz gibi, o future'ler dahili
referanslara sahip olabilir bu yüzden otomatik olarak `Unpin` uygulamazlar. Onları pimlediğimizde,
sonuçan `Pin` türünü `Vec` içine geçebiliriz, gelecekte future'lerin altındaki verinin _hareket
etmeyeceğine_ emin olabiliriz. Kod Listesi 17-24, `pin!` makrosunu çağırarak ve üç future'nin
her birinin tanımlandığı yerde ve özellik nesnesi türünü ayarlayarak kodu nasıl düzelteceğini gösterir.

<Listing number="17-24" caption="Pinning futures to enable moving them into vector">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-24/src/main.rs:here}}
```

</Listing>

Bu örnek artık derlenir ve çalışır ve gelecekte future'leri vektörden ekleyebilir veya
çıkarabilir ve hepsini bir araya getirebiliriz.

`Pin` ve `Unpin`, çoğu zaman alt seviye kütüphaneleri oluştururken veya bir
runtime'ı kendiniz oluştururken önemlidir, günlük Rust kodu için değil. Ancak bu
özellikleri hata mesajlarında gördüğünüzde, artık kodunuzu nasıl düzelteceğinize dair
daha iyi bir fikriniz olacak!

> Not: Bu `Pin` ve `Unpin` kombinasyonu, Rust'te aksi takdirde kanıtlayıcı olan
> çok karmaşık bir tür sınıfını güvenli bir şekilde uygulamayı mümkün kılar çünkü onlar
> öz referanslıdır. `Pin` gerektiren türler async Rust'te bugün en yaygın gösterir ancak
> her zaman zaman zaman başka bağlamlarda da görebilirsiniz.
>
> `Pin` ve `Unpin`'in nasıl çalıştığı ve onları tutmak zorunda oldukları kurallar,
> `std::pin` için API belgelerinde kapsamlı olarak kaplanmıştır bu yüzden daha fazla öğrenmek
> isterseniz başlamak için harika bir yerdir.
>
> Altında şeylerin daha fazla ayrıntıyla nasıl çalıştığını anlamak isterseniz,
> [2][under-the-hood]<!-- ignore --> ve
> [4][pinning]<!-- ignore --> bölümlerine bakın
> [_Rust'te Asenkron Programlama_][async-book] kitabına bakın.

### `Stream` Özelliği (The `Stream` Trait)

Artık `Future`, `Pin` ve `Unpin` özellikleri hakkında daha derin bir anlayışınız
olduğuna göre, dikkatimizi `Stream` özelliğine çevirebiliriz. Bölümün başlarında öğrendiğiniz
gibi, akarslar asenkrone yineleyicilere benzerdir. Ancak `Yineleyici` ve `Future`'in
aksine, `Stream`'in bu yazma sırasında standart kütüphanede hiç tanımlaması yoktur ancak
ekosistem genelinde kullanılan `futures` crate'ten gelen çok yaygın bir tanımı _vardır_.

`Stream` özelliğine bakmadan önce `Yineleyici` ve `Future` özelliklerinin tanımlamalarını
gözden geçirelim ki bir `Stream` özelliği bunları nasıl bir araya getirebilir. `Yineleyici`'den,
bir dizi fikrimiz var: onun `next` yöntemi bir `Option<Self::Item>` sağlar. `Future`'den,
zamanla hazır olma fikrimiz var: onun `poll` yöntemi bir `Poll<Self::Output>` sağlar.
Zamanla hazır olan ögeler dizisini temsil etmek için, bu özellikleri bir araya getiren bir `Stream`
özelliği tanımlarız:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

trait Stream {
    type Item;

    fn poll_next(
        self: Pin<&mut Self>,
        cx: &mut Context<'_>
    ) -> Poll<Option<Self::Item>>;
}
```

`Stream` özelliği, akar tarafından üretilen ögelerin türü için `Item` adında bir ilişkili tür
tanımlar. Bu, sıfırdan çok sayıda öge olabileceği `Yineleyici`'e benzerdir ve birim
tane `Output` olan `Future`'in aksine, o birim bile `()` birim türü olsa.

`Stream` ayrıca bu ögeleri almak için bir yöntem tanımlar. Bunu `poll_next` olarak
adlandırıyoruz çünkü bu, `Future::poll`'in yaptığı gibi aynı şekilde pollar ve `Yineleyici::next`'in
yaptığı gibi aynı şekilde bir öge dizisi üretir. Dönüş türü, `Poll` ile `Option`'u birleştirir.
Dıştaki tür `Poll`'tür çünkü bir future gibi hazır olma kontrol edilmesi gerekir. İçteki tür `Option`
tür çünkü başka mesaj olup olmadığını belirtmeli gerekir, tıpkı bir yineleyici gibi.

Buna çok benzer bir tanımlama muhtemelen Rust'ın standart kütüphanesinin bir parçası
olarak sonlanacak. Bu arada, çoğu runtime'ların araç setinin bir parçasıdır bu yüzden ona
güvenebilirsiniz ve kapladığımız her şey genellikle uygulanır!

[“Akarslar: Sıral Future'ler”][streams]<!-- ignore --> bölümünde gördüğümüz
örneklerde, `poll_next`'i _veya_ `Stream`'i kullanmadık ve bunun yerine
`next` ve `StreamExt` kullandık. Elbette `poll_next` API cinsinden doğrudan çalışmak
`_mümkün_ tıpkı kendi `Stream` durum makinesimizi elle yazarak future'lerle doğrudan çalışabilir
_mümkün_ tıpkı `poll` yöntemi ile. Ancak `await` kullanmak çok daha hoştır ve
`StreamExt` özelliği `next` yöntemini sağlar böylece tam bunu yapabiliriz:

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-stream-ext/src/lib.rs:here}}
```

> Not: Bölümde önce kullandığımız gerçek tanımla biraz farklı görünüyor
> çünkü özelliklerinde async fonksiyonları desteklemeyen Rust sürümlerini destekliyor.
> Sonuç olarak şu şekilde görünüyor:
> ```rust,ignore
> fn next(&mut self) -> Next<'_, Self> where Self: Unpin;
> ```
>
> Bu `Next` türü, `Future` uygular ve `Next<'_, Self>` ile `self` referansının
> ömrünü adlandırabilmemizi sağlayan bir `struct`'tur, bu yüzden `await` bu yöntemle
> çalışabilir.

`StreamExt` özelliği ayrıca akarslarla kullanılabilir ilginç yöntemlerin evidiridir.
`StreamExt`, `Stream` uygulayan her tür için otomatik olarak uygulanır ancak bu özellikler ayrı
tanımlanmıştır ki topluluk, temel özelliği etkilemeden kolaylık API'lar üzerinde yinelenmesi
sağlanabilir.

`trpl` crate'inde kullanılan `StreamExt` sürümünde, özellik yalnızca `next` yöntemini
tanımlamaz ayrıca `Stream::poll_next`'i çağırmanın ayrıntılarını doğru şekilde işleyen varsayılan bir
`next` uygulamasını da sağlar. Bu, kendi akış veri türünüzü yazmanız gerekse bile _yalnızca_
`Stream`'i uygulamanız gerektiği ve veri türünüzü kullanan herkes `StreamExt` ve onun
yöntemleri ile otomatik olarak kullanabilir demektir.

Bu özellikler hakkında alt seviye ayrıntıları kapsayacaklarımız bu kadar. Sonlandırmak
için, future'leri (akarslar dahil), görevleri ve iş parçacıklarının tümünü nasıl bir araya
getirdiğini düşünelim!

[message-passing]: ch17-02-concurrency-with-async.md#sending-data-between-two-tasks-using-message-passing
[ch-18]: ch18-00-oop.html
[async-book]: https://rust-lang.github.io/async-book/
[under-the-hood]: https://rust-lang.github.io/async-book/02_execution/01_chapter.html
[pinning]: https://rust-lang.github.io/async-book/04_pinning/01_chapter.html
[first-async]: ch17-01-futures-and-syntax.html#our-first-async-program
[any-number-futures]: ch17-03-more-futures.html#working-with-any-number-of-futures
[streams]: ch17-04-streams.html