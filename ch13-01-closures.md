<!-- Old headings. Do not remove or links may break. -->

<a id="closures-anonymous-functions-that-can-capture-their-environment"></a>
<a id="closures-anonymous-functions-that-capture-their-environment"></a>

## Kapanışlar (Closures)

Rust'in kapanışları isimsiz fonksiyonlardır ki onları bir değişkende saklayabilir veya başka fonksiyonlara argüman olarak geçirebilirsiniz. Bir kapanışı bir yerde oluşturabilir ve sonra onu farklı bir bağlamda değerlendirmek için elsewhere çağırabilirsiniz. Fonksiyonların aksine, kapanışlar tanımlandıkları kapsamdan değerleri yakalabilir. Bu kapanış özelliklerinin kod yeniden kullanım ve davranış özelleştirmeye nasıl izin verdiğini göstereceğiz.

<a id="creating-an-abstraction-of-beavior-with-closures"></a>
<a id="refactoring-using-functions"></a>
<a id="refactoring-with-closures-to-store-code"></a>
<a id="capturing-the-environment-with-closures"></a>

### Kapsam Yakalama (Capturing the Environment)

Önce, kapanışları kullanarak tanımlandıkları kapsamdan değerleri sonraki kullanım için nasıl yakalayacağımızı inceleyeceğiz. İşte senaryo: Her sıklıkla, T-shirt şirketimiz posta listemizdeki birine bir özel, sınırlı edisyon T-shirt hediye ediyor. Posta listesindeki kişiler proflillerine sevdiği rengi isteğe bağlı olarak ekleyebilirler. Bedava T-shirt için seçilen kişinin sevdiği rengi ayarlanmışsa, o rengin T-shirt'ini alırlar. Eğer kişi bir sevdiği renk belirlememişse, şirketin şu anda çoğunda hangi renge sahipse onu alırlarlar.

Bunu uygulamak için birçok yol vardır. Bu örnekte, kullanacağımız `ShirtColor` adında bir numaralandırmayı kullanıyoruz ki `Red` ve `Blue` varyantları var (basitlik için mevcut renklerin sayısını sınırlıyoruz). Şirketin stokunu `shirts` adında bir alan içeren bir `Vec<ShirtColor>` temsil eden bir `Inventory` struct'ı ile temsil ediyoruz. `giveaway` yöntemi `Inventory` üzerinde tanımlanmış, bedava T-shirt kazananının isteğe bağlı T-shirt rengi tercihini alır ve kişinin alacağı T-shirt rengini döner. Bu ayarlamayı Kod Listesi 13-1'de görüyoruz.

