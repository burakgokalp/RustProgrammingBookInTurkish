## Yorumlar

Tüm programcılar kodlarının kolayca anlaşılmasını yapmaya çalışır, ancak bazen ekstra açıklama gerekir. Bu durumlarda, programcılar kaynak kodlarına derleyicinin yok sayacağı ancak kaynak kodu okuyan insanlar için yararlı bulabilecekleri _yorumlar_ (comments) bırakır.

İşte basit bir yorum:

```rust
// hello, world
```

Rust'ta, idiyomatik yorum stili bir yorumu iki eğik çizgi ile başlatır ve yorum satırın sonuna kadar devam eder. Tek satırdan daha uzun yorumlar için, her satıra `//` eklemeniz gerekir, şöyle:

```rust
// Burada karmaşık bir şey yapıyoruz, bunun için
// birden fazla satır yoruma ihtiyacımız var! Of! Umarım bu yorum
// ne olduğunu açıklayacak.
```

Yorumlar kod içeren satırların sonuna da yerleştirilebilir:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-24-comments-end-of-line/src/main.rs}}
```

Ancak yorumları bu formatta, açıkladığı kodun üzerindeki ayrı bir satırda kullanıldığını daha sık göreceksiniz:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-25-comments-above-line/src/main.rs}}
```

Rust ayrıca başka bir yorum türüne, dokümantasyon yorumlarına sahiptir, bunları Bölüm 14'teki ["Bir Crate'ı Crates.io'ya Yayınlama"][publishing]<!-- ignore --> bölümünde tartışacağız.

[publishing]: ch14-02-publishing-to-crates-io.html