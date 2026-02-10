## Genel Veri Tipleri (Generic Data Types)

Fonksiyon imzaları veya struct'lar gibi öğelerin tanımlarını oluşturmak için jenerikleri kullanıyoruz, sonra bunları birçok farklı somut veri tipi ile kullanabiliriz. Önce fonksiyonların, struct'ların, enum'ların ve yöntemlerin jenerikleri kullanarak nasıl tanımlanacağına bakalım. Sonra, jeneriklerin kod performansını nasıl etkilediğini tartışacağız.

### Fonksiyon Tanımlarında (In Function Definitions)

Jenerik kullanan bir fonksiyon tanımlarken, jenerikleri genellikle parametrelerin ve dönüş değerinin veri tiplerini belirttiğimiz fonksiyon imzasında yerleştiriyoruz. Bunu yapmak kodumuzu daha esnek hale getirir ve kod tekrarını önlerken fonksiyonun çağıranlarına daha fazla işlevsellik sağlar.

`largest` fonksiyonumuzla devam ederek, Kod Listesi 10-4, her ikisi de bir dilimdeki en büyük değeri bulan iki fonksiyon gösterir. Sonra bunları jenerik kullanan tek bir fonksiyonda birleştireceğiz.

<Listing number="10-4" file-name="src/main.rs" caption="Sadece isimlerinde ve imzalarındaki tiplerde farklı olan iki fonksiyon">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-04/src/main.rs:here}}
```

</Listing>

`largest_i32` fonksiyonu, Kod Listesi 10-3'te çıkardığımız ve bir dilimdeki en büyük `i32` değerini bulan fonksiyondur. `largest_char` fonksiyonu, bir dilimdeki en büyük `char` değerini bulur. Fonksiyon gövdeleri aynı koda sahiptir, bu yüzden tek bir fonksiyonda bir jenerik tip parametresi tanıtarak tekrarlamayı kaldıralım.

Yeni tek bir fonksiyondaki tipleri parametrize etmek için, fonksiyona değer parametrelerini adlandırdığımız gibi tip parametresini de adlandırmamız gerekir. Tip parametresi adı olarak herhangi bir tanımlayıcıyı kullanabilirsiniz. Ancak `T` kullanacağız, çünkü kural olarak Rust'taki tip parametresi adları kısadır, genellikle tek bir harftir ve Rust'ın tip adlandırma kuralı UpperCamelCase'dir. _Type_'ın (tip) kısaltması olan `T`, çoğu Rust programcısının varsayılan seçimidir.

Fonksiyon gövdesinde bir parametre kullandığımızda, derleyicinin bu adın ne anlama geldiğini bilmesi için parametre adını imzada bildirmemiz gerekir. Benzer şekilde, bir fonksiyon imzasında bir tip parametresi adı kullandığımızda, kullanmadan önce tip parametresi adını bildirmemiz gerekir. Genel `largest` fonksiyonunu tanımlamak için, tip adı bildirimlerini açılı ayraçlar içine, `<>`, fonksiyon adı ve parametre listesi arasına şu şekilde yerleştiriyoruz:

```rust,ignore
fn largest<T>(list: &[T]) -> &T {
```

Bu tanımlamayı "Fonksiyon `largest`, bir tür `T` üzerinden jeneriktir" şeklinde okuruz. Bu fonksiyon `list` adında bir parametre sahiptir ki bu, `T` tipindeki değerlerin bir dilimidir. `largest` fonksiyonu, aynı `T` tipindeki bir değere bir referans döndürecektir.

Kod Listesi 10-5, imzasında genel veri tipi kullanan birleştirilmiş `largest` fonksiyon tanımını gösterir. Liste ayrıca fonksiyonu `i32` değerlerinin dilimiyle veya `char` değerlerinin dilimiyle nasıl çağırabileceğimizi de gösterir. Bu kodun henüz derlenmediğini unutmayın.

<Listing number="10-5" file-name="src/main.rs" caption="Jenerik tip parametrelerini kullanan `largest` fonksiyonu; bu henüz derlenmez">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/src/main.rs}}
```

</Listing>

Şu anda bu kodu derlersek, şu hatayı alırız:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/output.txt}}
```

Yardım metni `std::cmp::PartialOrd`'den bahseder ki bu bir trait'tir ve bir sonraki bölümde trait'lerden bahsedeceğiz. Şimdilik, bu hatanın `largest` fonksiyon gövdesinin `T`'nin olabileceği tüm olası tipler için çalışmayacağını söylediğini bilin. Çünkü fonksiyon gövdesinde `T` tipindeki değerleri karşılaştırmak istiyoruz, sadece değerleri sıralanabilen tipleri kullanabiliriz. Karşılaştırmaları etkinleştirmek için, standart kütüphane tipler üzerinde uygulayabileceğiniz `std::cmp::PartialOrd` trait'ine sahiptir (bu trait hakkında daha fazla bilgi için Ek C'ye bakın). Kod Listesi 10-5'i düzeltmek için, yardım metninin önerisini takip edebilir ve `T` için geçerli tipleri sadece `PartialOrd` uygulayanlarla sınırlayabiliriz. Liste sonra derlenecektir, çünkü standart kütüphane hem `i32` hem de `char` üzerinde `PartialOrd` uygular.

### Struct Tanımlarında (In Struct Definitions)

Ayrıca, struct tanımlarında bir veya daha fazla alanda genel tip parametresi kullanmak için `<>` sözdizimini kullanarak struct'ları da tanımlayabiliriz. Kod Listesi 10-6, `x` ve `y` koordinat değerlerini herhangi bir tipinde tutan bir `Point<T>` struct'ı tanımlar.

<Listing number="10-6" file-name="src/main.rs" caption="`T` tipindeki `x` ve `y` değerlerini tutan `Point<T>` struct'ı">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-06/src/main.rs}}
```

