## Yaşam Süreleri ile Referansları Doğrulama (Validating References with Lifetimes)

Yaşam süreleri (lifetimes), zaten kullandığımız başka bir genel türdür. Bir tipin sahip olduğu davranışı sağlamak yerine, referansların ihtiyacımız sürece kadar geçerli olmasını sağlarlarlar.

Bölüm 4'teki ["Referanslar ve Ödünç Alma"][references-and-borrowing]<!-- ignore --> bölümünde tartışmadığımız bir detay şu ki Rust'taki her referansın bir yaşam süresi vardır ki bu, referansın geçerli olduğu kapsamdır (scope). Çoğu zaman, yaşam süreleri örtüktür (implicit) ve çıkarılır (inferred), tıpkı tiplerin çoğu zaman çıkarıldığı gibi. Yalnızca birden fazla tip mümkün olduğunda tipleri not etmeliyiz. Benzer şekilde, referansların yaşam süreleri birkaç farklı şekilde ilişkili olabileceğinde yaşam sürelerini not etmeliyiz. Rust, çalışma zamanında kullanılan gerçek referansların kesinlikle geçerli olacağından emin olmak için genel yaşam süresi parametrelerini kullanarak ilişkileri not etmemizi zorlar.

Yaşam süresini not etme, çoğu diğer programlama dillerinde olan bile bir kavram değildir, bu yüzden bu alışılmadık gelecektir. Bu bölümde yaşam sürelerinin tamamını kapsamayacak olsak da, kavrama alışkan olmanız için karşılaşabileceğiniz yaygın yaşam süresi sözdizimini tartışacağınız.

<!-- Old headings. Do not remove or links may break. -->

<a id="preventing-dangling-references-with-lifetimes"></a>

### Sarkan Referanslar (Dangling References)

Yaşam sürelerinin temel amacı, sarkan referansları önlemektir ki bunlara varlardığı izin verilirse, bir programın amaçladığı veriden başka bir veriye referans vermesine neden olurlar. Kod Listesi 10-16'deki programa, bir dış kapsam ve bir iç kapsam var, bakın.

<Listing number="10-16" caption="Değeri kapsam dışına çıkan bir referansı kullanma girişimi">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/src/main.rs}}
```

</Listing>

> Not: Kod Listeleri 10-16, 10-17 ve 10-23'teki örnekler değişkenlere
> ilk değer vermeden bildirir, bu yüzden değişken adı dış kapsamda var.
> İlk bakışta bu, Rust'ın boş (null) değerleri olmamasıyla çelişmiş gibi
> görünebilir. Ancak, bir değişkene değer vermeden önce kullanmayı denersek,
> bir derleme zamanı hatası alırız, bu da Rust'ın gerçekten boş değerlere
> izin vermediğini gösterir.

Dış kapsam `r` adında ilk değeri olmayan bir değişken bildirir ve iç kapsam `5` ilk değerine sahip `x` adında bir değişken bildirir. İç kapsamın içinde, `r`'nin değerini `x`'e bir referans olarak ayarlamaya çalışırız. Sonra, iç kapsam biter ve `r`'deki değeri yazdırmaya çalışırız. Bu kod derlenmez, çünkü `r`'nin referans verdiği değer kullanmayı denediğimizden önce kapsam dışına çıkmıştır. İşte hata mesajı:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/output.txt}}
```

Hata mesajı `x` değişkeninin "yeterince kadar yaşamadığını" söylüyor. Sebebi, `x` 7. satırda iç kapsam bittiğinde kapsam dışına çıkacak. Ancak `r` hala dış kapsam için geçerli; kapsamı daha büyük olduğu için, onun "daha uzun yaşadığını" söylüyoruz. Eğer Rust bu kodun çalışmasına izin verseydi, `r` `x` kapsam dışına çıktığında serbest bırakılan (deallocated) hafızaya referans verecek ve `r` ile yapmaya çalıştığımız her şey doğru çalışmaz. O zaman, Rust bu kodun geçersiz olduğunu nasıl belirliyor? Ödünç denetleyicisi (borrow checker) kullanır.

### Ödünç Denetleyicisi (The Borrow Checker)

Rust derleyicisi kapsamları karşılaştırarak tüm ödünçlerin geçerli olup olmadığını belirleyen bir _ödünç denetleyicisine_ sahiptir. Kod Listesi 10-17, değişkenlerin yaşam sürelerini gösteren notlarla Kod Listesi 10-16 ile aynı kodu gösterir.

