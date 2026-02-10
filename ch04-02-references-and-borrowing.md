## Başvurular ve Ödünç Alma (References and Borrowing)

Kod Listesi 4-5'teki tüple koduyla sorun şu ki `calculate_length` çağrısından sonra hala `String` kullanabilmemiz için `String`'i çağıran fonksiyona geri dönmek zorundayız, çünkü `String` `calculate_length`'e taşınmış. Bunun yerine, bir `String` değerine bir başvuru (reference) sağlayabiliriz. Bir başvuru, takip edebileceğimiz bir adres gibidir ki o adreste saklanan verilere erişmemizi sağlar; bu veri başka bir değişken tarafından sahiptir. Bir göstericiden farklı olarak, bir başvuru belirli bir tipin geçerli değerine işaret edeceği, o başvurunun yaşamı boyunca garantilidir.

İşte `calculate_length` fonksiyonunu, bir değerin sahipliğini almak yerine bir nesneye başvuruya sahip bir parametreyi sahip olacak şekilde nasıl tanımlayacağınızı ve kullanacağınız:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:all}}
```

</Listing>

İlk olarak, tüm tüple kodunun değişken bildiriminde ve fonksiyon dönüş değerinde yok olduğunu fark edin. İkinci olarak, `&s1`'i `calculate_length`'e geçtiğimizi ve onun tanımlamasında `String` yerine `&String` aldığımızı fark edin. Bu ampersanlar başvuruları temsil eder ve size bir değerin sahipliğini almadan atıfta etmenizi izin verirler. Şekil 4-6 bu kavramı tasvir eder.

<img alt="Üç tablo: s tablosu sadece s1 tablosuna bir gösterici içeriyor. s1 tablosu s1 için yığın verisini içerir ve yığındaki dize verisine işaret eder." src="img/trpl04-06.svg" class="center" />

<span class="caption">Şekil 4-6: `&String` `s`'nin `String` `s1`'e işaret ettiğini gösteren bir diyagram</span>

> Not: `&` kullanarak başvurmanın tersi _referanssızlaştırma_ (dereferencing)'dir, bu referanssızlaştırma operatörü olan `*` ile gerçekleştirilir. Referanssızlaştırma operatörünün bazı kullanımlarını Bölüm 8'de göreceğiz ve referanssızlaştırmanın ayrıntılarını Bölüm 15'te konuşacağız.

Buradaki fonksiyon çağrısına daha yakından bakalım:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:here}}
```

`&s1` sözdizimi, bizi `s1`'in değerine _atıfta eden_ ancak onu sahiplenmeyen bir başvuru oluşturmamıza izin verir. Çünkü başvuru ona sahiptir değil, başvurunun işaret ettiği değer başvuru kullanmayı bittiğinde serbest bırakılacaktır.

Benzer şekilde, fonksiyonun imzası parametrenin `s`'nin bir başvuru olduğunu belirtmek için `&` kullanır. Bazı açıklama notasyonları ekleyelim:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-08-reference-with-annotations/src/main.rs:here}}
```

`s` değişkeninin geçerli olduğu kapsam, herhangi bir fonksiyon parametresinin kapsamıyla aynıdır ancak başvurunun işaret ettiği değer, `s` kullanmayı bittiğinde `s` sahiplik sahibi olmadığından serbest bırakılmayacaktır. Fonksiyonlar parametre olarak gerçek değerler yerine başvurulara sahip olduğunda, sahipliği asla geri dönmek zorunda değiliz, çünkü asla sahiplik sahibi olmadık.

Bir başvuru oluşturma eylemine _ödünç alma_ (borrowing) diyoruz. Gerçek hayatta, bir kişi bir şeye sahiptir ise, ondan ondan ödünç alabilirsiniz. İşiniz bittiğinde, onu geri vermelisiniz. Ona sahip değilsiniz.

Peki, ödünç aldığımız bir şeyi değiştirmeye çalışırsak ne olur? Kod Listesi 4-6'teki kodu deneyin. Spoiler uyarısı: Çalışmıyor!

<Listing number="4-6" file-name="src/main.rs" caption="Ödünç alınan bir değeri değiştirmeye çalışma">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-06/src/main.rs}}
```

