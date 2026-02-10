<!-- Old headings. Do not remove or links may break. -->

<a id="managing-growing-projects-with-packages-crates-and-modules"></a>

# Paketler, Kafalar ve Modüller (Packages, Crates, and Modules)

Büyük programlar yazarken, kodunuzu organize etmek giderek daha önemli hale gelir. İlgili işlevselliği gruplayarak ve farklı özelliklerle kodu ayırarak, belirli bir özelliği uygulayan kodun nerede bulunacağını ve bir özelliğin nasıl çalıştığını değiştirmek için nereye gideceğini netleştirirsiniz.

Şu ana kadar yazdığımız programlar tek bir modüldü tek bir dosyada olmuştur. Bir proje büyüdükçe, kodu çoklu modüllere ve sonra çoklu dosyalara bölerek organize etmelisiniz. Bir paket çoklu ikili kafaları ve isteğe bağlı olarak bir kütüphane kafası içerebilir. Bir paket büyüdükçe, parçaları dış bağımlılıklar haline ayrı kafaları dışarabilirsiniz. Bu bölüm tüm bu teknikleri kapsar. Birbirleriyle birlikte gelişen interrelated paket kümesinden oluşan çok büyük projeler için, Cargo çalışma alanları sağlar ki bunları Bölüm 14'te ["Cargo Çalışma Alanları" (Cargo Workspaces)][workspaces]<!-- ignore --> kapsamlıyacağız.

Ayrıca uygulama detaylarını kapsülüştürmeyi konuşacağız ki bu kodu daha yüksek bir seviyede yeniden kullanmanızı sağlar: Bir operasyonu uyguladıktan sonra, başka kod uygulamanın nasıl çalıştığını bilmek zorunda kalmadan genel arayüzü aracılığıyla kodunuzu çağırabilir. Kodu yazma şekliniz hangi parçaların başka kodun kullanması için genel olduğunu ve hangi parçaların değiştirme hakkını sakladığınız özel uygulama detayları olduğunu tanımlar. Bu, kafanızda tutmanız gereken detay miktarını sınırlamanın başka bir yoludur.

İlgili bir kavram kapsamdır (scope): Kodun yazıldığı gömülü bağlamın "kapsamda" olarak tanımlanmış bir isim kümesi vardır. Kod okururken, yazarken ve derlerken programcılar ve derleyicilerin belirli bir noktadaki belirli bir ismin bir değişken, fonksiyon, yapı, enum, modül, sabit veya başka bir ögeyi atıftığını ve bu ögenin ne anlama geldiğini bilmeleri gerekir. Kapsamlar oluşturabilir ve hangi isimlerin kapsamda veya kapsamdan dışarı olduğunu değiştirebilirsiniz. Aynı kapsamda aynı isimli iki ögeyi sahip olamazsınız; isim çakışmalarını çözmek için araçlar mevcuttur.

Rust'ta kodunuzun organizasyonunu yönetmenize izin veren bir sayı özellik vardır ki hangi detayların açığa sunulduğunu, hangi detayların özel olduğunu ve programlarınızdaki her kapsamda hangi isimlerin olduğunu içerir. Bu özellikler, bazen kollektif olarak _modül sistemi_ olarak adlandırılır, şunları içerir:

* **Paketler (Packages)**: Kafaları oluşturma, test etme ve paylaşmanızı sağlayan bir Cargo özelliği
* **Kafalar (Crates)**: Bir kütüphane veya yürütülebilir üreten modüller ağacı
* **Modüller ve use (Modules and use)**: Yolların organizasyonunu, kapsamını ve gizliliğini kontrol etmenizi sağlar
* **Yollar (Paths)**: Bir yapı, fonksiyon veya modül gibi bir ögeyi adlandırma yolu

Bu bölümde tüm bu özellikleri kapsamlıyacağız, nasıl etkileştiklerini konuşacağız ve kapsamları yönetmek için nasıl kullanacakları açıklayacağız. Sonuna kadar, modül sistemine sağlam bir anlayışa sahip olmalı ve kapsamları bir profesyonel gibi çalışabilmelisiniz!

[workspaces]: ch14-03-cargo-workspaces.html