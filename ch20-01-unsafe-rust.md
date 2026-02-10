## Güvenli Olmayan Rust (Unsafe Rust)

Şimdiye kadar tartıştığımız tüm kod, Rust'un bellek güvenliği garantilerinin
derleme zamanında uygulanmasını sağlamıştır. Ancak, Rust'un içinde saklı ve bu
bellek güvenliği garantilerini uygulamayan ikinci bir dili vardır: Buna _güvenli olmayan Rust_
(unsafe Rust) denir ve normal Rust gibi çalışır ancak bize ekstra süper güçler verir.

Güvenli olmayan Rust'ın var olma nedeni, doğası gereği, statik analiz
muhafazakâr olmasıdır. Derleyici, kodun garantileri karşılayıp karşılamadığını belirlemeye
çalıştığında, geçersiz bazı programları kabul etmek yerine geçerli bazı programları reddetmek
daha iyidir. Kod _olabilir_ normal olsa bile, eğer Rust derleyicisinin
kendiğine yeterli bilgi yoksa, kodu reddedecektir. Bu durumlarda, derleyiciye,
"Bana güven, ne yaptığımı biliyorum." diyerek güvenli olmayan kodu kullanabilirsiniz.
Ancak uyarın, güvenli olmayan Rust'ı kendi sorumluluğunuzda kullanırsınız:
Güvenli olmayan kodu yanlış kullanırsanız, null pointer'ın referansı kaldırılması
(dereferencing) gibi bellek güvenliği nedeniyle sorunlar oluşabilir.

Rust'un güvenli olmayan bir alter ego'ya sahip olmasının başka bir nedeni de,
alttaki bilgisayar donanımı doğası gereği güvenli değildir. Eğer Rust size güvenli
olmayan işlemleri yapmanıza izin vermezse, belirli görevleri yapamazsınız. Rust'un
düşük seviye sistem programlaması yapmanıza izin vermesi gerekir, örneğin doğrudan işletim sistemiyle
etkileşim veya kendi işletim sisteminizi yazmak bile. Düşük seviye sistem programlamasıyla
çalışmak dilin hedeflerinden biridir. Güvenli olmayan Rust ile ne yapabileceğimizi ve
nasıl yapabileceğimizi keşfedelim.

<!-- Old headings. Do not remove or links may break. -->

<a id="unsafe-superpowers"></a>

### Güvenli Olmayan Süper Güçlerini Gerçekleştirme (Performing Unsafe Superpowers)

Güvenli olmayan Rust'a geçmek için, `unsafe` anahtar kelimesini kullanın ve sonra güvenli
olmayan kod tutan yeni bir blok başlatın. Güvenli olmayan Rust'te, güvenli olmayan
Rust'te yapamayacağınız beş eylemi gerçekleştirebilirsiniz ki buna _güvenli olmayan
süper güçler_ diyoruz. Bu süper güçler şunları içerme yeteneği:

1. Ham bir referansı (raw pointer) referans kaldırma (dereferencing).
1. Güvenli olmayan bir fonksiyon veya yöntem çağırma.
1. Değiştirilebilir statik bir değişkene erişme veya değiştirme.
1. Güvenli olmayan bir trait'i uygulama.
1. `union`'ların alanlarına erişme.

Anlamanız önemli ki `unsafe`, ödünç kontrolcü (borrow checker)'ü kapatmaz veya Rust'un
başka güvenlik kontrollerini devre dışı bırakmaz: Güvenli olmayan kodda bir referans kullanırsanız,
yine kontrol edilecek. `unsafe` anahtar kelimesi size sadece bellek güvenliği için derleyici tarafından
kontrol edilmeyecek beş özelliğe erişim verir. Güvenli olmayan bir bloğun içinde hala
bir derecede güvenlik alacaksınız.

Ek olarak, `unsafe`, blok içindeki kodun gerekli tehlikeli olduğu veya kesinlikle bellek
güvenliği sorunları olacağı anlamına gelmez: Amaç şudur ki programcı olarak, bir
`unsafe` blok içindeki kodun belleği geçerli bir şekilde erişmesini sağlayacaksınız.

İnsanlar hatalıdır ve hatalar oluşacaktır ancak bu beş güvenli olmayan işlemin
`unsafe` ile açıklanmış bloklar içinde olmasını gerektirmesiyle, bellek güvenliğiyle ilgili
herhangi bir hatanın bir `unsafe` bloğu içinde olduğunu bileceksiniz. `unsafe` blokları
küçük tutun; bellek hatalarını araştırırken kendinize minnettar olacaksınız.

