## Hepsi Bir Arada: Future'ler, Görevler ve İş Parçacıkları (Putting It All Together: Futures, Tasks, and Threads)

[Bölüm 16][ch16]<!-- ignore -->'de gördüğümüz gibi, iş parçacıkları eşzamanlılık için bir yaklaşım sağlar.
Bu bölümde başka bir yaklaşım gördük: future'ler ve akarslarla async kullanma. Bir yöntem ile diğerini
ne zaman seçeceğinizi merak ediyorsanız, cevap: bağımlıdır! Ve çoğu durumda, seçenek iş
parçacıkları _veya_ async değil, iş parçacıkları _ve_ async.

Çoğu işletim sistemi artık iş parçacığı tabanlı eşzamanlılık modelleri sağlamaktadır ve
sonuç olarak çok programlama dili bunları destekler. Ancak bu modellerin değiş tokuşları yoktur.
Çoğu işletim sisteminde, her iş parçacığı için önemli miktarda bellek kullanırlar. İş parçacıkları
ayrıca yalnızca işletim sisteminiz ve donanımınız bunları desteklediğinde bir seçenektir.
Ana akım masaüstü ve mobil bilgisayarların aksine, bazı gömülü sistemler hiçbir işletim sistemine sahip
değildir bu yüzden iş parçacıkları da yoktur.

Async modeli farklı ve nihayetinde tamamlayıcı bir değiş tokuşlar seti sağlar. Async
modelde, eşzamanlı işlemler kendi iş parçacıklarını gerektirmez. Bunun yerine, akarslar bölümünde
bir senkron fonksiyondan işi başlatmak için `trpl::spawn_task` kullandığımız gibi görevler üzerinde
çalışabilirler. Bir görev, bir iş parçacığına benzer ancak işletim sistemi tarafından değil, kütüphane
seviyesinde kod tarafından yönetilir: runtime.

İş parçacığı oluşturma ve görev oluşturma API'ları bu kadar benzer olmasının bir nedeni var.
İş parçacıkları, senkron işlemler kümesi için bir sınır olarak hareket eder; eşzamanlılık iş
parçacıkları _arasında_ mümkündür. Görevler, _asenkron_ işlemler kümesi için bir sınır olarak hareket eder;
eşzamanlılık hem görevler _arasında_ hem de _içinde_ mümkündür çünkü bir görev, gövdesindeki future'ler
arasında değişebilir. Son olarak, future'ler Rust'ın en küçük eşzamanlılık birimidir ve her future
diğer future'lerin ağacını temsil edebilir. Runtime—özellikle onun yürütücüsü—görevleri yönetir
ve görevler future'leri yönetir. Bu bakımdan, görevler işletim sistemi tarafından değil bir runtime tarafından
yönetilen ek yeteneklerle birlikte hafif, runtime yönetilen iş parçacıkları benzerdir.

Bu demek değildir ki async görevler iş parçacıklarından her zaman daha iyidir (veya tersi).
İş parçacıkları ile eşzamanlılık bazı yollarla `async` ile eşzamanlılıktan daha basit bir programlama modelidir.
Bu bir güçlü veya zayıf olabilir. İş parçacıkları biraz "başlat ve unut" şeklindedir; onların
future'e doğal bir karşılığı yoktur bu yüzden işletim sistemi hariç bir şey tarafından kesilmeden
tamamlanana kadar çalışırlar.