</Listing>

Struct tanımlarında jenerikleri kullanma sözdizimi, fonksiyon tanımlarında kullanılan sözdizimine benzer. Önce, struct adının hemen arkasındaki açılı ayraçlar içinde tip parametresinin adını bildiriyoruz. Sonra, struct tanımında başka somut veri tipleri belirttiğimiz yerde genel tipi kullanıyoruz.

Yalnızca bir genel tip kullanarak `Point<T>`'yi tanımladığımız için, bu tanım `Point<T>` struct'ının bir tür `T` üzerinden jenerik olduğunu ve `x` ve `y` alanlarının her ikisinin de o tiplerin her ne olursa olsun aynı tip olduğunu söylediğini unutmayın. Kod Listesi 10-7'de gösterildiği gibi, farklı tiplerde değerlere sahip bir `Point<T>` örneği oluşturursak, kodumuz derlenmez.

<Listing number="10-7" file-name="src/main.rs" caption="`x` ve `y` alanları aynı genel veri tipi `T`'ye sahip olduğu için aynı tip olmalıdır.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/src/main.rs}}
```

</Listing>

Bu örnekte, `x`'e tamsayı değerini `5` atadığımızda, bu `Point<T>` örneği için genel tip `T`'nin bir tamsayı olacağını derleyiciye bildiriyoruz. Sonra, `x` ile aynı tipte olması gereken `y` için `4.0` belirttiğimizde, şöyle bir tip uyumsuzluk hatası alırız:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/output.txt}}
```

`x` ve `y` her ikisi de jenerik ancak farklı tiplerde olabilir bir `Point` struct'ı tanımlamak için, birden fazla genel tip parametresi kullanabiliriz. Örneğin, Kod Listesi 10-8'de, `x`'in `T` tipinde ve `y`'nin `U` tipinde olduğu `T` ve `U` tipleri üzerinden jenerik olan `Point` tanımını değiştiriyoruz.