<Listing number="13-1" file-name="src/main.rs" caption="T-shirt şirket hediyeli durumu">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-01/src/main.rs}}
```

</Listing>

`main`'de tanımlanan `store` bu sınırlı edisyon hediye için dağıtmak üzere iki mavi T-shirt ve bir kırmızı T-shirt var. Kırmızı T-shirt tercihine sahip bir kullanıc için ve herhangi bir tercih olmayan bir kullanıc için `giveaway` yöntemini çağırıyoruz.

Yine, bu kod birçok şekilde uygulanabilir ve burada, kapanışlara odaklanmak için, şimdiye kadar öğrendiğiniz kavramlara sadık kaldık, `giveaway` yönteminin gövdesi hariç ki onu bir kapanış kullanıyor. `giveaway` yönteminde, kullanıc tercihini `Option<ShirtColor>` tipinde bir parametre olarak alıyoruz ve `user_preference` üzerinde `unwrap_or_else` yöntemini çağırıyoruz. [`Option<T>` üzerindeki `unwrap_or_else` yöntemi][unwrap-or-else]<!-- ignore --> standart kütüphane tarafından tanımlanmıştır. Bir argüman alır: herhangi bir argüman olmayan ve `T` tipinde bir değer döndüren bir kapanış (bu durumda `Option<T>`'nin `Some` varyantında saklanan aynı tip, `ShirtColor`). Eğer `Option<T>` `Some` varyantıysa, `unwrap_or_else` `Some` içindeki değeri döner. Eğer `Option<T>` `None` varyantıysa, `unwrap_or_else` kapanışı çağırır ve kapanış tarafından dönen değeri döner.

Kapanış ifadesi `|| self.most_stocked()`'i `unwrap_or_else`'e bir argüman olarak belirliyoruz. Bu, hiç parametresi olmayan bir kapanıştır (eğer kapanışın parametresi varsa, onlar iki dikey borular arasında görünecektir). Kapanışın gövdesi `self.most_stocked()`'i çağırıyoruz. Burada bir kapanış tanımlıyoruz ve `unwrap_or_else`'in uygulaması sonuc gerekirse kapanışı değerlendirecektir.

Bu kodu çalıştırmak şunları yazdırır:

```console
{{#include ../listings/ch13-functional-features/listing-13-01/output.txt}}
```

Buradaki ilginç bir yön, geçerli `Inventory` örneği üzerinde `self.most_stocked()`'i çağıran bir kapanış geçirdiğimizdir. Standart kütüphane tanımladığımız `Inventory` veya `ShirtColor` tipleri veya bu senaryoda kullanmak istediğimiz mantık hakkında hiçbir şey bilmeye ihtiyaç duymadı. Kapanış, `self` `Inventory` örneğine değişmez bir referans yakalıyor ve onu `unwrap_or_else` yöntemine belirlediğimiz kodla geçiyoruz. Fonksiyonlar ise, bu şekilde kapsamlarını yakalayamazlar.

<a id="closure-type-inference-and-annotation"></a>

### Kapanış Tiplerini Çıkarma ve Anotasyon Etme (Inferring and Annotating Closure Types)

Fonksiyonlar ve kapanışlar arasında daha fazla farklar vardır. Kapanışlar genellikle `fn` fonksiyonlarının yaptığı gibi parametrelerin veya dönen değerin tiplerini anotasyon etmenizi gerektirmez. Fonksiyonlarda tip anotasyonları gereklidir çünkü tipler kullanıcılara maruz brürülen açık bir arayüzün bir parçasıdır. Bu arayüzü katıca tanımlamak, bir fonksiyonun hangi tiplerde değerler kullandığını ve döndürdüğü konusunda herkesin aynı fikirde olması için önemlidir. Kapanışlar ise, bu türde bir maruz arayüzde kullanılmazlar: Onlar değişkenlerde saklanır ve onlar adlandırılmadan ve kütüphanemizin kullanıcılara maruz bırakılmadan kullanılır.

Kapanışlar tipiksel olarak kısadır ve sadece dar bir bağlamda ilgilidir, herhangi keyfi bir senaryoda değil. Bu sınırlı bağlamlar içinde, derleyici parametrelerin tiplerini ve dönen tipi çıkabilir, çoğu değişkenlerin tiplerini çıkabildiği gibi (derleyicinin kapanış tip anotasyonlarına da ihtiyaç duyduğu nadir durumlar vardır).

Değişkenlerde olduğu gibi, eğer daha fazla açıklık ve netlik sağlamak istersek, katıca gerekenden daha uzun söylen pahasına tip anotasyonları ekleyebiliriz. Bir kapanış için tip anotasyonları eklemek Kod Listesi 13-2'de gösterilen tanıma benzer görünür. Bu örnekte, kapanışı tanımlıyor ve onu bir değişkende saklıyoruz, Kod Listesi 13-1'de yaptığımız gibi kapanışı bir argüman olarak geçirdiğimiz yere tanımlamak yerine.

<Listing number="13-2" file-name="src/main.rs" caption="Kapanışta parametre ve dönen değer tipleri için isteğe bağlı tip anotasyonları ekleme">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-02/src/main.rs:here}}
```

</Listing>

Tip anotasyonları eklenmiş olarak, kapanışların sözdizimi fonksiyonların sözdizimine daha çok benzer. Burada, parametresine 1 ekleyen bir fonksiyonu ve aynı davranışı olan bir kapanışı tanımlıyoruz, karşılaştırma için. İlgili parçaları hizalamak için bazı boşluklar ekledik. Bu, kapanış sözdizimin boruların kullanımı ve sözdizimin miktarının isteğe bağlı olması dışında fonksiyon sözdizimine nasıl benzediğini gösteriyor:

```rust,ignore
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

İlk satır bir fonksiyon tanımını ve ikinci satır tam anotasyon edilmiş bir kapanış tanımını gösterir. Üçüncü satırda, kapanış tanımından tip anotasyonlarını kaldırıyoruz. Dördüncü satırda, parantezleri kaldırıyoruz ki bunlar isteğe bağlıdır çünkü kapanış gövdesi sadece bir ifadesi vardır. Bunların hepsi çağrıldıklarında aynı davranışı üreten geçerli tanımlardır. `add_one_v3` ve `add_one_v4` satırları, derlenmesi için kapanışların değerlendirilmesini gerektirir çünkü tipler kullanımlarından çıkılacaktır. Bu, `let v = Vec::new();`'in tip anotasyonlarını veya `Vec`'e sokulacak bazı tipin değerlerini gerektirmesine benzemektedir.

Kapanış tanımları için, derleyici her parametreleri için ve dönen değerleri için bir somut tip çıkaracaktır. Örneğin, Kod Listesi 13-3, sadece bir parametre olarak aldığı değeri dönen kısa bir kapanışın tanımını gösterir. Bu kapanış, bu örneğin amaçları dışında pek faydalı değildir. Tanıma herhangi bir tip anotasyonu eklemediğimize dikkat edin. Tip anotasyonları olmadığından, herhangi bir tipteki kapanışı çağırabiliriz ki bunu burada ilk kez `String` ile yaptık. Eğer daha sonra `example_closure`'u bir tamsayı ile çağırmaya çalışırsak, bir hata alacağız.

<Listing number="13-3" file-name="src/main.rs" caption="Çıkarılan tipleri olan bir kapanışı iki farklı tiptle çağırmaya çalışma">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-03/src/main.rs:here}}
```

</Listing>

Derleyici bize bu hatayı verir:

```console
{{#include ../listings/ch13-functional-features/listing-13-03/output.txt}}
```

`example_closure`'u ilk kez `String` değeriyle çağırdığımızda, derleyici `x`'in tipini ve kapanışın dönen tipini `String` olarak çıkarır. Bu tipler daha sonra `example_closure`'taki kapanışa kilitlenir ve aynı kapanışı farklı bir tiptle kullanmaya çalıştığımızda bir tip hatası alırız.

### Referans Yakalama veya Mülkiyet Taşıma (Capturing References or Moving Ownership)

Kapanışlar kapsamlarından değerleri üç şekilde yakalayabilir ki bunlar bir fonksiyonun bir parametre alabileceği üç yolla birebir doğrudan eşlenir: değişmez ödünçleme, değiştirilebilir ödünçleme ve mülkiyet alma. Kapanış, gövdesinin yakalanmış değerlerle ne yaptığına dayanarak bunlardan hangisini kullanacağına karar verecektir.

Kod Listesi 13-4'te, sadece değeri yazdırmak için bir değişmez referansa ihtiyaç duyduğu `list` adındaki vektöre değişmez bir referans yakalayan bir kapanış tanımlıyoruz.

<Listing number="13-4" file-name="src/main.rs" caption="Değişmez bir referans yakalayan bir kapanış tanımlama ve çağırma">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-04/src/main.rs}}
```

</Listing>

Bu örnek ayrıca bir değişkenin bir kapanış tanımına bağlanabilceğini ve değişkenin adını ve parantezleri kullanarak daha sonra kapanışı bir fonksiyon adıymış gibi çağırabileceğimizi gösteriyor.

Aynı anda `list`'e birden fazla değişmez referansa sahip olabileceğimizden, `list` kapanış tanımından önceki koddan, kapanış tanımından sonra ancak kapanış çağrılmadan önce, ve kapanış çağrıldıktan sonra hala erişilebilir. Bu kod derlenir, çalışır ve şunları yazdırır:

```console
{{#include ../listings/ch13-functional-features/listing-13-04/output.txt}}
```

Sonra, Kod Listesi 13-5'te, kapanış gövdesini değiştiriyoruz ki onu `list` vektörüne bir eleman ekler. Kapanış şimdi değiştirilebilir bir referans yakalıyor.

<Listing number="13-5" file-name="src/main.rs" caption="Değiştirilebilir bir referans yakalayan bir kapanış tanımlama ve çağırma">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-05/src/main.rs}}
```

</Listing>

Bu kod derlenir, çalışır ve şunları yazdırır:

```console
{{#include ../listings/ch13-functional-features/listing-13-05/output.txt}}
```

`borrows_mutably` kapanışının tanımı ve çağrılması arasında artık bir `println!` olmadığına dikkat edin: `borrows_mutably` tanımlandığında, `list`'e değiştirilebilir bir referans yakalar. Kapanış çağrıldıktan sonra kapanışı tekrar kullanmayız, bu yüzden değiştirilebilir ödünçleme biter. Kapanış tanımı ve kapanış çağrısı arasında, değişmez ödünçleme yazdırmak izin verilmez çünkü değiştirilebilir ödünçleme varken başka ödünçlemelere izin verilmez. Oraya bir `println!` eklemeyi deneyip hangi hata mesajını aldığınızı görün!

Eğer kapanışın kapsamdan kullandığı değerleri, kapanış gövdesinin katıca mülkiyete ihtiyaç duymasa bile, kapanışa mülkiyetini zorlamak istiyorsanız, parametre listesinden önce `move` anahtar kelimesini kullanabilirsiniz.

Bu teknik genellikle kapanışı yeni bir ipliğa geçirerek veriyi taşıyıp mülkiyetini yeni ipliğa verdirmek için kullanışlıdır. İplikleri ve neden onları kullanmak isteyeceğinizi Bölüm 16'de, paralellik hakkında konuştuğumuzda ayrıntılı olarak tartışacağız ancak şimdilik, `move` anahtar kelimesine ihtiyaç duyan bir kapanış kullanarak yeni bir iplik spawn etmekle kısaca keşfedelim. Kod Listesi 13-6, Kod Listesi 13-4'ün değiştirilmiş versiyonunu gösteriyor ki onda vektörü yeni bir iplikte değil ana iplikte yazdırıyor.

<Listing number="13-6" file-name="src/main.rs" caption="`move` kullanarak kapanışın iplik için `list`'in mülkiyetini almasını zorlama">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-06/src/main.rs}}
```

</Listing>

Yeni bir iplik spawn ediyoruz, ipliğe bir argüman olarak çalıştıracağı kapanışı veriyoruz. Kapanış gövdesi listeyi yazdırır. Kod Listesi 13-4'te, kapanış onu yazdırmak için gereken en az erişim miktarı olduğu için sadece `list`'i değişmez referans olarak yakaladı. Bu örnekte, kapanış gövdesi hala sadece değişmez bir referansa ihtiyaç duymasına rağmen, `list`'in kapanışa taşınması gerektiğini belirtmemiz gerekiyor bu, kapanış tanımının başlangıcında `move` anahtar kelimesini koyarak. Eğer ana iplik, yeni iplik üzerindeki `join` çağırmadan önce daha fazla işlem gerçekleştirirse, yeni iplik ana iplik bitmeden önce bitebilir veya ana iplik önce bitebilir. Eğer ana iplik `list`'in mülkiyetini korusa da yeni iplikten önce bitip `list`'i düşürürse, iplikteki değişmez referans geçersiz olur. Bu yüzden, derleyici `list`'in yeni iplike verilene kapanışa taşınmasını gerektirir ki referans geçerli olsun. `move` anahtar kelimesini kaldırmayı veya kapanış tanımlandıktan sonra `main` ipliğinde `list`'i kullanmayı deneyip hangi derleyici hatalarını aldığınızı görün!

<a id="storing-closures-using-generic-parameters-and-the-fn-traits"></a>
<a id="limitations-of-the-cacher-implementation"></a>
<a id="moving-captured-values-out-of-the-closure-and-the-fn-traits"></a>
<a id="moving-captured-values-out-of-closures-and-the-fn-traits"></a>

### Yakalanmış Değerleri Kapanış Dışına Taşıma (Moving Captured Values Out of Closures)

Bir kapanış bir referans yakaladığında veya tanımlandığı kapsamdan bir değerin mülkiyetini (böylece neyin, herhangi bir şeyin, kapanış _içine_ taşınmasını etkileyen) yakaladığında, kapanış gövdesi referanslara veya değerlere ne olacağını tanımlar (böylece neyin, herhangi bir şeyin, kapanış _dışına_ taşınmasını etkileyen).

Bir kapanış gövdesi şunlardan herhangi birini yapabilir: Yakalanmış bir değeri kapanış dışına taşıma, yakalanmış değeri mutasyon etme, ne taşı ne de mutasyon etme, veya kapsamdan hiçbir şey yakalamamayı başla.

Bir kapanışın kapsamdan değerleri nasıl yakaladığı ve ele aldığı, hangi traitleri uyguladığını etkiler ve traitler, fonksiyonların ve structların hangi türde kapanışları kullanabileceklerini nasıl belirtebildikleridir. Kapanışlar, kapanış gövdesinin değerleri nasıl ele aldığına dayanarak, bir ekleme tarzında otomatik olarak bu `Fn` traitlerinin birini, ikisini veya hepsini uygulayacaktır:

* `FnOnce` sadece bir kez çağrılabilen kapanışlar için geçerlidir. Tüm kapanışlar en azından bu trait'ı uygular çünkü tüm kapanışlar çağrılabilir. Gövdesinden değerleri dışına taşıyan bir kapanış sadece `FnOnce`'i ve diğer `Fn` traitlerini uygulamayacak çünkü sadece bir kez çağrılabilir.
* `FnMut`, yakalanmış değerleri gövdesinden dışına taşımayan ancak yakalanmış değerleri mutasyon eden kapanışlar için geçerlidir. Bu kapanışlar birden fazla kez çağrılabilir.
* `Fn`, yakalanmış değerleri gövdesinden dışına taşımayan ve yakalanmış değerleri mutasyon etmeyen, kapsamlarından hiçbir şey yakalamayan kapanışlar için geçerlidir. Bu kapanışlar çevrelerini mutasyon etmeden birden fazla kez çağrılabilir ki bu gibi kapanışı aynı anda birden fazla kez paralellikle çağırma gibi durumlar için önemlidir.

Bölüm 13-1'de kullandığımız `Option<T>` üzerindeki `unwrap_or_else` yönteminin tanımına bakalım:

```rust,ignore
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

