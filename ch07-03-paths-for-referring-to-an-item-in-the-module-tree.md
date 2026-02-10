## Modül Ağacındaki Bir Ögeyi Atıfta Bulmak İçin Yollar (Paths for Referring to an Item in Module Tree)

Rust'a bir modül ağacındaki bir ögeyi nerede bulacağını göstermek için, bir dosya sisteminde gezinirken kullandığımız aynı şekilde bir yol kullanırız. Bir fonksiyonu çağırmak için yolunu bilmemiz gerekir.

Bir yol iki form alabilir:

- Bir _mutlak yol (absolute path)_, bir kafa kökünden başlayan tam yoldur; dış bir kafadan gelen kod için, mutlak yol kafa adıyla başlar ve mevcut kafadan gelen kod için, gerçek `crate` ile başlar.
- Bir _göreceli yol (relative path)_, mevcut modülden başlar ve mevcut modülden `self`, `super` veya bir tanımlayıcı kullanır.

Hem mutlak hem de göreceli yollar çift nokta (`::`) ile ayrılmış bir veya daha fazla tanımlayıcıyı takip eder.

Kod Listesi 7-1'e geri dönelim, `add_to_waitlist` fonksiyonunu çağırmak istediğimizi söyleyin. Bu, `add_to_waitlist` fonksiyonunun yolunun ne olduğuyla aynıdır. Kod Listesi 7-3, bazı modüllerin ve fonksiyonların kaldırılmış Kod Listesi 7-1'i içerir.

`eat_at_restaurant` adında yeni bir fonksiyondan `add_to_waitlist` fonksiyonunu çağırmanın iki yolunu göstereceğiz ki bu fonksiyon kafa kökünde tanımlanmıştır. Bu yollar doğrudur ancak örneğin olduğu gibi derlenmesini engelleyecek başka bir sorun kalır. Nedenini biraz açıklayacağız.

`eat_at_restaurant` fonksiyonumuzun kütüphane kafasının genel API'sinin bir parçası olduğu için, onu `pub` anahtar kelimesiyle işaretliyoruz. ["Yolları `pub` Anahtar Kelimesiyle Açığa Çıkarma" (Exposing Paths with `pub` Keyword)][pub]<!-- ignore --> bölümünde, `pub` hakkında daha fazla detaya gireceğiz.

<Listing number="7-3" file-name="src/lib.rs" caption="Mutlak ve göreceli yollar kullanarak `add_to_waitlist` fonksiyonunu çağırma">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-03/src/lib.rs}}
```

</Listing>

`eat_at_restaurant` içinde `add_to_waitlist` fonksiyonunu ilk kez çağırdığımızda, bir mutlak yol kullanıyoruz. `add_to_waitlist` fonksiyonu `eat_at_restaurant` ile aynı kafada tanımlanmıştır ki bu `crate` anahtar kelimesini kullanarak mutlak bir yol başlatmamızı sağlar. Sonra `add_to_waitlist`'a ulaşana kadar ardışık her modülü dahil ederiz. Aynı yapıya sahip bir dosya sistemini hayal edebilirsiniz: `add_to_waitlist` programını çalıştırmak için `/front_of_house/hosting/add_to_waitlist` yolunu belirtirdik; kafa kökünden başlatmak için `crate` adını kullanmak kablonuzda dosya sistemi kökünden başlatmak için `/` kullanmak gibi.

`eat_at_restaurant` içinde `add_to_waitlist`'i ikinci kez çağırdığımızda, bir göreceli yol kullanıyoruz. Yol `front_of_house` ile başlar ki bu `eat_at_restaurant` ile modül ağacının aynı seviyesinde tanımlanmış modül adıdır. Burada dosya sistemi eşdeğeri `front_of_house/hosting/add_to_waitlist` yolunu kullanmak olurdu. Bir modül adıyla başlamak yolun göreceli olduğu anlamına gelir.

Göreceli veya mutlak yol kullanıp kullanmamak seçimi projenize dayalı karar vermeniz gereken bir şeydir ve öge tanımlama kodunu ögeyi kullanan koddan ayrı veya birlikte taşıma olasılığınıza bağlıdır. Örneğin, eğer `front_of_house` modülünü ve `eat_at_restaurant` fonksiyonunu `customer_experience` adında bir modüle taşırsak, `add_to_waitlist`'e mutlak yolu güncellememiz gerekir ancak göreceli yol hala geçerli olur. Ancak, eğer `eat_at_restaurant` fonksiyonunu ayrı olarak `dining` adında bir modüle taşırsak, `add_to_waitlist` çağrısına mutlak yol aynı kalır ancak göreceli yol güncellenmesi gerekir. Genel olarak tercihimiz mutlak yollar belirtmektir çünkü kod tanımlarını ve öge çağrılarını birbirlerinden bağımsız taşımak istememiz daha olasıdır.

Kod Listesi 7-3'ü derlemeye çalışalım ve henüz derlenmediğini neden bulalım! Aldığımız hatalar Kod Listesi 7-4'te gösterilmiştir.

<Listing number="7-4" caption="Kod Listesi 7-3'teki kodu derlemeden gelen derleyici hataları">

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-03/output.txt}}
```