<Listing number="10-8" file-name="src/main.rs" caption="İki tip üzerinden jenerik olan `Point<T, U>` böylece `x` ve `y` farklı tiplerde değerler olabilir">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-08/src/main.rs}}
```

</Listing>

Artık gösterilen `Point`'in tüm örneklerine izin veriliyor! Bir tanımda istediğiniz kadar genel tip parametresi kullanabilirsiniz, ancak birkaçtan fazlasını kullanmak kodunuzun okunmasını zorlaştırır. Kodunuzda çok sayıda genel tip kullanmanız gerektiğini fark ederseniz, kodunuzun daha küçük parçalara yeniden yapılandırılması gerektiğini gösterebilir.

### Enum Tanımlarında (In Enum Definitions)

Struct'larla yaptığımız gibi, enum'ları da varyantlarında genel veri tipleri tutmak için tanımlayabiliriz. Standart kütüphanenin sağladığı ve Bölüm 6'da kullandığımız `Option<T>` enum'ına başka bir bakalım:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

Bu tanım artık size daha fazla anlam ifade etmelidir. Görebileceğiniz gibi, `Option<T>` enum'ı `T` tipi üzerinden jeneriktir ve iki varyantı vardır: `T` tipinde bir değer tutan `Some` ve herhangi bir değer tutmayan bir `None` varyantı. `Option<T>` enum'ını kullanarak, isteğe bağlı bir değerin soyut kavramını ifade edebiliriz ve `Option<T>` jenerik olduğu için, isteğe bağlı değerin tipi ne olursa olsun bu soyutlamayı kullanabiliriz.

Enum'lar da birden fazla genel tip kullanabilir. Bölüm 9'da kullandığımız `Result` enum'ının tanımı bunun bir örneğidir:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`Result` enum'ı `T` ve `E` olmak üzere iki tip üzerinden jeneriktir ve iki varyantı vardır: `T` tipinde bir değer tutan `Ok` ve `E` tipinde bir değer tutan `Err`. Bu tanım, bir şekilde başarılı olabilir (bir `T` tipinde değer döndürür) veya başarısız olabilir (bir `E` tipinde hata döndürür) herhangi bir işlemin olduğu yerlerde `Result` enum'ını kullanmayı kolaylaştırır. Aslında, bu, Kod Listesi 9-3'te bir dosyayı açmak için kullandığımız şeydir; dosya başarıyla açıldığında `T` `std::fs::File` tipiyle dolduruldu ve dosyayı açarken sorunlar olduğunda `E` `std::io::Error` tipiyle dolduruldu.

Kodunuzda, sadece tuttukları değerlerin tiplerinde farklı olan birden fazla struct veya enum tanımı olan durumları tanıdığınızda, jenerik tipleri kullanarak tekrarlamayı önleyebilirsiniz.

### Yöntem Tanımlarında (In Method Definitions)

Struct'lar ve enum'lar üzerinde yöntemler uygulayabiliriz (Bölüm 5'te yaptığımız gibi) ve tanımlarında jenerik tipleri de kullanabiliriz. Kod Listesi 10-9, Kod Listesi 10-6'da tanımladığımız `Point<T>` struct'ını ve onun üzerinde uygulanan `x` adında bir yöntem gösterir.

<Listing number="10-9" file-name="src/main.rs" caption="`T` tipindeki `x` alanına bir referans döndüren `Point<T>` struct'ında `x` adında bir yöntemin uygulanması">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-09/src/main.rs}}
```

</Listing>

Burada, `Point<T>` üzerinde `x` adında bir yöntem tanımladık ve bu yöntem, `x` alanındaki verilere bir referans döndürüyor.

`T`'yi `impl`'in hemen arkasından bildirmemiz gerektiğini unutmayın, böylece `T`'yi `Point<T>` tipi üzerinde yöntemler uyguladığımızı belirtmek için kullanabiliriz. `T`'yi `impl`'den sonra bir genel tip olarak bildirerek, Rust `Point`'teki açılı ayraçlardaki tipin bir genel tip olduğunu, somut bir tip olmadığını tanıyabilir. Bu genel parametre için struct tanımında bildirilen genel parametreden farklı bir ad seçebilirdik, ancak aynı adı kullanmak gelenekseldir. Genel bir tip bildiren bir `impl` içinde bir yöntem yazarsanız, o yöntem tipin herhangi bir örneği üzerinde tanımlanır, genel tipin yerine hangi somut tip geçerse geçsin.

Tip üzerinde yöntemler tanımlarken genel tipler üzerinde kısıtlamalar da belirtebiliriz. Örneğin, herhangi bir genel tipteki `Point<T>` örnekleri yerine sadece `Point<f32>` örneklerinde yöntemler uygulayabiliriz. Kod Listesi 10-10'da, `impl`'den sonra herhangi bir tip bildirmiyoruz, yani somut tip `f32` kullanıyoruz.

