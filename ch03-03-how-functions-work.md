## Fonksiyonlar

Fonksiyonlar Rust kodunda yaygındır. Dildeki en önemli fonksiyonlardan birini zaten gördünüz: `main` fonksiyonu, bu birçok programın giriş noktasıdır. Ayrıca `fn` anahtar kelimesini gördünüz, bu yeni fonksiyonlar bildirmeye izin verir.

Rust kodu fonksiyon ve değişken adları için geleneksel stil olarak _yılan kesenin (snake case)_ kullanır, tüm harfler küçük harftir ve kelimeler alt çizgi ile ayrılır. İşte bir örnek fonksiyon tanımı içeren bir program:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-16-functions/src/main.rs}}
```

Rust'ta `fn` ardından bir fonksiyon adı ve parantez seti girerek bir fonksiyon tanımlarız. Küme parantezleri derleyiciye fonksiyon gövdesinin nerede başladığını ve nerede bittiğini söyler.

Tanımladığımız herhangi bir fonksiyonu adını ardından parantez seti girerek çağırabiliriz. `another_function` programda tanımlandığından, onu `main` fonksiyonunun içinden çağrılabilir. Not edin ki `another_function`'ı kaynak kodda `main` fonksiyonundan _sonra_ tanımladık; onu önceden de tanımlayabilirdik. Rust fonksiyonlarınızı nerede tanımladığınızı umursamaz, sadece çağırıcı tarafından görülebilen bir scope'da bir yerde tanımlandıklarını önemser.

Fonksiyonları daha fazla keşfetmek için _functions_ adında yeni bir ikili proje başlatalım. `another_function` örneğini _src/main.rs_'ye yerleştirin ve çalıştırın. Aşağıdaki çıktıyı görmelisiniz:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-16-functions/output.txt}}
```

Satırlar `main` fonksiyonunda göründükleri sırada çalıştırılır. Önce "Hello, world!" mesajı yazdırılır ve sonra `another_function` çağrılır ve onun mesajı yazdırılır.

### Parametreler

Bir fonksiyonun bir parçası olan özel değişkenler olan _parametreleri_ olacak şekilde tanımlayabiliriz. Bir fonksiyon parametrelere sahip olduğunda, o parametreler için somut değerler sağlayabilirsiniz. Teknik olarak, somut değerler _argümanlar_ olarak adlandırılır, ancak gayri resmi konuşmalarda insanlar bir fonksiyonun tanımındaki değişkenler veya bir fonksiyonu çağırdığınızda geçilen somut değerler için _parametre_ ve _argüman_ kelimelerini birbirinin yerine kullanma eğilimindedirler.

`another_function`'ın bu sürümünde bir parametre ekliyoruz:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/src/main.rs}}
```

Bu programı çalıştırmayı deneyin; aşağıdaki çıktıyı almalısınız:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/output.txt}}
```

`another_function`'ın bildirimi `x` adında bir parametreye sahiptir. `x`'in tipi `i32` olarak belirtilmiştir. `another_function`'a `5` geçtiğimizde, `println!` makrosu `x`'i içeren küme parantezi çiftinin olduğu yere `5` koyar.

Fonksiyon imzalarında, her parametrenin tipini _zorunlu olarak_ bildirmelisiniz. Bu Rust'ın tasarımında bilinçli bir karardır: Fonksiyon tanımlarında tip notasyonları zorunlu kılması derleyicinin kodun başka yerlerinde kullanmaya zorun kalmadan ne tipi kastettiğinizi anlamasını neredeyse asla gerektirmemesi anlamına gelir. Derleyici ayrıca, fonksiyonun hangi tipleri beklediğini bilirse daha yararlı hata mesajları sağlayabilir.

Birden fazla parametre tanımlarken, virgül ile parametre bildirimlerini ayırın, şöyle:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/src/main.rs}}
```

Bu örnek iki parametreyi sahip `print_labeled_measurement` adında bir fonksiyon oluşturur. İlk parametre `value` adındadır ve bir `i32`'dir. İkincisi `unit_label` adındadır ve `char` tipindedir. Fonksiyon daha sonra hem `value` hem de `unit_label` içeren metni yazdırır.

Bu kodu çalıştırmayı deneyelim. Önleyen örnekle _functions_ projenizin _src/main.rs_ dosyasındaki programı değiştirin ve `cargo run` kullanarak çalıştırın:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/output.txt}}
```

Çünkü fonksiyonu `value` için `5` ve `unit_label` için `'h'` değerleriyle çağırdık, program çıktısı bu değerleri içerir.

### İfadeler ve Deyimler (Statements and Expressions)

Fonksiyon gövdeleri isteğe bağlı bir ifadeyle biten deyimler serisinden oluşur. Şimdiye kadar ele aldığımız fonksiyonlar biten bir ifade içermiyordu, ancak bir ifadeyi bir deyimin bir parçası olarak gördünüz. Çünkü Rust bir ifade tabanlı dildir, bu anlamanız gereken önemli bir ayrımdır. Diğer dillerde aynı ayrımlar yoktur, bu yüzden deyimlerin ve ifadelerin ne olduğunu ve farklarının fonksiyon gövdelerini nasıl etkilediğine bakalım.

- _Deyimler_ (Statements) bazı eylemi gerçekleştiren ve bir değer döndürmeyen talimatlardır.
- _İfadeler_ (Expressions) bir sonuç değere değerlendirir.

Bazı örnekleri bakalım.

Aslında deyimleri ve ifadeleri zaten kullandık. Bir değişken oluşturmak ve `let` anahtar kelimesiyle ona bir değer atamak bir deyimdir. Kod Listesi 3-1'de, `let y = 6;` bir deyimdir.