Güvenli olmayan kodu mümkün olduğunda izole etmek için, bu tür kodu güvenli bir
soyutlama (abstraction) içinde kapsatmak ve güvenli bir API sağlamak en iyidir ki bunu
bölümün ilerisinde güvenli olmayan fonksiyonlar ve yöntemleri incelerken tartışacağız.
Standart kütüphanenin bazı bölümleri, denetlenmiş güvenli olmayan kod üzerinde güvenli
soyutlamalar olarak uygulanır. Güvenli olmayan kodu güvenli bir soyutlama içinde sarmak
(wrapper), `unsafe` kullanımının sizin veya kullanıcılarınızın bu işlevselliği kullanmak isteyebileceği
tüm yerlere sızmasını engeller çünkü güvenli bir soyutlama kullanmak güvenlidir.

Her bir güvenli olmayan beş süper güce sırasıyla bakalım. Ayrıca güvenli olmayan koda
güvenli bir arayüz sağlayan bazı soyutlamalara da bakacağız.

### Ham Bir Referansın Referansını Kaldırma (Dereferencing a Raw Pointer)

Bölüm 4'te, [“Dangling References”][dangling-references]<!-- ignore
--> bölümünde, derleyicinin referansların her zaman geçerli olduğunu sağladığını söyledik.
Güvenli olmayan Rust, referanslara benzer _ham referanslar_ (raw pointers) adında iki
yeni türe sahiptir. Referanslarda olduğu gibi, ham referanslar sabit veya değiştirilebilir
olabilir ve sırasıyla `*const T` ve `*mut T` olarak yazılır. Yıldız, referans kaldırma
operatörü değildir; tür adının bir parçasıdır. Ham referanslar bağlamında, _sabit (immutable)_
demektir ki referansın referansı kaldırıldıktan (dereferenced) sonra doğrudan atanamaz.

Referanslardan ve akıllı işaretçilerden (smart pointers) farklı olarak, ham referanslar:

- Hem sabit ve değiştirilebilir referanslara veya aynı konuma çoklu değiştirilebilir
  referanslara sahip olarak ödünç kurallarını yoksayabilir
- Geçerli belleğe işaret ettikleri garanti edilmez
- Null olmaya izin verilir
- Herhangi bir otomatik temizleme (cleanup) gerçekleştirmezler

Rust'un bu garantileri uygulamamasından opt-out ederek, garantili güvenliği
daha büyük performans veya Rust'un garantilerinin geçerli olmadığı başka bir dil veya donanım
ile etkileşim yeteneği karşılığında verebilirsiniz.

Kod Listesi 20-1, bir sabit ve bir değiştirilebilir ham referans nasıl oluşturulacağını
gösterir.

<Listing number="20-1" caption="Creating raw pointers with the raw borrow operators">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-01/src/main.rs:here}}
```

</Listing>

Bu kodda `unsafe` anahtar kelimesini dahil etmediğimizi not edin. Güvenli kodda ham
referanslar oluşturabiliriz; sadece güvenli olmayan bir blok dışında ham referansların
referansını kaldıramayız (dereferencing), az sonra göreceksiniz.

Ham referansları ham ödünç operatörleri kullanarak oluşturduk: `&raw const num`
bir `*const i32` sabit ham referans oluşturur ve `&raw mut num` bir `*mut i32`
değiştirilebilir ham referans oluşturur. Çünkü onları doğrudan yerel bir değişkenden
oluşturduk, bu belirli ham referansların geçerli olduğunu biliyoruz ancak herhangi bir ham
referans hakkında bu varsayımı yapamayız.

Bunu göstermek için, şimdi `as` anahtar kelimesini kullanarak ham ödünç operatörü
yerine bir değeri cast ederek geçerliliğinden o kadar emin olamayacağımız bir ham
referans oluşturacağız. Kod Listesi 20-2, bellekteki keyfi bir konuma bir ham referans nasıl
oluşturulacağını gösterir. Keyfi belleği kullanmaya çalışmak tanımsızdır: O adreste veri
olabilir veya olmayabilir, derleyici kodu böyle optimize edebilir ki bellek erişimi yoktur veya
program bir bölme hatasıyla (segmentation fault) sonlanabilir. Genellikle, bu tür kodu
yazmak için iyi bir neden yoktur, özellikle ham ödünç operatörü kullanabileceğiniz
durumlarda ancak mümkündür.

<Listing number="20-2" caption="Creating a raw pointer to an arbitrary memory address">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-02/src/main.rs:here}}
```

</Listing>

Güvenli kodda ham referanslar oluşturabileceğimizi ancak ham referansların referansını
kaldırıp işaret ettiği veriyi okuyamayacağımızı hatırlayın. Kod Listesi 20-3'te,
bir `unsafe` bloğu gerektiren bir ham referans üzerinde referans kaldırma operatörü `*` kullanıyoruz.

