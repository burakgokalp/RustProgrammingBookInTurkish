<!-- Old headings. Do not remove or links may break. -->

<a id="shared-state-concurrency"></a>

## Paylaşılan Durum Eşzamanlılığı (Shared-State Concurrency)

Mesaj geçiri eşzamanlılığı ele almanın iyi bir yolu olmakla birlikte, bu tek yol değildir.
Başka bir yöntem, çoklu iş parçacıklarının aynı paylaşılan veriye erişmesi olabilir.
Go dili belgelerinden bu slogannın kısmını tekrar düşünelim: "Belleği
paylaşarak iletişim kurmayın."

Belleği paylaşarak iletişim kurmak nasıl görünirdi? Ayrıca, mesaj geçiri
merakzileri neden bellek paylaşımaması konusunda uyardı?

Bir şekilde, herhangi bir programlama dilindeki kanallar tek sahiplik gibidir çünkü bir değeri bir kanal üzerinden aktardıktan sonra, o değeri artık kullanmanmamlısınız.
Paylaşılan bellek eşzamanlılığı çoklu sahiplik gibidir: Çoklu iş parçacıkları
aynı anda bellek konumuna erişebilir. Bölüm 15'te gördüğümüz gibi, orı
akıllı göstericiler çoklu sahipligi mümkün kıldı, çoklu sahiplik karmaşıklık
ekleyebilir çünkü bu farklı sahiplerin yönetilmesi gerekir. Rust'ın tür sistemi
ve sahiplik kuralları bu yönetimyi doğru almada büyük ölçüde yardımcı olur. Bir
örnek için, mutexlere bakalım, bellek paylaşımı için daha yaygın eşzamanlılık
ilkelinden biri.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-mutexes-to-allow-access-to-data-from-one-thread-at-a-time"></a>

### Mutexlerle Erişimi Kontrol Etme (Controlling Access with Mutexes)

_Mutex_, _karşılıklı hariçetme_ (mutual exclusion) kısaltmasıdır, çünkü mutex sadece
bir iş parçacığının herhangi bir anda belirli bir veriye erişmesine izin verir.
Mutex içindeki veriye erişmek için, bir iş parçacığı önce mutexin kilidini elde etmek
istediğini belirtmelidir. _Kilit_ (lock), mutexin bir parçası olan ve şu anda veriye
özel erişim kime sahip olanı izleyen bir veri yapısıdır. Bu nedenle, mutex
kilitlenme sistemi üzerinden koruduğu veriyi _koruduyor_ olarak tanımlanır.

Mutexler kullanımı zor olduğu için ünleri vardır çünkü iki kuralı hatırlamanız gerekir:

1. Veriyi kullanmadan önce kilidini elde etmeye çalışmanız gerekir.
2. Mutexin koruduğu veri ile işiniz bitdiğinde, veriyi kilitmeniz gerekir
   böylece başka iş parçacıkları kilidi elde edebilir.

Gerçek dünyada bir mutex için bir benzetme, yalnızca bir mikrofoun olan bir panel
tartişması düşünün. Bir panelist konuşmadan önce, mikrofounu kullanmak
istediğini sormalı veya sinyallemlidir. Onlar mikrofounu aldıklarında, istedikleri
kadar konuşabilirler ve sonra mikrofounu konuşmak isteyen bir sonraki
paneliste verirlerler. Bir panelist mikrofounu bitirdiğinde onu devretmeyi
unutsa, başkası konuşamaz. Eğer paylaşılan mikrofounun yönetimi yanlış giderse,
panel planlandığı gibi çalışmaz!

Mutexlerin yönetimi doğru almak inanılmaz derecede zor olabilir, bu nedenle çok
çok insan kanallar konusunda heyecanlıdır. Ancak, Rust'ın tür sistemine ve
sahiplik kurallarına teşekkürler, kilitlenme ve kilidini serbest bırakmayı yanlış yapamazsınız.

#### `Mutex<T>` API'si

Mutex kullanımı hakkında bir örnek olarak, Kod Listesi 16-12'de gösterildiği gibi tek iş
parçacıklı bir bağlamda mutex kullanmayla başlayalım.

