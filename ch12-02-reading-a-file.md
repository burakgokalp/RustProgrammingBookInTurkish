## Dosya Okuma (Reading a File)

Şimdi `file_path` argümanında belirtilen dosyayı okuma yeteneği ekleyeceğiz. Önce, onu test etmek için bir örnek dosyaya ihtiyaçımız var: Birden fazla satırda bazı tekrarlanan kelimelerle küçük miktar metni içeren bir dosya kullanacağız. Kod Listesi 12-3 Emily Dickinson şiiri var ki iyi çalışacak! Projenizin kök seviyesinde _poem.txt_ adında bir dosya oluşturun ve şiir "I'm Nobody! Who are you?" girin.

<Listing number="12-3" file-name="poem.txt" caption="Emily Dickinson'tan bir şiir iyi bir test vakası yapar">

```text
{{#include ../listings/ch12-an-io-project/listing-12-03/poem.txt}}
```

</Listing>

Metni yerinde, _src/main.rs_'i düzenleyin ve Kod Listesi 12-4'de gösterildiği gibi dosyayı okumak için kod ekleyin.

<Listing number="12-4" file-name="src/main.rs" caption="İkinci argüman tarafından belirtilen dosyanın içeriklerini okuma">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/src/main.rs:here}}
```

</Listing>

Önce, `use` bildirimi ile standart kütüphanenin ilgili kısmını getiriyoruz: Dosyalarla uğraşmak için `std::fs` gerekiyor.

`main` içinde, yeni ifade `fs::read_to_string` `file_path` alır, o dosyayı açar ve dosyanın içeriklerini içeren `std::io::Result<String>` tipinde bir değer döndürür.

Sonra, dosya okunduktan sonra `contents`'in değerini yazdıran geçici bir `println!` ifadesi tekrar ekliyoruz ki şu ana kadar programın beklendiğimiz gibi çalıştığını kontrol edelim.

Bu kodu, ilk komut satırı argüman olarak herhangi bir dize ile (çünkü arama kısmını henüz uygulamamadık) ve _poem.txt_ dosyası ikinci argüman olarak çalıştıralım:

```console
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/output.txt}}
```

Harika! Kod dosyayı okudu ve sonra dosyanın içeriklerini yazdırdı. Ancak kodun birkaç kusuru var. Şu anda, `main` fonksiyonu birden fazla sorumluluğa sahip: Genel olarak, her fonksiyon sadece bir fikirden sorumlu olduğunda daha net ve bakımı yapmak daha kolaydır. Diğer sorun, yapabileceğimiz kadar iyi hataları elelemiyor olmamamızdır. Program hala küçük, bu yüzden bu kusurlar büyük bir sorun değil, ancak program büyüdükçe, onları temizce düzeltmek daha zor olacak. Bir program geliştirilirken yerefactor etmeye erken başlamak iyi bir uygulamadır, çünkü küçük miktar kodunu yerefactor etmek çok daha kolaydır. Bunu sonraki yapacağız.