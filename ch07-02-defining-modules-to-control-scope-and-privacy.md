<!-- Old headings. Do not remove or links may break. -->

<a id="defining-modules-to-control-scope-and-privacy"></a>

## Modüllerle Kapsam ve Gizlilik Kontrol Etme (Control Scope and Privacy with Modules)

Bu bölümde modüller ve modül sisteminin diğer parçaları, yani ögelere adlandırma izni veren _yollar (paths)_; bir yolu kapsama getiren `use` anahtar kelimesi; ve ögeleri genel yapan `pub` anahtar kelimesini konuşacağız. Ayrıca `as` anahtar kelimesini, dış paketleri ve glob operatörünü de konuşacağız.

### Modüller Hızlı Referans (Modules Cheat Sheet)

Modüller ve yolların detaylarına geçmeden önce, modüllerin, yolların, `use` anahtar kelimesinin ve `pub` anahtar kelimesinin derleyicide nasıl çalıştığı ve çoğu geliştiricinin kodunu nasıl organize ettiği hakkında hızlı bir referans sağlıyoruz. Bu bölüm boyunca bu kuralların her birinin örneklerini geçeceğiz ancak bu modüllerin nasıl çalıştığı hakkında hatırlatıcı olarak başvurmak için harika bir yer.

- **Kafa kökünden başlayın**: Bir kafayı derlerken, derleyici önce kafa kök dosyasına (kütüphane kafası için genellikle _src/lib.rs_, ikili kafa için _src/main.rs_) derlenecek kod için bakar.
- **Modül bildirimi**: Kafa kök dosyasında, yeni modüller bildirebilirsiniz; örneğin `mod garden;` ile "garden" modülü bildirebilirsiniz. Derleyici modülün kodunu şu yerlerde arar:
  - Satır içi, `mod garden`'i takip eden noktalı virgül yerine süslü parantezler içinde
  - _src/garden.rs_ dosyasında
  - _src/garden/mod.rs_ dosyasında
- **Alt modül bildirimi**: Kafa kökü dışındaki herhangi bir dosyada, alt modülleri bildirebilirsiniz. Örneğin, _src/garden.rs_'de `mod vegetables;` bildirebilirsiniz. Derleyici alt modülün kodunu şu yerlerde, ebeveyn modülün adı verilen klasör içinde arar:
  - Satır içi, `mod vegetables`'ı takiben doğrudan, noktalı virgül yerine süslü parantezler içinde
  - _src/garden/vegetables.rs_ dosyasında
  - _src/garden/vegetables/mod.rs_ dosyasında
- **Modüllerdeki kod yolları**: Bir modül kafanızın bir parçası olduğunda, gizlilik kurallarına izin verdiği sürece, aynı kafanın başka herhangi bir yerinden kodu koda yolu kullanarak atıfta bulunabilirsiniz. Örneğin, garden vegetables modülündeki `Asparagus` tipi `crate::garden::vegetables::Asparagus` adresinde bulunur.
- **Özel vs. Genel**: Bir modül içindeki kod varsayılan olarak ebeveyn modüllerinden özeldir (private). Bir modülü genel yapmak için `mod` yerine `pub mod` ile bildirin. Genel bir modül içindeki ögeleri de genel yapmak için, bildirimlerinin önünde `pub` kullanın.
- **`use` anahtar kelimesi**: Bir kapsamda, `use` anahtar kelimesi uzun yolların tekrarını azaltmak için ögelere kısayollar oluşturur. `crate::garden::vegetables::Asparagus`'e atıfta bulunabilen herhangi bir kapsamda, `use crate::garden::vegetables::Asparagus;` ile kısayol oluşturabilirsiniz ve o andan itibaren o tipi kapsamda kullanmak için sadece `Asparagus` yazmanız gerekir.

Burada, bu kuralları gösteren `backyard` adında bir ikili kafa oluşturuyoruz. Kafanın klasörü, aynı zamanda _backyard_, şu dosyaları ve klasörleri içerir:

```text
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

Bu durumda kafa kök dosyası _src/main.rs_'dir ve şunları içerir:

<Listing file-name="src/main.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/main.rs}}
```

</Listing>

`pub mod garden;` satırı derleyiciye _src/garden.rs_'de bulduğu kodu dahil etmesini söyler ki bu:

<Listing file-name="src/garden.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden.rs}}
```

</Listing>

Burada, `pub mod vegetables;` _src/garden/vegetables.rs_'deki kodun de dahil edildiği anlatır. O kod:

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden/vegetables.rs}}
```

Şimdi bu kuralların detaylarına geçelim ve onları eylemde gösterelim!

### Modüllerde İlgili Kod Gruplama (Grouping Related Code in Modules)