<Listing number="10-10" file-name="src/main.rs" caption="Sadece genel tip parametresi `T` için belirli bir somut tipi olan bir struct için geçerli olan bir `impl` bloğu">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-10/src/main.rs:here}}
```

</Listing>

Bu kod, `Point<f32>` tipinin bir `distance_from_origin` yöntemine sahip olacağı anlamına gelir; `T` tipi `f32` olmayan `Point<T>`'nin diğer örneklerinde bu yöntem tanımlanmayacaktır. Yöntem, noktamızın (0.0, 0.0) koordinatlarındaki noktadan ne kadar uzakta olduğunu ölçer ve sadece kayan nokta tiplerinde mevcut olan matematiksel işlemleri kullanır.

Struct tanımındaki genel tip parametreleri, aynı struct'ın yöntem imzalarında kullandığınız tip parametreleriyle her zaman aynı değildir. Kod Listesi 10-11, örneği daha netleştirmek için `Point` struct'ı için `X1` ve `Y1` genel tiplerini ve `mixup` yöntem imzası için `X2` ve `Y2` genel tiplerini kullanır. Yöntem, `self` `Point`'ten (`X1` tipinde) gelen `x` değerinden ve geçirilen `Point`'ten (`Y2` tipinde) gelen `y` değerinden yeni bir `Point` örneği oluşturur.

<Listing number="10-11" file-name="src/main.rs" caption="Struct'ının tanımından farklı genel tipler kullanan bir yöntem">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-11/src/main.rs}}
```

</Listing>

`main` içinde, `x` için `i32` (değer `5`) ve `y` için `f64` (değer `10.4`) olan bir `Point` tanımladık. `p2` değişkeni, `x` için bir string dilimi (değer `"Hello"`) ve `y` için bir `char` (değer `c`) olan bir `Point` struct'ıdır. `p1` üzerinde `p2` argümanıyla `mixup` çağırmak bize `p3` verir ki bu, `x`'in `p1`'den geldiği için `x` için `i32` olacaktır. `p3` değişkeni, `y`'nin `p2`'den geldiği için `y` için `char` olacaktır. `println!` makro çağrısı `p3.x = 5, p3.y = c` yazdıracaktır.

Bu örneğin amacı, bazı genel parametrelerin `impl` ile bildirildiği ve bazılarının yöntem tanımı ile bildirildiği bir durumu göstermektir. Burada, genel parametreler `X1` ve `Y1` `impl`'den sonra bildirilir, çünkü struct tanımıyla birlikte giderler. Genel parametreler `X2` ve `Y2` `fn mixup`'dan sonra bildirilir, çünkü sadece yöntemle ilgilidirler.

### Jenerikleri Kullanan Kodun Performansı (Performance of Code Using Generics)

Jenerik tip parametrelerini kullanmanın çalışma zamanı (runtime) maliyeti olup olmadığını merak ediyor olabilirsiniz. İyi haber şu ki, genel tipleri kullanmak programınızın somut tiplerle olduğundan daha yavaş çalışmasına neden olmaz.

Rust bunu, jenerikleri kullanan kod üzerinde derleme zamanında monomorfizasyon (monomorphization) gerçekleştirerek başarır. _Monomorfizasyon_, derlendiğinde kullanılan somut tipleri doldurarak genel kodu belirli koda dönüştürme işlemidir. Bu süreçte, derleyici Kod Listesi 10-5'te genel fonksiyonu oluşturmak için kullandığımız adımların tersini yapar: Derleyici, genel kodun çağrıldığı tüm yerlere bakar ve genel kodun çağrıldığı somut tipler için kod oluşturur.

Bunun nasıl çalıştığına standart kütüphanenin genel `Option<T>` enum'ını kullanarak bakalım:

```rust
let integer = Some(5);
let float = Some(5.0);
```

Rust bu kodu derlediğinde, monomorfizasyon gerçekleştirir. Bu süreçte, derleyici `Option<T>` örneklerinde kullanılan değerleri okur ve iki tür `Option<T>`'yi tanımlar: Biri `i32` ve diğeri `f64`. Bu nedenle, `Option<T>`'nin genel tanımını `i32` ve `f64`'e özelleştirilmiş iki tanıma genişletir, böylece genel tanımı belirli olanlarla değiştirir.

Kodun monomorfize edilmiş versiyonu aşağıdakine benzer (derleyici açıklama için kullandığımızdan farklı isimler kullanır):

<Listing file-name="src/main.rs">

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

</Listing>

Genel `Option<T>`, derleyici tarafından oluşturulan belirli tanımlarla değiştirilir. Çünkü Rust genel kodu her örnekte tipi belirten koda derler, jenerikleri kullanmak için çalışma zamanı maliyeti ödemeyiz. Kod çalıştığında, her tanımı elle tekrarlamış olsaydık gibi davranır. Monomorfizasyon süreci, Rust'ın jeneriklerini çalışma zamanında son derece verimli hale getirir.