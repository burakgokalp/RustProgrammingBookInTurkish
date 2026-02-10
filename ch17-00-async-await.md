# Asenkron Programlamanın Temelleri: Async, Await, Futures ve Streams (Fundamentals of Asynchronous Programming: Async, Await, Futures, and Streams)

Bilgisayara yaptırdığımız çok işlem tamamlanması biraz sürebilir.
Bu uzun süreçlerin tamamlanmasını beklerken başka bir şey yapabilseydik güzel olurdu.
Modern bilgisayarlar aynı anda birden fazla işlem üzerinde çalışmak için iki teknik sunar:
paralellik ve eşzamanlılık. Programlarımızın mantığı ise çoğunlukla
doğrusal bir şekilde yazılmış. Programın yapması gereken işlemleri ve bir
fonksiyonun durabileceği ve programın başka bir kısmı yerine çalışabileceği
noktaları belirtmek isteriz, her bir kod parçasının çalışması gereken tam sırayı ve
yöntemi baştan belirtmek zorunda kalmadan. _Asenkron programlama_, kodumuzu
olası durdurma noktaları ve sonuçları açısından ifade etmemize ve bizim
için koordinasyon detaylarını halletmemize izin veren bir soyutlamadır.

Bu bölüm, Bölüm 16'de kullanılan iş parçacıklarının paralellik ve
eşzamanlılık kullanımını, alternatif bir kod yazma yaklaşımını tanıtarak geliştirir:
Rust'ın futures, streams ve `async` ve `await` sözdizimi bize işlemlerin
asenkron olabileceğini nasıl ifade etmemizi sağlar ve asenkron runtimeleri (çalışma
zamanı) uygulayan üçüncü parti crate'ler: asenkron işlemlerin yürütülmesini
yöneten ve koordine eden kod.

Bir örnek düşünelim. Ailenizin kutlaması için oluşturduğunuz bir videoyu dışa
aktarıyorsunuz, bu işlem dakikadan saate kadar sürebilir. Video dışa aktarma, mümkün
olan kadar çok CPU ve GPU gücü kullanacaktır. Yalnızca bir CPU çekirdeginiz var ve işletim
sisteminiz dışa aktarmayı tamamlanana kadar durdurmazsa—yani, _eşzamanlı_
olarak yürütülseyse—o görev çalışırken bilgisayarınızda başka bir şey yapamazdınız.
Bu oldukça sinir edici bir deneyim olurdu. Neyse ki, bilgisayarınızın işletim sistemi,
diğer işleri aynı anda yapabilmeniz için dışa aktarmayı sık sık görünmez şekilde
durdurabilir ve şimdiye kadar yapıyor.

Şimdi başkasının paylaştığı bir videoyu indiriyorsunuz, bu da biraz sürebilir ancak
CPU süresi kadar çok almaz. Bu durumda, CPU verinin ağdan gelmesini beklemek zorundadır.
Veri gelmeye başladığı anda okumaya başlayabilirsiniz, ancak tümü belirsin için bazen biraz
sürebilir. Veri hepsi bile hazır olduktan sonra, video oldukça büyükse, tümünü yüklemek en az
bir iki saniye sürebilir. Bu pek çok gibi duyulmayabilir ancak modern bir işlemci için bu çok
uzun bir süredir ki her saniye milyarlarca işlem gerçekleştirebilir. Yine, işletim
sisteminiz programınızı görünmez şekilde durdurarak CPU'nun ağ çağrısının bitmesini beklerken
diğer işleri yapmasına izin verir.

Video dışa aktarma, bir _CPU-bağlı_ veya _hesap-bağlı_ (compute-bound)
işlemi örneğidir. CPU veya GPU içinde bilgisayarın olası veri işleme hızı ve o
hızın ne kadarını işleme adayabileceği ile sınırlıdır. Video indirme, bir _I/O-bağlı_
işlemi örneğidir, çünkü bilgisayarın _giriş ve çıiş_ hızı ile sınırlıdır; verinin
ağ üzerinden gönderilebildiği kadar hızlı gidebilir.