_Modüller_ okunabilirlik ve kolay yeniden kullanım için bir kafa içinde kodu organize etmemizi sağlar. Modüller ayrıca ögelerin _gizliliğini_ (privacy) kontrol etmemize izin verir çünkü bir modül içindeki kod varsayılan olarak özeldir. Özel ögeler dış kullanım için kullanılamayan iç uygulama detaylarıdır. Modülleri ve içindeki ögeleri genel yapmayı seçebiliriz ki bu onları dış kodun kullanmasına ve bağımlı olmasına izin vermek için açığa çıkarır.

Örnek olarak, bir restoranın işlevselliğini sağlayan bir kütüphane kafası yazalım. Fonksiyonların imzalarını tanımlayacağız ancak gövdelerini boş bırakacağız ki restoranın uygulaması yerine kodun organizasyonuna konsantre olalım.

Restoran endüstrisinde, restoranın bazı kısımları ön bölüm (front of house) ve diğerleri arka bölüm (back of house) olarak adlandırılır. _Ön bölüm_ müşterilerin olduğu yerdir; bu hostların müşterileri oturttuğu yerleri, sunucuların siparişleri ve ödemeleri aldığı yerleri ve barların içecekler yaptığı yerleri kapsar. _Arka bölüm_ şeflerin ve aşçıların mutfakta çalıştığı, bulaşçıların temizlik yaptığı ve yöneticilerin idari iş yaptığı yerdir.

Kafamızı bu şekilde yapılandırmak için, fonksiyonlarını iç içe geçmiş modüllere organize edebiliriz. `cargo new restaurant --lib` çalıştırarak `restaurant` adında yeni bir kütüphane oluşturun. Sonra, bazı modülleri ve fonksiyon imzalarını tanımlamak için _src/lib.rs_'ye Kod Listesi 7-1'deki kodu girin; bu kod ön bölüm bölümüdür.

<Listing number="7-1" file-name="src/lib.rs" caption="Fonksiyonları içeren diğer modülleri içeren bir `front_of_house` modülü">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-01/src/lib.rs}}
```

</Listing>

`mod` anahtar kelimesini ve ardından modül adını (bu durumda `front_of_house`) kullanarak bir modül tanımlarız. Modülün gövdesi süslü parantezlerin içine girer. Modüller içinde, `hosting` ve `serving` modülleri ile olduğu gibi, diğer modülleri yerleştirebiliriz. Modüller diğer ögeler için de tanımlar tutabilirler, örneğin yapılar, enumler, sabitler, traitler ve Kod Listesi 7-1'deki gibi fonksiyonlar.

Modülleri kullanarak, ilgili tanımları birlikte gruplayabilir ve niçin ilişkili olduklarını adlandırabiliriz. Bu kodu kullanan programcılar tüm tanımları okumak zorunda kalmak yerine gruplara göre kodda gezinebilirler ki onları ilgilendiren tanımları bulmayı kolaylaştırır. Bu koda yeni işlevsellik ekleyen programcılar programı organize tutmak için kodu nereye yerleştireceklerini bileceklerdir.

Daha önce, _src/main.rs_ ve _src/lib.rs_'nin _kafa kökleri_ olarak adlandırıldığını belirtmiştik. Adlarının sebebi, bu iki dosyanın her birinin içeriği kafanın modül yapısının kökünde `crate` adında bir modül oluşturmasıdır ki bu _modül ağacı_ olarak bilinir.

Kod Listesi 7-2, Kod Listesi 7-1'deki yapı için modül ağacını gösterir.

<Listing number="7-2" caption="Kod Listesi 7-1'deki kodun modül ağacı">

```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

</Listing>

Bu ağaç bazı modüllerin diğer modüllerin içine nasıl iç içe geçtiğini gösterir; örneğin, `hosting` `front_of_house` içinde iç içe geçer. Ağaç ayrıca bazı modüllerin _kardeşler_ olduğunu gösterir ki bu aynı modülde tanımlandıkları anlamına gelir; `hosting` ve `serving`, `front_of_house` içinde tanımlanmış kardeşlerdir. Eğer modül A modül B içinde kapsanıyorsa, modül A'nın modül B'nin _çocuğu_ olduğunu ve modül B'nin modül A'nın _ebeveyni_ olduğunu söyleriz. Tüm modül ağacının `crate` adında örtük modül altında köklendiğini fark edin.

Modül ağacı sizin bilgisayarınızdaki dosya sisteminin klasör ağacını hatırlatabilir; bu çok uygun bir karşılaştırma! Tıpkı bir dosya sistemindeki klasörler gibi, kodunuzu organize etmek için modülleri kullanırsınız. Ve tıpkı bir klasördeki dosyalar gibi, modüllerimizi bulmak için bir yola ihtiyacımız var.