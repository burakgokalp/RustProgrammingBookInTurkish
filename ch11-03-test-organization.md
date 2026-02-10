## Test Organizasyonu (Test Organization)

Bölümün başında bahsedildiği gibi, test etmek karmaşık bir disiplindir ve farklı kişiler farklı terminoloji ve organizasyon kullanır. Rust topluluğu testleri iki ana kategori açısından düşünür: birim testleri (unit tests) ve entegrasyon testleri (integration tests). _Birim testleri_ küçüktür ve daha odaklıdır, her seferinde bir modülü izole olarak test eder ve özel arayüzleri (private interfaces) test edebilir. _Entegrasyon testleri_ kütüphaneniz için tamamen dışarıdadır ve kodunuzu herhangi bir başka dış kodun kullanacağı şekilde kullanır, sadece genel arayüzü kullanır ve test başına potansiyel olarak birden fazla modülü kullanır.

Her iki tür testi yazmak, kütüphanenizin parçalarının beklediğiniz şekilde çalıştığından, ayrı ayrı ve birlikte, emin olmak için önemlidir.

### Birim Testleri (Unit Tests)

Birim testlerinin amacı, kodun geri kalanından izole olarak her kod birimini test etmektir böylece kodun beklediğiniz gibi ve olmadığı yerleri hızlı bir şekilde belirleyebilirsiniz. Birim testlerini _src_ dizininde test ettikleri kod ile aynı dosyaya koyacaksınız. Kural (convention), test fonksiyonlarını içeren `tests` adında bir modül oluşturmak ve modülü `cfg(test)` ile notlamaktır.

#### `tests` Modülü ve `#[cfg(test)]`

`tests` modülündeki `#[cfg(test)]` notu Rust'ı test kodunu sadece `cargo test` çalıştırdığınızda derlemesini ve çalıştırmasını söyler, `cargo build` çalıştırdığınızda değil. Bu, sadece kütüphane oluşturmak istediğinizde derleme süresini korur ve testler dahil edilmediği için ortaya çıkan derlenmiş eserde (compiled artifact) yer tasarruf eder. Entegrasyon testlerinin farklı bir dizine gittiğini göreceksiniz, bu yüzden `#[cfg(test)]` notuna ihtiyaçları yoktur. Ancak, birim testleri kod ile aynı dosyalara gittiği için, `#[cfg(test)]` kullanarak derlenmiş sonucun dahil edilmemesi gerektiğini belirteceksiniz.

Bu bölümün ilk kısmında yeni `adder` projesi ürettiğimizde, Cargo'nun bize bu kodu ürettiğini hatırlayın:

<span class="filename">Dosya Adı: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

Otomatik olarak üretilen `tests` modülünde, `cfg` özniteliği (attribute) _konfigürasyon_ anlamına gelir ve Rust'a belirli bir konfigürasyon seçeneği verildiğinde aşağıdaki öğenin sadece dahil edilmesi gerektiğini söyler. Bu durumda, konfigürasyon seçeneği `test`'tir ki bu, testleri derlemek ve çalıştırmak için Rust tarafından sağlanır. `cfg` özniteliğini kullanarak, Cargo test kodumuzu sadece `cargo test` ile testleri aktif olarak çalıştırırsak derler. Bu, `#[test]` ile notlanmış fonksiyonlara ek olarak bu modülün içinde bulunabilecek herhangi bir yardımcı fonksiyonu içerir.

<!-- Old headings. Do not remove or links may break. -->

<a id="testing-private-functions"></a>

#### Özel Fonksiyon Testleri (Private Function Tests)

Test topluluğunda özel fonksiyonların doğrudan test edilip edilmesi gerektiği konusunda bir tartışma var ve diğer diller özel fonksiyonları test etmeyi zor veya imkansız hale getirir. Hangi test ideolojisine uysanız uysanız, Rust'ın gizlilik kuralları özel fonksiyonları test etmenizi sağlar. Kod Listesi 11-12'deki kodu, `internal_adder` özel fonksiyonu ile düşünün.

