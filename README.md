## WiringPi Kullanımı
WiringPi, tüm Raspberry PI modelleri ve BCM2835, BCM2836 ve BCM2837 SoC cihazları için C ile yazılmış bir GPIO erişim kütüphanesidir. Kaynak kodu halka açık değildir fakat biz bulduk :)

WiringPI kullanmaya başlayan arkadaşlar ilk önce wiringPi kütüphanesini RPi üzerinde terminal vasıtasıyla doğrudan indirip kullanmıştır. Fakat biz geliştirmeyi RPi üzerinde yapmıyoruz. Bize HOST makinemizde compile edebileceğimiz bir ortam gerekiyor. Fakat HOST makinemizde de GPIO yok. Ayrıca HOST makinede derledğimiz uygulamayı başarılı şekilde nasıl RPi üzerinde çalıştıracağız? Çünkü işlemciler birbirinden tamamen farklı.

Bu sorunları aşmak için şöyle bir yol izlemeliyiz. HOST makinemiz için öncelikli olarak WiringPi kütüphanesini kurmamız gerekiyor. Yalnız dikkat edelim HOST makinemizde !!!GPIO YOK!!! Aynı zamanda HOST makinemizden RPi'ye cross compile etmemiz gerekecek, o zaman WiringPi'i de cross compile etmemiz gerekecek. Her neyse, anlatıma başlayalım.

Öncelikli olarak git projesini clone ediyoruz

> git clone https://github.com/muratcezan/WiringPi

Sonrasında HOST makine için kurulumunu devam eden komutlarla yapıyoruz.

> cd WiringPi
> ./build

sonrasında projenize doğrudan import edip kullanabilirsiniz.

### Cross-Compile işlemini nasıl yapacağız? 
Bunun için bizim bir toolchain'e ihtiyacımız var. Ben Linaro Toolchain kullanıyorum. Fakat siz bunları doğrudan terminal aracılığıyla da indirip kullanabilirsiniz. Her neyse Linaro Toolchain için;

`https://www.linaro.org/` adresinden sisteminiz için gerekli olan(Linux Kullanıyoruz ve Linux üzerine anlatıyoruz) toolchain'i indirebilirsiniz. Sonra inen dosyayı extract edin ve temiz bir yerde saklayın, buna ihtiyacımız olacak.

Sonra tekrar WiringPi kütüphane dosyalarına geliyoruz. Şimdi biraz değişiklik kodlarda değişiklik yapma zamanı. Öncelikle çalıştırdığımız `build` scriptinin ne yaptığına bakalım.

> cd WiringPi
> vi build

Satırlar arasında gezerken projenin nasıl clean ve uninstall edilmesiyle ilgili bir çok detay görüyoruz ve evet aradığımız satırı bulduk... `# Only if you know what you're doing! ` ile başlayan satır. Bu satırdan sonra WiringPi dizini altında bulunan `WiringPi, devLib ve gpio` dizinlerine girip bunları make `install-deb` ettiğini görüyoruz. Bizde bu dizinlerde ki Makefile'lara girip derleyiciyi değiştiriyoruz ve `Linaro Toolchain` 'imizin içinde gelen `C` için arm derleyicisini seçiyoruz. Benim örnek dizinim;

> CC	= /opt/gcc-linaro-7.5.0-2019.12-i686_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc

Daha sonra derlemeyi kaldırıyoruz, temizliyoruz ve build dosyasında görüğümüz komutları sırasıyla elle çalıştırmaya başlıyoruz.
> cd WiringPi
> ./build clean
> ./build uninstall
> cd wiring
> make install-deb

Bu işlemden sonra `HOME` dizinine gidip `~/wiringPi` dizinin oluşup oluşmadığını ve oluştuysa,
> file ~/wiringPi/debian-template/wiringPi/usr/lib/libwiringPi.so.2.60

komutuyla kütüphanenin arm için derlenip derlenmediğini kontrol ediyoruz. ` ... ELF 32-bit LSB pie executable, ARM, EABI5 version ... ` yazısını gördüysek sorunsuz şekilde kütüphanemizi Cross-Compile etmiş oluyoruz ve derlemeye devam ediyoruz.

> cd ..//devLib
> make install-deb INCLUDE='-I. -I../wiringPi'
> cd ../gpio
> make install-deb INCLUDE='-I../wiringPi -I../devLib' LDFLAGS=-L/home/murat/wiringPi/debian-template/wiringPi/usr/lib

Bu son satırda dikkat edilmesi gereken husus HOME dizinindeki wiringPi dizininden .so dosyalarımızı çağırmış olmamız.

Artık -lwiringPi parametresini ekleyerek HOST makinede geliştirip, Cross-Compile edip RPi boardumuz üzerinde uygulamalarımızı koşturabiliriz.
Bir not data, HOST makinede GPIO olmadığı için, GPIO yada wiringPi kütüphane fonksiyonlarının çağrıldığı yerlerde MACRO kullanıp HOST makinede gerçekten oraya girmişmi diye daha kısa yollardan test edebilirsiniz.

Başarılar.