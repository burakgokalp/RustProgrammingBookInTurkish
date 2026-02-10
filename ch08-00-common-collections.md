# Yaygın Koleksiyonlar (Common Collections)

Rust'ın standart kütüphanesi _koleksiyonlar_ (collections) adında çok kullanışlı çok sayıda veri yapısını içerir. Çoğu diğer veri tipleri belirli bir değeri temsil eder ancak koleksiyonlar çoklu değeler içerebilir. Yerleşik (built-in) dizi ve demet (tuple) tiplerinin aksine, bu koleksiyonların işaret ettiği veri yığıında depolanır ki bu veri miktarının derleme zamanında bilinmesine ve program çalışırken büyümesine veya küçülmesine izin verir. Her koleksiyon türü farklı yeteneklere ve maliyetlere sahiptir ve mevcut durumunuz için uygun birini seçmek zamanla geliştireceğiniz bir beceridir. Bu bölümde, Rust programlarında çok sık kullanılan üç koleksiyonu konuşacağız:

- Bir _vektör (vector)_, sizin çoklu değeleri birbirinin yanına saklamanızı sağlar.
- Bir _dize (string)_ karakterler koleksiyonudur. Daha önce `String` tipini bahsetmiştik ancak bu bölümde onu derinlikte konuşacağız.
- Bir _hash map_, sizin bir değeri belirli bir anahtarla ilişkilendirmenizi sağlar. Bu daha genel veri yapısı olan bir _harita_ (map)'nin belirli bir uygulamasıdır.

Standart kütüphane tarafından sağlanan diğer koleksiyon türleri hakkında öğrenmek için, [belgelerine][collections] bakın.

Vektörleri, dizeleri ve hash haritaları nasıl oluşturacağımızı ve güncelleyeceğimizi konuşacağız, ayrıca her birini özel yapan şeyleri.

[collections]: ../std/collections/index.html