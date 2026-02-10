# Genel Tipler, Traitler ve Yaşam Süreleri (Generic Types, Traits, and Lifetimes)

Her programlama dilinin kavramların tekrarını etkili bir şekilde ele alma araçları vardır. Rust'ta, böyle bir araç _jeneriklerdir_ (generics): somut tiplerin veya başka özelliklerin soyut yer tutucular. Jeneriklerin davranışını veya derleme ve kodu çalıştırdığında onların yerine ne olacağını bilmeden diğer jeneriklerle nasıl ilişkidebileceğimizi ifade edebiliriz.

Fonksiyonlar, `i32` veya `String` gibi somut bir tip yerine bazı jenerik tibin parametrelerini alabilirler, aynı şekilde bilinmeyen değerleri parametre olarak alıp birden fazla somut değerde aynı kodu çalıştırırlar. Aslında, biz daha önce Bölüm 6'da `Option<T>`, Bölüm 8'de `Vec<T>` ve `HashMap<K, V>`, ve Bölüm 9'da `Result<T, E>` ile jenerikler kullandık. Bu bölümde, kendi tiplerinizi, fonksiyonlarınızı ve yöntemlerinizi jeneriklerle nasıl tanımlayacağınızı keşfedeceksiniz!

Önce, kod tekrarını azaltmak için bir fonksiyonu nasıl çıkaracağımızı (extract) gözden geçireceğiz. Sonra, sadece parametrelerinin tiplerinde farklı olan iki fonksiyondan bir jenerik fonksiyon yapmak için aynı tekniği kullanacağız. Ayrıca struct ve enum tanımlarında jenerik tipleri nasıl kullanacağımızı açıklayacağız.

Sonra, traitleri kullanarak davranışı jenerik bir şekilde nasıl tanımlayacağınızı öğreneceksiniz. Traitleri jenerik tiplerle birleştirip bir jenerik tiyi sadece belirli bir davranışa sahip olan tipleri kabul etmesi için kısıtlayabilirsiniz, herhangi bir tipi kabul etmesine zıdde.

Son olarak, _yaşam sürelerini_ (lifetimes) tartışacağız: derleyiciye referansların birbirleriyle nasıl ilişkideceği hakkında bilgi veren bir çeşit jenerik. Yaşam süreleri bize ödünç değerler hakkında derleyiciye yeterli bilgi verir ki böylece bizim yardımımız olmadan daha fazla durumda referansların geçerli olacağını sağlar.

## Bir Fonksiyonu Çıkararak Tekrarı Kaldırmak (Removing Duplication by Extracting a Function)

Jenerikler bize birden fazla tipi temsil eden bir yer tutucu ile somut tipleri değiştirip kod tekrarını kaltırmamıza izin verir. Jenerikler sözdizimine (syntax) dalmadan önce, jenerik tipleri içermeyen bir şekilde tekrarı bir fonksiyonu çıkararak (extracting) ve somut değerleri birden fazla değeri temsil eden bir yer tutucu ile değiştirerek nasıl kaldırdığımıza bakalım. Sonra, aynı tekniği bir jenerik fonksiyonu çıkarmak için uygulayacağız! Kod tekrarını nasıl tanıyabileceğinizi ve fonksiyona çıkarabileceğinizi gözleyerek, jenerikleri kullanabilebilecek kod tekrarını tanımaya başlayacaksınız.

Kod Listesi 10-1'deki sayı listesinde en büyük sayıyı bulan kısa bir programla başlayacağız.

<Listing number="10-1" file-name="src/main.rs" caption="Bir sayı listesinde en büyük sayıyı bulma">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-01/src/main.rs:here}}
```

</Listing>

Tam sayıları bir `number_list` değişkeninde depoluyoruz ve listedeki ilk sayıyı `largest` adında bir değişkende bir referans yerleştiriyoruz. Sonra, listedeki tüm sayılar üzerinden yineliyoruz (iterate) ve eğer güncel sayı `largest` içinde saklanan sayıdan büyükse, bu değişkendeki referansı değiştiriyoruz. Ancak, eğer güncel sayı şu ana kadar görülen en büyük sayıdan küçük veya eşitse, değişken değişmez ve kod listedeki bir sonraki sayıya geçer. Listedeki tüm sayıları gözettikten sonra, `largest`, bu durumda 100 olan en büyük sayıya referan edecektir.

Şimdi iki farklı sayı listesinde en büyük sayıyı bulmakla görevlendirildik. Bunu yapmak için, Kod Listesi 10-1'deki kodu tekrarlamayı seçebiliriz ve programın iki farklı yerinde aynı mantığı, Kod Listesi 10-2'de gösterildiği gibi kullanabiliriz.

<Listing number="10-2" file-name="src/main.rs" caption="İki sayı listesinde en büyük sayıyı bulan kod">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-02/src/main.rs}}
```

</Listing>

Bu kod çalışmasına ragmen, kodu tekrarlamak sıkıcı ve hataya meyilli (error-prone)dir. Ayrıca kodu değiştirmek istediğimizde birden fazla yeri hatırlamalıyız.

Bu tekrarlamayı yok etmek için, parametre olarak geçirilen herhangi bir sayı listesinde çalışan bir fonksiyon tanımlayarak bir soyutlama (abstraction) oluşturacağız. Bu çözüm kodunuzu daha temiz hale getirir ve bir sayı listesinde en büyük sayıyı bulma kavramını soyut olarak ifade etmenizi sağlar.

Kod Listesi 10-3'te, en büyük sayıyı bulan kodu `largest` adında bir fonksiyona çıkarıyoruz. Sonra, Kod Listesi 10-2'deki iki listeden en büyük sayıyı bulmak için fonksiyonu çağırıyoruz. Gelecekte sahip olabilecek herhangi bir `i32` değer listesinde de bu fonksiyonu kullanabiliriz.

<Listing number="10-3" file-name="src/main.rs" caption="İki listede de en büyük sayıyı bulamak için soyutlanmış kod">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-03/src/main.rs:here}}
```

</Listing>

`largest` fonksiyonu `list` adında bir parametre sahiptir ki bu, fonksiyona geçirmiş olabileceğimiz `i32` değerlerinin herhangi bir somut dilimini (slice) temsil eder. Sonuç olarak, fonksiyonu çağırdığımızda, kod geçirdiğimiz somut değerler üzerinde çalışır.

Özetle, işte Kod Listesi 10-2'den Kod Listesi 10-3'e kodu değiştirmek için attığımız adımlar:

1. Tekrarlanan kodu tanıyın.
1. Tekrarlanan kodu fonksiyon gövdesine çıkarın ve fonksiyon imzasında (signature) o kodun giriler ve dönüş değerlerini belirtin.
1. Tekrarlanan kodun iki kopyasını fonksiyonu çağırmak için güncelleyin.

Sonra, kod tekrarını azaltmak için bu aynı adımları jeneriklerle kullanacağız. Aynı şekilde ki fonksiyon gövdesi somut değerler yerine soyut bir `list` üzerinde çalışabilir, jenerikler kodun soyut tipler üzerinde çalışmasına izin verir.

Örneğin, diyelim ki iki fonksiyonumuz var: biri `i32` değerlerinin diliminde en büyük öğeyi bulan ve diğeri `char` değerlerinin diliminde en büyük öğeyi bulan. Bu tekrarlamayı nasıl yok ederdik? Bulalım!
