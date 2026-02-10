# Ortak Programlama Kavramları

Bu bölüm, hemen hemen her programlama dilinde görünen kavramları ve bunların Rust'ta nasıl çalıştığını kapsar. Birçok programlama dili çekirdeğinde çok ortak noktaya sahiptir. Bu bölümde sunulan kavramların hiçbiri Rust'a özgü değildir, ancak bunları Rust bağlamında tartışacağız ve bunları kullanma etrafındaki sözleşmeleri açıklayacağız.

Özellikle, değişkenler, temel tipler, fonksiyonlar, yorumlar ve kontrol akışı hakkında öğreneceksiniz. Bu temeller her Rust programında olacak ve bunları erken öğrenmek size güçlü bir temel başlatacak.

> #### Anahtar Kelimeler
>
> Rust dili diğer dillerde olduğu gibi, sadece dil tarafından kullanım için ayrılmış bir _anahtar kelime_ kümesine sahiptir. Aklınızda bulundurun ki bu kelimeleri değişken veya fonksiyon adları olarak kullanamazsınız. Çoğu anahtar kelimenin özel anlamları vardır ve bunları Rust programlarınızda çeşitli görevleri yapmak için kullanacaksınız; birkaçının şu anda bunlarla ilgili işlevselliği yoktur ancak gelecekte Rust'a eklenebilir işlevsellik için ayrılmıştır. [Ek A][appendix_a]<!-- ignore -->'da anahtar kelimelerin listesini bulabilirsiniz.

[appendix_a]: appendix-01-keywords.md