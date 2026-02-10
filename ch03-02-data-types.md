## Veri Tipleri

Rust'taki her değer belirli bir _veri tipine_ sahiptir, bu Rust'ın hangi türden verinin belirtildiğini söyler böylece o veriyle nasıl çalışması gerektiğini bilir. İki veri tipi alt kümesine bakacağız: skaler ve bileşik.

Aklınızda bulundurun ki Rust bir _statik tipli_ dildir, bu derleme zamanında tüm değişkenlerin tiplerini bilmesi gerektiği anlamına gelir. Derleyici genellikle değer ve onu nasıl kullanmamıza dayanarak kullanmak istediğimiz tipi çıkarabilir (infer). Birçok tipin mümkün olduğu durumlarda, örneğin [Bölüm 2][comparing-the-guess-to-the-secret-number]<!-- ignore -->'de `parse` kullanarak bir `String`'i sayısal bir tipe dönüştürdüğümüzde, şuna benzer bir tip notasyonu eklememiz gerekir:

```rust
let guess: u32 = "42".parse().expect("Not a number!");
```

Önceki kodda gösterildiği gibi `: u32` tip notasyonunu eklemezsek, Rust aşağıdaki hatayı gösterecek, bu derleyicinin kullanmak istediğimiz tip hakkında bizden daha fazla bilgiye ihtiyaç duyduğunu anlamına gelir:

```console
{{#include ../listings/ch03-common-programming-concepts/output-only-01-no-type-annotations/output.txt}}
```

Başka veri tipleri için farklı tip notasyonlarını göreceksiniz.

### Skaler Tipler

Bir _skaler_ tip tek bir değeri temsil eder. Rust'ın dört temel skaler tipi vardır: tamsayılar, kayan noktalı sayılar, Booleanlar ve karakterler. Bu kelimeleri diğer programlama dillerinden tanıyabilirsiniz. Hemen bunların Rust'ta nasıl çalıştığına atlayalım.

#### Tamsayı Tipleri

Bir _tamsayı_ (integer) kesirli bir bileşen olmayan bir sayıdır. Bölüm 2'de bir tamsayı tipi kullandık, `u32` tipi. Bu tip bildirimi, ilişkili değerin imzasız (unsigned) bir tamsayı olmasını belirtir (imzalı tamsayı türleri `u` yerine `i` ile başlar) ve 32 bit yer kaplar. Tablo 3-1 Rust'taki yerleşik tamsayı tiplerini gösterir. Bir tamsayı değerinin tipini bildirmek için bu varyantlardan herhangi birini kullanabiliriz.

<span class="caption">Tablo 3-1: Rust'ta Tamsayı Tipleri</span>

| Uzunluk | İmzalı | İmzasız |
| ------- | ------- | -------- |
| 8-bit   | `i8`    | `u8`     |
| 16-bit  | `i16`   | `u16`    |
| 32-bit  | `i32`   | `u32`    |
| 64-bit  | `i64`   | `u64`    |
| 128-bit | `i128`  | `u128`   |
| Mimariye bağlı | `isize` | `usize`  |

Her varyant ya imzalı ya da imzasız olabilir ve açık bir boyuta sahiptir. _İmzalı_ ve _imzasız_ sayının negatif olup olamayaçağna atıfta—in diğer kelimelerle, sayının bir işaret (signed) ile birlikte mi olması gerektiği ya da sadece pozitif olacağı ve bu yüzden bir işaret olmadan temsil edilebileceğine (unsigned) atıfta. Bu kağıt üzerinde sayı yazmak gibidir: İşaret önemliyse, bir sayı artı veya eksi işaretiyle gösterilir; ancak sayının pozitif olduğunu varsaymanın güvenli olduğu zaman, işaret olmadan gösterilir. İmzalı sayılar [ikiye tümleme][twos-complement]<!-- ignore --> gösterimi kullanılarak saklanır.