`T`'nin `Option`'nun `Some` varyantındaki değerin tipini temsil eden genel bir tip olduğunu hatırlayın. `T` tipi ayrıca `unwrap_or_else` fonksiyonunun dönen tipidir: Örneğin bir `Option<String>` üzerinde `unwrap_or_else` çağıran kod bir `String` alacaktır.

Sonra, `unwrap_or_else` fonksiyonunun `F` adında ek genel tip parametresi olduğuna dikkat edin. `F` tipi `f` adında parametrenin tipidir ki bu, `unwrap_or_else`'i çağırırken sağladığımız kapanıştır.

Genel tip `F` üzerinde belirtilen trait sınırı `FnOnce() -> T`'dir ki bu, `F`'in bir kez çağrılabilmesini, hiç argüman almamasını ve bir `T` döndürmesini gerektirir. Trait sınırında `FnOnce` kullanmak, `unwrap_or_else`'in `f`'i birden fazla kez çağırmayacağını ifade eden bir kısıttır. `unwrap_or_else`'in gövdesinde, `Option` `Some` ise, `f` çağrılmayacığını görebiliriz. Eğer `Option` `None` ise, `f` bir kez çağrılacaktır. Tüm kapanışlar `FnOnce`'i uyguladığından, `unwrap_or_else` tüm üç türde kapanışı kabul eder ve olabildiği kadar esnektir.

> Not: Eğer yapmak istediğimiz bir şeyin kapsamdan değer yakalamayı gerektirmiyorsa, kapanış yerine birinin `Fn` traitlerinden birini uygulayan bir fonksiyonun adını kullanabiliriz. Örneğin, bir `Option<Vec<T>>` değeri üzerinde, değer `None` ise yeni, boş bir vektör almak için `unwrap_or_else(Vec::new)` çağırabiliriz. Derleyici, bir fonksiyon tanımı için geçerli olan `Fn` traitlerinin herhangi birini otomatik olarak uygular.

Şimdi standart kütüphane yöntemi olan dilimlerde tanımlanan `sort_by_key`'e bakalım ki bunun `unwrap_or_else`'den nasıl farklılaştığını ve `sort_by_key`'in trait sınırı için `FnOnce` yerine neden `FnMut` kullandığını görelim. Kapanış, işlemede olan geçerli öğeye bir referans şeklinde bir argüman alır ve `K` tipinde sıralanabilir bir değer döner. Bu yöntem, bir dilimi öğelerin her birinin belirli bir özelliğine göre sıralamak istediğinizde kullanışlıdır. Kod Listesi 13-7'de, `Rectangle` örneklerinden oluşan bir listemiz var ve onları `width` özelliğine göre alçatan yüksekten doğruya `sort_by_key` kullanarak sıralıyoruz.

