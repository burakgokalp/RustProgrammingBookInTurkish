## Referans Döngüleri Bellek Sızıntısı Yapabilir (Reference Cycles Can Leak Memory)

Rust'un bellek güvenliği garantileri, kazara bellek oluşturmayı zor ama imkansız hale getirir ki bu bellek hiç temizlenmez (bellek sızıntısı olarak bilinir). Bellek sızıntılarını tamamen önlemek, Rust'un garantilerinden biri değil, bu demektir ki bellek sızıntıları Rust'ta bellek güvenlidir. Bellek sızıntılarını `Rc<T>` ve `RefCell<T>` kullanarak Rust'un nasıl izin verdiğini görebiliriz: Öğelerin birbirlerine işaret ettiği döngüler oluşturmak mümkündür. Bu bellek sızıntıları yaratır çünkü döngüdeki her öğenin referans sayısı asla 0'a ulaşamayacak ve değerler asla bırakılmayacak.

### Bir Referans Döngüsü Oluşturma (Creating a Reference Cycle)

Bir referans döngüsünün nasıl oluşabileceğini ve nasıl önleneceğini inceleyelim, Kod Listesi 15-25'teki `List` numaralandırmasını ve `tail` yönteminden başlayarak.

<Listing number="15-25" file-name="src/main.rs" caption="`Cons` varyantının işaret ettiği şeyi değiştirebilmek için bir `RefCell<T>` tutan bir cons liste tanımı">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-25/src/main.rs:here}}
```

</Listing>

Kod Listesi 15-5'teki `List` tanımının başka bir varyasyonunu kullanıyoruz. `Cons` varyantındaki ikinci öğe artık `RefCell<Rc<List>>`, bu demektir ki Kod Listesi 15-24'teki gibi `i32` değerini değiştirme yeteneği yerine, bir `Cons` varyantının işaret ettiği `List` değerini değiştirmek istiyoruz.

Ayrıca `Cons` varyantına sahipsek ikinci öğeyi erişmeyi kolaylaştırmak için bir `tail` yöntemi ekliyoruz.

Kod Listesi 15-26'da, Kod Listesi 15-25'teki tanımları kullanan bir `main` fonksiyonu ekliyoruz. Bu kod `a` içinde bir list ve `b` içindeki listenin işaret ettiği listeyi oluşturur. Sonra, `a` içindeki listeyi `b`'ye işaret edecek şekilde değiştirir, bir referans döngüsü yaratır. Bu sürecin çeşitli noktalarda referans sayılarının ne olduğunu göstermek için `println!` bildirimleri vardır.

<Listing number="15-26" file-name="src/main.rs" caption="Birbirine işaret eden iki `List` değeri referans döngüsü oluşturma">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-26/src/main.rs:here}}
```

</Listing>

`a` değişkeninde `5, Nil` listesini tutan bir `Rc<List>` örneği oluştururuz. Sonra `10` değerini içerir ve `a` içindeki listeyi işaret eden bir `List` değerini tutan başka bir `Rc<List>` örneğini `b` değişkeninde oluştururuz.

`a`'yı `Nil` yerine `b`'ye işaret edecek şekilde değiştiririz ki bir döngü yaratır. Bunu `a` içindeki `RefCell<Rc<List>>`'e referans almak için `tail` yöntemini kullanarak yapıyoruz ki onu `link` değişkeninde koyaruz. Sonra, `RefCell<Rc<List>>` üzerinde `borrow_mut` yöntemini kullanarak içindeki değeri, `Nil` değeri tutan bir `Rc<List>`'den `b` içindeki `Rc<List>`'e değiştiririz.