Her imzalı varyant −(2<sup>n − 1</sup>)'ten 2<sup>n − 1</sup> − 1'e kadar dahil sayıları saklayabilir, burada _n_ varyantın kullandığı bit sayısıdır. Yani, bir `i8` −(2<sup>7</sup>)'den 2<sup>7</sup> − 1'e kadar, yani −128'den 127'ye kadar sayıları saklayabilir. İmzasız varyantlar 0'dan 2<sup>n</sup> − 1'e kadar sayıları saklayabilir, bu yüzden bir `u8` 0'dan 2<sup>8</sup> − 1'e kadar, yani 0'dan 255'e kadar sayıları saklayabilir.

Ek olarak, `isize` ve `usize` tipler programın çalıştığı bilgisayarın mimarisine bağlıdır: 64 bit mimaride bulunuyorsanız 64 bit ve 32 bit mimaride bulunuyorsanız 32 bit.

Tablo 3-2'de gösterilen formların herhangi birinde tamsayı değişmezleri yazabilirsiniz. Birden fazla sayısal tip olabilen sayı değişmezleri, örneğin `57u8`, tipi belirtmek için bir tip soneki kullanmalarına izin verir. Sayı değişmezleri sayıyı okumayı kolaylaştırmak için `_`'i görsel ayırıcı olarak da kullanabilirler, örneğin `1_000`, bu `1000` belirtmişsenizle aynı değere sahip olacaktır.

<span class="caption">Tablo 3-2: Rust'ta Tamsayı Değişmezleri</span>

| Sayı değişmezleri | Örnek       |
| ---------------- | ------------- |
| Ondalık          | `98_222`      |
| Onaltılı              | `0xff`        |
| Sekizli            | `0o77`        |
| İkili           | `0b1111_0000` |
| Bayt (`u8` only) | `b'A'`        |

Peki hangi tamsayı tipini kullanacağınızı nasıl bilirsiniz? Emin değilseniz, Rust'ın varsayılanları genellikle başlamak için iyi yerlerdir: Tamsayı tipler varsayılan olarak `i32`'dir. `isize` veya `usize` kullanacağınız temel durum, bir tür koleksiyon indekslemek (indexing) olduğundur.

> ##### Tamsayı Taşması (Integer Overflow)
>
> Diyelim ki 0 ile 255 arasında değerler tutabilen `u8` tipinde bir değişkeniniz var.
> Değişkeni o aralıktan dışarıdaki bir değere, örneğin 256, değiştirmeye
> çalışırsanız, _tamsayı taşması_ (integer overflow) oluşacak, bu iki davranıştan birinde sonuçlanabilir.
> Hata ayıklama modunda derlediğinizde, Rust bu davranış gerçekleşirse programın çalışma
> zamanında _panik_ (panic) yapmasına neden olanan tamsayı taşması kontrolleri içerir.
> Rust bir hata ile çıkan bir program için _panikleme_ (panicking) terimini
> kullanır; Bölüm 9'daki [“`panic!` ile Kurtarılamaz
> Hatalar”][unrecoverable-errors-with-panic]<!-- ignore --> bölümünde panikleri daha
> derinlikle tartışacağız.
>
> `--release` bayrağı ile sürüm modunda derlediğinizde, Rust paniklere
> neden olan tamsayı taşması kontrollerini _yapmaz_. Bunun yerine, taşma oluşursa, Rust
> _ikiye tümleme sarması_ (two's complement wrapping) gerçekleştirir. Kısaca, tipin
> tutabileceği maksimum değerden daha büyük değerler tipin tutabileceği minimum değere
> “sarmaları”. Bir `u8` durumunda, değer 256 0 olur, değer 257 1 olur ve
> böyle devam eder. Program panik yapmayacak, ancak değişkenin muhtemelen beklediğiniz
> bir değere sahip olacaktır. Tamsayı taşmasının sarma davranışına güvenilmesi bir hata olarak
> kabul edilir.
>
> Taşma olasılığını açıkça işlemek için, standart kütüphane tarafından sağlanan
> ilkel sayısal tipler için bu yöntem ailelerini kullanabilirsiniz:
>
> - `wrapping_*` yöntemleriyle, örneğin `wrapping_add`, tüm modlarda sarın.
> - `checked_*` yöntemleriyle, taşma olduğunda `None` değeri döndür.
> - `overflowing_*` yöntemleriyle, taşma olduğunda değeri ve taşma olup olmadığını
>   gösteren bir Boolean döndür.
> - `saturating_*` yöntemleriyle, değerin minimum veya maksimum değerlerinde doygun.

#### Kayan Nokta Tipleri

Rust ayrıca _kayan noktalı sayılar_ için iki ilkel tipe sahiptir, bunlar ondalık noktalı sayılardır. Rust'ın kayan noktalı tipleri `f32` ve `f64`'tür, bunlar sırasıyla 32 bit ve 64 bit boyutundadır. Varsayılan tip `f64`'tür çünkü modern CPU'larda, `f32` ile kabaca aynı hızdadır ancak daha fazla hassasiyet sunabilir. Tüm kayan noktalı tipler imzalıdır.

Kayan noktalı sayıların eylemde olduğunu gösteren bir örnek şudur:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-06-floating-point/src/main.rs}}
```

Kayan noktalı sayılar IEEE-754 standardına göre temsil edilirler.

#### Sayısal İşlemler

Rust tüm sayı tipleri için beklediğiniz temel matematiksel işlemleri destekler: toplama, çıkarma, çarpma, bölme ve kalan. Tamsayı bölmesi sıfıra doğru en yakın tamsayıya yuvarlar. Aşağıdaki kod her sayısal işlemi bir `let` ifadesinde nasıl kullanacağınızı gösterir:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-07-numeric-operations/src/main.rs}}
```