</Listing>

Hata mesajları `hosting` modülünün özel olduğunu söylüyor. Başka bir deyimle, `hosting` modülü ve `add_to_waitlist` fonksiyonu için doğru yollarımız var ancak Rust bunları kullanmamıza izin vermiyor çünkü özel bölümlere erişimi yok. Rust'ta tüm ögeler (fonksiyonlar, yöntemler, yapılar, enumler, modüller ve sabitler) varsayılan olarak ebeveyn modüllere özeldir. Bir fonksiyon veya yapı gibi bir ögeyi özel yapmak isterseniz, onu bir modüle koyarsınız.

Bir ebeveyn modüldeki ögeler alt modüller içindeki özel ögeleri kullanamaz ancak alt modüllerdeki ögeler atalar modüllerindeki ögeleri kullanabilir. Bunun sebebi alt modüller uygulama detaylarını kapsar ve gizler ancak alt modüller tanımlandıkları bağlamı görebilirler. Metaforumuzla devam etmek için, gizlilik kurallarını bir restoranın arka ofisi gibi düşünün: Orada olan şeyler restoran müşterilerine özeldir ancak ofis yöneticileri işlettikleri restorandaki her şeyi görebilir ve yapabilirler.

Rust modül sisteminin bu şekilde çalışmasını seçmiştir ki iç uygulama detaylarını gizlemek varsayılandır. Bu şekilde, dış kodu kırmadan hangi iç kod parçalarını değiştirebileceğinizi biliyorsunuz. Ancak, Rust size alt modüllerin kodunun iç parçalarını dış atalar modüllere `pub` anahtar kelimesini kullanarak açığa çıkama seçeneği verir.

### Yolları `pub` Anahtar Kelimesiyle Açığa Çıkarma (Exposing Paths with `pub` Keyword)

Kod Listesi 7-4'teki hataya geri dönelim ki bize `hosting` modülünün özel olduğunu söyledi. Ebeveyn modüldeki `eat_at_restaurant` fonksiyonunun alt modüldeki `add_to_waitlist` fonksiyonuna erişmesini istiyoruz, bu yüzden `hosting` modülünü `pub` anahtar kelimesiyle işaretliyoruz, Kod Listesi 7-5'te gösterildiği gibi.

<Listing number="7-5" file-name="src/lib.rs" caption="`hosting` modülünü `eat_at_restaurant`'tan kullanmak için `pub` olarak bildirme">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-05/src/lib.rs:here}}
```

</Listing>

Maalesef, Kod Listesi 7-5'teki kod hala derleyici hatalarıyla sonuçlanır, Kod Listesi 7-6'da gösterildiği gibi.

<Listing number="7-6" caption="Kod Listesi 7-5'teki kodu derlemeden gelen derleyici hataları">

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-05/output.txt}}
```

</Listing>

Ne oldu? `mod hosting` önüne `pub` anahtar kelimesi eklemek modülü genel yapar. Bu değişiklikle, `front_of_house`'e erişebilirsek, `hosting`'e de erişebiliriz. Ancak `hosting`'in _içeriği_ hala özeldir; modülü genel yapmak içeriğini genel yapmaz. Bir modüldeki `pub` anahtar kelimesi sadece atalar modüllerindeki kodun ona atıfta bulunmasını sağlar, iç koduna erişimi değil. Modüller kapsayıcılar olduğu için sadece modülü genel yapmakla yapabileceğimiz çok şey yok; modül içindeki bir veya daha fazla ögeyi de genel yapmayı seçmeliyiz.

Kod Listesi 7-6'daki hatalar `add_to_waitlist` fonksiyonunun özel olduğunu söylüyor. Gizlilik kuralları yapılar, enumler, fonksiyonlar ve yöntemlere olduğu gibi modüllere de uygulanır.

Ayrıca `add_to_waitlist` fonksiyonunu tanımlamasının önüne `pub` anahtar kelimesi ekleyerek genel yapalım, Kod Listesi 7-7'deki gibi.