<Listing number="13-7" file-name="src/main.rs" caption="Dikdörtgenleri genişliğe göre sıralamak için `sort_by_key` kullanma">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-07/src/main.rs}}
```

</Listing>

Bu kod şunları yazdırır:

```console
{{#include ../listings/ch13-functional-features/listing-13-07/output.txt}}
```

`sort_by_key`'in bir `FnMut` kapanışı almak üzere tanımlandığının sebebi, onu birden fazla kez çağırmasıdır: dilimdeki her öğe için bir kez. `|r| r.width` kapanışı herhangi bir şeyi yakalamaz, mutasyon eder veya kapsamdan dışına taşımaz, bu yüzden trait sınır gereksinimlerini karşılar.

Karşılaştırma olarak, Kod Listesi 13-8, sadece `FnOnce` trait'ini uygulayan bir kapanış örneği gösterir çünkü onu bir değeri kapsamdan dışına taşır. Derleyici bize bu kapanışı `sort_by_key` ile kullanmamıza izin vermez.

<Listing number="13-8" file-name="src/main.rs" caption="Bir `FnOnce` kapanışını `sort_by_key` ile kullanmaya çalışma">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-08/src/main.rs:here}}
```

</Listing>

Bu, `list`'i sıralarken `sort_by_key`'un kapanışı kaç kez çağırdığını saymaya çalışmanın (bu çalışmaz) zorakiştirilmiş, karmaşık bir yoldur. Bu kod, saymayı, kapanışın kapsamdan bir `value` - bir `String` - `sort_operations` vektörüne iterek yapmaya çalışır. Kapanış `value`'i yakalar ve sonra `value`'in mülkiyetini `sort_operations` vektörüne aktararak onu kapanış dışına taşır. Bu kapanış sadece bir kez çağrılabilir; ikinci kez çağırmaya çalışmak işe yaramaz çünkü `value` artık kapsamda `sort_operations`'e tekrar itmek için olmayacaktır! Bu yüzden, bu kapanış sadece `FnOnce`'i uygular. Bu kodu derlemeye çalıştığımızda, kapanış bir değeri kapsamdan dışına taşınamaz çünkü kapanış `FnMut` uygulamalıdır hatasını alırız:

```console
{{#include ../listings/ch13-functional-features/listing-13-08/output.txt}}
```

Hata, `value`'in kapanış gövdesinden kapsamdan dışına taşındığı kapanış gövdesindeki satıra işaret eder. Bunu düzeltmek için, kapanış gövdesini değerleri kapsamdan dışına taşımaması için değiştirmemiz gerekiyor. Kapsamda bir sayaç tutmak ve kapanış gövdesinde değerini arttırmak, kapanışın kaç kez çağrıldığını saymanın daha açık bir yoludur. Kod Listesi 13-9'deki kapanış `sort_by_key` ile çalışır çünkü onu sadece `num_sort_operations` sayaçına değiştirilebilir bir referans yakalıyor ve bu yüzden birden fazla kez çağrılabilir.

<Listing number="13-9" file-name="src/main.rs" caption="Bir `FnMut` kapanışını `sort_by_key` ile kullanmak izin verilir.">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-09/src/main.rs}}
```

</Listing>

Kapanışlar veya kapanış kullanan fonksiyonlar veya tipleri tanımlarken veya kullanarken `Fn` traitleri önemlidir. Sonraki bölümde, iteratörleri tartışacağız. Birçok iteratör yöntemi kapanış argümanları alır, bu yüzden bu kapanış detaylarını aklınızda tutarak devam edelim!

[unwrap-or-else]: ../std/option/enum.Option.html#method.unwrap_or_else