<Listing number="11-12" file-name="src/lib.rs" caption="Özel bir fonksiyon test etme">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-12/src/lib.rs}}
```

</Listing>

`internal_adder` fonksiyonunun `pub` olarak işaretlenmediğine dikkat edin. Testler sadece Rust kodudur ve `tests` modülü sadece başka bir modüldür. [“Modül Ağacındaki Bir Öğeye Atıf Yollar”][paths]<!-- ignore --> bölümünde tartıştığımız gibi, alt modüllerdeki öğeler üst (ancestor) modüllerindeki öğeleri kullanabilirler. Bu testte, `tests` modülünün üstüne ait tüm öğeleri `use super::*;` ile kapsama getiriyoruz ve sonra test `internal_adder` çağırabilir. Özel fonksiyonların test edilmesi gerektiğini düşünmüyorsanız, sizi bunu yapmaya zorlayacak hiçbir şey yoktur.

### Entegrasyon Testleri (Integration Tests)

Rust'te, entegrasyon testleri kütüphaneniz için tamamen dışarıdadır. Kodunuzu herhangi bir başka kodun kullanacağı şekilde kullanırlar ki bu, sadece kütüphanenizin genel API'sinin bir parçası olan fonksiyonları çağırabilecekleri anlamına gelir. Amaçları, kütüphanenizin birçok parçasının doğru birlikte çalışıp çalışmadığını test etmektir. Kendi başına doğru çalışan kod birimleri, entegre edildiğinde sorunlara sahip olabilir, bu yüzden entegre kodun test kapsamı (test coverage) de önemlidir. Entegrasyon testleri oluşturmak için, önce bir _tests_ dizinine ihtiyaçınız var.

#### _tests_ Dizini (The _tests_ Directory)

Proje dizinimizin en üstünde, _src_ yanında bir _tests_ dizini oluştururuz. Cargo'nun bu dizinde entegrasyon test dosyaları aramasını bilir. Sonra istediğimiz kadar test dosyası oluşturabiliriz ve Cargo dosyaların her birini bireysel bir crate olarak derleyecektir.

Bir entegrasyon testi oluşturalım. Kod Listesi 11-12'deki kod hala _src/lib.rs_ dosyasındayken, _tests_ dizini oluşturun ve _tests/integration_test.rs_ adında yeni bir dosya oluşturun. Dizin yapınız şöyle görünmelidir:

```text
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

Kod Listesi 11-13'teki kodu _tests/integration_test.rs_ dosyasına girin.

<Listing number="11-13" file-name="tests/integration_test.rs" caption="`adder` crate'inde bir fonksiyonun entegrasyon testi">

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-13/tests/integration_test.rs}}
```

</Listing>

_tests_ dizinindeki her dosya ayrı bir crate'dir, bu yüzden kütüphanemizi her test crate'ın kapsamına getirmemiz gerekir. Bu nedenle, kodun en üstüne `use adder::add_two;` ekliyoruz ki birim testlerde gerek yoktu.

_tests/integration_test.rs_ dosyasında herhangi bir kodu `#[cfg(test)]` ile notlamaya gerek yok. Cargo _tests_ dizinini özel davranır ve sadece `cargo test` çalıştırdığımızda bu dizindeki dosyaları derler. Şimdi `cargo test` çalıştırın:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-13/output.txt}}
```

Çıktının üç bölümü birim testleri, entegrasyon testi ve belgelem testlerini içerir. Bir bölümdeki herhangi bir test başarısız olursa, aşağıdaki bölümler çalıştırılmayacağına dikkat edin. Örneğin, bir birim test başarısız olursa, entegrasyon ve belgelem testleri için hiçbir çıktı olmayacak, çünkü bu testler sadece tüm birim testleri geçerse çalıştırılacaklar.

Birim testleri için ilk bölüm şimdiye kadar gördüğümüz gibi aynı: her birim test için bir satır (Kod Listesi 11-12'de eklediğimiz `internal` adlı bir) ve sonra birim testleri için bir özet satırı.

Entegrasyon testleri bölümü `Running tests/integration_test.rs` satırı ile başlar. Sonra, o entegrasyon testindeki her test fonksiyonu için bir satır ve entegrasyon testinin sonuçları için bir özet satırı vardır, hemen `Doc-tests adder` bölümü başlamadan önce.

Her entegrasyon test dosyasının kendi bölümü vardır, bu yüzden _tests_ dizininde daha fazla dosya eklersek, daha fazla entegrasyon test bölümü olacak.

Hala `cargo test`'e argüman olarak test fonksiyonunun adını belirterek belirli bir entegrasyon test fonksiyonu çalıştırabiliriz. Belirli bir entegrasyon test dosyasındaki tüm testleri çalıştırmak için, `cargo test`'in `--test` argümanını ve ardından dosyanın adını kullanın:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-05-single-integration/output.txt}}
```

Bu komut sadece _tests/integration_test.rs_ dosyasındaki testleri çalıştırır.

#### Entegrasyon Testlerinde Alt Modüller (Submodules in Integration Tests)

Daha fazla entegrasyon testi ekledikçe, onları organize etmelerine yardımcı olmak için _tests_ dizininde daha fazla dosya oluşturmak isteyebilirsiniz; örneğin, test fonksiyonlarını test ettikleri işlevselliğe göre gruplayabilirsiniz. Daha önce bahsedildiği gibi, _tests_ dizinindeki her dosya kendi ayrı crate'i olarak derlenir ki bu, son kullanıcıların crate'inizi kullanacakları yolu daha yakından taklit eden ayrı kapsamlar oluşturmak için kullanışlıdır. Ancak bu, _tests_ dizinindeki dosyaların _src_ içindekilerle aynı davranışı paylaşmadığı anlamına gelir, Bölüm 7'de kodu modüllere ve dosyalara ayırma hakkında öğrendiğiniz gibi.

