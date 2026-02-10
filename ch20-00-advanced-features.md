# Gelişmiş Özellikler (Advanced Features)

Şimdiye kadar, Rust programlama dilinin en yaygın kullanılan parçlarını öğrendiniz. Bölüm 21'de
bir başka proje yapmadan önce, arada sırada karşılaşabileceğiniz ancak her gün kullanmayabileceğiniz
dilin bazı yanlarına bakacağız. Herhangi bir bilmediğiniz şeyler karşılaştığınızda bu bölümü bir
referans olarak kullanabilirsiniz. Burada ele alınan özellikler çok belirli durumlar için kullanışlıdır.
Onlara sık sık ulaşmasanız bile, Rust'un sunabileceği tüm özelliklere bir kavraya sahip olduğunuzdan
emin olmak istiyoruz.

Bu bölümde şunları ele alacağız:

- Güvenli olmayan Rust (Unsafe Rust): Rust'un bazı garantilerinden nasıl opt-out edeceğinizi ve
  bu garantileri manuel olarak sürdürme sorumluluğunu nasıl alacağınız
- Gelişmiş trait'ler: İlişkili türler, varsayılan tür parametreleri, tam nitelikli sözdizimi,
  supertrait'ler ve trait'lerle ilişkili newtype deseni
- Gelişmiş türler: newtype deseni hakkında daha fazlası, tür takma adları (type aliases), never türü
  ve dinamik boyutlandırılmış türler
- Gelişmiş fonksiyonlar ve kapamalar: Fonksiyon işaretçileri ve kapamaları döndürme
- Makrolar: Derleme zamanında daha fazla kod tanımlayan kod tanımlama yolları

Herkes için bir şeyi olan Rust özelliklerinin bir panoplysi! Hadi dalalım!