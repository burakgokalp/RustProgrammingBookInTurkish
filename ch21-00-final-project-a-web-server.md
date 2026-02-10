# Final Projesi: Çok İş Parçacıklı (Multithreaded) Bir Web Sunucusu Oluşturma

Uzun bir yolculuktu ancak kitabın sonuna ulaştık. Bu bölümde, son
bölümlerde ele aldığımız bazı konseptleri göstermek ve daha önceki dersleri
özetlemek için birlikte bir proje daha yapacağız.

Final projemiz için, bir web tarayıcısında Şekil 21-1 gibi görünen "Hello!" diyen bir
web sunucusu yapacağız.

İşte web sunucusu oluşturma planımız:

1. TCP ve HTTP hakkında biraz öğrenin.
2. Bir sokette (socket) TCP bağlantılarını dinleyin.
3. Küçük bir sayıda HTTP isteğini ayrıştırın (parse).
4. Uygun bir HTTP yanıtı oluşturun.
5. Sunucumuzun iş hacmini (throughput) bir iş parçacığı havuzu (thread pool) ile geliştirin.

<img alt="Screenshot of a web browser visiting address 127.0.0.1:8080 displaying a webpage with text content “Hello! Hi from Rust”" src="img/trpl21-01.png" class="center" style="width: 50%;" />

<span class="caption">Figure 21-1: Our final shared project</span>

Başlamadan önce, iki detay belirtmeliyiz. İlk olarak, kullanacağımız yöntem Rust ile
bir web sunucusu oluşturmanın en iyi yolu olmayacak. Topluluk üyeleri,
[crates.io](https://crates.io/) adresinde, bizim yapacağımızdan daha tam web sunucusu ve iş
parçacığı havuzu (thread pool) uygulamaları sağlayan üretim hazırı (production-ready) crate'ler
yayınladı. Ancak, bu bölümdeki niyetimiz sizin öğrenmenize yardımcı olmak, kolay yolu seçmek değil.
Rust bir sistem programlama dili olduğu için, çalışmak istediğimiz soyutlama seviyesini seçebiliriz ve
diğer dillerde mümkün veya pratik olanın daha altına gidebiliriz.

İkinci olarak, burada async ve await kullanmayacağız. Bir iş parçacığı havuzu (thread pool) oluşturmak,
bir async çalışma zamanı (runtime) oluşturmaya eklemeden, tek başına büyük bir meydan okuma! Ancak,
bu bölümde göreceğimiz aynı sorunların bazılarına async ve await'in uygulanabilir olabileceğini
not edeceğiz. Sonuç olarak, Bölüm 17'de belirttiğimiz gibi, birçok async çalışma zamanı
(runtime), işlerini yönetmek için iş parçacığı havuzları kullanır.

Bu yüzden, gelecekte kullanabileceğiniz crate'lerin arkasındaki genel fikirleri ve teknikleri
öğrenebilesiniz diye temel HTTP sunucusunu ve iş parçacığı havuzunu (thread pool) manuel olarak
yazacağız.