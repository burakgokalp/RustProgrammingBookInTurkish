# Desenler ve Eşleşme (Patterns and Matching)

Desenler (patterns), Rust'ta türlerin yapısıyla eşleşmek için karmaşık ve basit
olan özel bir sözdizimidir. `match` ifadeleri ve başka yapılarla birlikte desenler kullanmak,
programınızın kontrol akışı üzerinde daha fazla kontrol sağlar. Bir desen aşağıdaki
bileşenlerden bazı kombinasyonundan oluşur:

- Sabit değerler (literals)
- Yapısı bozulmuş diziler, enum'lar, struct'lar veya demetler (tuples)
- Değişkenler
- Joker karakterler (wildcards)
- Yer tutucular (placeholders)

Bazı örnek desenler `x`, `(a, 3)`, ve `Some(Color::Red)` gibi. Desenlerin geçerli olduğu
bağlamlarda bu bileşenler verinin şeklini açıklar. Programımız sonra belirli bir kod parçasını
çalıştırmaya devam etmek için doğru veri şeklinde olup olmadığını belirlemek için desenlere karşı
değerleri eşleştirir.

Bir desen kullanmak için, onu bir değerle karşılaştırırız. Eğer desen değere eşleşirse,
kodemizde değer parçalarını kullanırız. Bölüm 6'daki, desenleri kullanan `match` ifadelerini
hatırlayın, örneğin para sıralama makinesi örneği gibi. Eğer değer desenin şekline uyarsa,
adlandırılmış parçaları kullanabiliriz. Uymazsa, desenle ilişkili kod çalışmaz.

Bu bölüm desenlerle ilgili her şeye referanstır. Desenlerin kullanılabileceği geçerli yerleri,
yanılmaz (refutable) ve yanılmaz (irrefutable) desenler arasındaki farkı ve görebileceğiniz farklı
türde desen sözdizimini ele alacağız. Bölümün sonuna doğru, desenleri birçok kavramı net bir
şekilde ifade etmek için nasıl kullanacağınızı bileceksiniz.