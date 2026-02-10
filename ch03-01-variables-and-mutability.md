## Değişkenler ve Değiştirilebilirlik

[Değişkenlerle Değerler Saklama][storing-values-with-variables]<!-- ignore --> bölümünde belirtildiği gibi, varsayılan olarak, değişkenler değiştirilemezdir (immutable). Bu, Rust'ın size kodunuzu Rust'ın sunduğu güvenlik ve kolay eşzamanlılık avantajlarından yararlanacak şekilde yazmanız için verdiği birçok teşviklerden biridir. Ancak, yine de değişkenlerinizi değiştirilebilir yapma seçeneğiniz var. Rust'ın neden değiştirilemezliği tercih etmeye teşvik ettiğini ve bazen neden dış çıkmak isteyebileceğinizi inceleyelim.

Bir değişken değiştirilemez olduğunda, bir değer bir ada bağlandığında, o değeri değiştiremezsiniz. Bunu göstermek için, _projects_ dizininizde `cargo new variables` kullanarak _variables_ adında yeni bir proje oluşturun.

Sonra, yeni _variables_ dizininizde, _src/main.rs_'yi açın ve kodunu aşağıdaki kodla değiştirin, bu henüz derlenmeyecek:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/src/main.rs}}
```

`cargo run` kullanarak programı kaydedin ve çalıştırın. Aşağıdaki çıktıda gösterildiği gibi, bir değiştirilemezlik hatası hakkında bir hata mesajı almalısınız:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/output.txt}}
```

Bu örnek derleyicinin programınızdaki hataları bulmanıza nasıl yardımcı olduğunu gösterir. Derleyici hataları can sıkıcı olabilir, ancak gerçekten sadece programınızın henüz istediğinizi güvenli bir şekilde yapmadığı anlamına gelirler; sizin iyi bir programcı olmadığınızı _anlamazlar_! Deneyimli Rustaceanlar yine de derleyici hataları alırlar.

Değiştirilemez `x` değişkenine iki kez atama yapmaya çalıştığınız için `cannot assign twice to immutable variable x` hata mesajını aldınız.

Değiştirilemez olarak belirlenmiş bir değeri değiştirmeye çalıştığımızda derleme zamanı hataları almanın önemli çünkü bu çok durum hatalara yol açabilir. Kodumuzun bir kısmı bir değerin asla değişmeyeceği varsayımında işler ve kodumuzun başka bir kısmı o değeri değiştirirse, kodun ilk kısmının tasarlandığı şeyi yapmaması mümkündür. Bu tür bir hatanın nedeni, özellikle ikinci kod parçası değeri sadece _bazen_ değiştirdiğinde, gerçeğin ardından takip etmek zor olabilir. Rust derleyicisi bir değerin değişmeyeceğini belirttiğinizde, gerçekten değişmeyeceğini garanti eder, bu yüzden kendinizi takip etmeniz gerekmez. Kodunuz böylece akıl yürütmesi daha kolaydır.

Ancak değiştirilebilirlik çok faydalı olabilir ve kodu yazmayı daha uygun hale getirebilir. Değişkenler varsayılan olarak değiştirilemez olsa da, [Bölüm 2][storing-values-with-variables]<!-- ignore -->'de yaptığınız gibi değişken adının önüne `mut` ekleyerek onları değiştirilebilir yapabilirsiniz. `mut` eklemek ayrıca kodun gelecekteki okuyucularına niyeti ileter, kodun başka kısımlarının bu değişkenin değerini değiştireceğini belirtir.

Örneğin, _src/main.rs_'yi şuna değiştirelim:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/src/main.rs}}
```

Programı şimdi çalıştırdığımızda, şunu alırız:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/output.txt}}
```

`mut` kullanıldığında, `x`'e bağlı değeri `5`'ten `6`'ya değiştirmeye izinliyoruz. Sonuç olarak, değiştirilebilirlik kullanıp kullanmamanın size ve belirli durumdaki en açık olan şeye bağlıdır.

<!-- Old headings. Do not remove or links may break. -->
<a id="constants"></a>

### Sabitleri Bildirme

Değiştirilemez değişkenler gibi, _sabitler_ (constants) bir ada bağlanan ve değiştirilmesine izin verilmeyen değerlerdir, ancak sabitler ve değişkenler arasında birkaç fark vardır.

Önce, `mut`'ü sabitlerle kullanmana izin verilmez. Sabitler varsayılan olarak sadece değiştirilemez değillerdir—onlar her zaman değiştirilemezdir. Sabitleri `let` anahtar kelimesi yerine `const` anahtar kelimesini kullanarak bildirirsiniz ve değerin tipi _zorunlu olarak_ not edilmelidir (annotated). Tipler ve tip notasyonlarını bir sonraki bölümde, ["Veri Tipleri"][data-types]<!-- ignore -->, kapsayacağız, bu yüzden şimdi detaylarla endişelenmeyin. Sadece her zaman tipi not etmeniz gerektiğini bilin.

Sabitler herhangi bir scope'da bildirilebilir, global scope dahil, bu onları kodun birçok kısmının bilmeye ihtiyaç duyduğu değerler için yararlı yapar.

Son fark, sabitlerin sadece bir sabit ifadeye ayarlanabileceği, sadece çalışma zamanında hesaplanabilen bir değerin sonucuna değil, olmasıdır.