Bu kodu çalıştırdığımızda, son `println!`'yi bir an için yorumlu tutarak, şu çıktıyı alacağız:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-26/output.txt}}
```

Hem `a` hem de `b` içindeki `Rc<List>` örneklerinin referans sayısı, `a` içindeki listeyi `b`'ye değiştirdiğimizde 2 olur. `main`'in sonunda, Rust `b` değişkenini bırakır ki bu `b`'nin `Rc<List>` örneğinin referans sayısını 2'den 1'e azaltır. `Rc<List>`'in öbek üzerinde sahip olduğu bellek bu noktada bırakılmayacak çünkü referans sayısı 1, 0 değil. Sonra, Rust `a`'yı bırakır ki bu `a`'nın `Rc<List>` örneğinin referans sayısını da 2'den 1'e azaltır. Bu örneğinin belleği de bırakılamaz çünkü diğer `Rc<List>` örneği hala işaret ediyor. Listeye ayrılan bellek sonsuza kadar kullanılmamayacak kalacak. Bu referans döngüsünü görselleştirmek için Şekil 15-4'teki diyagramı oluşturduk.

<img alt="Tamsayı 5 içeren bir dikdörtgeni işaret eden 'a' etiketli bir dikdörtgen. Tamsayı 10 içeren bir dikdörtgeni işaret eden 'b' etiketli bir dikdörtgen. 5 içeren dikdörtgen, 10 içeren dikdörtgeni işaret eder ve 10 içeren dikdörtgen, 5 içeren dikdörtgeni geri işaret eder, bir döngü yaratır." src="img/trpl15-04.svg" class="center" />

<span class="caption">Şekil 15-4: Birbirine işaret eden listeler `a` ve `b`'nin referans döngüsü</span>

Son `println!`'nin yorumunu kaldırıp programı çalıştırsanız, Rust `a`'nın `b`'yi işaret ettiğini `b`'nin `a`'yı işaret ettiğini ve böyle devam ederek yığın taşana kadar bu döngüyü yazmaya çalışacak.

Bu örnekteki gerçek dünya programına kıyaslandığında bir referans döngüsü oluşturmanın sonuçları çok ciddi değildir: Referans döngüsünü oluşturduğumuzda hemen program biter. Ancak, daha karmaşık bir program döngüde çok bellek ayırır ve uzun süre tutarsa, program ihtiyacından daha fazla bellek kullanabilir ve sistemi boğabilir, kullanılabilir belleğin dışına çıkmasına neden olabilir.

Referans döngülerini oluşturmak kolayca yapılmaz ama imkansız da değildir. Eğer `Rc<T>` değerlerini tutan `RefCell<T>` değerlerinize veya iç değişkenlik ve referans sayma özelliklerine sahip benzeri iç iç içermeleriniz varsa, döngüleri oluşturmadığınızdan emin olmalısınız; bunları yakalamak için Rust'a güvenilemezsiniz. Bir referans döngüsü oluşturmak, programınızdaki bir mantık hatası olur ve bunu en aza indirmek için otomatik testler, kod incelemeleri ve diğer yazılım geliştirme uygulamalarını kullanmalısınız.

Referans döngülerinden kaçınmak için başka bir çözüm, veri yapılarınızdır yeniden düzenlemektir ki bazı referanslar sahipliği ifade eder ve bazıları ifade etmez. Sonuç olarak, bazı sahiplik ilişkilerinden ve bazı sahiplik olmayan ilişkilerden oluşan döngüler yaratabilirsiniz ve sadece sahiplik ilişkileri, bir değerin bırakılıp bırakılmayacağını etkiler. Kod Listesi 15-25'te her zaman `Cons` varyantlarının listelerini sahiplenmesini istiyoruz, bu yüzden veri yapısını yeniden düzenlemek imkansız değil.

Çocuk düğümlerini ve ebeveyn düğümlerinden oluşan ağaçları kullanan bir örneğe bakalım ki sahiplik olmayan ilişkilerin referans döngülerini önlemek için ne zaman uygun bir yol olduğunu görebiliriz.

<!-- Old headings. Do not remove or links may break. -->

<a id="preventing-reference-cycles-turning-an-rct-into-a-weakt"></a>

### `Weak<T>` Kullanarak Referans Döngülerini Önleme (Preventing Reference Cycles Using `Weak<T>`)

Şu ana kadar, `Rc::clone` çağırarak bir `Rc<T>` örneğinin `strong_count`'ını nasıl artırdığımızı gösterdik ve bir `Rc<T>` örneği sadece `strong_count` 0 ise temizlenir. Ayrıca `Rc<T>` örneği içindeki bir değere `Rc::downgrade` çağırarak ve `Rc<T>`'e bir referans geçirerek bir zayıf referans oluşturabilirsiniz. *Güçlü referanslar*, bir `Rc<T>` örneğinin sahipliğini nasıl paylaşabileceğinizdir. *Zayıf referanslar* bir sahiplik ilişkisini ifade etmezler ve onların sayısı bir `Rc<T>` örneğinin ne zaman temizleneceğini etkilemez. Bir referans döngüsü yaratmazlar çünkü bazı zayıf referanslar içeren değerlerin güçlü referans sayısı 0 olduğunda kesilecektir.

`Rc::downgrade` çağırdığınızda, `Weak<T>` türünde bir akıllı gösterici alırsınız. `Rc<T>` örneğindeki `strong_count`'ı 1 artırmak yerine, `Rc::downgrade` çağırarak `weak_count`'ı 1 artırırsınız. `Rc<T>` türü, `strong_count`'e benzer olarak kaç `Weak<T>` referansının olduğunu takip etmek için `weak_count`'i kullanır. Fark şudur ki `Rc<T>` örneğinin temizlenebilmesi için `weak_count`'ın 0 olması gerekmez.

`Weak<T>` işaret ettiği değerin bırakılmış olabileceği için, `Weak<T>`'in işaret ettiği değeriyle bir şey yapmak için değerin hala var olduğundan emin olmalısınız. Bunu `Weak<T>` örneği üzerinde `upgrade` yöntemini çağırarak yaparsınız ki bu `Option<Rc<T>>` döndürür. `Rc<T>` değeri henüz bırakılmadıysa `Some` sonucunu ve `Rc<T>` değeri bırakılmışsa `None` sonucunu alırsınız. `upgrade` bir `Option<Rc<T>>` döndürdüğü için, Rust'in `Some` durumunu ve `None` durumunu işlediğinden emin olması ve geçersiz bir gösterici olmayacağı anlamına gelir.

Bir örnek olarak, sadece sonraki öğe hakkında bilen bir liste yerine, çocuk öğelerini _ve_ ebeveynlerini bilen bir ağaç oluşturacağız.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-a-tree-data-structure-a-node-with-child-nodes"></a>

#### Bir Ağaç Veri Yapısı Oluşturma (Creating a Tree Data Structure)

Başlamak için, çocuk düğümlerini bildiği düğümlerine sahip olan ağaç oluşturacağız. `Node` adında bir struct oluşturacağız ki kendi `i32` değerinin yanı sıra çocuk `Node` değerlerine referanslar içerir:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:here}}
```