Bu iki örnekte de, işletim sisteminin görünmez durdurmaları bir eşzamanlılık
biçimi sunar. Bu eşzamanlılık sadece tüm program seviyesinde gerçekleşir: İşletim
sistemi bir programı durdururken diğer programların iş yapmasını sağlar. Çoğu durumda,
programlarımızı işletim sisteminden çok daha granüler bir seviyede anladığımız için, işletim
sisteminin göremediği eşzamanlılık fırsatlarını görebiliriz.

Örneğin, dosya indirmelerini yönetmek için bir araç yapıyorsak, programımızı
bir indirmenin UI'yi kilitlenmesin ve kullanıcıların aynı anda birden fazla indirmeyi
başlatabilmesi için yazabilmeliyiz. Ağ ile etkileşmek için çok işletim sistemi API'si
_engelleyici_ (blocking) tir; yani, işledikleri veriler tamamen hazır olana kadar
programın ilerlemesini engeller.

> Not: Bunu düşünürseniz bu _çok_ fonksiyon çağrısının çalışma biçimidir.
> Ancak, _engelleyici_ terimi genellikle dosyalar, ağ veya bilgisayardaki diğer
> kaynaklarla etkileşen fonksiyon çağrıları için ayrılır, çünkü bu bireysel
> programların işlemin _engellemeyen_ (non-blocking) olmasından fayda
> sağlayacağı durumlar.

Ana iş parçacığımızı engellemekten kaçınmak için her dosyayı indirmek için
özel bir iş parçacığı dönderebiliriz. Ancak, bu iş parçacıkları tarafından kullanılan
sistem kaynaklarının yükü sonunda sorun haline gelebilir. Çağrının ilk yerde engellememesi
tercih edilir, bunun yerine programımızın tamalamasını istediğimiz görev sayısını
tanımlayabilir ve runtime'a en iyi sırayı ve yürütme biçimini seçmesini sağlayabiliriz.

Bu tam olarak Rust'ın _async_ (asynchronous için kısaltma) soyutlaması bize
verdiğidir. Bu bölümde, aşağıdaki konuları kapsarken async hakkında her şeyi öğreneceksiniz:

- Rust'ın `async` ve `await` sözdizimini ve asenkron fonksiyonları
  bir runtime ile nasıl kullanacağız
- Asenkron modeli kullanarak Bölüm 16'de baktığımız bazı aynı sorunları nasıl
  çözeceğiz
- Çok iş parçacıklılığın ve async'in birçok durumda birleştirebileceğiniz
  tamamlayıcı çözümler nasıl sağladığı

Ancak pratikte async'in nasıl çalıştığına bakmadan önce, paralellik ve eşzamanlılık arasındaki
farkları tartışmak için kısa bir ayrılmaya ihtiyacımız var.

## Paralellik ve Eşzamanlılık (Parallelism and Concurrency)

Şimdiye kadar paralellik ve eşzamanlılığı çoğunlukla birbirinin yerine kullanılabilir
olarak ele aldık. Şimdi bunları daha kesin olarak ayırmamız gerekir çünkü farklar
çalışmaya başladıkça ortaya çıkacaktır.

Bir yazılım projesi üzerinde çalışmayı nasıl bölebileceğini düşünelim.
Tek bir üyeye birden fazla görev atayabilir, her üyeye bir görev atayabilir veya bu iki
yaklaşımın bir karışımını kullanabiliriz.

Bir bireysel, tamamlanmadan önce birkaç farklı görev üzerinde çalışıyorsa, bu
_eşzamanlılıktır_. Eşzamanlılığı uygulamanın bir yolu, bilgisayarınızda iki farklı
projeyi check-out yapmış ve bir projede sıkılıp ya da sıkıldığınızda diğerine geçmiş
olmanıza benzerdir. Tek kişisiniz, bu yüzden aynı anda her iki görevde ilerleme
yapamazsınız ancak çok görev yapabilirsiniz, onlar arasında geçerek birinde birinde ilerleme
yaparsınız (Bkz. Şekil 17-1).

<figure>