</Listing>

İşte hata:

```console
{{#include ../listings/ch04-understanding-ownership/listing-04-06/output.txt}}
```

Tıpkı değişkenler varsayılan olarak değişmez (immutable) olduğu gibi başvurular da öyledirler. Bir başvuruya sahip olduğumuz bir şeyi değiştirmemeyize izin verilmez.

### Değiştirilebilir Başvurular (Mutable References)

Kod Listesi 4-6'daki kodu, birkaç küçük ayarlamayla, ödünç aldığımız değeri değiştirmemize izin veren _değiştirilebilir başvuru_ (mutable reference) kullanarak düzeltebiliriz:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-09-fixes-listing-04-06/src/main.rs}}
```

</Listing>

İlk olarak, `s`'i `mut` olacak şekilde değiştiriyoruz. Sonra, `change` fonksiyonunu çağırdığımızda `&mut s` ile bir değiştirilebilir başvuru oluşturuyoruz ve fonksiyon imzasını `&mut String` ile bir değiştirilebilir başvuru kabul etmek üzere güncelliyoruz. Bu, `change` fonksiyonunun ödünç aldığı değeri değiştireceğini çok netleştirir.

Değiştirilebilir başvuruların bir büyük kısıtlaması vardır: Bir değere değiştirilebilir başvurunuz varsa, o değere başka başvurularınız olamaz. `s`'ye iki değiştirilebilir başvuru oluşturmaya çalışan bu kod başarısız:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/src/main.rs:here}}
```

</Listing>

İşte hata:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/output.txt}}
```

Bu hata, bu kodun geçersiz olduğunu söylüyor çünkü `s`'yi bir kerede birden fazla kez değiştirilebilir olarak ödünç alamayız. İlk değiştirilebilir ödünç `r1`'de ve kullanılıncaya kadar kalmalıdır ancak o değiştirilebilir başvurunun oluşturulması ve kullanımı arasından, `r1` ile aynı veriyi ödünç alan başka bir değiştirilebilir başvuru `r2`'yi oluşturmaya çalıştık.

Aynı zamanda aynı verilere birden fazla değiştirilebilir başvuruyu önleyen kısıtlama, değiştirmeyi ancak çok kontrollü bir şekilde izin verir. Bu, yeni Rust öğrenenlerinin çoğu dilin istediğiniz zaman değiştirmenize izin verdiği bir şeyle baş etmeleri. Bu kısıtlamanın faydası şudur ki Rust, derleme zamanında veri yarışmalarını (data races) önleyebilir. Bir _veri yarışması_ (data race), bir koşul durumuna benzerdir ve bu üç davranışın gerçekleştiğinde olur:

- İki veya daha fazla gösterici aynı zamanda aynı verilere erişir.
- Göstericilerin en az biri veriyi yazmak için kullanılıyordur.
- Veriye erişimi senkronize etmek için kullanılan bir mekanizma yoktur.

Veri yarışmaları belirsiz davranışa neden olur ve çalışma zamanında onları takip etmeyi ve düzeltmeyi denemekte zor olabilir; Rust, derleme zamanında veri yarışmaları olan kodu reddederek bu probleminin önler!

Her zaman olduğu gibi, küme parantezleri kullanarak yeni bir kapsam oluşturabiliriz, bu aynı zamanda ancak birden fazla değiştirilebilir başvuru değil, _eşzamanlı_ (simultaneous) olmalarına izin verir:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-11-muts-in-separate-scopes/src/main.rs:here}}
```

Rust, değiştirilebilir ve değişmez başvuruları birleştirmek için benzer bir kuralı zorunlar. Bu kod bir hatada sonuçlanır:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/src/main.rs:here}}
```

İşte hata:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/output.txt}}
```

Vay! Aynı değere bir değişmez başvurumuz varken bir değiştirilebilir başvurumuz da _olamaz_!

