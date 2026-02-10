## Release Profilleriyle Yapılandırmaları Özelleştirme (Customizing Builds with Release Profiles)

Rust'te, _release profilleri_, programcının kod derleme için çeşitli seçenekler üzerinde daha fazla kontrole sahip olmasına izin veren önceden tanımlanmış, özelleştirilebilir farklı yapılandırmalarla profillerdir. Her profil diğerlerinden bağımsız yapılandırılır.

Cargo'nun iki ana profili vardır: `cargo build` çalıştırdığınızda Cargo'nun kullandığı `dev` profili ve `cargo build --release` çalıştırdığınızda Cargo'nun kullandığı `release` profili. `dev` profili geliştirme için iyi varsayılanlarla tanımlanmıştır ve `release` profili release yapılandırmaları için iyi varsayılanlara sahiptir.

Bu profil adları yapılandırmalarınızın çıktısından tanıdık olabilir:

<!-- manual-regeneration
anywhere, run:
cargo build
cargo build --release
and ensure output below is accurate
-->

```console
$ cargo build
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
$ cargo build --release
    Finished `release` profile [optimized] target(s) in 0.32s
```

`dev` ve `release`, derleyici tarafından kullanılan bu farklı profillerdir.

Cargo, projenizin _Cargo.toml_ dosyasında açıkça herhangi bir `[profile.*]` bölümü eklemediğinizde uygulanan, her profil için varsayılan ayarlara sahiptir. Özelleştirmek istediğiniz herhangi bir profil için `[profile.*]` bölümleri ekleyerek, varsayılan ayarların herhangi bir alt kümesini geçersiz kılarsınız. Örneğin, işte `dev` ve `release` profilleri için `opt-level` ayarının varsayılan değerleri:

<span class="filename">Dosya Adı: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

`opt-level` ayarı Rust'ın kodunuza uygulayacağı optimizasyonların sayısını kontrol eder ki bu 0 ile 3 arasındadır. Daha fazla optimizasyon uygulamak derleme süresini uzatır, bu yüzden geliştirmedeyseniz ve kodunuzu sık sık derliyorsanız, sonuç kod daha yavaş çalışsa bile daha hızlı derlemek için daha az optimizasyon istersiniz. Bu yüzden `dev` için varsayılan `opt-level` `0`'dır. Kodunuzu yayınlamaya hazır olduğunuzda, derlemek için daha fazla zaman harcamak en iyisidir. Release modunda sadece bir kez derliyecek ama derlenen programı birçok kez çalıştıracaksınız, bu yüzden release modu daha uzun derleme süresini daha hızlı çalışan kod için takas eder. Bu yüzden `release` profili için varsayılan `opt-level` `3`'tür.

Varsayılan bir ayarı _Cargo.toml_'da onun için farklı bir değer ekleyerek geçersiz kılabilirsiniz. Örneğin, geliştirme profilinde optimizasyon seviyesi 1 kullanmak istiyorsak, projemizin _Cargo.toml_ dosyasına şu iki satırı ekleyebiliriz:

<span class="filename">Dosya Adı: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 1
```

Bu kod `0`'ın varsayılan ayarını geçersiz kılar. Şimdi `cargo build` çalıştırdığımızda, Cargo `dev` profili için varsayılanları ve `opt-level`'e özelleştirmemizi kullanacaktır. `opt-level`'i `1` olarak ayarladığımızdan, Cargo varsayılana göre daha fazla optimizasyon uygulayacak ama release yapılandırmasındakiler kadar değil.

Her profil için yapılandırma seçeneklerinin ve varsayılanların tam listesi için [Cargo'nun belgesine](https://doc.rust-lang.org/cargo/reference/profiles.html) bakın.