Bu ifadelerdeki her ifade bir matematiksel operatör kullanır ve tek bir değere değerlendirir, sonra bu değer bir değişkene bağlanır. [Ek B][appendix_b]<!-- ignore --> Rust'ın sağladığı tüm operatörlerin listesini içerir.

#### Boolean Tipi

Çoğu diğer programlama dilinde olduğu gibi, Rust'taki Boolean tipinin iki olası değeri vardır: `true` ve `false`. Booleanlar bir bayt boyutundadır. Rust'taki Boolean tipi `bool` kullanılarak belirtilir. Örneğin:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-08-boolean/src/main.rs}}
```

Boolean değerleri kullanmanın temel yolu koşullar üzerinden, örneğin bir `if` ifadesidır. Biz ["Kontrol Akışı"][control-flow]<!-- ignore --> bölümünde `if` ifadelerinin Rust'ta nasıl çalıştığını kapsayacağız.

#### Karakter Tipi

Rust'ın `char` tipi dilin en ilkel alfabetik tipidir. İşte `char` değerlerini bildirme bazı örnekler:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-09-char/src/main.rs}}
```

Dize değişmezlerinin çift tırnak işaretlerini kullandıklarının aksine `char` değişmezlerini tek tırnak işaretleriyle belirttiğimizi not edin. Rust'ın `char` tipi 4 bayt boyutundadır ve Unicode skaler bir değeri temsil eder, bu sadece ASCII'den çok daha fazla şeyi temsil edebileceği anlamına gelir. Aksanlı harfler; Çince, Japonca ve Korece karakterler; emojiler; ve sıfır genişlik boşluklar Rust'ta geçerli tüm `char` değerleridir. Unicode skaler değerleri `U+0000`'dan `U+D7FF`'e ve `U+E000`'dan `U+10FFFF`'e kadar dahildir. Ancak, Unicode'de bir "karakter" gerçekten bir kavram değildir, bu yüzden insan sezgileriniz bir "karakter"in ne olduğu `char`'in Rust'ta olduğu ile uyuşmayabilir. Bunu Bölüm 8'deki ["UTF-8 Kodlanmış Metin Dizeleriyle Saklama"][strings]<!-- ignore --> bölümünde detaylı olarak tartışacağız.

### Bileşik Tipler

_Bileşik tipler_ birden fazla değeri bir tipte gruplayabilir. Rust'ın iki ilkel bileşik tipi vardır: tuple'lar ve diziler.

#### Tuple Tipi