<Listing number="7-7" file-name="src/lib.rs" caption="`mod hosting` ve `fn add_to_waitlist`'e `pub` anahtar kelimesi eklemek bize `eat_at_restaurant`'tan fonksiyonu çağırmamızı sağlar.">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-07/src/lib.rs:here}}
```

</Listing>

Şimdi kod derlenecek! `eat_at_restaurant` içinde bu yolları gizlilik kurallarına neden kullanabildiğimizi görmek için, mutlak ve göreceli yolları bakalım.

Mutlak yolda, kafamızın modül ağacının kökü olan `crate` ile başlarız. `front_of_house` modülü kafa kökünde tanımlanmıştır. `front_of_house` genel olmasa da, `eat_at_restaurant` fonksiyonu `front_of_house` ile aynı modülde tanımlanmıştır (yani, `eat_at_restaurant` ve `front_of_house` kardeşlerdir), bu yüzden `eat_at_restaurant`'tan `front_of_house`'a atıfta bulunabiliriz. Sonra `pub` ile işaretlenmiş `hosting` modülü var. `hosting`'in ebeveyn modülüne erişebiliriz, bu yüzden `hosting`'e erişebiliriz. Son olarak, `add_to_waitlist` fonksiyonu `pub` ile işaretlenmiştir ve ebeveyn modülüne erişebiliriz, bu yüzden bu fonksiyon çağrısı çalışır!

Göreceli yolda, mantık mutlak yol ile aynıdır ilk adım hariç: Kafa kökünden başlamak yerine, yol `front_of_house`'den başlar. `front_of_house` modülü `eat_at_restaurant` ile aynı modül içinde tanımlanmıştır, bu yüzden `eat_at_restaurant`'in tanımlandığı modülden başlayan göreceli yol çalışır. Sonra, `hosting` ve `add_to_waitlist` `pub` ile işaretlenmiş olduğu için yolun geri kalanı çalışır ve bu fonksiyon çağrısı geçerlidir!

Eğer kütüphane kafanızı başka projelerin kodunuzu kullanabilmesi için paylaşmayı planlıyorsanız, genel API'niz kafanızın kullanıcılarıyla olan kontratınızdır ki kullanıcılarınızın kodunuzla nasıl etkileşebileceklerini belirler. Kafanıza güvenmeyi kolaylaştırmak için genel API'nizi yönetmek hakkında birçok husus vardır. Bu hususlar bu kitabın kapsamının dışındadır; bu konuya ilgi duyuyorsanız, [Rust API Rehberi][api-guidelines]'ne bakın.

> #### İkili ve Kütüphaneli Paketler İçin En İyi Uygulamalar
>
> Bir paketin hem _src/main.rs_ ikili kafa kökü hem de _src/lib.rs_ kütüphane kafa kökü içerebileceğini ve her ikisinin varsayılan olarak paket adına sahip olacağını belirttik. Tipik olarak, hem bir kütüphane hem de bir ikili kafa içeren bu desenle paketler ikili kafa içinde yeterli kodun yürütülebilir başlatır ve kütüphane kafasında tanımlanmış kodu çağırır. Bu, diğer projelerin paketin sağladığı çoğu işlevselliğinden fayda sağlamasına izin verir çünkü kütüphane kafasının kodu paylaşılabilir.
>
> Modül ağacı _src/lib.rs_'de tanımlanmalıdır. Sonra, paket adıyla yollar başlatarak herhangi bir genel öge ikili kafada kullanılabilir. İkili kafa tamamen dış bir kafa gibi kütüphane kafasını kullanır: Sadece genel API'yi kullanabilir. Bu size iyi bir API tasarlamaya yardım eder; Sadece yazar değilsiniz, aynı zamanda bir müşterisiniz!
>
> Bölüm 12'de[ch12]<!-- ignore -->, hem bir ikili kafa hem de bir kütüphane kafa içeren bir komut satır programıyla bu organizasyonel uygulamayı göstereceğiz.

### Yolları `super` ile Başlatma (Starting Relative Paths with `super`)

Mevcut modül veya kafa kökü yerine ebeveyn modülden başlayan göreceli yollar, yolun başında `super` kullanarak oluşturabiliriz. Bu ebeveyn klasöre gitmek anlamına gelen `..` sözdizimiyle başlayan bir dosya sistemi yolu gibidir. `super` kullanmak bize ebeveyn modülde olduğunu bildiğimiz bir ögeyi atıfta bulmamızı sağlar ki bu, modül ebeveyniyle yakından ilişkili ancak ebeveyn gelecekte modül ağacında başka bir yere taşınabilir ise modül ağacını yeniden düzenlemeyi kolaylaştırır.

Kod Listesi 7-8'deki kodu düşünün ki bir şefin yanlış bir siparişi düzelttiğini ve kişisel olarak müşteriye getirdiğini modellemektedir. `back_of_house` modülünde tanımlanmış `fix_incorrect_order` fonksiyonu, `deliver_order`'in yolunu `super` ile başlatarak ebeveyn modülünde tanımlanmış `deliver_order` fonksiyonunu çağırır.

<Listing number="7-8" file-name="src/lib.rs" caption="`super` ile başlayan bir göreceli yol kullanarak bir fonksiyon çağırma">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-08/src/lib.rs}}
```

