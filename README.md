# WSL2 Custom Kernel: Wi-Fi Monitor Mode & USB Entegrasyonu

![WSL2](https://img.shields.io/badge/Platform-WSL2-blue) ![Linux](https://img.shields.io/badge/OS-Kali%20Linux-black) ![Kernel](https://img.shields.io/badge/Kernel-6.6.x-green) ![Status](https://img.shields.io/badge/Status-Working-success)

Bu proje, WSL2 üzerinde **Monitor Mode (Ağ Dinleme)** ve **Packet Injection** özelliklerini kullanabilmek için özel olarak derlenmiş bir Linux çekirdeğidir. Standart WSL2 çekirdeğinde bulunmayan USB Wi-Fi sürücülerini ve donanım erişim protokollerini içerir.

---

## Projenin Amacı ve Çözülen Sorunlar

Siber güvenlik eğitimlerinde ve Penetrasyon testlerinde WSL2 kullanırken karşılaşılan en büyük engel, harici USB Wi-Fi adaptörlerinin (özellikle **Ralink/MediaTek** serisi) tanınmaması ve Monitor moda geçememesidir.

**Bu proje ile çözülen teknik engeller:**
1.  **Donanım Desteği:** Standart çekirdekte olmayan `vhci-hcd` (USB/IP) ve `mt7601u` (Wi-Fi) sürücüleri çekirdeğe **gömülü (built-in)** olarak eklendi.
2.  **Derleme Hataları:** GCC 13+ derleyicilerinde eski kod tabanının yarattığı `false/true keyword` çakışmaları ve `objtool` hataları, Makefile yapılandırması C standartlarına (`gnu11`) göre optimize edilerek çözüldü.
3.  **Performans:** Gereksiz Debug sembolleri kaldırılarak çekirdek boyutu optimize edildi (~14MB).

---

## Kurulum (Hızlı Yöntem - Derleme Gerektirmez)

Eğer teknik detaylarla uğraşmadan sadece sistemi kullanmak istiyorsanız

### 1. Çekirdeği İndirin
Sağ taraftaki **[Releases]** kısmından en son yayınlanan **`bzImage`** dosyasını indirin.

### 2. Dosyayı Yerleştirin
İndirdiğiniz dosyayı Windows kullanıcı klasörünüze taşıyın.
`C:\Users\KullaniciAdiniz\bzImage`

### 3. WSL'i Yapılandırın
Aynı klasörde (`C:\Users\KullaniciAdiniz\`) **`.wslconfig`** adında bir dosya oluşturun (veya varsa açın) ve şu satırları ekleyin:

```ini
[wsl2]
kernel=C:\\Users\\KullaniciAdiniz\\bzImage 
```
(wsl`i yeniden başlatın)


### Adım 2: USB Cihazını Listeleme ve Linux'a Aktarma

WSL2, taktığınız USB cihazlarını (Wi-Fi Adaptörü, Mouse vb.) varsayılan olarak görmez. Windows üzerindeki `usbipd` aracı ile bu cihazları Linux'a "köprü" yapmamız gerekir.

### 1. Windows Tarafı 

Öncelikle Wi-Fi adaptörünüzü bilgisayara takın ve PowerShell'i **Yönetici Olarak** çalıştırın.

**A. Cihazları Listeleme:**
Hangi cihazın hangi ID'ye sahip olduğunu görmek için şu komutu yazın:
```powershell
usbipd list
```
bu komut 2-4 gibi bir ID verecek HATIRLA

**B. Cihazı Paylaşıma Açma (Bind):**
```powershell
usbipd bind --busid 2-4
```
**C. Cihazı Linux'a Bağlama (Attach)**
```powershell
usbipd attach --wsl --busid 2-4
```
(Eğer hata vermeden alt satıra geçiyorsa veya "Attached" diyorsa işlem başarılıdır)


### 2. Linux Tarafı 
`lsusb`
Listede adaptörünüzün markasını (Ralink, MediaTek, Realtek vb.) görüyorsanız, donanım fiziksel olarak Linux'a bağlanmış demektir

`iwconfig`

Çıktıda `wlan0` veya benzeri bir ibare görüyorsanız tebrikler! Monitor moda geçmeye hazırsınız.