<Listing number="16-12" file-name="src/main.rs" caption="Exploring the API of `Mutex<T>` in a single-threaded context for simplicity">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-12/src/main.rs}}
```

</Listing>

Çok türde olduğu gibi, ilişkilendirilmiş bir `new` fonksiyonu kullanarak `Mutex<T>`
oluşturuyoruz. Mutex içindeki veriye erişmek için, kilidi elde etmek için `lock`
yöntemini kullanıyoruz. Bu çağrı şu an iş parçacığını engelleyecek böylece
kilidi bizim olana kadar hiçbir iş yapamaz.

`lock` çağrısı, kilidi tutan başka bir iş parçacığı paniklerse başarısız olur.
Bu durumda, hiç kimse kilidi elde edemezdiğinden bu durumdaysa `unwrap`
çağırıyoruz ve bu iş parçacığı paniklesin.

Kilidi elde ettikten sonra, dönüş değeri olan ve bu durumda `num` adlandırılan
değeri, mutex içindeki veriye değiştirilebilir bir referans olarak ele alabiliriz.
Tür sistemi, `m` içindeki değeri kullanmadan önce kilidi elde etmediğimizi sağlar.
`m`'nin türü `Mutex<i32>`'dir, `i32` değildir, bu nedenle `i32` değerini
kullanabilmek için kilidini elde etmek zorundayız. Unutamayız; tür sistemi bize
iç `i32`'ye erişmemize izin vermez.

`lock` çağrısı bizden `MutexGuard` adlandırılan ve `LockResult` içinde sarılmış
bir tür döner ki bunu `unwrap` çağrısıyla ele aldık. `MutexGuard` türü
iç verimizize işaret etmek için `Deref` uygular; tür ayrıca bir `MutexGuard`
kapsam dışına çıktığında kilidi otomatik olarak bırakan bir `Drop` uygulamasına
sahiptir ki bu iç kapsamın sonunda gerçekleşir. Sonuç olarak, kilidi serbest
bırakmayı unutma ve mutexi diğer iş parçacıkların kullanmasını engelleme riski
taşımayız çünkü kilid serbest bırakma otomatik gerçekleşir.

Kilidi bıraktıktan sonra, mutex değerini yazdırabilir ve iç `i32`'yi `6`'ya
değiştirebildiğimizi görebiliriz.

<!-- Old headings. Do not remove or links may break. -->

<a id="sharing-a-mutext-between-multiple-threads"></a>

#### `Mutex<T>`'ye Paylaşılan Eriş (Shared Access to `Mutex<T>`)

Şimdi `Mutex<T>` kullanarak çoklu iş parçacıkları arasında bir değeri paylaşmayı
deneyelim. 10 iş parçacığı döndereceğim ve her biri bir sayaç değerini 1 artırsın,
böylece sayaç 0'dan 10'a gider. Kod Listesi 16-13'teki örnek bir derleyici hataya
sahiptir olacak ve bu hatayı `Mutex<T>` kullanımı ve Rust'ın bizi doğru kullanmada nasıl
yardımcı olduğunu öğrenmek için kullanacağız.

<Listing number="16-13" file-name="src/main.rs" caption="Ten threads, each incrementing a counter guarded by a `Mutex<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-13/src/main.rs}}
```

</Listing>

`counter` değişkenini bir `Mutex<T>` içinde `i32` tutmak için oluşturuyoruz, Kod
Listesi 16-12'de yaptığımız gibi. Sonra, sayı aralığı üzerinde yineleyerek 10 iş
parçacığı oluşturuyoruz. `thread::spawn` kullanıyoruz ve tüm iş parçacıklarına aynı
kapanğı sağlıyoruz: `counter`'ı iş parçacığına taşıyan, `Mutex<T>` üzerinde
kilidi elde etmek için `lock` yöntemini çağıran ve sonra mutex içindeki değeri 1
artıran. Bir iş parçacığı kapanığını çalıstırdığında, `num` kapsam dışına çıkacak
ve kilidi serbest bırakacak böylece başka bir iş parçacığı onu elde edebilir.

Ana iş parçacığında, tüm birleşme tutamaclarını topluyoruz. Sonra, Kod Listesi
16-2'de yaptığımız gibi, tüm iş parçacıklarının bitmesini sağlamak için her tutamaça
üzerinde `join` çağırıyoruz. O noktada, ana iş parçacığı kilidi elde edecek ve
bu programın sonucunu yazdıracak.

Bu örneğin derlenmeyeceğini ima ettik. Şimdi nedenini bulalım!

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-13/output.txt}}
```

Hata mesajı `counter` değerinin döngünün önceki yinelenmesinde taşındığını
belirtiyor. Rust bize `counter` kilidinin sahipliğini çoklu iş parçacıklarına
taşıyamayacağımızı söylüyor. Derleyici hatayı Bölüm 15'te tartıştığımız
çoklu sahiplik yöntemiyle düzelteyelim.

#### Çoklu İş Parçacıklarına Çoklu Sahiplik (Multiple Ownership with Multiple Threads)

Bölüm 15'te, bir değeri çoklu sahiplere vermek için akıllı gösterici olan `Rc<T>`
kullanarak referans sayılmış bir değer oluşturduk. Burada aynısını yapalım ve ne
olduğunu görelim. Kod Listesi 16-14'te `Mutex<T>`'yi `Rc<T>` ile sarıp
iş parçacığına sahiplik taşımadan önce `Rc<T>`'yi klonlayacağız.

<Listing number="16-14" file-name="src/main.rs" caption="Attempting to use `Rc<T>` to allow multiple threads to own the `Mutex<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-14/src/main.rs}}
```

</Listing>

Yine, derleyelimiz ve... farklı hatalar! Derleyici bize çok şey öğretiyor:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-14/output.txt}}
```

Vay canım, bu hata mesajı çok kelimeyi! İşte odaklanmanız gereken önemli kısım:
`` `Rc<Mutex<i32>>` iş parçacıkları arasında güvenli şekilde gönderilemez ``.
Derleyici bize bunun nedenini de söylüyor: `` `Send` özelliği `Rc<Mutex<i32>>` için uygulanmamış ``.
`Send` özelliğini sonraki bölümde tartışacağız: Bu, iş parçacıklarıyla
kullandığımız türlerin eşzamanlı durumlar için kullanımak üzere olduğunu sağlayan
özelliklerden biri.

Maalesef, `Rc<T>` iş parçacıkları arasında paylaşmak için güvenli değildir. `Rc<T>`
referans sayısını yönetdiğinde, her `clone` çağrısında sayıya ekler ve her
klonun bırakıldığında sayıdan çıkarır. Ancak referans sayısına yapılan
değişikliklerin başka bir iş parçacığı tarafından kesilemeyeceği sağlamak için hiçbir
eşzamanlılık ilkelini kullanmaz. Bu yanlış sayılara yol açabilir—ince
buglar ki bunlar bellek sızıntıra veya biz onunla işimiz bitmeden önce
bir değerin bırakılmasına yol açabilir. İhtiyacımız olan, `Rc<T>`'ye tamamen
benzer olan ancak referans sayısına iş parçacıkları güvenli şekilde yaptığı
değişiklikleri yapan bir türdür.

#### `Arc<T>` ile Atomik Referans Sayma (Atomic Reference Counting with `Arc<T>`)

Şanslı ki, `Arc<T>` eşzamanlı durumlarda kullanmak için güvenli olan `Rc<T>` gibi bir
türdür. _a_, _atomik_ anlamına gelir, yani bu _atomik olarak referans sayılan_
bir türdür. Atomikler bu bölümde detaylı olarak kapsamayacağımız ek bir eşzamanlılık
ilkelidir: Daha fazla bilgi için standart kütüphane belgelerine bakınız.
[`std::sync::atomic`][atomic]<!-- ignore -->. Bu noktada bilmeniz gereken tek şey,
atomiklerin ilkel türler gibi çalıştığı ancak iş parçacıkları arasında paylaşmak için
güvenli olduğudur.

Sonra tüm ilkel türlerin neden atomik olmadığını ve standart kütüphane türlerinin
neden öntanımlı olarak `Arc<T>` kullanmak üzere uygulanmadığını merak edebilirsiniz.
Sebep, iş parçacığı güvenliği gerçekten ihtiyacınızda ödemek istediğiniz bir
performans bedeliyle gelir. Sadece tek bir iş parçacığı içinde değerler üzerinde işlemler
yapıyorsanız, kodunuz atomikler sağlayan garantileri uygulamak zorunda kalmazsa daha
hızlı çalışabilir.

Örneğimize dönelim: `Arc<T>` ve `Rc<T>` aynı API'ye sahiptir, bu nedenle
programımızı `use` satırını, `new` çağrısını ve `clone` çağrısını değiştirerek
düzeltiyoruz. Kod Listesi 16-15'teki kod sonunda derleyecek ve çalışacak.

<Listing number="16-15" file-name="src/main.rs" caption="Using an `Arc<T>` to wrap the `Mutex<T>` to be able to share ownership across multiple threads">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-15/src/main.rs}}
```

</Listing>

Bu kod aşağıdakini yazdıracak:

```text
Result: 10
```

Başardık! 0'dan 10'a kadar saydık, bu pek etkileyici görünmeyebilir ancak bize
`Mutex<T>` ve iş parçacığı güvenliği hakkında çok şey öğretti. Bu programın yapısını
sadece bir sayaçı artırmaktan daha karmaşık işlemler yapmak için de kullanabilirsiniz.
Bu strateji kullanarak, bir hesaplamayı bağımsız paralara bölebilir, bu parçaları
iş parçacıkları arasında yayabilir ve sonra bir `Mutex<T>` kullanarak her iş
parçacığının sonucu kendi parçasıyla güncellemesini sağlayabilirsiniz.

Basit sayısal işlemler yapıyorsanız, standart kütüphanenin [`std::sync::atomic`
modülü][atomic]<!-- ignore --> tarafından sağlanan `Mutex<T>` türlerinden daha basit türler
vardığını not edin. Bu türler ilkel türler için güvenli, eşzamanlı, atomik erişim sağlar.
Bu örnekte `Mutex<T>`'yi bir ilkel türle kullandık ki `Mutex<T>`'nin nasıl çalıştığına
odaklanabilim.

<!-- Old headings. Do not remove or links may break. -->

<a id="similarities-between-refcelltrct-and-mutextarct"></a>

### `RefCell<T>`/`Rc<T>` ve `Mutex<T>`/`Arc<T>` Karşılaştırılması

`counter`'ın değiştirilemez olduğunu ancak içindeki değereye değiştirilebilir bir
referans elde edebildiğimizi fark etmiş olabilirsiniz; bu `Mutex<T>`'nin, `Cell`
ailesi gibi içsal değiştirilebilirlik sağladığı anlamına gelir. `Rc<T>` içindeki
içerikleri Bölüm 15'te `RefCell<T>` kullanarak değiştirebildiğimiz gibi, burada
`Mutex<T>` kullanarak `Arc<T>` içindeki içerikleri değiştiriyoruz.

Dikkat edilecek başka bir detay da şudur ki `Mutex<T>` kullanırken sizin tüm türden
mantık hatalardan Rust sizi koruyamaz. Bölüm 15'ten `Rc<T>` kullanmanın
referans döngüleri oluşturma riskiyle birlikte geldiğini hatırlayın, burada iki
`Rc<T>` değeri birbirine referans verir, bellek sızıntlarına neden olur. Benzer şekilde,
`Mutex<T>` de _kilitlemeler_ (deadlocks) oluşturma riskiyle birlikte gelir. Bu durum,
bir işlemin iki kaynağı kilitlemesi ve iki iş parçacığının her biri bir kilidi
elde ettiği ve böylece birbirlerini sonsuza beklemeyecekleri durumda gerçekleşir.
Kilitlemelerle ilgilisinizyorsanız, bir kilitlemesi olan bir Rust programı oluşturmayı
deneyin; sonra, herhangi bir dildeki mutexler için kilitleme azaltma stratejilerini
arştırın ve bunları Rust'te uygulamayı deneyin. `Mutex<T>` ve `MutexGuard`
için standart kütüphane API belgeleri faydalı bilgi sunar.

Bu bölümü `Send` ve `Sync` özelliklerini ve bunları özel türlerle nasıl
kullanabileceğimizi tartışarak tamamlayacağız.

[atomic]: ../std/sync/atomic/index.html