<Listing number="10-17" caption="`r` ve `x`'in yaşam sürelerine, sırasıyla `'a` ve `'b` adı verilmiş notlar">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-17/src/main.rs}}
```

</Listing>

Burada, `r`'nin yaşam süresini `'a` ile ve `x`'in yaşam süresini `'b` ile notladık. Görebileceğiniz gibi, iç `'b` bloğu dış `'a` yaşam süresi bloğundan çok daha küçük. Derleme zamanında, Rust iki yaşam süresinin boyutunu karşılaştırır ve `r`'nin `'a` yaşam süresine sahip olduğunu ancak `'b` yaşam süresine sahip hafızaya referans verdiğini görür. Program reddedilir çünkü `'b`, `'a`'den kısadır: Referansın konusu, referans kadar uzun yaşamaz.

Kod Listesi 10-18, kodun sarkan referansa sahip olmamasını ve hiçbir hata olmadan derlenmesini düzeltir.

<Listing number="10-18" caption="Geçerli bir referans çünkü veri referanstan daha uzun yaşam süresine sahip">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-18/src/main.rs}}
```

</Listing>

Burada, `x` `'b` yaşam süresine sahip ki bu durumda `'a`'den daha büyük. Bu, `r`'nin `x`'e referans verebileceği anlamına gelir çünkü Rust `r`'deki referansın `x` geçerli olduğu sürece her zaman geçerli olacağını bilir.

Artık referansların yaşam sürelerinin nerede olduğunu ve Rust'ın referansların her zaman geçerli olacağını emin olmak için yaşam sürelerini nasıl analiz ettiğini bildiğinize göre, fonksiyon parametreleri ve dönüş değerlerinde genel yaşam sürelerini keşfedelim.

### Fonksiyonlarda Genel Yaşam Süreleri (Generic Lifetimes in Functions)

İki dize diliminden daha uzun olanı döndüren bir fonksiyon yazacağız. Bu fonksiyon iki dize dilimi alacak ve tek bir dize dilimi döndürecek. `longest` fonksiyonunu uyguladıktan sonra, Kod Listesi 10-19'deki kod `The longest string is abcd` yazdırmalı.

<Listing number="10-19" file-name="src/main.rs" caption="İki dize diliminden daha uzun olanı bulmak için `longest` fonksiyonunu çağıran bir `main` fonksiyonu">

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-19/src/main.rs}}
```

</Listing>

Fonksiyonun dize dilimlerini, dize yerine referanslar olarak almasını istediğimizi not edin, çünkü `longest` fonksiyonunun parametrelerine sahip olmasını istemiyoruz. Kod Listesi 10-19'de kullandığımız parametrelerin neden istediğimiz parametreler oldukları hakkında daha fazla tartışma için Bölüm 4'teki ["Parametre Olarak Dize Dilimleri"][string-slices-as-parameters]<!-- ignore --> bölümüne bakın.

Kod Listesi 10-20'de gösterildiği gibi `longest` fonksiyonunu uygulamayı denerseniz, bu derlenmez.

<Listing number="10-20" file-name="src/main.rs" caption="İki dize diliminden daha uzun olanı döndüren ancak henüz derlenmeyen bir `longest` fonksiyonunun uygulaması">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/src/main.rs:here}}
```

</Listing>

Bunun yerine, yaşam süreleri hakkında konuşan şu hatayı alırız:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/output.txt}}
```

Yardım metni, dönüş tipinin üzerinde bir genel yaşam süresi parametresine sahip olması gerektiğini açığlar çünkü Rust, dönen referansın `x`'e mi yoksa `y`'ye mi referans verdiğini söyleyemez. Aslında, biz de bilmiyoruz, çünkü bu fonksiyonun gövdesindeki `if` bloğu `x`'e bir referans döndürür ve `else` bloğu `y`'ye bir referans döndürür!

Bu fonksiyonu tanımlarken, bu fonksiyona geçirilecek gerçek değerleri bilmiyoruz, bu yüzden `if` durumunun mu yoksa `else` durumunun mu icra edeceğini bilmiyoruz. Ayrıca geçirilecek referansların gerçek yaşam sürelerini de bilmiyoruz, bu yüzden Kod Listeleri 10-17 ve 10-18'de yaptığımız gibi kapsamlara bakarak döndürdüğümüz referansın her zaman geçerli olacağını belirleyemeyiz. Ödünç denetleyicisi de bunu belirleyemez, çünkü `x` ve `y`'nin yaşam sürelerinin dönüş değerinin yaşam süresiyle nasıl ilişkilendiğini bilmez. Bu hatayı düzeltmek için, ödünç denetleyicisinin analizini yapabilmesi için referanslar arasındaki ilişkiyi tanımlayan genel yaşam süresi parametreleri ekleyeceğiz.

