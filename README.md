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
```bash
lsusb
```
Listede adaptörünüzün markasını (Ralink, MediaTek, Realtek vb.) görüyorsanız, donanım fiziksel olarak Linux'a bağlanmış demektir

monitormode aktif mi kontrol edin
```bash
iwconfig
```

Çıktıda `wlan0` veya benzeri bir ibare görüyorsanız tebrikler! Monitor moda geçmeye hazırsınız.



---

## Geliştirici Rehberi: Kendi Çekirdeğinizi Derleyin

Bu çekirdeği kendiniz derlemek, farklı sürücüler eklemek veya süreci öğrenmek istiyorsanız aşağıdaki adımları takip edebilirsiniz. Bu rehber, **GCC 13+** derleyicilerinde karşılaşılan hataları (`false keyword`, `objtool`, `BTF`) giderecek şekilde hazırlanmıştır.

### 1. Gerekli Paketlerin Kurulumu
Kali Linux (veya Ubuntu) üzerinde derleme araçlarını kurun:

```bash
sudo apt update
sudo apt install build-essential flex bison libssl-dev libelf-dev libncurses-dev autoconf libudev-dev zlib1g-dev bc dwarves
```
###2. Kaynak Kodun İndirilmesi
# Kullandığınız WSL çekirdek sürümünü öğrenin (Örn: 6.6.87.2)
```bash
uname -r
```

# O sürüme ait kaynak kodunu indirin (Microsoft Reposundan)
```bash
git clone --depth 1 --branch linux-msft-wsl-6.6.87.2 [https://github.com/microsoft/WSL2-Linux-Kernel.git](https://github.com/microsoft/WSL2-Linux-Kernel.git)
cd WSL2-Linux-Kernel
```
###3. Konfigürasyonun Hazırlanması
# Çalışan config dosyasını kopyala
```bash
cp /proc/config.gz config.gz
gunzip config.gz
mv config .config
```
# Menüyü aç ve Wi-Fi / USB ayarlarını yap
```bash
make menuconfig
```

### Menuconfig Yol Haritası (Adım Adım)

Çekirdek ayar menüsünü (`make menuconfig`) açtıktan sonra, Wi-Fi kartınızı (MT7601U) aktif etmek için aşağıdaki yolu izleyin.

**Navigasyon İpuçları:**
* **Girmek için:** `Enter`
* **Geri dönmek için:** `Esc` (iki kez)
* **Seçmek (Gömülü yapmak) için:** `Y` tuşu (Yanında `<*>` işareti olmalı)

#### 1. Wi-Fi Sürücüsünü Aktif Etme
Şu yolu takip edin:
1.  `Device Drivers` --->
2.  `Network device support` --->
3.  `Wireless LAN` --->
4.  `MediaTek devices` --->
5.  **`MediaTek MT7601U (USB) support`** seçeneğini bulun.
    * Üzerine gelip **`Y`** tuşuna basın.
    * Sol tarafında **`<*>`** işaretini gördüğünüzden emin olun. (`<M>` değil!)

#### 2. Temel Wi-Fi Desteğini Açma (Önemli)
Sürücünün çalışması için ana Wi-Fi yığınının da gömülü olması gerekir.
1.  Ana menüye dönün.
2.  `Networking support` --->
3.  `Wireless` --->
4.  **`cfg80211 - wireless configuration API`** ---> **`<*>` (Yıldızla)**
5.  **`Generic IEEE 802.11 Networking Stack (mac80211)`** ---> **`<*>` (Yıldızla)**

#### 3. USBIP Desteğini Kontrol Etme
Windows ile USB bağlantısının kopmaması için:
1.  Ana menüye dönün.
2.  `Device Drivers` --->
3.  `USB support` --->
4.  **`Support for Host-side USB`** ---> **`<*>` (Yıldızla)**
5.  **`USB/IP support`** ---> **`<*>` (Yıldızla)**

---
*(İşlemler bitince alttaki **`< Save >`** butonuna basıp kaydedin ve **`< Exit >`** ile çıkın.)*

###4. Derleme ve Optimizasyon
### 1. Gereksiz Debug bilgilerini kapat (Linker hatalarını ve devasa boyutu önler)
scripts/config --disable CONFIG_DEBUG_INFO
scripts/config --disable CONFIG_DEBUG_INFO_BTF
scripts/config --disable CONFIG_DEBUG_INFO_DWARF4
scripts/config --disable CONFIG_DEBUG_INFO_DWARF5
scripts/config --disable CONFIG_PAHOLE_HAS_SPLIT_BTF

### 2. Derlemeyi Başlat (GNU11 standardı zorlaması ile)
```bash
make -j$(nproc) KCONFIG_CONFIG=.config KCFLAGS="-std=gnu11"
```

İşlem bittiğinde derlenmiş çekirdeğiniz şu yolda hazır olacaktır: `arch/x86/boot/bzImage`

---

## Derleme Sonrası: Windows Tarafında Kurulum

Derleme işlemi başarıyla tamamlandığında ve `bzImage is ready` mesajını gördüğünüzde, yeni çekirdeği Windows'a tanıtmanız gerekir.

### 1. Çekirdeği Windows'a Kopyalayın
Derlenen çekirdek dosyası (`bzImage`) Linux dosya sistemindedir. Bunu Windows kullanıcı klasörünüze taşımak için terminalde şu komutu kullanın:

```bash
# "KullaniciAdiniz" kısmını kendi Windows kullanıcı adınızla değiştirin
cp arch/x86/boot/bzImage /mnt/c/Users/KullaniciAdiniz/bzImage-wifi

### 2. .wslconfig Dosyasını Oluşturun/Düzenleyin
WSL2'nin varsayılan çekirdek yerine sizin derlediğinizi kullanması için bir ayar dosyası oluşturmalısınız.

1.  Windows'ta `C:\Users\KullaniciAdiniz\` (Kullanıcı Klasörü) dizinine gidin.
2.  Burada **`.wslconfig`** adında bir dosya oluşturun (Eğer yoksa).
3.  Dosyayı Not Defteri ile açın ve şunları ekleyin:

```ini
[wsl2]
kernel=C:\\Users\\KullaniciAdiniz\\bzImage-wifi
```

### 3. WSL'i Yeniden Başlatın
Yeni ayarların geçerli olması için WSL'i tamamen kapatıp açmak şarttır.

**PowerShell (Yönetici)** açın ve şu komutu girin:
```powershell
wsl --shutdown
```


### 4. Doğrulama (Büyük An) 
Kali Linux (veya kullandığınız dağıtımı) tekrar açın ve terminale şu komutu yazın:

```bash
uname -r
```
Çıktının sonunda + işareti veya derleme tarihini görüyorsanız (Örn: `6.6.87.2-microsoft-standard-WSL2+`), tebrikler! Artık kendi derlediğiniz çekirdek üzerindesiniz.