Ve iş çıkıyor ki iş parçacıkları ve görevler genellikle birlikte çok iyi çalışır çünkü görevler
(en azından bazı runtime'larda) iş parçacıkları arasında etrafında hareket edilebilir. Aslında, kullanmakta
olduğumuz runtime'ın kapa altında—including `spawn_blocking` ve `spawn_task` fonksiyonları—varsayılan
olarak çok iş parçacıklıdır! Çoğu runtime, iş parçacıklarının şu an kullanıldığı şekilde, sistemin genel
performansını iyileştirmek için iş parçacıkları arasında görevleri şeffafça hareket ettiren _iş çalma_
(work stealing) adı verilen bir yaklaşım kullanır. Bu yaklaşım aslında hem iş parçacıklarını hem de görevleri
ve dolayısıyla future'leri gerektirir.

Ne zaman hangi yöntemi kullanacağınızı düşünürken şu başparmak kurallarını göz önünde bulundurun:

- Eğer iş _çok parallelizasyona müsait_ ise (yani, CPU-bağlı), her bir parçasının ayrı ayrı işlenebileceği
  çok veri işlenmesi gibi, iş parçacıkları daha iyi bir seçenektir.
- Eğer iş _çok eşzamanlı_ ise (yani, I/O-bağlı), farklı zaman aralıklarında veya farklı oranlarda gelebilecek
  çok farklı kaynaklardan mesajları işlemek gibi, async daha iyi bir seçenektir.

Ve hem parallelizm hem de eşzamanlılık ihtiyacınız varsa, iş parçacıkları ve async arasında seçim yapmak
zorunda değilsiniz. Onları özgürce birlikte kullanabilirsiniz, her biri en iyi olduğu rolü oynatmak için.
Örneğin, Kod Listesi 17-25, gerçek dünya Rust kodunda bu tür bir karışımın oldukça yaygın bir örneğini
gösterir.

<Listing number="17-25" caption="Sending messages with blocking code in a thread and awaiting messages in an async block" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-25/src/main.rs:all}}
```

</Listing>

Bir async kanalı oluşturarak başlıyoruz, sonrasında kanalın alıcı tarafını `move` anahtar kelimesini kullanarak
sahiplik alan bir iş parçacığı oluşturuyoruz. İş parçacığı içinde, 1'den 10'a kadar sayıları gönderiyoruz,
her biri arasında bir saniye uyuyoruz. Son olarak, bu bölüm boyunca yaptığımız gibi bir `trpl::block_on`'a
geçilen bir async bloğuyla oluşturulan bir future yürütüyoruz. Bu future içinde, gördüğümüz diğer
mesaj geçirme örneklerinde olduğu gibi o mesajları bekliyoruz.

Bu bölümü başlattığımız senaryoya geri dönmek için, bir video kodlama görevleri kümesini, video kodlamanın
hesaplama-bağlı olduğu için ayrılmış bir iş parçacığı kullanarak ancak bir async kanal ile o işlemlerin
bittiğini UI'ye bildiren olarak çalıştırdığınızı hayal edin. Gerçek dünya kullanım durumlarında bu tür
karşılaşmaların sayısız örneği vardır.

## Özet (Summary)

Bu kitapta göreceğiniz eşzamanlılık bu değil. [Bölüm 21][ch21]<!--
ignore -->'deki proje, burada tartışılan daha basit örneklerden daha gerçekçi bir durumda bu kavramları
uygulayacak ve iş parçacıkları ve görevler ve future'ler ile problem çözme daha doğrudan karşılaştırarak.

Bu yaklaşımların hangisini seçerseniz seçin, Rust size güvenli, hızlı, eşzamanlı kod yazmak için
ihtiyacınız olan araçları verir—ister yüksek verimli bir web sunucusu olsun ister gömülü bir işletim
sistemi.

Sıradan, Rust programlarınız büyüdükçe sorunları yerleşik olarak modellemenin ve çözümleri yapılandırmanın
yollarını konuşacağız. Ayrıca, Rust'ın deyimlerinin, nesne yönelimli programlamadan tanıdığınız
deyimlerle nasıl ilişkili olduğunu tartışacağız.

[ch16]: http://localhost:3000/ch16-00-concurrency.html
[combining-futures]: ch17-03-more-futures.html#building-our-own-async-abstractions
[streams]: ch17-04-streams.html#composing-streams
[ch21]: ch21-00-final-project-a-web-server.html