<Listing number="20-3" caption="Dereferencing raw pointers within an `unsafe` block">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-03/src/main.rs:here}}
```

</Listing>

Bir referans oluşturmak zararsızdır; sadece işaret ettiği değere erişmeye çalıştığımızda
geçersiz bir değerle uğraşabiliriz.

Ayrıca Kod Listeleri 20-1 ve 20-3'te, `num`'un depolandığı aynı bellek konumuna
işaret eden `*const i32` ve `*mut i32` ham referanslar oluşturduğumuzu not edin.
Bunun yerine, `num` için bir sabit ve bir değiştirilebilir referans oluşturmaya
çalışsaydık, kod derlenmezdi çünkü Rust'un sahiplik (ownership) kuralları aynı
zaman değiştirilebilir bir referansa izin vermez. Ham referanslarla, aynı konuma bir
değiştirilebilir ve bir sabit referans oluşturabilir ve veriyi değiştirilebilir referans
üzerinden değiştirebiliriz, potansiyel olarak bir veri yarışı (data race) oluşturabilir.
Dikkatli olun!

Bu tüm tehlikelerle, neden ham referansları asla kullanırsınız? Bir büyük kullanım
durumu C koduyla etkileşimdir, bir sonraki bölümde göreceksiniz. Başka bir durum,
ödünç kontrolcüsünün anlamadığı güvenli soyutlamalar oluştururken. Güvenli olmayan
fonksiyonları tanıtacağız ve sonra güvenli olmayan kod kullanan güvenli bir soyutlama
örneğine bakacağız.

### Güvenli Olmayan Bir Fonksiyon veya Yöntem Çağırma (Calling an Unsafe Function or Method)

Bir güvenli olmayan blokta gerçekleştirebileceğiniz ikinci tür işlem, güvenli olmayan
fonksiyonları çağırmaktır. Güvenli olmayan fonksiyonlar ve yöntemler normal fonksiyonlar
ve yöntemler gibi görünür ancak tanımının geri kalanından önce ek bir `unsafe`'e sahiptir.
Bu bağlamda `unsafe` anahtar kelimesi, fonksiyonun biz bu fonksiyonu çağırdığımızda
korumamız gereken gereksinmleri olduğunu gösterir çünkü Rust bu gereksinimleri karşıladığımızı
garanti edemez. Bir güvenli olmayan fonksiyonu bir `unsafe` bloğu içinde çağırarak,
bu fonksiyonun belgelerini okuduğumuzu ve fonksiyonun sözleşmelerini
sürdürme sorumluluğunu aldığımızı söylüyoruz.

Gövdesinde hiçbir şey yapmayan `dangerous` adında güvenli olmayan bir fonksiyon:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-01-unsafe-fn/src/main.rs:here}}
```

`dangerous` fonksiyonunu ayrı bir `unsafe` bloğu içinde çağırmalıyız. Eğer
`dangerous`'i `unsafe` bloğu olmadan çağırmaya çalışırsak bir hata alacağız:

```console
{{#include ../listings/ch20-advanced-features/output-only-01-missing-unsafe/output.txt}}
```

`unsafe` bloğuyla, Rust'a bu fonksiyonun belgelerini okuduğumuzu, doğru kullanmayı
bildiğimizi ve fonksiyonun sözleşmesini yerine getirdiğimizi doğrulduğumuzu iddia ediyoruz.

Bir `unsafe` fonksiyonun gövdesinde güvenli olmayan işlemleri gerçekleştirmek için,
normal bir fonksiyonun içinde olduğu gibi yine bir `unsafe` bloğu kullanmanız gerekir ve unutursanız
derleyici sizi uyaracaktır. Bu bize `unsafe` blokları mümkün olduğunda küçük
tutmamıza yardımcı olur çünkü güvenli olmayan işlemler tüm fonksiyon gövdesi boyunca
gerekli olmayabilir.

#### Güvenli Olmayan Kod Üzerinde Güvenli Bir Soyutlama Oluşturma (Creating a Safe Abstraction over Unsafe Code)

Bir fonksiyon güvenli olmayan kod içerdiği, tüm fonksiyonu güvenli olmayan olarak
işaretlememiz gerektiği anlamına gelmez. Aslında, güvenli olmayan kodu güvenli bir
fonksiyonda sarmak ortak bir soyutlamadır. Bir örnek olarak, standart kütüphaneden
`split_at_mut` fonksiyonunu inceleyelim ki bu bazı güvenli olmayan kod gerektirir.
Nasıl uygulayabileceğimizi keşfedelim. Bu güvenli yöntem, değiştirilebilir dilimlerde
(mutable slices) tanımlanır: Bir dilim alır ve bir argüman olarak verilen indekste dilimi ikiye
ayarlar. Kod Listesi 20-4, `split_at_mut` nasıl kullanılacağını gösterir.

