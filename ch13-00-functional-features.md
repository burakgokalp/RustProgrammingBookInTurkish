# Fonksiyonel Dil Özellikleri: İteratörler ve Kapanışlar (Functional Language Features: Iterators and Closures)

Rust'in tasarımı birçok mevcut dil ve teknitten ilham almıştır ve önemli bir etkisi _fonksiyonel programlama_dır. Fonksiyonel bir tarzda programlama genellikle fonksiyonları değer olarak kullanmayı içerir: onları argümanlarda geçirmek, onları başka fonksiyonlardan döndürmek, onları daha sonraki yürütme için değişkenlere atamak ve bunun gibi.

Bu bölümde, fonksiyonel programlamanın ne olduğu veya olmadığı tartışmak yerine, onları genellikle fonksiyonel denilen birçok dillerdeki özelliklere benzer bazı Rust özelliklerini tartışacağız.

Daha spesifik olarak şunları kaplayacağız:

- _Kapanışlar_, bir değişkende saklayabileceğiniz fonksiyon benzeri bir yapı
- _İteratörler_, bir seriyi elemanları işlemenin bir yolu
- Bölüm 12'deki I/O projesini iyileştirmek için kapanışlar ve iteratörleri nasıl kullanılır
- Kapanışların ve iteratörlerin performansı (spoiler uyarısı: Düşünceğinizden daha hızlılar!)

Şimdiye kadar bazı diğer Rust özelliklerini, örneğin desen eşleştirme ve numaralandırmaları da kapsadık ki bunlar da fonksiyonel tarzdan etkilenmiştir. Kapanışları ve iteratörleri ustalaştırmak hızlı, idiyomatik Rust kodu yazmanın önemli bir parçası olduğundan, tüm bu bölümü onlara ayıracağız.