Bir _tuple_ çeşitli tiplerdeki bir sayıdaki değeri tek bir bileşik tipe gruplamanın genel bir yoludur. Tuple'ların sabit bir uzunluğu vardır: Bir kez bildirilirler, boyutlarında büyüyüp küçülemezler.

Parantez içinde virgülle ayrılmış bir değer listesi yazarak bir tuple oluştururuz. Tuple'daki her pozisyonun bir tipi vardır ve tuple'daki farklı değerlerin tiplerinin aynı olması gerekmez. Bu örnekte isteğe bağlı tip notasyonları ekledik:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-10-tuples/src/main.rs}}
```

`tup` değişkeni, tuple tek bir bileşik eleman olarak kabul edildiği için tüm tuple'a bağlanır. Bir tuple'dan bireysel değerleri almak için, şuna benzer tuple değerini ayırmak için desen eşleştirmesini kullanabiliriz:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-11-destructuring-tuples/src/main.rs}}
```

Bu program önce bir tuple oluşturur ve onu `tup` değişkenine bağlar. Sonra `tup`'ü alıp onu üç ayrı `x`, `y` ve `z` değişkenlerine çevirmek için `let` ile bir desen kullanır. Buna _ayırma_ (destructuring) denir çünkü tek tuple'ı üç parçaya böler. Son olarak, program `y` değerini, yani `6.4`'ü, yazdırır.

Ayrıca bir tuple elemanını nokta (`.`) kullanıp ardından erişmek istediğimiz değerin indeksini kullanarak doğrudan erişebiliriz. Örneğin:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-12-tuple-indexing/src/main.rs}}
```

Bu program tuple `x` oluşturur ve sonra ilgili indekslerini kullanarak tuple'ın her elemanına erişir. Çoğu programlama dilinde olduğu gibi, tuple'daki ilk indeks 0'dır.

Herhangi bir değeri olmayan tuple'ın özel bir adı vardır, _unit_. Bu değer ve karşılık tipi her ikisi de `()` ile yazılır ve boş bir değeri veya boş bir dönüş tipini temsil eder. Başka bir değer döndürmezse ifadeler örtük olarak birim değeri döndürür.

#### Dizi Tipi

Birden fazla değerin bir koleksiyonuna sahip olmanın başka bir yolu bir _dizi_ (array) kullanmaktır. Bir tuple'dan farklı olarak, bir dizinin her elemanı aynı tipe sahip olmalıdır. Bazı diğer dillerdeki dizilerden farklı olarak, Rust'taki dizilerin sabit bir uzunluğu vardır.

Bir dizideki değerleri köşeli parantez içinde virgülle ayrılmış bir liste olarak yazarız:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-13-arrays/src/main.rs}}
```