<Listing number="20-4" caption="Using the safe `split_at_mut` function">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-04/src/main.rs:here}}
```

</Listing>

Sadece güvenli Rust kullanarak bu fonksiyonu uygulayamayız. Bir girişim, Kod Listesi
20-5 gibi görünebilir ki derlenmeyecek. Basitlik için, `split_at_mut`'i bir yöntem
yerine bir fonksiyon olarak ve genel bir tür `T` yerine sadece `i32` değerlerinin dilimleri
için uygulayacağız.

<Listing number="20-5" caption="An attempted implementation of `split_at_mut` using only safe Rust">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-05/src/main.rs:here}}
```

</Listing>

Bu fonksiyon önce dilimin toplam uzunluğunu alır. Sonra, bir parametre olarak
verilen indeksin dilim içinde olup olmadığını belirleyerek (less than or equal to length) doğrular.
Doğrulama, dilimi ayırmak için uzunluğundan büyük bir indeks geçirirsek fonksiyon o
indeksi kullanmaya çalışmadan önce panikleyeceğini (panic) anlamına gelir.

Sonra, bir demet içinde iki değiştirilebilir dilim döndürüyoruz: biri orijinal dilimin
başlangıcından `mid` indeksine ve diğeri `mid`'ten dilimin sonuna kadar.

Kod Listesi 20-5'teki kodu derlemeye çalıştığımızda bir hata alacağız:

```console
{{#include ../listings/ch20-advanced-features/listing-20-05/output.txt}}
```

Rust'un ödünç kontrolcüsü, dilimin farklı parçalarını ödünçlediğimizi anlamayabilir;
sadece aynı dilimden iki kez ödünçlediğimizi bilir. Dilimin farklı parçalarını ödünçlemek
temelde uygundur çünkü iki dilim çakışmıyor ancak Rust bunu bilmek için yeterince
akıllı değil. Kodun uygunda biliyoruz ancak Rust bilmiyor, güvenli olmayan koda
ulaşmak zamanı.

Kod Listesi 20-6, bir `unsafe` blok, bir ham referans ve `split_at_mut`'in
uygulamasının çalışmasını sağlamak için bazı güvenli olmayan fonksiyon çağrıları nasıl kullanılacağını
gösterir.

<Listing number="20-6" caption="Using unsafe code in implementation of `split_at_mut` function">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-06/src/main.rs:here}}
```

</Listing>

Bölüm 4'teki [“The Slice Type”][the-slice-type]<!-- ignore --> bölümünden
hatırlayın ki bir dilim bazı verilere işaret eden bir referans ve dilimin uzunluğudur.
Bir dilimin uzunluğunu almak için `len` yöntemini ve bir dilimin ham referansına erişmek için
`as_mut_ptr` yöntemini kullanıyoruz. Bu durumda, `i32` değerlerine değiştirilebilir bir
dilimimiz olduğu için `as_mut_ptr` `*mut i32` türünde bir ham referans döndürüyor ki bunu
`ptr` değişkeninde sakladık.

`mid` indeksin dilim içinde olduğunu doğrulamayı koruyoruz. Sonra güvenli olmayan
koda geliyoruz: `slice::from_raw_parts_mut` fonksiyonu bir ham referans ve bir uzunluk alır ve
bir dilim oluşturur. Bu fonksiyonu, `ptr`'den başlayan ve `mid` öğe uzunluğunda olan bir
dilim oluşturmak için kullanıyoruz. Sonra, `ptr` üzerindeki `add` yöntemini `mid` ile
çağırarak `mid`'te başlayan bir ham referans alıyoruz ve o referansı ve `mid`'ten sonraki
kalıcı öğe sayısını uzunluk olarak kullanarak bir dilim oluşturuyoruz.

`slice::from_raw_parts_mut` fonksiyonu güvenli olmaz çünkü bir ham referans alır ve bu
referansın geçerli olduğuna güvenmeli. Ham referanslardaki `add` yöntemi de güvenli
olmaz çünkü offset konumunun da geçerli bir referans olduğuna güvenmeli. Bu yüzden,
çağırabilmek için `slice::from_raw_parts_mut` ve `add` çağrılarımızın etrafına bir
`unsafe` bloğu koymak zorunda kaldık. Koda bakarak ve `mid`'in `len`'den küçük veya
eşit olması doğrulamasını ekleyerek, `unsafe` bloğu içinde kullanılan tüm ham referansların
dilim içindeki verilere geçerli referanslar olduğunu söyleyebiliriz. Bu `unsafe`'in kabul
edilebilir ve uygun bir kullanımıdır.

Sonuçta oluşan `split_at_mut` fonksiyonunu `unsafe` olarak işaretlememiz
gerekmediğini not edin ve bu fonksiyonu güvenli Rust'ten çağırabiliriz. Güvenli olmayan
koda güvenli bir soyutlama oluşturduk çünkü fonksiyonun uygulaması `unsafe` kodu güvenli
bir şekilde kullanıyor çünkü bu fonksiyona erişimi olan veriden sadece geçerli referanslar
oluşturuyor.

Buna karşın, Kod Listesi 20-7'teki `slice::from_raw_parts_mut` kullanımı dilim kullanıldığında
muhtemelen çökürecektir. Bu kod, keyfi bir bellek konumunu alır ve 10,000 öğe uzunluğunda
bir dilim oluşturur.

<Listing number="20-7" caption="Creating a slice from an arbitrary memory location">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-07/src/main.rs:here}}
```

