<!-- Old headings. Do not remove or links may break. -->

<a id="comparing-performance-loops-vs-iterators"></a>

## Döngüler ve İteratörlerde Performans (Performance in Loops vs. Iterators)

Döngüler veya iteratörler kullanmak arasında seçim yapmak için, hangi uygulamanın daha hızlı olduğunu bilmeniz gerekiyor: açık bir `for` döngüsü olan `search` fonksiyonunun sürümü veya iteratör kullanan sürüm.

Bir benchmark çalıştırdık ki bu, Sir Arthur Conan Doyle'un _Sherlock Holmes'un Maceraları_ kitabının tüm içeriğini bir `String`'e yükleyip içeride _the_ kelimesini arıyor. İşte `for` döngüsü kullanan `search` sürümü ve iteratör kullanan sürüm için benchmark sonuçları:

```text
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
```

İki uygulamanın benzer performansları var! Burada benchmark kodunu açıklamayacağız çünkü amaç iki sürümün eşdeğer olduğunu kanıtlamak değil ama bu iki uygulamanın performans açısından nasıl karşılaştırıldığı hakkında genel bir his almak.

Daha kapsamlı bir benchmark için, `contents` olarak çeşitli boyutlardaki çeşitli metinleri, `query` olarak çeşitli kelime ve farklı uzunluklardaki kelimeleri ve diğer tüm çeşitli varyasyonları kullanarak kontrol etmelisiniz. Amaç şu: İteratörler, yüksek düzey bir soyutlama olmalarına rağmen, alt düzey kodu kendiniz yazmışsınız gibi kabaca aynı koda derlenir. İteratörler Rust'ın _sıfır maliyetli soyutlamalarından_ (zero-cost abstractions) biridir ki bu, soyutlamayı kullanmanın ek bir çalışma zamanı ek yükü getirmemesi anlamına gelir. Bu, C++'ın orijinal tasarımcısı ve uygulayıcısı Bjarne Stroustrup'un 2012 ETAPS keynote konuşması "C++'ın Temelleri"nde sıfır ek yükü (zero-overhead) tanımladığı şeye benzer:

> Genel olarak, C++ uygulamaları sıfır ek yük ilkesine uyar: Kullanmadığınız
> için ödemezsiniz. Ve dahası: Kullandığınız için, elle daha iyi kod
> yazamazsınız.

Birçok durumda, iteratörler kullanan Rust kodu, elle yazacağınız aynı assemble'yi derler. Döngü açma ve dizi erişimi üzerinde sınır kontrolü gibi optimizasyonlar uygulanır ve sonuç kodu son derece verimli yapar. Şimdi bunu bildiğinize, tereddüt olmadan iteratörleri ve kapanışları kullanabilirsiniz! Onlar kodun yüksek düzeyde görünmesini sağlar ama bunu yapmak için bir çalışma zamanı performans cezası getirmez.

## Özet (Summary)

Kapanışlar ve iteratörler fonksiyonel programlama dili fikirlerinden ilham alınan Rust özellikleridir. Onlar, Rust'ın düşük düzey performansla yüksek düzey fikirleri net bir şekilde ifade etme yeteneğine katkıda bulunurlar. Kapanışların ve iteratörlerin uygulamaları çalışma zamanı performansını etkilemez. Bu, sıfır maliyetli soyutlamalar sağlamak için çabalama Rust'ın hedefinin bir parçasıdır.

I/O projemizin ifade etkinliğini iyileştirdiğimize göre, projeyi dünyayla paylaşmamıza yardımcı olacak `cargo`'nun bazı daha fazla özelliğine bakalım.