Bir `Node`'un çocuklarını sahiplenmesini istiyoruz ve o sahipliğini değişkenlerle paylaşmak istiyoruz ki böylece ağaçtaki her `Node`'e doğrudan erişebiliriz. Bunu yapmak için, `Rc<Node>` türünde değerleri olan `Vec<T>` öğelerini tanımlıyoruz. Ayrıca hangi düğümlerin başka bir düğümün çocukları olduğunu değiştirmek istiyoruz bu yüzden `children`'i `Vec<Rc<Node>>` çevresinde bir `RefCell<T>`'e sahiyoruz.

Sonra, struct tanımımızı kullanıp `leaf` adında değer `3` ve hiç çocuğu olmayan, `branch` adında değer `5` ve `leaf`'i çocuklarından biri olan başka bir örnek oluşturacağız ki bu Kod Listesi 15-27'de gösterildiği gibi.

<Listing number="15-27" file-name="src/main.rs" caption="Çocuğu olmayan bir `leaf` düğümü ve `leaf`'i çocuklarından biri olan bir `branch` düğümü oluşturma">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:there}}
```

</Listing>

`leaf`'teki `Rc<Node>`'ü klonlayıp `branch` içinde saklarız ki bu demektir ki `leaf` içindeki `Node` artık iki sahibi var: `leaf` ve `branch`. `branch.children` üzerinden `branch`'ten `leaf`'e gelebiliriz ama `leaf`'ten `branch`'e gelmenin yolu yok. Neden şu ki `leaf`'in `branch`'e bir referansı yok ve ilgili olduklarını bilmiyor. `leaf`'in `branch`'in ebeveyni olduğunu bilmesini istiyoruz. Bunu sonraki yapacağız.

#### Bir Çocuktan Ebeveynine Bir Referans Ekleme (Adding a Reference from a Child to Its Parent)

Çocuk düğümünün ebeveynini bilmesini sağlamak için, `Node` struct tanımına bir `parent` alanı eklememiz gerekir. Sorun, `parent`'in hangi türde olması gerektiğine karar vermektir. `Rc<T>` içeremeyeceğini biliyoruz çünkü bu `leaf.parent`'ın `branch`'e ve `branch.children`'ın `leaf`'e işaret etmesiyle `strong_count` değerlerinin asla 0 olmasına neden olacağı ki bu bir referans döngüsü yaratır.

İlişkileri başka bir şekilde düşünürsek, ebeveyn düğüm çocuklarını sahiplenmeli: Bir ebeveyn düğüm bırakılırsa, çocuk düğümleri de bırakılmalıdır. Ancak, bir çocuk ebeveynini sahiplenmemelidir: Bir çocuk düğümü bırakırsak, ebeveynin hala var olması gerekir. Bu, zayıf referanslar için bir durumdur!

Bunun yerine, `Rc<T>`'den, `parent`'ın türünü `Weak<T>` kullanmak için yapacağız, özellikle bir `RefCell<Weak<Node>>`. Şimdi `Node` struct tanımımız şöyle görünür:

<span class="filename">Dosya Adı: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:here}}
```