</Listing>

Bu keyfi konumdaki belleğe sahip değiliz ve bu kodun oluşturduğu dilimin geçerli
`i32` değerleri içerdiğini garanti yoktur. `values`'i geçerli bir dilimmiş gibi kullanmaya
çalışmak tanımsız davranışla sonuçlanır.

#### `extern` Fonksiyonlarıyla Dış Kod Çağırma (Using `extern` Functions to Call External Code)

Bazen Rust kodunuz başka bir dilde yazılmış kodla etkileşmesi gerekebilir. Bunun için
Rust'un başka bir dilden tanımlanan fonksiyonları ve farklı (yabancı) programlama
dilinin bu fonksiyonları çağırmasını sağlayan bir yol olan _Yabancı Fonksiyon Arayüzü (FFI)_
için kolaylık sağlayan bir `extern` anahtar kelimesi vardır.

Kod Listesi 20-8, C standart kütüphanesinden `abs` fonksiyonuyla bir entegrasyon
nasıl kurulacağını gösterir. `extern` blokları içinde tanımlanan fonksiyonlar genel olarak
Rust kodundan çağırmak için güvenli değildir bu yüzden `extern` blokları da `unsafe`
olarak işaretlenmelidir. Sebep şudur ki diğer diller Rust'un kurallarını ve garantilerini
uygulamaz ve Rust bunları kontrol edemez bu yüzden güvenliği sağlamak sorumluluğu
programcıya düşer.

<Listing number="20-8" file-name="src/main.rs" caption="Declaring and calling an `extern` function defined in another language">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-08/src/main.rs}}
```

</Listing>

`unsafe extern "C"` bloğu içinde, başka bir dilden çağırmak istediğimiz dış fonksiyonların adlarını
ve imzalarını listeliyoruz. `"C"` parçası, dış fonksiyonun hangi _uygulama ikili arayüzünü
(ABI)_ kullandığını tanımlar: ABI, fonksiyonu assembly seviyesinde nasıl çağıracağını tanımlar.
`"C"` ABI en yaygın olanıdır ve C programlama dilinin ABI'yi takip eder. Rust'un
desteklediği tüm ABI'ler hakkında bilgi [Rust Reference][ABI] içinde mevcuttur.

Bir `unsafe extern` bloğu içinde tanımlanan her madde dolaylı olarak güvenli olamazdır.
Ancak, bazı FFI fonksiyonları *çağırmak için* güvenlidir. Örneğin, C'nin standart
kütüphanesinden `abs` fonksiyonunun herhangi bir bellek güvenliği dikkate almaz ve herhangi
`i32` ile çağırılabileceğini biliriz. Bu gibi durumlarda, `safe` anahtar kelimesini
kullanabiliriz ki bu belirli fonksiyon `unsafe extern` bloğu içinde olsa bile güvenle çağırılabilir.
Bu değişikliği yaptıktan sonra, Kod Listesi 20-9'da gösterildiği gibi çağırmak artık bir
`unsafe` bloğu gerektirmez.

<Listing number="20-9" file-name="src/main.rs" caption="Explicitly marking a function as `safe` within an `unsafe extern` block and calling it safely">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-09/src/main.rs}}
```

</Listing>

Bir fonksiyonu `safe` olarak işaretlemek doğası gereği güvenli yapmaz! Bunun yerine,
Rust'a bunun güvenli olduğu bir vaat gibi davranır. Bu sözü koruma sorumluluğu hala
sizin!

#### Başka Dillerden Rust Fonksiyonlarını Çağırma (Calling Rust Functions from Other Languages)