_tests_ dizini dosyalarının farklı davranışı, birden fazla entegrasyon test dosyasında kullanmak için bir set yardımcı fonksiyonunuz olduğunda ve Bölüm 7'nin [“Modülleri Farklı Dosyalara Ayırma”][separating-modules-into-files]<!-- ignore --> bölümündeki adımları takip edip bunları ortak bir modüle çıkarmaya çalıştığınızda en belirgin olur. Örneğin, _tests/common.rs_ oluştururuz ve içine `setup` adında bir fonksiyon yerleştiriz, `setup` içine birden fazla test dosyasındaki birden fazla test fonksiyonundan çağırmak istediğimiz bazı kod ekleyebiliriz:

<span class="filename">Dosya Adı: tests/common.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/tests/common.rs}}
```

Testleri tekrar çalıştırdığımızda, _common.rs_ dosyası için test çıktısında yeni bir bölüm göreceksiniz, hem bu dosya herhangi bir test fonksiyonu içermez hem de `setup` fonksiyonunu herhangi bir yerden çağırmadık:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/output.txt}}
```

`common`'in test sonuçlarında görünmesi ve `running 0 tests` göstermesi istediğimiz değil. Sadece diğer entegrasyon test dosyalarıyla bazı kod paylaşmak istedik. `common`'in test çıktısında görünmesinden kaçınmak için, _tests/common.rs_ oluşturmak yerine, _tests/common/mod.rs_ oluşturacağız. Proje dizini şimdi şöyle görünür:

```text
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    ├── common
    │   └── mod.rs
    └── integration_test.rs
```

Bu, Rust'ın da anladığı ve Bölüm 7'de [“Alternatif Dosya Yolları”][alt-paths]<!-- ignore --> bölümünde bahsettiğimiz eski bir adlandırma kuralıdır. Dosyayı bu şekilde adlandırmak, Rust'a `common` modülünü entegrasyon test dosyası olarak muamele etmemesini söyler. `setup` fonksiyon kodunu _tests/common/mod.rs_ içine taşıdığımızda ve _tests/common.rs_ dosyasını sildiğimizde, test çıktısındaki bölüm artık görünmeyecektir. _tests_ dizinin alt dizinlerindeki dosyalar ayrı crate'ler olarak derlenmez veya test çıktısında bölümlere sahip değildir.

_tests/common/mod.rs_ oluşturduktan sonra, onu herhangi bir entegrasyon test dosyasından bir modül olarak kullanabiliriz. İşte _tests/integration_test.rs_ dosyasındaki `it_adds_two` testinden `setup` fonksiyonu çağırmanın bir örneği:

<span class="filename">Dosya Adı: tests/integration_test.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-13-fix-shared-test-code-problem/tests/integration_test.rs}}
```

`mod common;` deklarasının Kod Listesi 7-21'de gösterdiğimiz modül deklarası ile aynı olduğunu unutmayın. Sonra, test fonksiyonunda, `common::setup()` fonksiyonunu çağırabiliriz.

#### İkili Crate'ler İçin Entegrasyon Testleri (Integration Tests for Binary Crates)

Projemiz sadece bir _src/main.rs_ dosyası içeren ve bir _src/lib.rs_ dosyasına sahip olmayan bir ikili crate ise, _tests_ dizininde entegrasyon testleri oluşturamazız ve `use` bildirimi ile _src/main.rs_ dosyasında tanımlanan fonksiyonları kapsama getiremeyiz. Sadece kütüphane crate'leri diğer crate'lerin kullanabileceği fonksiyonları ortaya çıkarır; ikili crate'ler kendi başlarına çalışıtırılmak için tasarlanmıştır.

Bu, ikili sağlayan Rust projelerinin, _src/lib.rs_ dosyasında yaşayan mantığı çağıran basit bir _src/main.rs_ dosyasına sahip olmasının nedenlerinden biridir. Bu yapıyı kullanarak, entegrasyon testleri `use` kullanarak önemli işlevselliği kullanılabilir hale getirebilir şekilde kütüphane crate'ini test edebilirler. Önemli işlevsellik çalışırsa, _src/main.rs_ dosyasındaki küçük miktar kod da çalışacaktır ve o küçük miktar kodun test edilmesi gerekmez.

## Özet (Summary)

Rust'ın test özellikleri, kodun nasıl çalışması gerektiğini belirtme yolu sağlar böylece değişiklik yapmanıza rağmen beklendiğiniz şekilde çalışmaya devam ettiğinden emin olabilirsiniz. Birim testleri bir kütüphanenin farklı bölümlerini ayrı ayrı kullanır ve özel uygulama detaylarını test edebilir. Entegrasyon testleri kütüphanenin birçok bölümünün doğru birlikte çalıştığını kontrol eder ve kodu dış kodun kullanacağı şekilde kullanmak için kütüphanenizin genel API'sini kullanır. Rust'ın tip sistemi ve mülkiyet kuralları bazı hata türlerini önlemeye yardımcı olsa da, testler yine de kodun beklendiği şekilde davranmakla ilgili mantık hatalarını azaltmak için önemlidir.

Bu bölümde ve önceki bölümlerde öğrendiğiniz bilgiyi bir projede çalışmak için birleştirelim!

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
[separating-modules-into-files]: ch07-05-separating-modules-into-different-files.html
[alt-paths]: ch07-05-separating-modules-into-different-files.html#alternate-file-paths