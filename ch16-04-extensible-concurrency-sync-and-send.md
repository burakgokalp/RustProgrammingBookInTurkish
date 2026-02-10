<!-- Old headings. Do not remove or links may break. -->

<a id="extensible-concurrency-with-the-sync-and-send-traits"></a>
<a id="extensible-concurrency-with-the-send-and-sync-traits"></a>

## `Send` ve `Sync` ile Genişletilebilir Eşzamanlılık (Extensible Concurrency with `Send` and `Sync`)

İlginç şekilde, bu bölümde şimdiye kadar tartıştığımız neredeyse her eşzamanlılık özelliği standart kütüphanenin parçası değil dilin. Eşzamanlılığı ele almanız için seçenekleriniz dil veya standart kütüphane ile sınırlı değildir; kendi eşzamanlılık özelliklerinizi yazabilir veya başkaları tarafından yazılanları kullanabilirsiniz.

Ancak, dilde standart kütüphaneden ziyade gömülü olan anahtar eşzamanlılık kavramları arasında `std::marker` özellikleri olan `Send` ve `Sync` bulunur.

<!-- Old headings. Do not remove or links may break. -->

<a id="allowing-transference-of-ownership-between-threads-with-send"></a>

### İş Parçacıkları Arasında Sahiplik Transfer Etmek (Transferring Ownership Between Threads)

`Send` işaretçi özelliği, `Send` uygulayan türün değerlerinin sahipliğinin iş parçacıkları arasında transfer edilebileceğini gösterir. Neredeyse her Rust türü `Send` uygular, ancak bazı istisnalar vardır, `Rc<T>` dahil: Bu `Send` uygulayamaz çünkü bir `Rc<T>` değerini klonlar ve klonu başka bir iş parçacığına sahiplik transfer etmeye çalışırsanız, her iki iş parçacığı aynı anda referans sayısını güncelleyebilir. Bu nedenle, `Rc<T>` iş parçacığı güvenli performans bedelini ödemek istemediğiniz tek iş parçacıklı durumlarda kullanmak için uygulanır.

Bu nedenle, Rust'ın tür sistemi ve özellik sınırları, hiçbir zaman kasıtsızca `Rc<T>` değerini iş parçacıkları arasında güvenli olmayan şekilde gönderemeyeceğinizi sağlar. Bunu Kod Listesi 16-14'te yapmayı denediğimizde, `` özellik `Send`, `Rc<Mutex<i32>>` için uygulanmamış `` hatasını aldık. `Send` uygulayan `Arc<T>`'ye geçtiğimizde, kod derlendi.

`Send` türlerinden tamamen oluşan herhangi bir tür otomatik olarak `Send` olarak da işaretlenir. Ham göstericiler hariç (Bölüm 20'de tartışacağız), neredeyse tüm ilkel türler `Send`'dir.

<!-- Old headings. Do not remove or links may break. -->

<a id="allowing-access-from-multiple-threads-with-sync"></a>

### Çoklu İş Parçacıklarından Eriş (Accessing from Multiple Threads)

`Sync` işaretçi özelliği, `Sync` uygulayan türün çoklu iş parçacıklarından referans edilmesinin güvenli olduğunu gösterir. Başka bir deyimle, herhangi bir `T` türü, `&T` (`T`'ye değiştirilemez bir referans) `Send` uyguluyorsa `Sync` uygular, bu da referansın başka bir iş parçacığına güvenli şekilde gönderilebileceği anlamına gelir. `Send`'e benzer şekilde, ilkel türlerin tümü `Sync` uygular ve `Sync` uygulayan türlerden tamamen oluşan türler de `Sync` uygular.

Akıllı gösterici `Rc<T>` de, `Send` uygulamadığı aynı nedenlerle `Sync` uygulamaz. `RefCell<T>` türü (Bölüm 15'te bahsettik) ve ilgili `Cell<T>` türlerin ailesi `Sync` uygulamaz. `RefCell<T>`'nin çalışma zamanında yaptığı ödünç kontrol uygulaması iş parçacığı güvenli değildir. `Mutex<T>` akıllı göstericisi `Sync` uygular ve [ "`Mutex<T>`'ye Paylaşılan Eriş"] [shared-access]<!-- ignore -->'te gördüğümüz gibi çoklu iş parçacıkları ile paylaşılan erişim için kullanılabilir.

### `Send` ve `Sync`'yi Manuel Olarak Uygulamak Güvensizdir (Implementing `Send` and `Sync` Manually Is Unsafe)

`Send` ve `Sync` özelliklerini uygulayan diğer türlerden tamamen oluşan türler de otomatik olarak `Send` ve `Sync` uyguladığı için, bu özellikleri manuel olarak uygulamamıza gerek yoktur. İşaretçi özellikleri olarak, uygulayacak yöntemleri bile yoktur. Bunlar sadece eşzamanlılıkla ilgili sabitleri dayatmak için kullanışlıdır.

Bu özellikleri manuel olarak uygulamak güvensiz Rust kodunu uygulamayı içerir. Bölüm 20'de güvensiz Rust kodunu kullanmayı tartışacağız; şimdilik önemli bilgi şudur ki, `Send` ve `Sync` parçalarından oluşmayan yeni eşzamanlı türler oluşturmak güvenlik garantilerini korumak için dikkatli düşünüm gerektir. ["Rustonomicon"][nomicon] bu garantiler ve bunları nasıl koruyacağız hakkında daha fazla bilgiye sahiptir.

## Özet (Summary)

Bu, bu kitapta eşzamanlılık olarak göreceğiniz son değil: Sonraki bölüm asenkron programlamaya odaklanacak ve Bölüm 21'deki proje burada tartışılan küçük örneklerden daha gerçekçi bir durumda bu bölümdeki kavramları kullanacak.

Daha önce belirtildiği gibi, Rust'ın eşzamanlılığı nasıl ele aldığının çok azı dilin parçası olduğundan, çok fazla eşzamanlılık çözümü crate olarak uygulanır. Bunlar standart kütüphaneden daha hızlı gelişir, bu nedenle çok iş parçacıklı durumlarda kullanmak için güncel, en gelişmiş crate'leri çevrimiçi aradığınızdan emin olun.

Rust standart kütüphanesi mesaj geçiri için kanallar ve `Mutex<T>` ve `Arc<T>` gibi, eşzamanlı bağlamlarda kullanmak için güvenli olan akıllı gösterici türleri sağlar. Tür sistemi ve ödünç kontrolü, bu çözümleri kullanan kodun veri yarışlarına veya geçersiz referanslarla sonuçlanmayacağını sağlar. Kodunuzu derlediğinizde, onun çok iş parçacığında mutlu bir şekilde çalışacağından ve diğer dillerde ortak olan iz sürmesi zor hata türlerinden kaynaklanmayacağından emin olabilirsiniz. Eşzamanlı programlama artık korkulması gereken bir kavram değil: İleri gidin ve programlarınızı eşzamanlı yapın, korkmadan!

[shared-access]: ch16-03-shared-state.html#shared-access-to-mutext
[nomicon]: ../nomicon/index.html