Ayrıca, diğer dillerin Rust fonksiyonlarını çağırmasını sağlayan bir arayüz oluşturmak için
`extern`'i kullanabiliriz. Tüm bir `extern` bloğu oluşturmak yerine, ilgili fonksiyon için
`fn` anahtar kelimesinden hemen önce `extern` anahtar kelimesini ekleyerek kullanılacak ABI'yi belirtiriz.
Ayrıca `#[unsafe(no_mangle)]` açıklamasını eklememiz gerekir ki Rust derleyicisine bu
fonksiyonun adını mangle etmemesini söyler. _Mangling_, derleyicinin fonksiyona verdiğimiz
adı derleme sürecinin diğer bölümleri için daha fazla bilgi içeren ancak insan tarafından daha az
okunabilir bir ada değiştirmesidir. Her programlama dili derleyicisi isimleri hafifçe farklı
mangle eder bu yüzden bir Rust fonksiyonunun diğer diller tarafından adlandırılabilmesi için Rust
derleyicisinin name mangling'ini devre dışı bırakmalıyız. Bu güvenli olmaz çünkü
yerleşik mangling olmadan kütüphaneler arasındaki isim çakışmaları olabilir bu yüzden mangling olmadan
ihraç etmek için seçtiğimiz ismin güvenli olması sorumluluğumuzdur.

Aşağıdaki örnekte, paylaşılan bir kütüphaneye derlendikten ve C'den bağlandıktan sonra
`call_from_c` fonksiyonunu C kodundan erişilebilir hale getiriyoruz:

```
#[unsafe(no_mangle)]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```

`extern`'in bu kullanımı, `extern` bloğu üzerinde değil sadece özellikte `unsafe` gerektirir.

### Değiştirilebilir Statik Bir Değişkene Erişme veya Değiştirme (Accessing or Modifying a Mutable Static Variable)

Bu kitapta henüz küresel değişkenler (global variables) hakkında konuşmadık ki Rust bunları
destekler ancak Rust'un sahiplik kurallarıyla sorunlu olabilir. Eğer iki thread aynı
değiştirilebilir küresel değişkene erişirse, bir veri yarışı (data race) oluşabilir.

Rust'ta küresel değişkenler _statik_ değişkenler olarak adlandırılır. Kod Listesi 20-10,
bir string dilimini değer olarak kullanan bir statik değişkenin örnek bildirimini ve kullanımını
gösterir.

<Listing number="20-10" file-name="src/main.rs" caption="Defining and using an immutable static variable">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-10/src/main.rs}}
```

</Listing>

Statik değişkenler, Bölüm 3'teki [“Declaring Constants”][constants]<!--
ignore --> bölümünde tartıştığımız sabitlere benzer. Statik değişkenlerin adları geleneksel olarak
`SCREAMING_SNAKE_CASE` formatındadır. Statik değişkenler sadece `'static` ömrüyle referansları
saklayabilir ki bu demektir ki Rust derleyicisi ömrü belirleyebilir ve bizden açıkça açıklama
(notate) yapmamız gerekmez. Sabit bir statik değişkene erişmek güvenlidir.

Sabitler ve sabit statik değişkenler arasında ince bir fark şudur ki statik
değişkendeki değerler bellekte sabit bir adres sahiptir. Değeri kullanmak her zaman aynı
veriye erişecektir. Öte yandan, sabitler kullanıldıklarında verilerini kopyalamaya
izin verilir. Başka bir fark şudur ki statik değişkenler değiştirilebilir olabilir. Değiştirilebilir
statik değişkenlere erişmek ve bunları değiştirmek _güvenli değildir_ (unsafe). Kod
Listesi 20-11, `COUNTER` adında değiştirilebilir bir statik değişken nasıl bildirileceğini,
erişileceğini ve değiştirileceğini gösterir.

