## Kurtarılamaz Hatalar `panic!` ile (Unrecoverable Errors with `panic!`)

Bazen kodunuzda kötü şeyler olur ve bunun hakkında yapabileceğiniz hiçbir şey yoktur. Bu durumlarda, Rust'ın `panic!` makrosu vardır. Pratikte bir panik iki şekilde oluşur: kodunuzu paniklemeye neden olan bir eylem alarak (örneğin dizinin sonunun ötesine erişmek gibi) veya açıkça `panic!` makrosunu çağırarak. Her iki durumda, programımızda bir panik neden oluruz. Varsayılan olarak, bu panikler bir başarısızlık mesajı yazdırır, geriye sarar, yığını temizler ve çıkar. Bir çevre değişkeni aracılıyla, ayrıca bir panik olduğunda Rust'a çağrı yığını görüntülemesini sağlayabilirsiniz ki bu, panikın kaynağını aşağı takip etmeyi kolaylaştırır.

> ### Panike Yanıtı Olarak Yığını Geriye Sarmak veya İşletme Durması (Unwinding the Stack or Aborting in Response to a Panic)
> Varsayılan olarak, bir panik oluştuğunda, program _geriye sarma_ (unwinding) başlar ki bu, Rust'ın yığını geriye yürüdüğünü ve karşılaştığı her fonksiyonun veriyi temizlediğini belirler. Ancak, geriye yürütmek ve temizlemek çok işdir. Bu nedenle, Rust size hemen _işletme_ (aborting) alternatifini seçmenizi sağlar ki bu, temizlemeksiz programı biter.
> Programın kullandığı belleği sonra işletim sistemi tarafından temizlenmesi gerekir. Eğer projenizde oluşturulan ikili dosyası mümkün olduğunca küçük olmasını gerektiriyorsa, panik durumunda geriye sarmadan işletmeye geçebilirsiniz bunu _Cargo.toml_ dosyanızdaki uygun `[profile]` bölümlerine `panic = 'abort'` ekleyerek. Örneğin, yayın modunda panik durumunda işletmek isterseniz, bunu ekleyin:
> ```toml
> [profile.release]
> panic = 'abort'
> ```

Basit bir programda `panic!`'i çağmayı deneyelim:

<Listing file-name="src/main.rs">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-01-panic/src/main.rs}}
```

</Listing>

Programı çalıştırdığınızda, şuna benzer bir şey göreceksiniz:

```console
{{#include ../listings/ch09-error-handling/no-listing-01-panic/output.txt}}
```

`panic!` çağrısı, son iki satırda içeren hata mesajına neden olur. İlk satır panik mesajımızı ve kaynak kodumuzdaki panik oluştuğu yeri gösterir: _src/main.rs:2:5_, ikinci satır, beşinci karakterimizin _src/main.rs_ dosyasında olduğunu belirler.

Bu durumda, belirtilen satır kodumuzun bir parçasıdır ve o satıra gidersek, `panic!` makro çağrısını görürüz. Başka durumlarda, `panic!` çağrısı kodumuzun çağırdığı kodda olabilir ve hata mesajı tarafından raporlanan dosya adı ve satır sayısı, `panic!` makrosunun çağrıldığı başkası kod, sonunda `panic!` çağrısına neden olan kodumuzun satırı olacaktır.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-a-panic-backtrace"></a>

Bir Panikten Geldiği Fonksiyonların Yığını Kullanabiliriz ki problemi neden olan kodumuzun parçasını anlamak için. Bir `panic!` yığınınasını nasıl kullanacağımızı anlamak için başka bir örneğe bakalım ve bir panik doğrudan kodumuzun makroyu çağırmak yerine kodumuzdaki bir hatadan dolayı bir kütüphaneden geldiğinde neye benzedğini görelim. Kod Listesi 9-1, geçerli indekslerin aralığının ötesindeki bir vektördeki bir indekse erişmeyi deneyen bazı kod sahiptir.

<Listing number="9-1" file-name="src/main.rs" caption="Bir vektörün sonunun ötesindeki bir öğeye erişmeyi deneme, bu bir `panic!` çağrısına neden olacak">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-01/src/main.rs}}
```

</Listing>

Burada, vektörümüzün yüzüncü öğesine erişmeyi deniyoruz (bu, indeksleme sıfırdan başladığı için indeks 99'dedir) ancak vektörümüz sadece üç öğe sahiptir. Bu durumda, Rust panikleyecek. `[]` kullanmak bir öğe dönmek içindir ancak geçersiz bir indeks geçirirseniz, Rust'ın burada dönebileceği doğru olabilecek bir öğe yoktur.

C'de, bir veri yapısının sonunun ötesini okumak tanımsız davranıştır. Veri yapısındaki o öğeye karşılık gelen bellekte ne olursa olsun alabilirsiniz, bellek o yapının ait olmasa rağmen. Bu _buffer aşırma_ (buffer overread) olarak adlandırılır ve bir saldırganın indeksi veri yapısından sonra saklanmış olana izin veriyi okumamasını sağlayacak şekilde manipüle edebileceği durumda güvenlik açıklarına yol açabilir.

Programınızı bu türde bir açıklıktan korumak için, varolmayan bir indeksteki bir öğeyi okumayı çalışırsanız, Rust yürütümeyi durduracak ve devam etmeyi reddedecek. Deneyelim ve görelim:

```console
{{#include ../listings/ch09-error-handling/listing-09-01/output.txt}}
```

Bu hata hatanın _main.rs_'in 4. satırına işaret eder ki burada `v` içindeki vektörün 99. indeksine erişmeye çalışıyoruz.

`note:` satır bize, hataya neden olan tam olarak ne olduğunu anlamak için `RUST_BACKTRACE` çevre değişkenini ayarlayabileceğimizi söylüyor. Bir _yığınıas_ (backtrace), bu noktaya ulaşmak için çağrılmış tüm fonksiyonların listesidir. Rust'teki yınınaslar, başka dillerde oldukları gibi çalışır: Bir yığınıasını okumak için anahtar en üstten başlamak ve yazdığınız dosyaları görene kadar okumaktır. Bu, sorunun orijinlandığı yerdir. O noktanın üstündeki satırlar, kodunuzun çağırdığı koddur; aşağıdaki satırlar, kodunuzu çağıran koddur. Bu öncesi-sonrası satırlar çekirdek Rust kodunu, standart kütüphane kodunu veya kullandığınız kafaları içerebilir. `RUST_BACKTRACE` çevre değişkenini `0`'dan başka herhangi bir değere ayarlayarak bir yığınıas almaya çalışalım. Kod Listesi 9-2, göreceğinize benzer çıktıyı gösterir.

<!-- manual-regeneration
cd listings/ch09-error-handling/listing-09-01
RUST_BACKTRACE=1 cargo run
copy backtrace output below
check backtrace number mentioned in text below listing
-->

<Listing number="9-2" caption="`panic!` çağrısı tarafından oluşturulan yınınası `RUST_BACKTRACE` çevre değişkeni ayarlandığında görüntülenen">

```console
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at src/main.rs:4:6:
index out of bounds: the len is 3 but the index is 99
stack backtrace:
   0: rust_begin_unwind
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/std/src/panicking.rs:692:5
   1: core::panicking::panic_fmt
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:75:14
   2: core::panicking::panic_bounds_check
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:273:5
   3: <usize as core::slice::index::SliceIndex<[T]>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:274:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:16:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/alloc/src/vec/mod.rs:3361:9
   6: panic::main
             at ./src/main.rs:4:6
   7: core::ops::function::FnOnce::call_once
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/ops/function.rs:250:5
note: Bazı detaylar atlanmıştır, ayrıntılı bir yınınas için `RUST_BACKTRACE=full` ile çalıştırın.
```

</Listing>

Bu çok fazla çıktı! Tam olarak göreceğiniz çıktı işletim sisteminiz ve Rust sürümünüze göre farklı olabilir. Bu bilgide yınınaslar almak için, hata ayıklama sembolleri etkinleştirilmiş olmalıdır. Hata ayıklama sembolleri, burada yaptığımız gibi `--release` bayrağı olmadan `cargo build` veya `cargo run` kullanırken varsayılan olarak etkindir.

Kod Listesi 9-2'deki çıktıda, yınınasın 6. satırı projemizdeki problemi neden olan satıra işaret eder: _src/main.rs_'ın 4. satırı. Eğer programımızın paniklemesini istemiyorsak, araştırmamızı, dosyamızı yazdığını belirten ilk satırın işaret ettiği yerde başlamalıyız. Kod Listesi 9-1'de, paniklemeyecek kodu kasıtlı yazdığımızda, panik düzeltmenin yolu, vektör indekslerinin aralığının ötesindeki bir öğe istememektir. Kodunuz gelecekte paniklediğinde, kodun hangi eylemi hangi değerlerle alıp paniklemeyeceğini ve kodun bunun yerine ne yapması gerektiğini anlamanız gerekecek.

Hata durularını ele almak için `panic!`'i ne zaman ve ne zaman kullanmamamız gerektiğini, bu bölümün ilerleyen kısmında [ "`panic!` Kullanmak veya Kullanmamak" (To `panic!` or Not to `panic!`][to-panic-or-not-to-panic]<!-- ignore --> bölümünde tekrar bakacağız. Sonra, `Result` kullanarak bir hatadan kurtarmayı nasıl yapacağımıza bakacağız.

[to-panic-or-not-to-panic]: ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic