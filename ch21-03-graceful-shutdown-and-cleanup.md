## Nazik Kapanış ve Temizleme (Graceful Shutdown and Cleanup)

Kod Listesi 21-20'deki kod, bir iş parçacığı havuzu kullanarak isteklere eşzamanlı (asynchronously) yanıt
veriyor, planlandığı gibi. `workers`, `id` ve `thread` alanları hakkında doğrudan bir şekilde kullanmadığımızı
hatırlatan bazı uyarılar alıyoruz ki bizim bir şeyi temizlemediğimizi hatırlatıyoruz. <kbd>ctrl</kbd>-<kbd>C</kbd> yöntemini kullanarak ana
iş parçacığını durdurduğumuzda, tüm diğer iş parçacıkları da hemen durduruluyor, bir isteği sunmakta
olmasalar bile.

Sonra ise, havuzdaki her iş parçacığında `join`'ı çağırmak için `Drop` trait'ini uygulayacağız ki onlar,
kapanmadan üzerinde çalıştıkları isteklerini bitirebilsinler. Sonra iş parçacıklarına yeni istekleri kabul
etmemeleri söylemenin ve kapanmaları söyeceğiz bir yöntem uygulayacağız. Bu kodu eylemde görmek için,
sunucumuzu, iş parçacığı havuzunu nazikçe kapanmadan önce sadece iki isteği kabul edecek şekilde değiştireceğiz.

İlerlerken dikkat etmeniz gereken bir şey: Bu hiçbiri, kapanmaları çalıştıran kodun kısımlarını etkilemez, bu yüzden buradaki her şey,
bir iş parçacığı havuzu kullanan bir async çalışma zamanı (runtime) kullanıyormuşsak gibi olurdu.

### `ThreadPool` üzerinde `Drop` Trait'ini Uygulama (Implementing `Drop` Trait on `ThreadPool`)

İş parçacığı havuzumuz üzerinde `Drop` uygulamayarak başlayalım. Havuz düşürüldüğinde,
iş parçacıklarımızın tümü birleşmeleri ve çalışmalarını bitirmeler. Kod Listesi 21-22,
bir `Drop` uygulamasına ilk denemesini gösterir; bu kod henüz tam çalışmaz.

<Listing number="21-22" file-name="src/lib.rs" caption="Joining each thread when thread pool goes out of scope">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-22/src/lib.rs:here}}
```

</Listing>

Önce, iş parçacığı havuzunun `workers` üzerinden döngüyoruz. Bunu için `&mut` kullanıyoruz çünkü
`self`, değiştirilebilir bir referanstır ve ayrıca `worker`'i değiştirebilmemize ihtiyaç duyuyoruz. Her
`worker` için, bu belirli `Worker` örneğinin kapanacağını söyleyen bir mesaj yazdırıyor ve sonra o
`Worker` örneğinin iş parçacığı üzerinde `join`'ı çağırıyoruz. `join` çağrısı başarısızsa,
Rust'ın paniklemesi ve nazik olmayan bir kapanışa gitmesi için `unwrap` kullanıyoruz.

İşte bu kodu derlediğimizde aldığımız hata:

```console
{{#include ../listings/ch21-web-server/listing-21-22/output.txt}}
```

Hata bize `join`'ı çağıramadığımızı söylüyor çünkü her `worker`'in sadece değiştirilebilir bir ödününe
sahibiz ve `join`, argümanının sahipliğini alır. Bu sorunu çözmek için, `thread`'ı, `thread`'u tutan
`Worker` örneğinden dışarı taşımamıza ihtiyaç duyuyoruz ki `join`, iş parçacığını tüketebilsin. Bunu
yapmanın bir yolu, Kod Listesi 18-15'te aldığımız aynı yaklaşımı kullanmaktır. Eğer `Worker`, bir
`Option<thread::JoinHandle<()>>` tutuyorsa, `Option` üzerindeki `take` yöntemini çağırarak değeri
`Some` varyantından dışarı taşıyabiliriz ve yerine `None` varyantını bırakabiliriz. Diğer
sözlerle, çalışan bir `Worker`, `thread`'ta `Some` varyantına sahip olacak ve bir `Worker`'ı temizlemek
istediğimizde, `Worker`'un çalıştırabileceği bir iş parçacığı olmaması için `Some`'i `None` ile
değiştirebiliriz.

Ancak bu yaklaşımın karşacağı zaman _sadece_ `Worker` düşürüldüğünde olur. Bunun karşılığında,
`worker.thread` eriştiğimiz her yerde bir `Option<thread::JoinHandle<()>>` ile uğraşmak zorunda
kalacağız. Rust, `Option`'yi oldukça sık kullanır ancak kendinizi, bildiğiniz her zaman var olacak bir şeyi
bir çalışma yolu olarak bu şekilde bir `Option`'ye sardığınızı bulursanız, kodunuzu daha temiz ve
daha az hataya eğilimli yapmak için alternatif yaklaşımlara bakmak iyi bir fikirdir.

Bu durumda, daha iyi bir alternatif var: `Vec::drain` yöntemi. Vektörden hangi öğeleri
kaldıracağını belirlemek için bir aralık parametresi kabul eder ve bu öğelerin bir yineleyicisini (iterator) döner.
`..` aralık söz dizimini (range syntax) geçerek *tüm* değerleri vektörden kaldırır.

Bu yüzden, `ThreadPool`'un `drop` uygulamasını şöyle güncellememiz gerekiyor:

<Listing file-name="src/lib.rs">

```rust
{{#rustdoc_include ../listings/ch21-web-server/no-listing-04-update-drop-definition/src/lib.rs:here}}
```

</Listing>

Bu, derleyici hatasını çözer ve kodumuza başka bir değişiklik gerektirmez. Unutmayın ki
drop, paniklerken çağrılabilir bu yüzden `unwrap` da panikleyebilir ve çift paniklemeye (double panic)
neden olabilir ki bu, programı hemen çöker ve devam etmekteki temizlemeyi sonlandırır. Bu, örnek bir
program için iyidir ancak üretim kodu için tavsiye edilmez.

### İş Parçacıklarına İş Dinlemeyi (Listening for Jobs) Bırkmalarını Söyleme (Signaling to Threads to Stop Listening for Jobs)

Yaptığımız tüm değişikliklerle kodumuz herhangi bir uyarı olmadan derleniyor. Ancak
kötü haber şu ki kod hala istediğimiz şekilde çalışmıyor. Anahtar, `Worker` örneklerinin
iş parçacıkları tarafından çalıştırılan kapanmalardaki mantık: şu anda `join` çağırıyoruz ancak bu iş
parçacıklarını kapatmıyor çünkü onlar işler için sonsuza `loop`'ta (döngüde). `ThreadPool`'umuzu
mevcut `drop` uygulamasıyla düşürmeye çalışırsak, ana iş parçacığı, ilk iş parçacığının bitmesini
beklerken sonsuza blokecek.

Bu sorunu çözmek için, `ThreadPool`'un `drop` uygulamasında bir değişiklik ve sonra
`Worker` döngüsünde bir değişiklik gerekiyor.

Önce, `ThreadPool`'un `drop` uygulamasını, iş parçacıklarını bitmesini beklemenden önce
`sender`'ı açıkça düşürecek şekilde değiştireceğiz. Kod Listesi 21-23, `ThreadPool` üzerindeki
değişiklikleri açıkça `sender` düşürmeyi gösterir. İş parçacıklarının aksine, burada iş
parçacığını `ThreadPool`'dan taşıyabilmek için `Option` kullanmak _zorundayız_ `Option::take`
kullanmakla.

<Listing number="21-23" file-name="src/lib.rs" caption="Explicitly dropping `sender` before joining `Worker` threads">

```rust,noplayground,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-23/src/lib.rs:here}}
```

</Listing>

`sender`'ı düşürmek kanalı kapatıyor ki bu, başka mesajın gönderilmeyeceğini gösteriyor. Bu
olduğunda, `Worker` örneklerinin sonsuz döngüde yaptığı `recv` çağrılarının tümü bir hata dönecektir.
Kod Listesi 21-24'te, `Worker` döngüsünü, o durumda döngüden nazikçe çıkmak için değiştiriyoruz ki bu
demek ki `ThreadPool`'un `drop` uygulaması, `Worker` örneklerinin üzerinde `join`'ı çağırdığında iş
parçacıkları bitirecek.

<Listing number="21-24" file-name="src/lib.rs" caption="Explicitly breaking out of loop when `recv` returns an error">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-24/src/lib.rs:here}}
```

</Listing>

Bu kodu eylemde görmek için, `main`'i, sunucuyu nazikçe kapanmadan önce sadece iki isteği kabul
edecek şekilde değiştirelim, Kod Listesi 21-25'te gösterildiği gibi.

<Listing number="21-25" file-name="src/main.rs" caption="Shutting down server after serving two requests by exiting loop">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/listing-21-25/src/main.rs:here}}
```

</Listing>

Gerçek dünyadaki bir web sunucusunun sadece iki istek sunma'dan sonra kapanmasını istemessiniz.
Bu kod sadece nazik kapanış ve temizlemenin doğru sırada çalıştığını gösteriyor.

`take` yöntemi, `Iterator` trait'inde tanımlanır ve yinelemeyi en fazla ilk iki öğe ile
sınırlar. `ThreadPool`, `main`'in sonunda kapsamdan (scope) çıkacak ve `drop` uygulaması
çalıştırılacak.

Sunucuyu `cargo run` ile başlatın ve üç istek yapın. Üçüncü istek hatasız olmalı ve terminalinizde buna
benzer çıktı görmelisiniz:

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-25
cargo run
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
third request will error because server will have shut down
copy output below
Can't automate because of output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.41s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

Farklı `Worker` ID'leri ve yazdırılan mesajların sıralamasını görebilirsiniz. Bu kodun
nasıl çalıştığını mesajlardan görebilirsiniz: `Worker` örnekleri 0 ve 3, ilk iki isteği aldı.
Sunucu ikinci bağlantıdan sonra bağlantıları kabul etmeyi durdurdu ve `ThreadPool` üzerindeki `Drop`
uygulaması, `Worker` 3 işini başlatmadan önce çalıştırmaya başladı. `sender`'ı düşürmek, tüm
`Worker` örneklerini koparıyor ve onlara kapanmaları söylüyor. Her `Worker` örneği, kopuğunda bir mesaj
yazdırıyor ve sonra iş parçacığı havuzu, her `Worker` iş parçacığının bitmesini beklemek için `join`
çağırıyor.

Bu belirli yürütmenin ilginç bir yönüne dikkat edin: `ThreadPool`, `sender`'ı düşürdü ve
herhangi bir `Worker` bir hata almadan önce, biz `Worker 0`'ı `join` yapmaya çalıştık. `Worker 0`,
henüz `recv`'ten bir hata almamıştı bu yüzden ana iş parçacığı, `Worker 0`'ın bitmesini beklerken
blokecek. O sırada, `Worker 3` bir iş aldı ve sonra tüm iş parçacıkları bir hata aldı.
`Worker 0` bitirdiğinde, ana iş parçacığı, geriye kalan `Worker` örneklerinin bitmesini bekledi.
O noktada, onların tümü döngülerinden çıkmış ve durmuşlardı.

Tebrikler! Projemizimizi tamamladık; bir iş parçacığı havuzu kullanarak eşzamanlı (asynchronously)
yanıt veren temel bir web sunucusumuz var. Sunucunun nazik kapanışını gerçekleştirebiliyoruz ki bu,
havuzdaki tüm iş parçacıklarını temizler.

İşte referans için tam kod:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/main.rs}}
```

</Listing>

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/lib.rs}}
```

</Listing>

Buradan daha fazlasını yapabiliriz! Bu projeyi iyileştirmeye devam etmek istiyorsanız, işte bazı fikirler:

- `ThreadPool` ve genel yöntemlerine daha fazla dokümantasyon ekleyin.
- Kütüphanenin fonksiyonalitesi için testler ekleyin.
- `unwrap` çağrılarını daha sağlam hata ele alma ile değiştirin.
- `ThreadPool`'ı web isteklerini sunmak dışında bazı görevleri gerçekleştirmek için kullanın.
- [crates.io](https://crates.io/) adresinde bir iş parçacığı havuzu crate'i bulun ve
onun yerine crate kullanarak benzer bir web sunucusu uygulayın. Sonra, API'sini ve
sağlamlığını, uyguladığımız iş parçacığı havuzu ile karşılaştırın.

## Özet (Summary)

Harika! Kitabın sonuna ulaştınız! Bu Rust turu sırasında bize katıldığınız için teşekkür etmek
istiyoruz. Artık kendi Rust projelerinizi uygulayabilir ve diğer kişilerin projelerine yardımcı olabilirsiniz. Aklınızda,
karşalarla karşılaştığınızda size yardımcı olmayı seven hoş karşılamalar (welcoming community) bulunduğunu
tutun.

[integer-types]: ch03-02-data-types.html#integer-types
[moving-out-of-closures]: ch13-01-closures.html#moving-captured-values-out-of-closures
[type-aliases]: ch20-03-advanced-types.html#type-synonyms-and-type-aliases
[builder]: ../std/thread/struct.Builder.html
[builder-spawn]: ../std/thread/struct.Builder.html#method.spawn