### Yaşam Süresi Not Sözdizimi (Lifetime Annotation Syntax)

Yaşam süresi notları, referansların ne kadar yaşadığı değiştirmez. Aksine, referansların yaşam sürelerini birbirleriyle ilişkisini yaşam sürelerini etkilemeden açıklarlar. Tıpkı imzada genel bir tip parametresi belirtilirse fonksiyonlar herhangi bir tipi kabul edebiliyorsa, genel bir yaşam süresi parametresi belirterek herhangi bir yaşam süresine sahip referansları kabul edebilirler.

Yaşam süresi notlarının biraz alışılmadık sözdizimi vardır: Yaşam süresi parametresi adları kesme işareti (`'`) ile başlamalı ve genellikle tümü küçük harf ve çok kısadır, tıpkı genel tipler gibi. Çoğu insanlar birinci yaşam süresi notu için `'a` adını kullanır. Bir referansın `&`'sinden sonra, notu referansın tipinden ayırmak için bir boşluk kullanarak yaşam süresi parametresi notlarını koyarız.

İşte birkaç örnek - yaşam süresi parametresi olmayan bir `i32` referansı, `'a` adı verilmiş bir yaşam süresi parametresine sahip bir `i32` referansı ve ayrıca `'a` yaşam süresine sahip değişilebilir (mutable) bir `i32` referansı:

```rust,ignore
&i32        // bir referans
&'a i32     // belirgin bir yaşam süresine sahip bir referans
&'a mut i32 // belirgin bir yaşam süresine sahip bir değişilebilir referans
```

Tek başına bir yaşam süresi notunun çok fazla anlamı yoktur, çünkü notlar Rust'a birkaç referansın genel yaşam süresi parametrelerinin birbirleriyle nasıl ilişkili olduğunu söylemek içindir. Yaşam süresi notlarının `longest` fonksiyonu bağlamında birbirleriyle nasıl ilişkili olduğuna bakın.

<!-- Old headings. Do not remove or links may break. -->

<a id="lifetime-annotations-in-function-signatures"></a>

### Fonksiyon İmzalarında (In Function Signatures)

Fonksiyon imzalarında yaşam süresi notlarını kullanmak için, genel tip parametreleriyle yaptığımız gibi, fonksiyon adı ve parametre listesi arasındaki açılı ayraçlar içinde genel yaşam süresi parametreleri bildirmemeliyiz.

İmzanın şu kısıtlamayı ifade etmesini istiyoruz: Dönen referans, her iki parametre de geçerli olduğu sürece geçerli olacaktır. Bu, parametrelerin ve dönüş değerinin yaşam süreleri arasındaki ilişkidir. Yaşam süresini `'a` olarak adlandıracağız ve ardından her bir referansa ekleyeceğiz, Kod Listesi 10-21'de gösterildiği gibi.

<Listing number="10-21" file-name="src/main.rs" caption="İmzadaki tüm referansların aynı `'a` yaşam süresine sahip olması gerektiğini belirten `longest` fonksiyonu tanımı">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-21/src/main.rs:here}}
```

</Listing>

Bu kod derlenmeli ve Kod Listesi 10-19'deki `main` fonksiyonu ile kullandığımızda istediğimiz sonucu üretmeli.

Fonksiyon imzası artık Rust'a şu şekilde söylüyor ki bazı yaşam süresi `'a` için, fonksiyon iki parametre alıyor ki bunların ikisi de en az `'a` yaşam süresi kadar yaşayan dize dilimleridir. Fonksiyon imzası ayrıca Rust'a şu şekilde söylüyor ki fonksiyondan dönen dize dilimi en az `'a` yaşam süresi kadar yaşayacak. Pratikte bu, `longest` fonksiyonu tarafından dönen referansın yaşam süresinin, fonksiyon argümanları tarafından referans verilen değerlerin yaşam sürelerinin daha kücüne eşit olduğu anlamına gelir. Bu ilişkiler, Rust'ın bu kodu analiz ederken kullanmasını istediğimiz ilişkilerdir.