Bir değişmez başvurunun kullanıcıları, değerin altlarından aniden değişmesini beklemeler! Ancak, birden fazla değişmez başvuruya izin verilir çünkü veriyi okuyan hiç kimse başka birinin veri okumasını etkileme yeteneği sahiptir.

Bir başvurunun kapsamının tanıtılan yerden başladığını ve başvurunun son kullanıldığı zamana kadar devam ettiğini fark edin. Örneğin, bu kod derlenecektir çünkü değişmez başvuruların son kullanımı `println!`'de ve değiştirilebilir başvuru tanıtılmadan öncedir:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-13-reference-scope-ends/src/main.rs:here}}
```

Değişmez başvuruların `r1` ve `r2`'nin kapsamları, en son kullanıldıkları yerde olan `println!`'de biter ki bu değiştirilebilir başvuru `r3` oluşturulmadan öncedir. Bu kapsamlar çakışmıyor, bu yüzden bu koda izin verilir: Derleyici, başvurunun kapsamın sonunda kullanılmadığı bir noktada söyleyebilir.

Ödünç alma hataları bazen can sıkıcı olsa da, bunun Rust derleyicisinin potansiyel bir hatayı erken (çalışma zamanı yerine derleme zamanında) işaret etmesi ve sorunu tam olarak nerede olduğunu göstermesi olduğunu unutmayın. Sonra, verinizin düşündüğünüz gibi olmamasının nedenini iz sürmek zorunda kalmazsınız.

### Sarkan Başvurular (Dangling References)

Göstericileri olan dillerde, bir _sarkan gösterici_ (dangling pointer) oluşturmak kolaydır bir hatadır - bir gösterici bu bellekteki bir konuma başvurur ki o belleğin başkasına verilmiş olabilir - bazı belleği serbest bırakırken o belleğe bir göstericiyi koruyarak. Buna karşın olarak Rust, derleyici başvuruların asla sarkan başvurular olmayacağını garantiler: Eğer bazı verilere bir başvurunuz varsa, derleyici, verinin o başvurudan önce kapsam dışına çıkmayacağını sağlayacaktır.

Sarkan bir başvuru oluşturmaya ve Rust'in bunları bir derleme zamanı hatasıyla nasıl önlediğini görmek için deneyelim:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/src/main.rs}}
```

</Listing>

İşte hata:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/output.txt}}
```

Bu hata mesajı, henüz ele almadığımız bir özelliğe atıfta eder: yaşam süreleri (lifetimes). Yaşam sürelerini Bölüm 10'da ayrıntılarıyla konuşacağız. Ancak, yaşam süreleriyle ilgili parçaları yok sayarsanız, mesaj bu kodun bir sorun olduğu için anahtarı içerir:

```text
bu fonksiyonun dönüş tipi bir ödünç alınmış değer içeriyor ancak ödünç alınacak bir değer yok
```

`dangle` kodumuzun her aşamasında tam olarak ne olduğunu daha yakından bakalım:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-15-dangling-reference-annotated/src/main.rs:here}}
```

</Listing>

Çünkü `s` `dangle` içinde oluşturulmuştur, `dangle`'nin kodu bittiğinde `s` serbest bırakılacaktır. Ancak ona bir başvuru dönmeye çalıştık. Bu şu demektir ki bu başvuru geçersiz bir `String`'e işaret edecektir. Bu iyi değil! Rust bizi bunu yapmaya izin vermeyecek.

Buradaki çözüm doğrudan `String`'i dönmektir:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-16-no-dangle/src/main.rs:here}}
```

Bu hiçbir problem olmadan çalışır. Sahiplik dışarı taşınmıştır ve hiçbir şey serbest bırakılmamıştır.

### Başvuruların Kuralları

Başvurular hakkında konuştuğumuz şeyleri özetleyelim:

- Herhangi bir kerede, _ya_ tek bir değiştirilebilir başvuru _veya_ herhangi bir sayıda değişmez başvuruya sahip olabilirsiniz.
- Başvurular her zaman geçerli olmalıdır.

Şimdi, farklı bir başvuru türüne bakacağız: dilimler (slices).