İşte bir sabit bildirme örneği:

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

Sabitin adı `THREE_HOURS_IN_SECONDS`'dir ve değeri 60 (bir dakikadaki saniye sayısı) ile 60 (bir saatteki dakika sayısı) ile 3'ün (bu programda saymak istediğimiz saat sayısı) çarpımının sonucuna ayarlanmıştır. Rust'ın sabitler için adlandırma kuralı, kelimeler arasında alt çizgi ile tüm büyük harf kullanmaktır. Derleyici derleme zamanında sınırlı bir işlem setini değerlendirebilir, bu bize bu değeri anlaması ve doğrulaması daha kolay bir şekilde yazmayı seçmemizi sağlar, bu sabiti 10,800 değerine ayarlamak yerine. Sabitleri bildirirken hangi işlemlerin kullanılabileceği hakkında daha fazla bilgi için [Rust Referansının sabit değerlendirme bölümüne][const-eval] bakın.

Sabitler, programın çalıştığı tüm süre boyunca geçerlidir, bildirildikleri scope içinde. Bu özellik sabitleri uygulama alanınızdaki, programın birçok kısmının bilmeye ihtiyaç duyabileceği, örneğin bir oyunun herhangi bir oyuncusunun kazanmaya izin verilen maksimum puan sayısı veya ışık hızı gibi değerler için yararlı yapar.

Programınız boyunca kullanılan sabit kodlanmış değerleri sabitler olarak adlandırmak, o değerin anlamını kodun gelecekteki bakımcılarına iletmek için yararlıdır. Ayrıca gelecekte sabit kodlanmış değerin güncellenmesine ihtiyaç duyulursa kodunuzdaki değiştirmeniz gerekecek tek bir yere sahip olmanıza yardımcı olur.

### Gölgeleme (Shadowing)

[Bölüm 2][comparing-the-guess-to-the-secret-number]<!-- ignore -->'de tahmin oyunu öğreticisinde gördüğünüz gibi, önceki bir değişkenle aynı adda yeni bir değişken bildirebilirsiniz. Rustaceanlar ilk değişkenin ikinci tarafından _gölgelendiğini_ (shadowed) söyler, bu, ikinci değişkenin, değişken adını kullandığınızda derleyicinin göreceği şey olduğu anlamına gelir. Etkili olarak, ikinci değişkeni gölgeler, değişken adının tüm kullanımlarını kendine alır ya da o kendisi gölgelene kadar veya scope bitene kadar. Aynı değişkenin adını kullanarak ve `let` anahtar kelimesinin kullanımını şu şekilde tekrarlayarak bir değişkeni gölgeleyebiliriz:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/src/main.rs}}
```

Bu program önce `x`'i `5` değerine bağlar. Sonra, `let x =` tekrarlayarak yeni bir `x` değişkeni oluşturur, orijinal değeri alıp `1` ekler böylece `x`'in değeri `6` olur. Sonra, küme parantezleriyle oluşturulan bir iç scope içinde, üçüncü `let` ifadesi de `x`'i gölgeler ve yeni bir değişken oluşturur, önceki değeri `2` ile çarparak `x`'e `12` değeri verir. O scope bittiğinde, iç gölgeleme biter ve `x` `6` olmaya döner. Bu programı çalıştırdığımızda, şunu çıktılayacaktır:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/output.txt}}
```

Gölgeleme, bir değişkeni `mut` olarak işaretlemekten farklıdır çünkü `let` anahtar kelimesini kullanmadan yanlışlıkla bu değişkene tekrar atama yapmaya çalışırsak bir derleme zamanı hatası alırız. `let` kullanarak, bir değer üzerinde birkaç dönüşüm gerçekleştirebiliriz ancak bu dönüşümler tamamlandıktan sonra değişken değiştirilemez olabilir.

`mut` ve gölgeleme arasındaki diğer fark, `let` anahtar kelimesini tekrar kullandığımızda etkili olarak yeni bir değişken oluşturduğumuzdan, değerin tipini değiştirebiliriz ancak aynı adı yeniden kullanabiliriz. Örneğin, programımız bir kullanıcının bazı metinler arasında kaç boşluk istediklerini girerek boşluk karakterleri göstermesini sorunsun, sonra o girdiyi bir sayı olarak saklamak isteyelim:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-04-shadowing-can-change-types/src/main.rs:here}}
```

İlk `spaces` değişkeni bir dize tipidir ve ikinci `spaces` değişkeni bir sayı tipidir. Gölgeleme böylece bizi `spaces_str` ve `spaces_num` gibi farklı adlar bulmak zorunda kalmaktan kurtarır; bunun yerine, daha basit `spaces` adını yeniden kullanabiliriz. Ancak, bunun için `mut` kullanmaya çalışırsak, burada gösterildiği gibi, bir derleme zamanı hatası alırız:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/src/main.rs:here}}
```

Hata, bir değişkenin tipini değiştirmeye izin verilmediğini söyler:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/output.txt}}
```

Değişkenlerin nasıl çalıştığını incelediğimize göre, sahip olabilecekleri daha fazla veri tipine bakalım.

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[data-types]: ch03-02-data-types.html#data-types
[storing-values-with-variables]: ch02-00-guessing-game-tutorial.html#storing-values-with-variables
[const-eval]: ../reference/const_eval.html