Unutmayın, bu fonksiyon imzasında yaşam süresi parametreleri belirttiğimizde, geçirilen veya dönen değerlerin yaşam sürelerini değiştirmiyoruz. Aksine, bu kısıtlamalara uymayan değerleri reddetmesi gerektiğini ödünç denetleyicisine belirtiyoruz. `longest` fonksiyonunun `x` ve `y`'nin tam olarak ne kadar yaşayacağını bilmesine gerekmediğini, sadece bu imzayı sağlayan bir kapsamın `'a` yerine geçebileceğini not edin.

Fonksiyonlarda yaşam sürelerini notlarken, notlar fonksiyon imzasına değil, fonksiyon gövdesine gider. Yaşam süresi notları, imzadaki tipler gibi fonksiyonun sözleşmesinin bir parçası haline gelir. Fonksiyon imzalarının yaşam süresi sözleşmesi içermesi, Rust derleyicisinin yaptığı analizi daha basit hale getirir. Fonksiyonun nasıl notlandığı veya nasıl çağrıldığı ile ilgili bir sorun varsa, derleyici hataları kodumuzun parçasına ve kısıtlamalara daha kesin olarak işaret edebilir. Aksine, Rust derleyicisi yaşam sürelerinin ilişkilerinin ne olması gerektiği hakkında daha fazla çıkarımlar yapsaydı, derleyici yalnızca sorunun nedeninden çok adım uzakta kodumuzun kullanımına işaret edebilirdi.

`longest`'e gerçek referanslar geçirdiğimizde, `'a` yerine geçen gerçek yaşam süresi, `x`'in kapsamının `y`'nin kapsamı ile çakışan bir parçasıdır. Başka bir deyişle, genel yaşam süresi `'a`, `x` ve `y`'nin yaşam sürelerinin daha kücüne eşit olan gerçek yaşam süresini alacak. Dönen referansı aynı yaşam süresi parametresi `'a` ile notladığımız için, dönen referans da `x` ve `y`'nin yaşam sürelerinin daha kücünün uzunluğu için geçerli olacaktır.

Farklı gerçek yaşam sürelerine sahip referansları geçirerek yaşam süresi notlarının `longest` fonksiyonunu nasıl kısıtladığına bakın. Kod Listesi 10-22 basit bir örnektir.

<Listing number="10-22" file-name="src/main.rs" caption="Farklı gerçek yaşam sürelerine sahip `String` değerlerine referanslarla `longest` fonksiyonunu kullanma">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-22/src/main.rs:here}}
```

</Listing>

Bu örnekte, `string1` dış kapsamın sonuna kadar geçerlidir, `string2` iç kapsamın sonuna kadar geçerlidir ve `result` iç kapsamın sonuna kadar geçerli olan bir şeye referans verir. Bu kodu çalıştırırsanız ve görürsünüz ki ödünç denetleyicisi onaylıyor; derlenecek ve `The longest string is long string is long` yazdıracak.

Sonra, `result`'taki referansın yaşam süresinin iki argümanın daha küçük yaşam süresi olması gerektiğini gösteren bir örneği deneyelim. `result` değişkeninin bildirimini iç kapsamın dışına taşıyacağız ancak `result` değişkenine değer atamasını `string2` olan kapsam içinde bırakacağız. Sonra, `result` kullanan `println!`'i iç kapsam dışına taşıyacağız, iç kapsam bittikten sonra. Kod Listesi 10-23'teki kod derlenmez.

<Listing number="10-23" file-name="src/main.rs" caption="`string2` kapsam dışına çıktıktan sonra `result` kullanmaya çalışma">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/src/main.rs:here}}
```

</Listing>

Bu kodu derlemeyi denediğimizde şu hatayı alırız:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/output.txt}}
```

Hata, `result`'ün `println!` ifadesi için geçerli olması için `string2`'nin dış kapsamın sonuna kadar geçerli olması gerektiğini gösterir. Rust bunu biliyor çünkü fonksiyon parametrelerinin ve dönüş değerlerinin yaşam sürelerini aynı yaşam süresi parametresi `'a` kullanarak notladık.

İnsanlar olarak, bu koda bakabiliriz ve `string1`'in `string2`'den daha uzun olduğunu ve bu yüzden `result`'ün `string1`'e bir referans içereceğini görebiliriz. `string1` henüz kapsam dışına çıkmadığı için, `string1` referansı `println!` ifadesi için hala geçerli olacaktır. Ancak, derleyici bu durumda referansın geçerli olduğunu göremez. Rust'a `longest` fonksiyonu tarafından dönen referansın yaşam süresinin, geçirilen referansların yaşam sürelerinin daha kücüne eşit olduğunu söyledik. Bu yüzden, ödünç denetleyicisi, geçersiz bir referansa sahip olabileceği için Kod Listesi 10-23'teki kodu reddeder.

