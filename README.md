# Rust Programlama Kitabı - Türkçe Çeviri

## 📚 Kitabın HTML Sürümünü İndirin (https://doc.rust-lang.org/book/)

Kitabın oluşturulmuş HTML sürümünü doğrudan indirmek için repository'de bulunan **`RustProgrammingBookAsHtml.7z`** dosyasını indirebilirsiniz. Bu dosyayı indirdikten sonra çıkararak tarayıcınızda kitabı görüntüleyebilirsiniz.

---

Bu repository, [Rust Programlama Kitabı](https://github.com/rust-lang/book)'nın `src` dalı altındaki metinlerin **Türkçe çevirisini** içermektedir.

## Kaynak

Bu çeviri, resmi Rust Programming Language kitabının şu sürümüne dayanmaktadır:
- Orijinal Repository: https://github.com/rust-lang/book
- Kaynak Dizin: https://github.com/rust-lang/book/tree/main/src
- Çeviriye Konu Olan Kitabın İngilizce Sürümü: https://doc.rust-lang.org/book/

## İçerik

Kitap, Rust programlama dilini öğrenmek için kapsamlı bir kaynak sunar. Türkçe çevirisi şu bölümleri içerir:

- **Giriş ve Başlangıç**: Kurulum, Hello World ve Cargo kullanımı
- **Temel Programlama Kavramları**: Değişkenler, veri türleri, fonksiyonlar, yorumlar ve kontrol akışı
- **Ownership (Sahiplik)**: Rust'ın en önemli özelliklerinden biri
- **Structs ve Enums**: Veri yapıları ve desen eşleştirme
- **Modül Sistemi**: Paketler, crate'ler ve modüller
- **Koleksiyonlar**: Vektörler, stringler ve hash map'ler
- **Hata Yönetimi**: Panic ve Result tipi
- **Generics, Traits ve Lifetime**: Gelişmiş tip sistemi
- **Test Yazma**: Birim testler ve entegrasyon testleri
- **IO Projesi**: Komut satırı programı geliştirme
- **Fonksiyonel Özellikler**: Closures ve iterator'lar
- **Cargo ve Crates.io**: Paket yönetimi ve yayınlama
- **Akıllı İşaretçiler (Smart Pointers)**: Box, Rc, RefCell ve daha fazlası
- **Eşzamanlılık (Concurrency)**: Thread'ler, mesaj geçişi ve paylaşılan durum
- **Async Await**: Asenkron programlama
- **Nesne Yönelimli Programlama**: Trait nesneleri ve tasarım desenleri
- **Desenler (Patterns)**: Pattern matching ve refutability
- **Gelişmiş Özellikler**: Unsafe Rust, gelişmiş traits ve makrolar
- **Final Proje**: Web sunucusu geliştirme
- **Ekler**: Anahtar kelimeler, operatörler, türetilebilir traits ve daha fazlası

## Kullanım

Bu çeviri, Rust'ı öğrenmek isteyen Türkçe konuşan kullanıcılar için hazırlanmıştır. Orijinal İngilizce metinlerle karşılaştırarak daha iyi anlayabilirsiniz.

## Kitabı Oluşturma

### Kitap Haline Çevirme

Orijinal Rust Programming Book repository'sindeki markdown dosyalarını kitap formatına dönüştürmek için **mdBook** aracı kullanılır.

### Gerekli Araçlar

1. **mdBook**: Rust ile yazılmış bir kitap oluştirma aracıdır. Yüklemek için:
   ```bash
   cargo install mdbook
   ```

2. **Git**: Repository'yi klonlamak için gerekli
3. **Rust**: mdBook'u yüklemek için Rust toolchain'inin yüklü olması gerekir

### Türkçe Çeviri Kitabını Oluşturma Adımları

1. **Orijinal Repository'yi Klonlayın**:
   ```bash
   git clone https://github.com/rust-lang/book.git
   cd book
   ```

2. **Çeviri Repository'sini Klonlayın veya Kullanın**:
   ```bash
   git clone https://github.com/burakgokalp/RustProgrammingBookInTurkish.git
   cd RustProgrammingBookInTurkish
   ```

3. **Kitabı Oluşturun**:
   Türkçe çeviri repository'sinin kök dizininde (src klasörünün üstü) şu komutu çalıştırın:
   ```bash
   mdbook build
   ```

   Bu komut `book/` dizini oluşturacak ve tüm markdown dosyalarını HTML formatına dönüştürecektir.

4. **Kitabı Görüntüleyin**:
   ```bash
   mdbook serve
   ```
   
   Ardından tarayıcınızda `http://localhost:3000` adresine giderek kitabı görüntüleyebilirsiniz.

### Çeviri Repository'sini Kopyalama ve Yapıştırma

Eğer kendi çevirinizi yapacaksanız veya çeviriyi güncelleyecekseniz:

1. Türkçe çeviri repository'sini klonlayın:
   ```bash
   git clone https://github.com/burakgokalp/RustProgrammingBookInTurkish.git
   cd RustProgrammingBookInTurkish
   ```

2. Çevirilen dosyalar `src/` dizininde bulunur. Orijinal repository'den kopyaladığınız dosyaları buraya yapıştırabilirsiniz:
   ```
   RustProgrammingBookInTurkish/
   ├── src/                    # Çeviri dosyalarının bulunduğu dizin
   │   ├── ch00-00-introduction.md
   │   ├── ch01-00-getting-started.md
   │   ├── ...
   ├── book/                   # mdbook build komutundan sonra oluşur
   ├── book.toml              # Kitap yapılandırma dosyası
   └── README.md              # Bu dosya
   ```

3. Dosyaları düzenledikten sonra `mdbook build` komutunu çalıştırarak kitabı güncelleyebilirsiniz.

## Katkıda Bulunma

Çeviriyi geliştirmek veya düzeltmek için katkıda bulunabilirsiniz. Lütfen pull request göndererek veya issue açarak yardımcı olun.

## Lisans

Bu çeviri, orijinal Rust Programlama Kitabı ile aynı lisans altındadır.

---

**Not**: Bu çeviri, Rust topluluğuna katkıda bulunmak amacıyla hazırlanmıştır. Orijinal sürüm ile karşılaştırarak en güncel bilgiye ulaşabilirsiniz.