<Listing number="20-11" file-name="src/main.rs" caption="Reading from or writing to a mutable static variable is unsafe.">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-11/src/main.rs}}
```

</Listing>

Normal değişkenlerde olduğu gibi, `mut` anahtar kelimesini kullanarak değiştirilebilirliği belirtiriz.
`COUNTER`'den okuyan veya yazan herhangi bir kod `unsafe` bloğu içinde olmalıdır. Kod Listesi
20-11 derlenir ve beklediğimiz gibi `COUNTER: 3` yazdırır çünkü tek threadlidır.
`COUNTER`'e erişen çoklu threadleri olması muhtemelen veri yarışlarına sonuçlanır bu yüzden
tanımsız davranıştır. Bu yüzden, tüm fonksiyonu `unsafe` olarak işaretlemeli ve güvenlik
sınırlamasını belgelemeliyiz ki fonksiyonu çağıran herkesin neye izin verdiklerini ve güvenle
yapamayacaklarını bilsin.

Güvenli olmayan bir fonksiyon yazdığımızda, `SAFETY` ile başlayan ve arayanın fonksiyonu
güvenle çağırmak için ne yapması gerektiğini açıklayan bir yorum yazmak idiomatiktir. Benzer
şekilde, güvenli olmayan bir işlem gerçekleştirdiğimizde, güvenlik kurallarının nasıl karşılandığını
açıklamak için `SAFETY` ile başlayan bir yorum yazmak idiomatiktir.

Ek olarak, derleyici, derleyici lint (compiler lint) aracılığıyla varsayılan olarak
değiştirilebilir statik bir değişkene referans oluşturma girişimini reddedecektir.
Ya bu lint'in korumalarından açıkça opt-out yapmalısınız ya da `#[allow(static_mut_refs)]`
açıklamasını eklemeliyiz ya da ham ödünç operatörlerinden biriyle oluşturulan bir ham referans
aracılığıyla değiştirilebilir statik değişkene erişmelisiniz. Bu, referansın görünmez bir şekilde
oluşturulduğu durumları da içerir, örneğin kod listesindeki `println!` içinde kullanıldığında.
Statik değiştirilebilir değişkenlere referansların ham referanslar yoluyla oluşturulmasını
gerektirmek, bunları kullanmanın güvenlik gereksinimlerini daha belirgin hale getirmeye yardımcı olur.

Dünya çapında erişilebilir değiştirilebilir veriyle, veri yarışlarının olmadığından emin olmak
zor olduğu için Rust değiştirilebilir statik değişkenleri güvenli olmayan olarak kabul eder. Mümkün olduğunda,
Bölüm 16'da tartıştığımız eşzamanlılık tekniklerini ve thread-safe akıllı işaretçileri kullanmak
daha yeğ edilebilir ki derleyici farklı threadlerden veri erişiminin güvenli bir şekilde yapıldığını
kontrol eder.

### Güvenli Olmayan Bir Trait Uygulama (Implementing an Unsafe Trait)

Bir trait'i uygulamak için `unsafe` kullanabiliriz. Bir trait en azından bir yönteminin derleyicinin
doğrulayamadığı bazı invariantsa sahip olduğunda güvenli olmazdır. Bir trait'in `unsafe` olduğunu
trait'ten önce `unsafe` anahtar kelimesi ekleyerek ve trait'in uygulamasını da `unsafe` olarak
işaretleyerek bildiririz, Kod Listesi 20-12'da gösterildiği gibi.