`longest` fonksiyonuna geçirilen referansların değerlerini ve yaşam sürelerini ve dönen referansın nasıl kullanıldığını değiştiren daha fazla deneme tasarlamayı deneyin. Deneylerinizin ödünç denetleyicisini geçip geçmeyeceği hakkında derlemeden önce varsayımlar yapın; sonra, doğru olup olmadığınızı kontrol edin!

<!-- Old headings. Do not remove or links may break. -->

<a id="thinking-in-terms-of-lifetimes"></a>

### İlişkiler (Relationships)

Yaşam süresi parametrelerini belirtmeniz gereken yol, fonksiyonunuzun ne yaptığıya bağlıdır. Örneğin, `longest` fonksiyonunun uygulamasını her zaman ilk parametreyi döndürecek şekilde, en uzun dize dilimini döndürmek yerine değiştirsek, `y` parametresinde bir yaşam süresi belirtmeye gerekmezdi. Aşağıdaki kod derlenecektir:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-08-only-one-reference-with-lifetime/src/main.rs:here}}
```

</Listing>

`x` parametresi ve dönüş tipi için bir yaşam süresi parametresi `'a` belirttik ancak `y` parametresi için değil, çünkü `y`'nin yaşam süresinin `x`'in veya dönüş değerinin yaşam süresiyle hiçbir ilişkisi yok.

Bir fonksiyondan bir referans dönerken, dönüş tipi için yaşam süresi parametresinin parametrelerden birinin yaşam süresi parametresiyle eşleşmesi gerekir. Dönen referans parametrelerden birine _referans vermezse_, bu fonksiyonun içinde oluşturulan bir değere referans vermelidir. Ancak bu bir sarkan referans olurdu çünkü değer fonksiyonun sonunda kapsam dışına çıkacaktı. Derlenmeyen bu `longest` fonksiyonu uygulaması deneyini düşünün:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/src/main.rs:here}}
```

</Listing>

Burada, dönüş tipi için bir yaşam süresi parametresi `'a` belirtmiş olmamıza rağmen, bu uygulama derlenmeyi başaracaktır çünkü dönüş değeri yaşam süresi parametrelerin yaşam süreleriyle hiçbir ilişkisi yok. İşte aldığımız hata mesajı:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/output.txt}}
```

Sorun şu ki `result` `longest` fonksiyonunun sonunda kapsam dışına çıkar ve temizlenir (clean up). Ayrıca fonksiyondan `result`'e bir referans döndürmeye çalışıyoruz. Sarkan referansı değiştirecek yaşam süresi parametrelerini belirlemenin bir yolu yok ve Rust bize bir sarkan referans oluşturmamıza izin vermez. Bu durumda en iyi düzeltme, referans yerine sahibi olunan bir veri tipi döndürmek olacaktır böylece çağıran fonksiyon değeri temizlemekten sorumlu olur.

Sonuç olarak, yaşam süresi sözdizimi, fonksiyonların çeşitli parametrelerinin ve dönüş değerlerinin yaşam sürelerini bağlamakla ilgilidir. Bir kez bağlandıklarında, Rust hafız güvenli işlemlere izin vermek için yeterli bilgiye sahiptir ve sarkan işaretçileri veya hafıza güvenliğini ihlal eden başka işlemlere izin vermez.

<!-- Old headings. Do not remove or links may break. -->

<a id="lifetime-annotations-in-struct-definitions"></a>

### Struct Tanımlarında (In Struct Definitions)

Şimdiye kadar tanımladığımız tüm struct'lar sahibi olunan tipleri tutarlar. Struct'ları referansları tutmak için tanımlayabiliriz ancak bu durumda, struct'ın tanımındaki her bir referansa bir yaşam süresi notu eklemeliyiz. Kod Listesi 10-24'te dize dilimi tutan `ImportantExcerpt` adında bir struct var.

<Listing number="10-24" file-name="src/main.rs" caption="Bir referans tutan bir struct, yaşam süresi notu gerektirir">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-24/src/main.rs}}
```

</Listing>

Bu struct'ın dize dilimi referans olan tek bir alanı `part` vardır. Genel veri tipleriyle olduğu gibi, struct adının arkasındaki açılı ayraçlar içinde genel yaşam süresi parametresinin adını bildiririz böylece struct tanımının gövdesinde yaşam süresi parametresini kullanabiliriz. Bu not, bir `ImportantExcerpt` örneğinin kendi `part` alanında tuttuğu referanstan daha uzun yaşayamayacağı anlamına gelir.

