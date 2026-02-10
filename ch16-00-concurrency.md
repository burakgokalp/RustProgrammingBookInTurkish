# Cesursuz Eşzamanlılık (Fearless Concurrency)

Eşzamanlı programlamayı güvenli ve verimli bir şekilde ele almak, Rust'un ana hedeflerinden biridir. Bir programın farklı bölümlerinin bağımsız olarak yürütüldüğü _eşzamanlı programlama_ ve bir programın farklı bölümlerinin aynı anda yürütüldüğü _paralel programlama_, daha fazla bilgisayarın çok işlemcisinden yararlandıkça giderek daha önemli hale gelmektedir. Tarihsel olarak, bu bağlamlarda programlama zor ve hataya açıktı. Rust bunu değiştirmeyi umuyor.

Başlangıçta, Rust ekibi bellek güvenliğini sağlamanın ve eşzamanlılık sorunlarını önlemenin farklı yöntemlerle çözülmesi gereken iki ayrı zorluk olduğunu düşündü. Zamanla, ekibi sahipliğin ve tür sisteminin bellek güvenliğini _ve_ eşzamanlılık sorunlarını yönetmeye yardımcı olmak için güçlü bir araç seti olduğunu keşfetti! Sahiplik ve tür kontrolünden yararlanarak, birçok eşzamanlılık hatası Rust'ta çalışma zamanı hatası yerine derleme zamanı hatasıdır. Bu nedenle, çalışma zamanı bir eşzamanlılık hatasının hangi tam koşullar altında gerçekleştiğini yeniden üretmek için çok fazla zaman harcamak yerine, yanlış kod derlenmeyi reddedecek ve sorunu açıklayan bir hata sunacak. Sonuç olarak, kodunuzu üzerinde çalışırken düzeltebilirsiniz, muhtemelen üretim ortamına gönderildikten sonra değil. Bu Rust yönünü _cesursuz eşzamanlılık_ olarak adlandırdık. Cesursuz eşzamanlılık, ince hatalardan arınmış kodu ve yeni hatalar getirmeden kolayca yeniden düzenlemeyi sağlar.

> Not: Basitlik uğruna, sorulardan çoğuna daha doğrucu olma için _eşzamanlı ve/veya paralel_ demek yerine _eşzamanlı_ olarak referans vereceğiz. Bu bölüm için, biz _eşzamanlı_ kullandığımızda zihinsel olarak _eşzamanlı ve/veya paralel_ ile değiştirin. Bir sonraki bölümde, ayrımın daha önemli olduğu yerde, daha spesifik olacağız.

Çok sayıda dil, sundukları çözümlerde eşzamanlı sorunları ele almada dogmatiktir. Örneğin, Erlang mesaj geçişli eşzamanlılık için zarif işlevselliğe sahiptir ancak iş parçacıkları arasında durumu paylaşmanın sadece tıknaz yolları vardır. Olası çözümlerin bir alt kümesini desteklemek, üst düzey bir dil için mantıklı bir stratejidir çünkü üst düzey bir dil soyutlamaları kazanmak için bazı kontrolü vermek fayda sağlar. Ancak, alt düzey dillerin verilen herhangi bir durumda en iyi performansla çözüm sunması ve donanım üzerinde daha az soyutlaması beklenir. Bu nedenle, Rust, durumunuzu ve gereksinimlerinize uygun herhangi bir şekilde sorunları modellemek için çeşitli araçlar sunar.

Bu bölümde ele alacağımız konular şunlardır:

- Aynı anda birden fazla kod parçasını çalıştırmak için iş parçacıkları nasıl oluşturulur
- _Mesaj geçişli_ eşzamanlılık, kanalların iş parçacıkları arasında mesajlar gönderdiği
- _Paylaşılan durumlu_ eşzamanlılık, birden fazla iş parçacığının verilerin bir kısmına eriştiği
- `Sync` ve `Send` nitelikleri, Rust'un eşzamanlılık garantilerini kullanıcı tanımlı türlerine ve standart kütüphane tarafından sağlanan türlere uzatır