<Listing number="3-1" file-name="src/main.rs" caption="Bir deyim içeren bir `main` fonksiyon bildirimi">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-01/src/main.rs}}
```

</Listing>

Fonksiyon tanımları da deyimlerdir; tüm önleyen örnek kendi başına bir deyimdir. (Kısa süre içinde göreceğimiz gibi, bir fonksiyonu çağırmak bir deyim değildir, ancak.)

Deyimler değer döndürmezler. Bu nedenle, aşağıdaki kodun yapmaya çalıştığı gibi, bir `let` deyimini başka bir değişkene atayamazsınız; bir hata alacaksınız:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/src/main.rs}}
```

Bu programı çalıştırdığınızda, alacağınız hata şuna benzer:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/output.txt}}
```

`let y = 6` deyimi bir değer döndürmediğinden, `x`'in bağlanabileceği hiçbir şey yoktur. Bu C ve Ruby gibi diğer dillerde olanlardan farklıdır, atamanın atanma değerini döndürdüğü. Bu dillerde, `x = y = 6` yazabilirsiniz ve hem `x` hem de `y`'in `6` değerine sahip olmasını sağlayabilirsiniz; bu Rust'ta durum böyle değildir.

İfadeler bir değere değerlendirir ve Rust'ta yazacağınız kodun geri kalanının çoğunu oluştururlar. `5 + 6` gibi bir matematik işlem düşünün, bu `11` değerine değerlendiren bir ifadedir. İfadeler deyimlerin bir parçası olabilirler: Kod Listesi 3-1'de, `let y = 6;` deyimindeki `6`, `6` değerine değerlendiren bir ifadedir. Bir fonksiyonu çağırmak bir ifadedir. Bir makroyu çağırmak bir ifadedir. Küme parantezleri ile oluşturulan yeni bir scope bloğu bir ifadedir, örneğin:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-20-blocks-are-expressions/src/main.rs}}
```

Bu ifade:

```rust,ignore
{
    let x = 3;
    x + 1
}
```

bu durumda `4` değerine değerlendiren bir bloktur. O değer `let` deyiminin bir parçası olarak `y`'e bağlanır. Not edin ki `x + 1` satırının sonunda noktalı virgül yoktur, şimdiye kadar gördüğünüz çoğu satırdan farklı. İfadeler biten noktalı virgül içermezler. Bir ifadenin sonuna noktalı virgül eklerseniz, onu bir deyime çevirirsiniz ve daha sonra bir değer döndürmeyecek. Sonraki fonksiyon dönüş değerlerini ve ifadeleri keşfederken bunu aklınızda tutun.

### Dönüş Değerine Sahip Fonksiyonlar

Fonksiyonlar çağıran kodlara değerler döndürebilir. Dönüş değerlerini isimlendirmeyiz ancak dönüş tiplerini bir ok (`->`) sonra bildirmeliyiz. Rust'ta, bir fonksiyonun dönüş değeri fonksiyonun gövdesinin bloğundaki final ifadenin değeriyle eşanlamlıdır. Bir fonksiyondan erken dönmek için `return` anahtar kelimesini kullanabilir ve bir değer belirtebilirsiniz, ancak çoğu fonksiyon son ifadeyi örtük olarak döndürür. İşte bir değer döndüren bir fonksiyon örneği:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/src/main.rs}}
```

`five` fonksiyonunda hiçbir fonksiyon çağrısı, makro veya hatta `let` deyimi yoktur—sadece kendisi başına `5` sayısıdır. Bu Rust'ta tamamen geçerli bir fonksiyondur. Fonksiyonun dönüş tipinin de `-> i32` olarak belirtildiğini not edin. Bu kodu çalıştırmayı deneyin; çıktı şöyle görünmelidir:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/output.txt}}
```

`five`'teki `5` fonksiyonun dönüş değeridir, bu yüzden dönüş tipi `i32`'dir. Bunu daha detaylı inceleyelim. Önemli iki bit var: Önce, `let x = five();` satırı bir fonksiyonun dönüş değerini bir değişkeni başlatmakta kullandığımızı gösterir. Çünkü `five` fonksiyonu bir `5` döndürür, o satır aşağıdaki ile aynıdır:

```rust
let x = 5;
```

İkinci, `five` fonksiyonunun parametresi yoktur ve dönüş değerinin tipini tanımlar ancak fonksiyonun gövdesi noktalı virgül olmayan yalnızca `5`'tir çünkü dönmek istediğimiz değerin ifadesidir.

Başka bir örnek bakalım:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-22-function-parameter-and-return/src/main.rs}}
```

Bu kodu çalıştırmak `The value of x is: 6` yazdıracak. Ancak `x + 1` içeren satırın sonuna noktalı virgül koyarsak, onu bir ifadeden bir deyime çevirirsek ne olur?

<span class="filename">Dosya Adı: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/src/main.rs}}
```

Bu kodu derlemek bir hata üretecek, şöyle:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/output.txt}}
```

Ana hata mesajı, `mismatched types`, bu kodla temel soruyu ortaya koyar. `plus_one` fonksiyonunun tanımı, bir `i32` döndüreceğini söyler, ancak deyimler bir değere değerlendirmez, bu `()` ile ifade edilir, birim tipidir. Bu nedenle, hiçbir şey döndürülmez, bu fonksiyon tanımıyla çelişir ve bir hatada sonuçlanır. Bu çıktıda, Rust bu sorunu düzeltmeye yardımcı olmak için bir mesaj sağlar: noktalı virgülü kaldırmayı önerir, bu hatayı düzeltecektir.