Buradaki `main` fonksiyonu, `novel` değişkenine sahip olan bir `String`'in ilk cümlesine referans tutan bir `ImportantExcerpt` struct örneği oluşturur. `novel`'deki veri `ImportantExcerpt` örneği oluşturulmadan önce vardır. Ayrıca, `novel`, `ImportantExcerpt` kapsam dışına çıktıktan sonra kapsam dışına çıkmaz, bu yüzden `ImportantExcerpt` örneğindeki referans geçerlidir.

### Yaşam Süresi İhmal (Lifetime Elision)

Her referansın bir yaşam süresi olduğunu ve referans kullanan fonksiyonlar veya struct'lar için yaşam süresi parametreleri belirtmeniz gerektiğini öğrendiniz. Ancak, Kod Listesi 4-9'da ve Kod Listesi 10-25'te yeniden gösterilen, yaşam süresi notları olmadan derlenen bir fonksiyonumuz vardı.

<Listing number="10-25" file-name="src/lib.rs" caption="Parametresi ve dönüş tipi referans olmasına rağmen yaşam süresi notları olmadan derlenen, Kod Listesi 4-9'da tanımladığımız bir fonksiyon">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-25/src/main.rs:here}}
```

</Listing>

Bu fonksiyonun yaşam süresi notları olmadan derlenmesinin sebebi tarihseldir: Rust'ın erken sürümlerinde (öncesi-1.0), bu kod derlenmezdi çünkü her referans belirgin bir yaşam süresine ihtiyaç duyardı. O zaman fonksiyon imzası şöyle yazılmış olacaktı:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
}
```

Çok fazla Rust kodu yazdıktan sonra, Rust ekibi belirli durumlarda programcıların aynı yaşam süresi notlarını tekrar tekrar girdiğini buldu. Bu durumlar tahmin edilebilirdi ve birkaç belirleyici (deterministic) kalıbı takip ediyordu. Geliştiriciler bu kalıpları derleyicinin koduna programladılar böylece ödünç denetleyicisi bu durumlarda yaşam sürelerini çıkarabilecek ve belirgin notlara ihtiyaç duymayacaktı.

Bu Rust tarihinin bir parçasıdır çünkü daha fazla belirleyici kalıpları çıkıp derleyiciye eklenebilir. Gelecekte, daha az yaşam süresi notu gerekebilir.

Rust'ın referans analizine programlanan kalıplara _yaşam süresi ihmal kuralları_ denir. Bunlar programcıların takip etmesi gereken kurallar değildir; derleyicinin düşüneceği ve kodunuz bu durumlarla uyarsa yaşam sürelerini açıkça yazmanıza gerekmez.

İhmal kuralları tam çıkarım sağlamaz. Rust kuralları uyguladıktan sonra referansların yaşam süreleri hakkında hala belirsizlik varsa, derleyici kalan referansların yaşam süresinin ne olması gerektiğini tahmin etmez. Tahmin etmek yerine, derleyici yaşam süresi notları ekleyerek çözebileceğiniz bir hata verecektir.

Fonksiyon veya yöntem parametrelerindeki yaşam sürelerine _giriş yaşam süreleri_ ve dönüş değerlerindeki yaşam sürelerine _çıkış yaşam süreleri_ denir.

Derleyici, açık notlar olmadığında referansların yaşam sürelerini belirlemek için üç kural kullanır. Birinci kural giriş yaşam sürelerine, ikinci ve üçüncü kurallar ise çıkış yaşam sürelerine uygulanır. Derleyici üç kuralın sonuna gelirse ve hala yaşam sürelerini belirleyemediği referanslar varsa, bir hata ile durur. Bu kurallar hem `fn` tanımlarına hem de `impl` bloklarına uygulanır.

Birinci kural şu ki derleyici, referans olan her parametreye bir yaşam süresi parametresi atar. Başka bir deyişle, tek parametresi olan bir fonksiyon tek bir yaşam süresi parametresi alır: `fn foo<'a>(x: &'a i32)`; iki parametresi olan bir fonksiyon iki ayrı yaşam süresi parametresi alır: `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`; ve benzeri.

İkinci kural şu ki, tam olarak bir giriş yaşam süresi parametresi varsa, bu yaşam süresi tüm çıkış yaşam süresi parametrelerine atanır: `fn foo<'a>(x: &'a i32) -> &'a i32`.

Üçüncü kural şu ki, birden fazla giriş yaşam süresi parametresi varsa ancak bunlardan biri `&self` veya `&mut self` ise çünkü bu bir yöntemdir, `self`'in yaşam süresi tüm çıkış yaşam süresi parametrelerine atanır. Bu üçüncü kural yöntemleri okumak ve yazmak çok daha güzelleştirir çünkü daha az sembole gereklidir.

