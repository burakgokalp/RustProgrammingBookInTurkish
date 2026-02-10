## Testlerin Nasıl Çalıştığını Kontrol Etme (Controlling How Tests Are Run)

`cargo run` kodunuzu derler ve sonra ortaya çıkan ikiliyi (binary) çalıştırır, `cargo test` ise kodunuzu test modunda derler ve ortaya çıkan test ikiliyi çalıştırır. `cargo test` tarafından üretilen ikiliyi varsayılan davranışı tüm testleri paralel olarak çalıştırmak ve test çalışmaları sırasında üretilen çıktıyı yakalamaktır, çıktının görüntülenmesini önlemek ve test sonuçlarıyla ilgili çıktıyı okumayı kolaylaştırmak. Ancak, bu varsayılan davranışı değiştirmek için komut satırı seçenekleri belirtebilirsiniz.

Bazı komut satırı seçenekleri `cargo test`'e gider, bazıları ise ortaya çıkan test ikiliye gider. Bu iki argüman türünü ayırmak için, `cargo test`'e giden argümanları ve ardından `--` ayırıcısı ve ardından test ikiliye giden argümanları listelersiniz. `cargo test --help` çalıştırmak `cargo test` ile kullanabileceğiniz seçenekleri görüntüler ve `cargo test -- --help` çalıştırmak ayırıcısından sonra kullanabileceğiniz seçenekleri görüntüler. Bu seçenekler ayrıca [_The `rustc` Book_'nın "Testler" bölümünde][tests] belgelendirilmiştir.

[tests]: https://doc.rust-lang.org/rustc/tests/index.html

### Paralel veya Sırayla Testleri Çalıştırma (Running Tests in Parallel or Consecutively)

Çok sayıda test çalıştırdığınızda, varsayılan olarak iş parçaları (threads) kullanılarak paralel olarak çalışırlar, bu da daha hızlı bitmeleri ve daha hızlı geri bildirim anlamına gelir. Çünkü testler aynı anda çalışırlar, testlerinize birbirlerine veya ortak bir duruma, ortak bir ortam gibi geçerli çalışma dizini veya ortam değişkenlerine bağlı olmadığından emin olmalısınız.

Örneğin, her testinizin disk üzerinde _test-output.txt_ adında bir dosya oluşturan ve o dosyaya bazı veri yazan bir kod çalıştırdığını söyleyin. Sonra, her test o dosyadaki veriyi okur ve dosyanın belirli bir değeri içerdiğini iddia eder ki bu, her testte farklıdır. Çünkü testler aynı anda çalışırlar, bir test başka bir testin yazdığı ile okuma arasında dosyanın üzerine yazma ihtimali vardır. İkinci test o zaman başarısız olur çünkü kod yanlış değil, testler paralel çalışırken birbirleriyle karışmışlar. Bir çözüm, her testin farklı bir dosyaya yazdığından emin olmaktır; başka bir çözüm ise testleri bir anda çalıştırmaktır.

Eğer testleri paralel olarak çalıştırmak istemiyorsanız veya kullanılan iş parçacılarının sayısı üzerinde daha ince kontrole sahip olmak istiyorsanız, `--test-threads` bayrağı (flag) ve kullanmak istediğiniz iş parçacı sayısını test ikiliye gönderebilirsiniz. Aşağıdaki örneğe bakın:

```console
$ cargo test -- --test-threads=1
```

Test iş parçacı sayısını `1`'e ayarladık ki programa herhangi bir paralellik kullanmamasını söylüyoruz. Bir iş parçacığını kullanarak testleri çalıştırmak, paralel olarak çalıştırmaktan daha uzun sürecektir ancak testler ortak durum paylaşırlarsa birbirleriyle karışmazlar.

### Fonksiyon Çıktısını Gösterme (Showing Function Output)

Varsayılan olarak, bir test geçerse, Rust'ın test kütüphanesi standart çıktıya yazılan herhangi bir şeyi yakalar. Örneğin, bir testte `println!` çağırız ve test geçerse, terminale `println!` çıktısını görmeyiz; sadece testin geçtiğini gösteren satırı görüyoruz. Bir test başarısız olursa, başarısızlık mesajının geri kalanıyla standart çıktıya yazılan her şeyi görüyoruz.

Bir örnek olarak, Kod Listesi 11-10 parametresinin değerini yazan ve 10 döndüren saçma bir fonksiyon ile geçen bir test ve başarısız olan bir test içerir.

<Listing number="11-10" file-name="src/lib.rs" caption="`println!` çağıran bir fonksiyon için testler">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-10/src/lib.rs}}
```

</Listing>

Bu testleri `cargo test` ile çalıştırdığımızda şu çıktıyı görüyoruz:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-10/output.txt}}
```

