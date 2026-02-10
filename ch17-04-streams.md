<!-- Old headings. Do not remove or links may break. -->

<a id="streams"></a>

## Akarslar (Streams): Sıral Future'ler (Streams: Futures in Sequence)

Bu bölümün önceki kısmında async kanalımız için alıcıyı nasıl kullandığımızı
hatırlayın [“Mesaj Geçirme”][17-02-messages]<!-- ignore --> bölümünde. Async
`recv` yöntemi zamanla bir öge dizisi üretir. Bu, çok daha genel bir desen olan
bir _akar_ın (stream) örneğidir. Çok kavram doğal olarak akarslar olarak
temsil edilir: sıradaki ögeler, bilgisayarın belleği için çok büyük olan tam veri kümesinden dosya
sistemi üzerinden artan şekilde çekilen veri paraları, veya zamanla ağ üzerinden gelen veri.
Çünkü akarslar future'lardır, onları diğer herhangi bir tür future ile kullanabiliriz
ve ilginç yollarla birleştirebiliriz. Örneğin, çok ağ çağrısı tetiklemekten kaçınmak için
olayları toplayabilir, uzun süren işlemlerin sıraları üzerinde timeout'lar ayarlayabiliriz,
veya gereksiz iş yapmaktan kaçınmak için kullanıcı arayüz olaylarını kısıtlayabiliriz.

Bölüm 13'te bir öge dizisi gördük, [Yineleyici Özelliği ve `next` Yöntemi]
[iterator-trait]<!-- ignore --> bölümünde Yineleyici özelliğine baktığımızda, ancak
yineleyiciler ve async kanal alıcı arasında iki fark vardır. İlk fark zamandır: yineleyiciler
senkronizedir kanal alıcı ise asenkrondur. İkinci fark API'dır. Doğrudan
`Iterator` ile çalışırken, onun senkronize `next` yöntemini çağırız. Özellikle
`trpl::Receiver` akarsı ile, yerine asenkron `recv` yöntemini çağırıyoruz. Aksi takdirde,
bu API'lar çok benzer hissettirir ve bu benzerlik tesadüf değildir. Bir akar,
yinelemenin asenkron bir biçimidir. Ancak `trpl::Receiver` özellikle mesajları almayı
beklerken genel amaçlı akar API'si çok daha geniştir: `Yineleyici`'in yaptığı gibi
sonraki ögeyi sağlar ancak asenkronda.

Rust'teki yineleyiciler ve akarslar arasındaki benzerlik, herhangi bir
yineleyiciden bir akar oluşturabileceğimiz anlamına gelir. Bir yineleyici ile olduğu gibi,
`next` yöntemini çağırarak ve sonrasında çıktıyı bekleyerek bir akars ile çalışabiliriz,
Kod Listesi 17-21'de ki henüz derlenmez.

<Listing number="17-21" caption="Creating a stream from an iterator and printing its values" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-21/src/main.rs:stream}}
```

</Listing>

Sayılardan oluşan bir dizi ile başlıyoruz, bunu bir yineleyiciye dönüştürüyoruz
sonra tüm değerleri iki katına çıkarmak için `map` çağırıyoruz. Sonra yineleyiciyi
`trpl::stream_from_iter` fonksiyonu kullanarak bir akara dönüştürüyoruz. Sonrasında,
`while let` döngüsü ile akardaki ögeler geldikçe döngüyoruz.

Maalesef, kodu çalıştırmaya çalıştığımızda, derlenmez yerine `next` yönteminin
kullanılamadığı bildirir:

```text
error[E0599]: no method named `next` found for struct `tokio_stream::iter::Iter` in the current scope
  --> src/main.rs:10:40
   |
10 |         while let Some(value) = stream.next().await {
   |                                        ^^^^
   |
   = help: items from traits can only be used if trait is in scope
help: following traits which provide `next` are implemented but not in scope; perhaps you want to import one of them
   |
1  + use crate::trpl::StreamExt;
   |
1  + use futures_util::stream::stream::StreamExt;
   |
1  + use std::iter::Iterator;
   |
1  + use std::str::pattern::Searcher;
   |
help: there is a method `try_next` with a similar name
   |
10 |         while let Some(value) = stream.try_next().await {
   |                                        ~~~~~~~~
```

Bu çıktının açıkladığı gibi, derleyici hatasının nedeni `next` yöntemini
kullanabilmemiz için doğru özelliğin kapsam içinde olmamızdır. Şimdiye kadar olan tartışmamız
göre, bu özelliğin `Stream` olmasını makul bir şekilde bekleyebilirsiniz ancak
aslında `StreamExt`. _uzantı_ için kısa olan `Ext`, Rust topluluğunda bir özelliği
başka biriyle genişletmek için ortak bir desendir.

`Stream` özelliği, etkili olarak `Yineleyici` ve `Future` özelliklerini birleştiren
düşük seviyeli bir arayüz tanımlar. `StreamExt`, `Stream` üzerinde üst düzey bir API seti
sağlar ki bu `next` yöntemini ve `Yineleyici` özelliği tarafından sağlananlara benzer
diğer yardımcı yöntemleri de içerir. `Stream` ve `StreamExt` henüz Rust'ın standart
kütüphanesinin bir parçası değildir ancak çoğu ekosistem crate'leri benzer tanımlamalar
kullanır.

Derleyici hatasının çözümü `trpl::StreamExt` için bir `use` bildirimi eklemektir,
Kod Listesi 17-22'deki gibi.

<Listing number="17-22" caption="Successfully using an iterator as basis for a stream" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-22/src/main.rs:all}}
```

</Listing>

Tüm bu parçalar bir araya getirildiğinde, bu kod istediğimiz gibi çalışıyor! Dahası,
şimdi `StreamExt`'i kapsamızda olduğu için, onun tüm yardımcı yöntemlerini
kullanabiliriz, tıpkı yineleyiciler ile olduğu gibi.

[17-02-messages]: ch17-02-concurrency-with-async.html#message-passing
[iterator-trait]: ch13-02-iterators.html#the-iterator-trait-and-the-next-method