Derleyici olduğunu varsayın. Kod Listesi 10-25'teki `first_word` fonksiyonunun imzasındaki referansların yaşam sürelerini belirlemek için bu kuralları uygulayacağız. İmza referanslarla ilişkili hiçbir yaşam süresi olmadan başlar:

```rust,ignore
fn first_word(s: &str) -> &str {
}
```

Sonra, derleyici birinci kuralı uygular ki her parametrenin kendi yaşam süresini belirtir. Geleneksel olduğu gibi bunu `'a` olarak adlandıralım, şimdi imza şöyle:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &str {
}
```

İkinci kural uygulanır çünkü tam olarak bir giriş yaşam süresi var. İkinci kural, tek giriş parametresinin yaşam süresinin çıkış yaşam süresine atandığını belirtir, böylece imza şimdi şöyle:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
}
```

Artık bu fonksiyon imzasındaki tüm referansların yaşam süreleri var ve derleyici programcıya bu fonksiyon imzasındaki yaşam sürelerini notlamadan analizini sürdürebilir.

Başka bir örneğe bakın, bu sefer Kod Listesi 10-20'de çalışmaya başladığımızda yaşam süresi parametresi olmayan `longest` fonksiyonunu kullanarak:

```rust,ignore
fn longest(x: &str, y: &str) -> &str {
}
```

Birinci kuralı uygulayalım: Her parametre kendi yaşam süresini alır. Bu sefer bir yerine iki parametremiz var, bu yüzden iki yaşam süremiz var:

```rust,ignore
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
}
```

Görebileceğiniz gibi ikinci kural uygulanmaz çünkü birden fazla giriş yaşam süresi var. Üçüncü kural da uygulanmaz çünkü `longest` bir yöntem değil, bu yüzden parametrelerden hiçbiri `self` değil. Üç kuralın hepsini çalıştıktan sonra hala dönüş tipinin yaşam süresinin ne olduğunu belirleyemedik. Bu yüzden Kod Listesi 10-20'deki kodu derlemeyi çalığından bir hata aldık: Derleyici yaşam süresi ihmal kurallarıyla çalıştı ancak imzadaki referansların tüm yaşam sürelerini hala belirleyemedi.

Üçüncü kural gerçekten sadece yöntem imzalarında uygulanır, bu yüzden üçüncü kuralın neden yöntem imzalarında sık sık yaşam sürelerini not etmemize gerekmediğini görmek için bir sonraki bölümde yöntemlerdeki yaşam sürelerine bakacağız.

<!-- Old headings. Do not remove or links may break. -->

<a id="lifetime-annotations-in-method-definitions"></a>

### Yöntem Tanımlarında (In Method Definitions)

Yaşam süreleri olan bir struct üzerinde yöntemler uygularken, Kod Listesi 10-11'de gösterildiği gibi genel tip parametreleriyle aynı sözdizimi kullanırız. Yaşam süresi parametrelerini nerede bildirip kullanacağımız, struct alanlarıyla mı yoksa yöntem parametreleri ve dönüş değerleriyle mi ilişkili olduğuna bağlıdır.

Struct alanlarının yaşam süresi adları her zaman `impl` anahtar kelimesinden sonra bildirilmelidir ve ardından struct'ın adından sonra kullanılmalıdır çünkü bu yaşam süreleri struct'ın tipinin bir parçasıdır.

`impl` bloğunun içindeki yöntem imzalarında, referanslar struct'ın alanlarındaki referansların yaşam süresine bağlı olabilir veya bağımsız olabilirler. Ayrıca, yaşam süresi ihmal kuralları genellikle yöntem imzalarında yaşam süresi notlarının gereksiz olmasına neden olur. Kod Listesi 10-24'te tanımladığımız `ImportantExcerpt` adındaki struct'ı kullanan birkaç örneğe bakın.

Önce, tek parametresi `self`'e bir referans ve dönüş değerinin herhangi bir şeye referans olmayan bir `i32` olduğu `level` adında bir yöntemi kullanacağız:

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:1st}}
```

`impl`'ten sonraki yaşam süresi parametresi bildirimi ve tip adından sonraki kullanımı gerekir, ancak birinci ihmal kuralı nedeniyle, `self`'e referansın yaşam süresini notlamaya zorunlu değiliz.

İşte üçüncü yaşam süresi ihmal kuralının uygulandığı bir örnek:

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:3rd}}
```