Bir düğüm, ebeveyn düğümüne referans edebilecek ama ebeveynini sahiplenmeyecek. Kod Listesi 15-28'de, `leaf` düğümünün ebeveynine referans edebilecek şekilde `main`'i güncelliyoruz.

<Listing number="15-28" file-name="src/main.rs" caption="Ebeveyn düğümüne zayıf referansı olan bir `leaf` düğümü, `branch`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:there}}
```

</Listing>

`leaf` düğümünü oluşturmak, `parent` alanı dışında Kod Listesi 15-27'ye benzer ancak `parent` alanı ile: `leaf` ebeveyn olmadan başlar bu yüzden yeni, boş bir `Weak<Node>` referans örneği oluştururak.

Bu noktada, `upgrade` yöntemini kullanarak `leaf`'in ebeveynine referans almaya çalıştığımızda, `None` değeri alırsınız. Bunu ilk `println!` bildiriminden gelen çıktıda görürsünüz:

```text
leaf parent = None
```

`branch` düğümünü oluşturduğumuzda, ebeveyn düğümü olmadığı için `parent` alanında de yeni bir `Weak<Node>` referansı olacak çünkü `branch`'in ebeveyn düğümü yok. Yine `branch`'in çocuklarından biri olarak `leaf`'e hala sahip oluyoruz. Bir kere `branch` içinde `Node` örneği olduktan sonra, `leaf`'i ebeveynine bir `Weak<Node>` referansı vermek için değiştirebiliriz. `leaf`'in `parent` alanındaki `RefCell<Weak<Node>>` üzerinde `borrow_mut` yöntemini kullanır ve sonra `branch` içindeki `Rc<Node>`'den `branch`'e bir `Weak<Node>` referansı oluşturmak için `Rc::downgrade` fonksiyonunu kullanırız.

`leaf`'in ebeveynini tekrar yazdırdığımızda, bu seferde `branch`'i tutan `Some` varyantını alacağız: Şimdi `leaf` ebeveynine erişebilir! `leaf`'i yazdırdığımızda ayrıca Kod Listesi 15-26'da sonunda yığın taşmasına neden olan döngüden kaçınırız; `Weak<Node>` referansları `(Weak)` olarak yazdırılır:

```text
leaf parent = Some(Node { value: 5, parent: RefCell { value: (Weak) },
children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) },
children: RefCell { value: [] } }] } })
```

Sonsuz çıktının olmayışı bu kodun bir referans döngüsü oluşturmadığını gösterir. Ayrıca `Rc::strong_count` ve `Rc::weak_count` çağırarak aldığımız değerlere bakarak bunu söyleyebiliriz.

#### `strong_count` ve `weak_count` Değişiklerini Görselleştirme (Visualizing Changes to strong_count and weak_count)

Yeni bir iç kapsam oluşturup ve `branch` oluşturmasını bu kapsam içine taşıyarak `Rc<Node>` örneklerinin `strong_count` ve `weak_count` değerlerinin nasıl değiştiğine bakalım. Böyle yaparak, `branch` oluşturulduğunda ve kapsam dışına çıktığında ne olduğunu görebiliriz. Değişiklikler Kod Listesi 15-29'da gösterilmiştir.

<Listing number="15-29" file-name="src/main.rs" caption="Güçlü ve zayıf referans sayılarını incelemek için bir iç kapsamda `branch` oluşturma">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-29/src/main.rs:here}}
```