<img src="img/trpl17-01.svg" class="center" alt="A diagram with stacked boxes labeled Task A and Task B, with diamonds in them representing subtasks. Arrows point from A1 to B1, B1 to A2, A2 to B2, B2 to A3, A3 to A4, and A4 to B3. The arrows between subtasks cross boxes between Task A and Task B." />

<figcaption>Şekil 17-1: Görev A ve Görev B arasında geçiş yapan eşzamanlı bir iş akışı</figcaption>

</figure>

Ekip üyelerinin herinin bir görev alıp yalnız çalışması ile bir grup görevi
böldüğünde, bu _paralelliktir_. Ekipteki her kişi aynı anda ilerleme
yapabilir (Bkz. Şekil 17-2).

<figure>

<img src="img/trpl17-02.svg" class="center" alt="A diagram with stacked boxes labeled Task A and Task B, with diamonds in them representing subtasks. Arrows point from A1 to A2, A2 to A3, A3 to A4, B1 to B2, and B2 to B3. No arrows cross between boxes for Task A and Task B." />

<figcaption>Şekil 17-2: Görev A ve Görev B üzerinde işin bağımsız olarak gerçekleştiği paralel bir iş akışı</figcaption>

</figure>

Bu iki iş akışında da, farklı görevler arasında koordinasyon yapmak zorunda kalabilirsiniz.
Belki bir kişiye atanan görevi herkesin işinden tamamen bağımsız düşünmüşsünüzdür ancak
aslında ekipteki başka bir kişinin önce görevini bitirmesini gerektir. Çalışmanın bazı
parçaları paralel olarak yapılabilir ancak bazıları aslında _seri_ (serial): Şekil 17-3'teki
gibi, birbiri ardınca, yalnızca bir seride gerçekleşebilir.

<figure>

<img src="img/trpl17-03.svg" class="center" alt="A diagram with stacked boxes labeled Task A and Task B, with diamonds in them representing subtasks. In Task A, arrows point from A1 to A2, from A2 to a pair of thick vertical lines like a "pause" symbol, and from that symbol to A3. In task B, arrows point from B1 to B2, from B2 to B3, from B3 to A3, and from B3 to B4." />

<figcaption>Şekil 17-3: Kısmen paralel bir iş akışı, Görev A ve Görev B üzerinde iş bağımsız olarak gerçekleşir ta ki Görev A3 Görev B3'ün sonuçlarında bekletilir.</figcaption>

</figure>

Benzer şekilde, kendi görevlerinizden birinin başka görevinize bağımlı olduğunu fark
edebilirsiniz. Şimdi eşzamanlı işiniz de seri olmuş.

Paralellik ve eşzamanlılık da birbirleriyle kesişebilir. Meslekdaşınızın,
görevlerinizden birini bitirip bitirmeyeceğini öğrendiğinizde, muhtemelen tüm
çabalarınızı o göreve odaklayarak meslekdaşınızı "kilitsiz" yaparsınız. Siz ve iş
arkadaşınız artık paralel çalışamaz ve kendi görevleriniz üzerinde artık eşzamanlı olarak da
çalışamazsınız.

Aynı temel dinamikler yazılım ve donanımla da devreye girer. Tek bir CPU çekirge
olan bir makinede, CPU bir seferde yalnızca bir işlem gerçekleştirebilir ancak hala
eşzamanlı çalışabilir. İş parçacıkları, süreçler ve async gibi araçları kullanarak,
bilgisayar bir faaliyeti durdurabilir ve diğerlerine geçebilir ve son olarak ilk faaliyete
döndürebilir. Çoklu CPU çekirge olan bir makinede, aynı zamanda paralel olarak
iş de yapabilir. Bir çekirge bir görev yaparken başka bir çekirge tamamen ilişkisiz bir görev
yapabilir ve bu işlemler aynı anda gerçekleşir.

Rust'ta async kodunun çalışması genellikle eşzamanlı olur. Kullandığımız donanım,
işletim sistemi ve async runtime'a (hakkında daha fazlası yakında) bağlı olarak bu eşzamanlılık
arkaplanda paralellik de kullanabilir.

Şimdi, Rust'ta asenkron programlamanın nasıl çalıştığına dalalım.