İki giriş yaşam süresi vardır, bu yüzden Rust birinci yaşam süresi ihmal kuralını uygular ve hem `&self` hem de `announcement` kendi yaşam sürelerini verir. Sonra, parametrelerden biri `&self` olduğu için, dönüş tipi `&self`'in yaşam süresini alır ve tüm yaşam süreleri hesaplanmıştır.

### Statik Yaşam Süresi (The Static Lifetime)

Tartışmamız gereken özel bir yaşam süresi `'static`'dir ki bu, etkilen referansın programın tüm süresince yaşayabileceğini belirtir. Tüm dize değişmezleri (string literals) `'static` yaşam süresine sahiptir ki bunları şu şekilde notlayabiliriz:

```rust
let s: &'static str = "I have a static lifetime.";
```

Bu dizenin metni doğrudan programın ikiliğinde (binary) depolanır ki bu her zaman mevcuttur. Bu yüzden tüm dize değişmezlerinin yaşam süresi `'static`'ir.

Hata mesajlarında `'static` yaşam süresi kullanma önerileri görebilirsiniz. Ancak bir referans için yaşam süresi olarak `'static` belirtmeden önce, sahip olduğunuz referansın gerçekten programın tüm yaşam süresince yaşayıp yaşamadığını ve bunu isteyip istemediğinizi düşünün. Çoğu zaman, `'static` yaşam süresi öneren bir hata mesajı sarkan bir referans oluşturma girişiminden veya mevcut yaşam sürelerinin bir uyumsuzluğundan kaynaklanır. Bu tür durumlarda, çözüm `'static` yaşam süresini belirtmek değil, bu sorunları düzeltmektir.

<!-- Old headings. Do not remove or links may break. -->

<a id="generic-type-parameters-trait-bounds-and-lifetimes-together"></a>

## Genel Tip Parametreleri, Trait Sınırları ve Yaşam Süreleri (Generic Type Parameters, Trait Bounds, and Lifetimes)

Genel tip parametrelerini, trait sınırlarını ve yaşam sürelerinin hepsini tek bir fonksiyonda belirlemenin sözdizimine kısaca bakalım!

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-11-generics-traits-and-lifetimes/src/main.rs:here}}
```

Bu, iki dize diliminden daha uzun olanı döndüren Kod Listesi 10-21'deki `longest` fonksiyonudur. Ancak şimdi `where` tümcesinde belirtildiği gibi `Display` trait'ini uygulayan herhangi bir tiple doldurulabilen genel tip `T`'ye sahip `ann` adında ekstra bir parametresi vardır. Bu ekstra parametre `{}` kullanılarak yazdırılacaktır, bu yüzden `Display` trait sınırı gereklidir. Çünkü yaşam süreleri bir genel türdür, yaşam süresi parametresi `'a`'nin ve genel tip parametresi `T`'nin bildirimleri fonksiyon adından sonraki açılı ayraçların içindeki aynı listeye gider.

## Özet (Summary)

Bu bölümde çok şeyi kapsadık! Artık genel tip parametrelerini, trait'leri ve trait sınırlarını ve genel yaşam süresi parametrelerini bildiğinize göre, birçok farklı durumda çalışan tekrarsız kod yazmaya hazırsınız. Genel tip parametreleri kodunuzu farklı tiplere uygulamanıza izin verir. Trait'ler ve trait sınırları, tipler genel olsalar bile, kodunuzun ihtiyaç duyduğu davranışı sahip olacaklarını sağlar. Bu esnek kodun hiçbir sarkan referansına sahip olmayacağını emin olmak için yaşam süresi notlarını nasıl kullanacağınızı öğrendiniz. Ve tüm bu analiz derleme zamanında gerçekleşir ki bu çalışma zamanı performansını etkilemez!

İnanır mısın mısınız, bu bölümde tartıştığımız konularda öğrenilecek çok daha fazla şey var: Bölüm 18 trait nesnelerini (trait objects) tartışır ki bu trait'leri kullanmanın başka bir yoludur. Ayrıca çok gelişmiş senaryolarda sadece ihtiyaç duyacağınız daha karmaşık senaryolarda yaşam süresi notları vardır; bunlar için [Rust Başvurusunu][reference] okumalısınız. Ancak sonraki, kodunuzun doğru çalıştığından emin olmak için Rust'te testleri nasıl yazacağınızı öğreneceksiniz.

[references-and-borrowing]: ch04-02-references-and-borrowing.html#references-and-borrowing
[string-slices-as-parameters]: ch04-03-slices.html#string-slices-as-parameters
[reference]: ../reference/trait-bounds.html