Bu çıktının hiçbir yerinde `I got value 4` görmediğimizi unutmayın ki bu, geçen test çalıştığında yazılır. Bu çıktı yakalanmıştır. Başarısız olan testin çıktısı olan `I got value 8`, test özeti çıktısının bir bölümünde görünür ki ayrıca test başarısızlığının sebebini de gösterir.

Geçen testler için yazılan değerleri de görmek istiyorsanız, Rust'ı başarılı testlerin çıktısını da göstermesi için `--show-output` kullanarak söyleyebilirsiniz:

```console
$ cargo test -- --show-output
```

Kod Listesi 11-10'deki testleri `--show-output` bayrağı ile tekrar çalıştırdığımızda şu çıktıyı görüyoruz:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-01-show-output/output.txt}}
```

### Ada Göre Testlerin Bir Alt Kümesini Çalıştırma (Running a Subset of Tests by Name)

Tam bir test paketini çalıştırmak bazen uzun sürebilir. Belirli bir alanda kod üzerinde çalışıyorsanız, sadece o kodla ilgili testleri çalıştırmak isteyebilirsiniz. Bir argüman olarak çalıştırmak istediğiniz test(lerin) adını veya adını `cargo test`'e geçirerek hangi testleri çalıştıracağınızı seçebilirsiniz.

Testlerin bir alt kümesini nasıl çalıştıracağınızı göstermek için, önce Kod Listesi 11-11'de gösterildiği gibi `add_two` fonksiyonumuz için üç test oluşturacağız ve hangilerini çalıştıracağızı seçeceğiz.

<Listing number="11-11" file-name="src/lib.rs" caption="Üç farklı ada sahip üç test">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-11/src/lib.rs}}
```

</Listing>

Herhangi bir argüman geçirmeden testleri çalıştırdığımızda, daha önce gördüğümüz gibi tüm testler paralel olarak çalışır:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-11/output.txt}}
```

#### Tek Testleri Çalıştırma (Running Single Tests)

Sadece o testi çalıştırmak için `cargo test`'e herhangi bir test fonksiyonunun adını geçebilirsiniz:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-02-single-test/output.txt}}
```

Sadece `one_hundred` adlı test çalıştı; diğer iki test bu ada uymadı. Test çıktısı bize çalıştırılmayan daha fazla test olduğunu sonunda `2 filtered out` görüntüleyerek bizi bilgilendirir.

Bu yolla birden fazla test adı belirtemeyiz; `cargo test`'e verilen sadece ilk değer kullanılır. Ancak birden fazla test çalıştırmanın bir yolu vardır.

#### Çoklu Testleri Çalıştırmak İçin Filtreleme (Filtering to Run Multiple Tests)

Bir test adının bir parçasını belirtebilir ve bu değeri eşleşen herhangi bir test çalıştırılacaktır. Örneğin, testlerimizin ikisinin adı `add` içerdiği için, `cargo test add` çalıştırarak bunların ikisini çalıştırabiliriz:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-03-multiple-tests/output.txt}}
```

Bu komut, adı `add` içeren tüm testleri çalıştırdı ve `one_hundred` adlı testi filtreledi. Ayrıca, bir testin göründüğü modülün, testin adının bir parçası olduğunu unutmayın, bu yüzden bir modülün tüm testlerini modülün adını filtreleyerek çalıştırabilirsiniz.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-some-tests-unless-specifically-requested"></a>

### Özellikle İstenmediği Sürece Testleri Yoksayma (Ignoring Tests Unless Specifically Requested)

Bazen birkaç belirli test çalıştırmak çok zaman alabilir, bu yüzden `cargo test`'in çoğu çalışması sırasında bunları hariç tutmak isteyebilirsiniz. Çalıştırmak istediğiniz tüm testleri argümanlar olarak listelemek yerine, burada gösterildiği gibi `ignore` özniteliğini kullanarak zaman alıcı testleri hariç tutabilirsiniz:

<span class="filename">Dosya Adı: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/src/lib.rs:here}}
```

`#[test]`'ten sonra, hariç tutmak istediğimiz testin `#[ignore]` satırını ekliyoruz. Şimdi testlerimizi çalıştırdığımızda, `it_works` çalışır ancak `expensive_test` çalışmaz:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/output.txt}}
```

`expensive_test` fonksiyonu `ignored` (yoksayılmış) olarak listelenmiş. Sadece yoksayılmış testleri çalıştırmak istiyorsanız, `cargo test -- --ignored` kullanabilirsiniz:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-04-running-ignored/output.txt}}
```

Hangi testlerin çalıştığını kontrol ederek, `cargo test` sonuçlarınız hızlı dönecektir. Yoksayılmış testlerin sonuçlarını kontrol etmenin mantıklı geldiği noktada sonuçları beklemek için zamanınız varsa, `cargo test -- --ignored` çalıştırabilirsiniz. Yoksayılmış veya yok bakmaksız tüm testleri çalıştırmak istiyorsanız, `cargo test -- --include-ignored` çalıştırabilirsiniz.