Diziler, verinizin yığın (stack) üzerinde ayrılmasını istediğinizde yararlıdır, şimdiye kadar gördüğümüz diğer tiplerle aynı, yığın (heap) üzerinde olmasına göre ([Bölüm 4][stack-and-heap]<!-- ignore -->'de yığın ve yığınu daha tartışacağız) veya her zaman sabit sayıda elemanlara sahip olduğunuzdan emin olmak istediğinizde. Bir dizi, içerikleri yığın üzerinde yaşadığından boyutu büyüyüp küçülebilen _vektör_ tipi kadar esnek değildir. Vektör, standart kütüphane tarafından sağlanan benzer bir koleksiyon tipidir ve boyutu _izin verilir_ büyüyüp küçülmek için çünkü içerikleri yığın üzerinde yaşar. Bir dizi mi vektör mü kullanacağızdan emin değilseniz, şanslar vektör kullanmanız gerek. Bölüm 8'de vektörleri daha detaylı olarak tartışıyor.

Ancak, eleman sayısının değiştirmeyeceğini bildiğinizde diziler daha yararlıdır. Örneğin, bir programda ay isimlerini kullanıyorsanız, muhtemelen bir vektör yerine bir dizi kullanırsınız çünkü her zaman 12 eleman içereceğini bilirsiniz:

```rust
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
```

Bir dizinin tipini, her elemanın tipini, bir noktalı virgül ve ardından dizideki eleman sayısı ile köşeli parantezler kullanarak yazarsınız, şöyle:

```rust
let a: [i32;5] = [1,2,3,4,5];
```

Burada, `i32` her elemanın tipidir. Noktalı virgülden sonra, sayı `5` dizinin beş eleman içerdiğini belirtir.

Ayrıca bir dizinin her elemanı için aynı değeri içerecek şekilde başlatabilirsiniz, başlangıç değerini, ardından bir noktalı virgül ve ardından köşeli parantez içinde dizinin uzunluğunu belirterek, burada gösterildiği gibi:

```rust
let a = [3;5];
```

`a` adındaki dizi, başlangıçta hepsi `3` değerine ayarlanmış `5` eleman içerir. Bu, `let a = [3,3,3,3,3];` yazmakla aynıdır ancak daha öz bir yoldur.

<!-- Old headings. Do not remove or links may break. -->
<a id="accessing-array-elements"></a>

#### Dizi Elemanına Erişim

Bir dizi, yığın üzerinde ayrılabilecek bilinen, sabit boyutlu tek bir bellek parçasıdır. Bir dizinin elemanlarına indeksleme kullanarak erişebilirsiniz, şöyle:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-14-array-indexing/src/main.rs}}
```

Bu örnekte, `first` adındaki değişken `1` değerini alacak çünkü bu dizinin `[0]` indeksindeki değerdir. `second` adındaki değişken dizinin `[1]` indeksinden `2` değerini alacak.

#### Geçersiz Dizi Elemanı Erişimi

Bir dizinin sonundan geçen bir dizinin elemanına erişmeyi çalıştığınızda ne olacağına bakalım. Bu kodu çalıştırın diyelim, Bölüm 2'deki tahmin oyununa benzer, kullanıcından bir dizi indeksi alın:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust,ignore,panics
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access/src/main.rs}}
```

Bu kod başarıyla derlenir. Bu kodu `cargo run` kullanarak çalıştırırsanız ve `0`, `1`, `2`, `3` veya `4` girerseniz, program dizideki o indekste ilgili değeri yazdıracaktır. Bunun yerine dizinin sonundan geçen bir sayı, örneğin `10`, girerseniz, şuna benzer bir çıktı göreceksiniz:

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access
cargo run
10
-->

```console
thread 'main' panicked at src/main.rs:19:19:
index out of bounds: len is5 but the index is 10
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Program indeksleme işlemi sırasında geçersiz bir değer kullanılan noktada çalışma zamanı hatasıyla sonuçlandı. Program bir hata mesajıyla çıktı ve final `println!` ifadesini çalıştırmadı. Bir elemana indeksleme kullanarak erişmeye çalıştığınızda, Rust belirttiğiniz indeksin dizi uzunluğundan küçük olduğunu kontrol eder. Eğer indeks dizi uzunluğundan büyük veya ona eşitse, Rust panik yapar. Bu kontrolün çalışma zamanında olması gerekir, özellikle bu durumda, çünkü derleyici bir kullanıcının kodu daha sonra çalıştıracağı zaman ne değer gireceğini muhtemelen bilamaz.

Bu, Rust'ın bellek güvenliği ilkelerinin eylemde bir örneğidir. Çoğu düşük seviyeli dillerde bu tür bir kontrol yapılmaz ve yanlış bir indeks sağladığınızda, geçersiz bellek erişilebilir. Rust, bellek erişimine izin vermek ve devam etmek yerine, bu tür hataya karşı sizi korumak için hemen çıkar. Bölüm 9'de Rust'ın hata işlemeden daha fazlasını ve neyin ne panik yapmayan ne de geçersiz bellek erişimi içeren okunabilir, güvenli kodu nasıl yazabileceğinizi tartışır.

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[twos-complement]: https://en.wikipedia.org/wiki/Two%27s_complement
[control-flow]: ch03-05-control-flow.html#control-flow
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[stack-and-heap]: ch04-01-what-is-ownership.html#the-stack-and-the-heap
[vectors]: ch08-01-vectors.html
[unrecoverable-errors-with-panic]: ch09-01-unrecoverable-errors-with-panic.html
[appendix_b]: appendix-02-operators.md