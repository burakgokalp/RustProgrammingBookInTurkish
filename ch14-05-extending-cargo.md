## Özel Komutlarla Cargo'yu Genişletme (Extending Cargo with Custom Commands)

Cargo, onu değiştirmek zorunda kalmadan yeni altkomutlarla genişletilebilecek şekilde tasarlanmıştır. Eğer `$PATH` içindeki bir ikili `cargo-something` adındaysa, onu `cargo something` çalıştırarak bir Cargo altkomutmuş gibi çalıştırabilirsiniz. Bu gibi özel komutlar da `cargo --list` çalıştırdığınızda listelenir. `cargo install` ile uzantılar yükleyip sonra onları yerleşik Cargo araçları gibi çalışabilmeniz, Cargo'nun tasarımının süper uygun bir faydasıdır!

## Özet (Summary)

Cargo ve [crates.io](https://crates.io/)<!-- ignore --> ile kod paylaşmak, Rust ekosistemini bir çok farklı görev için faydalı yapanın parçasıdır. Rust'ın standart kütüphanesi küçük ve sabittir ama crate'ler paylaşmak, kullanmak ve dilden farklı bir zaman çizelgesinde iyileştirmek kolaydır. [crates.io](https://crates.io/)<!-- ignore --> üzerinde sizin için faydalı olan kodu paylaşmaktan çekinmeyin; başkası için de faydalı olabilir!