# Bir G/Ç Projesi: Komut Satırı Programı Oluşturma (An I/O Project: Building a Command Line Program)

Bu bölüm, şu ana kadar öğrendiğiniz birçok becerinin tekrarı ve birkaç daha fazla standart kütüphane özelliğinin keşfidir. Şu ana sahip olduğunuz Rust kavramlarını pratik etmek için dosya ve komut satırı girdi/çıktısı ile etkileşen bir komut satırı aracı inşa edeceğiz.

Rust'ın hızı, güvenliği, tek ikili çıktısı ve çapraz-platform desteği, komut satırı araçları oluşturmak için ideal bir dil yapar, bu yüzden projemiz için, klasik komut satırı aracı `grep`'in (**g**lobally search a **r**egular **e**xpression and **p**rint) kendi versiyonunu yapacağız. En basit kullanım durumunda, `grep` belirli bir dosyada belirli bir dize için arar. Bunu yapmak için, `grep` argüman olarak bir dosya yolu ve bir dize alır. Sonra, dosyayı okur, dize argümanını içeren satırları bulur ve o satırları yazdırır.

Yol boyunca, komut satırı aracımızın birçok başka komut satırı aracının kullandığı terminal özelliklerini nasıl kullanacağını göstereceğiz. Aracın davranışını ayarlaması için kullanıcının ortam değişkeninin değerini okuyacağız. Ayrıca, örneğin kullanıcının başarılı çıktıyı bir dosyaya yönlendirirken ekranda hata mesajlarını hala görebilmesi için, standart çıktı (`stdout`) yerine standart hata konsol akımına (`stderr`) hata mesajlarını yazdıracağız.

Bir Rust topluluğu üyesi, Andrew Gallant, `ripgrep` adında `grep`'in tam özellikli, çok hızlı bir versiyonunu zaten oluşturmuştur. Kıyaslamayla, bizim versiyonumuz oldukça basit olacak, ancak bu bölüm `ripgrep` gibi gerçek dünya bir projeyi anlamak için ihtiyaç duyacağınız bazı arkaplan bilgisini size verecektir.

Bizim `grep` projemiz şu ana kadar öğrendiğiniz birçok kavramı birleştirecektir:

- Kod organizasyonu ([Bölüm 7][ch7]<!-- ignore -->)
- Vektör ve dizeler kullanma ([Bölüm 8][ch8]<!-- ignore -->)
- Hatalar ele alma ([Bölüm 9][ch9]<!-- ignore -->)
- Trait'leri ve ömürleri (lifetimes) uygun yerlerde kullanma ([Bölüm 10][ch10]<!-- ignore -->)
- Test yazma ([Bölüm 11][ch11]<!-- ignore -->)

Ayrıca kapanışları (closures), iteratörleri (iterators) ve trait nesnelerini (trait objects) kısaca tanıştıracağız ki bunları [Bölüm 13][ch13]<!-- ignore --> ve [Bölüm 18][ch18]<!-- ignore --> ayrıntılı olarak kaplayacaktır.

[ch7]: ch07-00-managing-growing-projects-with-packages-crates-and-modules.html
[ch8]: ch08-00-common-collections.html
[ch9]: ch09-00-error-handling.html
[ch10]: ch10-00-generics.html
[ch11]: ch11-00-testing.html
[ch13]: ch13-00-functional-features.html
[ch18]: ch18-00-oop.html