</Listing>

`fix_incorrect_order` fonksiyonu `back_of_house` modülündedir, bu yüzden `back_of_house`'in ebeveyn modülüne gitmek için `super` kullanabiliriz ki bu durumda `crate`, köktür. Oradan, `deliver_order`'i ararız ve buluruz. Başarı! `back_of_house` modülünün ve `deliver_order` fonksiyonunun birbirleriyle aynı ilişkiyi kalacağını ve kafanın modül ağacını yeniden düzenlemeye karar verirsek birlikte taşınacaklarını düşünüyoruz. Bu yüzden, bu kod farklı bir modüle taşınırsa gelecekte güncelleme yerlerimiz daha az olması için `super` kullandık.

### Yapıları ve Enumleri Genel Yapma (Making Structs and Enums Public)

Yapıları ve enumleri genel olarak belirtmek için de `pub` kullanabiliriz ancak `pub`'in yapılar ve enumlerle kullanımı hakkında birkaç ekstra detay vardır. Bir yapı tanımlamasının önünde `pub` kullanırsak, yapıyı genel yaparız ancak yapının alanları hala özeldir. Her alanı duruma göre genel veya özel yapabiliriz. Kod Listesi 7-9'da, genel bir `toast` alanı ve özel bir `seasonal_fruit` alanı olan genel bir `back_of_house::Breakfast` yapısı tanımladık. Bu, bir müşterinin yemekle gelen ekmek tipini seçebildiği ancak şefin mevsime ve stokta olan şeylere dayalı yemeği hangi meyvenin eşlik ettiğini karar verdiği restoran durumunu modellemektedir. Meysever meyve hızla değişir, bu yüzden müşteriler meyve seçemez veya hangi meyveyi alacaklarını bile göremezler.

<Listing number="7-9" file-name="src/lib.rs" caption="Bazı genel alanları ve bazı özel alanları olan bir yapı">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-09/src/lib.rs}}
```

</Listing>

`back_of_house::Breakfast` yapısındaki `toast` alanı genel olduğu için, `eat_at_restaurant` içinde nokta gösterimi kullanarak `toast` alanına yazabilir ve okuyabiliriz. `eat_at_restaurant` içinde `seasonal_fruit` alanını kullanamadığımızı fark edin çünkü `seasonal_fruit` özeldir. `seasonal_fruit` alan değerini değiştiren satırı açıklamaya çalışın ve hangi hatayı aldığınızı görün!

Ayrıca, `back_of_house::Breakfast` özel bir alana sahip olduğu için, yapının `Breakfast` bir örneğini oluşturan genel bir ilişkili fonksiyon sağlaması gerekir (burada `summer` olarak adlandırdık). Eğer `Breakfast` böyle bir fonksiyon sahip olmasaydı, `eat_at_restaurant` içinde `Breakfast` bir örneğini oluşturamazdık çünkü `eat_at_restaurant` içinde özel `seasonal_fruit` alanının değerini ayarlayamazdık.

Buna karşın, bir enumı genel yaparsak, tüm varyantları o zaman geneldir. Sadece `enum` anahtar kelimesinden önce `pub` gerekir, Kod Listesi 7-10'da gösterildiği gibi.

<Listing number="7-10" file-name="src/lib.rs" caption="Bir enumı genel olarak belirtmek tüm varyantlarını genel yapar.">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-10/src/lib.rs}}
```

</Listing>

`Appetizer` enumını genel olarak yaptığımız için, `eat_at_restaurant` içinde `Soup` ve `Salad` varyantlarını kullanabiliriz.

Enumler varyantları genel olmadan çok kullanışlı değildir; her enum varyantını `pub` ile açıklamak sinir bozucu olur, bu yüzden enum varyantları için varsayılan geneldir. Yapılar alanları genel olmadan da kullanışlı olabilir, bu yüzden yapı alanları `pub` ile açıklanmadıkça her şeyin özel olması genel kuralı takip eder.

`pub` içeren ve henüz kapsamadığımız bir durum daha var ve bu bizim son modül sistem özelliğimizdir: `use` anahtar kelimesi. Önce `use`'yi başına sonra `pub` ve `use`'yi nasıl birleştireceğimizi göstereceğiz.

[pub]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[api-guidelines]: https://rust-lang.github.io/api-guidelines/
[ch12]: ch12-00-an-io-project.html