<Listing number="20-12" caption="Defining and implementing an unsafe trait">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-12/src/main.rs:here}}
```

</Listing>

`unsafe impl` kullanarak, derleyicinin doğrulayamadığı invariantsı karşılayacağımızı vaat ediyoruz.

Bir örnek olarak, Bölüm 16'deki [“Extensible Concurrency with `Send` and `Sync`”][send-and-sync]<!--
ignore --> bölümünde tartıştığımız `Send` ve `Sync` işaretçi trait'lerini hatırlayın:
Eğer türlerimiz tamamen `Send` ve `Sync` uygulayan diğer türlerden oluşuyorsa derleyici bu trait'leri
otomatik olarak uygular. `Send` veya `Sync` uygulamayan bir tür içeren bir tür uygularsak ve
o türü `Send` veya `Sync` olarak işaretlemek istersek, örneğin ham referanslar, `unsafe`
kullanmalıyız. Rust, türümüzün güvenli bir şekilde threadler arasında gönderilip gönderilemeyeceğini veya
birden çok threadten erişilebileceğini garanti edemez; bu yüzden bu kontrolleri manuel olarak yapmalı
ve `unsafe` ile belirtmeliyiz.

### Bir Union'ın Alanlarına Erişme (Accessing Fields of a Union)

Sadece `unsafe` ile çalışan son işlem, bir union'ın alanlarına erişmektir. Bir *union*
bir `struct` gibidir ancak belirli bir örnekte bir seferde sadece bildirilen bir alan kullanılır.
Union'lar özellikle C kodundaki union'larla etkileşim için kullanılır. Union alanlarına erişmek
güvenli değildir çünkü Rust, union örneğinde şu an depolanan verinin türünü garanti edemez.
Union'lar hakkında daha fazla bilgiyi [Rust Reference][unions]'ten öğrenebilirsiniz.

### Miri Kullanarak Güvenli Olmayan Kodu Kontrol Etme (Using Miri to Check Unsafe Code)

Güvenli olmayan kod yazarken, yazdığınızın aslında güvenli ve doğru olduğunu kontrol etmek
isteyebilirsiniz. Bunu yapmanın en iyi yollarından biri Miri kullanmaktır ki bu, tanımsız
davranışı (undefined behavior) tespit eden resmi bir Rust aracıdır. Ödünç kontrolcüsü
derleme zamanında çalışan bir _statik_ araçtır oysa Miri çalışma zamanında çalışan bir _dinamik_ araçtır.
Programınızı veya test paketini çalıştırarak ve Rust'un nasıl çalışması gerektiği hakkında
kuralları ihlal ettiğinizde tespit ederek kodunuzu kontrol eder.

Miri kullanmak, Rust'un nightly bir derlemesini gerektir ki bunu [Ek G: Rust'un Nasıl Yapılır
ve "Nightly Rust""][nightly]<!-- ignore --> bölümünde daha fazla konuşuyoruz. Hem nightly Rust
versiyonunu hem de Miri aracını `rustup +nightly component add miri` yazarak yükleyebilirsiniz.
Bu projenizin kullandığı Rust versiyonunu değiştirmez; sadece aracı sisteminize ekler böylece
istediğinizde kullanabilirsiniz. Bir projede Miri'yi `cargo +nightly miri run` veya
`cargo +nightly miri test` yazarak çalıştırabilirsiniz.

Bunun ne kadar yardımcı olabileceğine bir örnek olarak, Kod Listesi 20-7'ye karşı
çalıştırdığında ne olacağını düşünün.

```console
{{#include ../listings/ch20-advanced-features/listing-20-07/output.txt}}
```

Miri bize bir tamsayıyı bir referansa cast ettiğimizi doğru şekilde uyarır ki bu bir sorun
olabilir ancak Miri, referansın nasıl kökenlediğini bilmediği için bir sorunun olup olmadığını
belirleyemez. Sonra, Miri Kod Listesi 20-7'nin tanımsız davranışa sahip olduğu bir hata
döndürür çünkü bir sarkan referansımız var (dangling pointer). Miri sayesinde, artık
tanımsız davranış riski olduğunu biliyoruz ve kodu güvenli yapmak hakkında düşünebiliriz. Bazı durumlarda,
Miri hataları nasıl düzeltileceği konusunda öneriler bile verebilir.

Miri, güvenli olmayan kod yazarken yanlış alabileceğiniz her şeyi yakalamaz. Miri bir
dinamik analiz aracıdır bu yüzden sadece aslında çalışan kod sorunlarını yakalar. Bu demektir ki
yazdığınız güvenli olmayan kod hakkında güveninizi artırmak için onu iyi test teknikleriyle
birlikte kullanmanız gerekecek. Miri ayrıca kodunuzun güvenli olamayabileceği her olası yolu
kapsamaz.

Başka bir deyişle: Eğer Miri bir sorunu _yakalarsa_, bir bug olduğunu bilirsiniz ancak
Miri bir bug'ı _yakalamazsa_ bu bir sorunun olmadığı anlamına gelmez. Ancak çok şey
yakalayabilir. Bu bölümdeki güvenli olmayan kodun diğer örneklerinde çalışmayı deneyin ve ne söylediğine bakın!

Miri hakkında daha fazla bilgiyi [GitHub depozitunda][miri] öğrenebilirsiniz.

<!-- Old headings. Do not remove or links may break. -->

<a id="when-to-use-unsafe-code"></a>

### Güvenli Olmayan Kodu Doğru Kullanma (Using Unsafe Code Correctly)

Az önce tartıştığımız beş süper güçten birini kullanmak için `unsafe` kullanmak yanlış
veya hatta kınanmaz ancak derleyicinin bellek güvenliğini sağlayamaması nedeniyle `unsafe`
kodunu doğru yapmak daha zordur. Güvenli olmayan kodu kullanmak için bir nedeniniz olduğunda,
bunu yapabilirsiniz ve açık `unsafe` açıklaması, sorunlar oluştuğunda kaynağı bulmayı kolaylaştırır.
Güvenli olmayan kod yazdığınızda, yazdığınız kodun Rust'un kurallarını karşıladığından daha emin
olmanıza yardımcı olması için Miri'yi kullanabilirsiniz.

Güvenli olmayan Rust ile etkili bir şekilde nasıl çalışmak hakkında çok daha derin bir keşif için,
Rust'un resmi `unsafe` kılavuzunu, [The Rustonomicon][nomicon] okuyun.

[dangling-references]: ch04-02-references-and-borrowing.html#dangling-references
[ABI]: ../reference/items/external-blocks.html#abi
[constants]: ch03-01-variables-and-mutability.html#declaring-constants
[send-and-sync]: ch16-04-extensible-concurrency-sync-and-send.html
[the-slice-type]: ch04-03-slices.html#the-slice-type
[unions]: ../reference/items/unions.html
[miri]: https://github.com/rust-lang/miri
[editions]: appendix-05-editions.html
[nightly]: appendix-07-nightly-rust.html
[nomicon]: https://doc.rust-lang.org/nomicon/