## Modülleri Farklı Dosyalara Ayırma (Separating Modules into Different Files)

Bu bölüme kadar, tüm örnekler bu bölümde bir dosyada çoklu modüller tanımladı. Modüller büyüdükçe, tanımlarını ayrı bir dosyaya taşıyarak kodun gezinmesini kolaylaştırmak isteyebilirsiniz.

Örneğin, Kod Listesi 7-17'deki kodu çoklu restoran modülleriyle başlayalım. Tüm modülleri kafa kök dosyasında tanımlamak yerine modülleri dosyalara çıkaracağız. Bu durumda, kafa kök dosyası _src/lib.rs_'dir ancak bu prosedür kafa kök dosyası _src/main.rs_ olan ikili kafalar için de çalışır.

İlk, `front_of_house` modülünü kendi dosyasına çıkaracağız. `front_of_house` modülü için süslü parantez içindeki kodu kaldırın, sadece `mod front_of_house;` bildirimini bırakın ki _src/lib.rs_ Kod Listesi 7-21'de gösterilen kodu içerir. _src/front_of_house.rs_ dosyasını Kod Listesi 7-22'de oluşturana kadar derlenmeyeceğini fark edin.

<Listing number="7-21" file-name="src/lib.rs" caption="Gövdesi *src/front_of_house.rs*'de olacak `front_of_house` modülünü bildirme">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/lib.rs}}
```

</Listing>

Sonra, süslü parantez içindeki kodu _src/front_of_house.rs_ adında yeni bir dosyaya koyun, Kod Listesi 7-22'de gösterildiği gibi. Derleyici bu dosyaya bakmasını bilir çünkü kafa kökünde `front_of_house` adıyla modül bildirmesiyle karşılaştı.

<Listing number="7-22" file-name="src/front_of_house.rs" caption="*src/front_of_house.rs*'de `front_of_house` modülü içindeki tanımlar">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/front_of_house.rs}}
```

</Listing>

Fark edin ki bir dosyayı yüklemek için modül ağacınızda bir `mod` bildirimini _bir kez_ yapmanız gerekir. Derleyici dosyanın projenin bir parçası olduğunu (ve kodun modül ağacındaki nerede oturduğunu `mod` ifadesini nereye koyduğunuza göre bilir) öğrendiğinde, projenizdeki diğer dosyalar yüklenen dosyanın koduna, ["Modül Ağacındaki Bir Ögeyi Atıfta Bulmak İçin Yollar" (Paths for Referring to an Item in Module Tree)][paths]<!-- ignore --> bölümünde kapsandığı şekilde bildirildiği yolla atıfta bulunmalıdır. Başka bir deyimle, `mod` başka programlama dillerinde görebileceğiniz "include" operasyonu _değildir_.

Sonra, `hosting` modülünü kendi dosyasına çıkaracağız. Prosedür biraz farklıdır çünkü `hosting`, kök modülünün değil `front_of_house`'un bir alt modülüdür. `hosting` için dosyayı modül ağacındaki ataları için adlandırılacak yeni bir klasöre koyacağız, bu durumda _src/front_of_house_'e.

`hosting`'i taşımaya başlamak için, _src/front_of_house.rs_'yi sadece `hosting` modülünün bildirimini içermesi için değiştiririz:

<Listing file-name="src/front_of_house.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house.rs}}
```

</Listing>

Sonra, bir _src/front_of_house_ klasörü ve _hosting.rs_ dosyası oluşturun ki `hosting` modülünde yapılan tanımları içersin:

<Listing file-name="src/front_of_house/hosting.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house/hosting.rs}}
```

</Listing>

Bunun yerine _hosting.rs_'i _src_ klasörüne koysaydık, derleyici _hosting.rs_ kodunun kafa kökünde bildirilen `hosting` modülünde olmasını ve `front_of_house` modülünün çocu olarak bildirilmemesini beklerdi. Derleyicinin hangi dosyalar için hangi modüllerin kodunu kontrol etme kuralları, modül ağacıyla klasörleri ve dosyaları daha yakından eşleştirir.

> ### Alternatif Dosya Yollar
>
> Şu ana kadar Rust derleyicisinin kullandığı çoğu idiyomatik dosya yollarını kapsadık ancak Rust ayrıca dosya yolunun eski bir stilini destekler. Kafa kökünde bildirilen `front_of_house` adında bir modül için, derleyici modülün kodunu şurarda arar:
> - _src/front_of_house.rs_ (bizim kapsadığımız)
> - _src/front_of_house/mod.rs_ (eski stil, hala desteklenen yol)
> `front_of_house`'un bir altmodülü olan `hosting` adında bir modül için, derleyici modülün kodunu şurarda arar:
> - _src/front_of_house/hosting.rs_ (bizim kapsadığımız)
> - _src/front_of_house/hosting/mod.rs_ (eski stil, hala desteklenen yol)
> Eğer aynı modül için her iki stili kullanırsanız, derleyici hatası alırsınız. Aynı projedeki farklı modüller için her iki stilin karışımını kullanmak izin verilir ancak projenizi gezen kişiler için kafa karıştırıcı olabilir.
> _mod.rs_ adında dosyalar kullanan stilin ana dezavantajı, projenizde _mod.rs_ adında birçok dosyayla sonlanmasıdır ki bunları düzenleyicinizde aynı anda açtığınızda kafa karıştırıcı olabilir.

Her modülün kodunu ayrı bir dosyaya taşıdık ve modül ağacı aynı kaldı. `eat_at_restaurant` içindeki fonksiyon çağrıları herhangi bir değişiklik olmadan çalışacak, tanımlar farklı dosyalarda olmasa bile. Bu teknik, modüller büyüdükçe onları yeni dosyalara taşımanızı sağlar.

Fark edin ki _src/lib.rs_'deki `pub use crate::front_of_house::hosting` ifadesi de değişmedi, ayrıca `use`'in kafa kısmı olarak derlenen dosyalar üzerinde hiçbir etkisi yoktur. `mod` anahtar kelimesi modülleri bildirir ve Rust o modüle giren kod için modülle aynı adda sahip bir dosyaya bakar.

## Özet

Rust size bir paketi çoklu kafalara ve bir kafayı modüllere bölmenizi sağlar ki bir modülde tanımlanmış ögelere başka bir modülden atıfta bulabilirsiniz. Bunu mutlak veya göreceli yollar belirterek yapabilirsiniz. Bu yollar `use` ifadesiyle kapsama getirilebilir ki bu kapsamada ögeyi çoklu kullanımları için daha kısa yol kullanabilirsiniz. Modül kodu varsayılan olarak özeldir ancak `pub` anahtar kelimesi ekleyerek tanımları genel yapabilirsiniz.

Bir sonraki bölümde, düzenli organize etmiş kodunuzda kullanabileceğiniz standart kütüphanedeki bazı veri yapısı yapılarına bakacağız.

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html