</Listing>

`leaf` oluşturulduktan sonra, onun `Rc<Node>`'unun güçlü sayısı 1 ve zayıf sayısı 0 olur. İç kapsamda, `branch` oluşturup onu `leaf` ile ilişkilendiriyoruz ki bu noktada sayıları yazdırdığımızda, `branch` içindeki `Rc<Node>` 1 güçlü ve 1 zayıf sayıya sahip olacak (`leaf.parent`'ın `branch`'i `Weak<Node>` ile işaret etmesi yüzünden). `leaf` içindeki sayıları yazdırdığımızda, onun güçlü sayısının 2 olacağını görürsünüz çünkü `branch` artık `branch.children` içinde saklanan `Rc<Node>`'un bir kopyasına sahip ama hala 0 zayıf sayısına sahip olacak.

İç kapsam bittiğinde, `branch` kapsam dışına çıkar ve `Rc<Node>`'ünün güçlü sayısı 0'a düşer, bu yüzden onun `Node` bırakılır. `leaf.parent`'dan gelen 1 zayıf sayısının, `Node`'nun bırakılıp bırakılmayacağı üzerinde hiçbir etkisi yoktur, bu yüzden hiçbir bellek sızıntısı almazız!

Kapsam sonunun sonrasında `leaf`'in ebeveynine erişmeye çalışırsak, tekrar `None` alırsınız. Programın sonunda, `leaf` içindeki `Rc<Node>` artık sadece `Rc<Node>`'a bir referans olduğu için 1 güçlü ve 0 zayıf sayısına sahip olacak çünkü `leaf` değişkeni artık sadece `Rc<Node>`'a bir referans.

Sayıları ve değer bıraklamayı yöneten tüm mantık, `Rc<T>` ve `Weak<T>` ve onların `Drop` niteliği uygulamalarına yerleşiktir. `Node` tanımında bir çocuktan ebeveynine ilişkinin `Weak<T>` referansı olmasını belirleyerek, ebeveyn düğümlerin çocuk düğümlerine işaret etmelerini ve tersini referans döngüsü ve bellek sızıntısı yaratmadan yapabilirsiniz.

## Özet (Summary)

Bu bölüm, akıllı göstricileri kullanarak Rust'un varsayılan olarak sıradan referanslarla yaptığı farklı garantileri ve takasları nasıl yapacağınızı kapsadı. `Box<T>` türü bilinen bir boyuta sahiptir ve öbek üzerinde ayrılmış veriye işaret eder. `Rc<T>` türü, öbek üzerindeki verilere olan referansların sayısını takip eder ki böylece verinin çok sahibi olabilir. İç değişkenliği ile `RefCell<T>` türü, değişmez bir tür ihtiyaç duyduğumuzda ancak o türün iç değerini değiştirmemiz gereken bir türü verir; ayrıca ödünç kurallarını derleme zamanı yerine çalışma zamanında zorlar.

Ayrıca `Deref` ve `Drop` niteliklerini tartıştık ki bu nitelikler akıllı göstricilerinin çok işlevlliğini etkinleştirir. Bellek sızıntılarına neden olabilecek referans döngülerini ve bunları `Weak<T>` kullanarak nasıl önleyeceğimizi inceledik.

Eğer bu bölüm ilginizi çektiyse ve kendi akıllı göstricilerinizi uygulamak istiyorsanız, ["Rustonomicon"][nomicon] daha fazla yararlı bilgi için kontrol edin.

Sonra, Rust'ta eşzamanlıktan konuşacağız. Hatta birkaç yeni akıllı göstriciler hakkında öğreneceksiniz.

[nomicon]: ../nomicon/index.html