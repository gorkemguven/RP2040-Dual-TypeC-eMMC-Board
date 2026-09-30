# RP2040 Dual Type-C eMMC High-Speed Development & Telemetry Board (PicoVault)

Bu proje; **Raspberry Pi RP2040** çift çekirdekli mikrodenetleyicisi, **Microchip USB2244** yüksek hızlı USB 2.0 köprüsü ve **Micron eMMC (BGA-153)** yüksek yoğunluklu depolama birimini tek bir 4 katmanlı PCB üzerinde birleştiren endüstriyel standartta bir akıllı veri depolama ve geliştirme kartıdır.

Kart; çift USB Type-C mimarisi, donanımsal güç telemetrisi (çift INA226 akım/voltaj monitörü ve Kelvin algılama), yüksek doğruluklu sıcaklık izleme (TMP119) ve durum bilgilendirme OLED ekran arayüzü ile hem bağımsız bir yüksek hızlı USB flash sürücü hem de derinlemesine analiz yapabilen bir gömülü geliştirme platformu olarak tasarlanmıştır.

---

## 📌 Öne Çıkan Özellikler

* **Çift Çekirdekli İşlem Gücü:** Raspberry Pi RP2040 (Çift ARM Cortex-M0+ @ 133 MHz, 264 KB dahili SRAM).
* **Yüksek Hızlı Depolama (eMMC):** Micron BGA-153 paket yüksek hızlı eMMC bellek.
* **Yüksek Hızlı USB Köprüsü:** Microchip USB2244 High-Speed USB 2.0 SD/eMMC kontrolcüsü (480 Mbps).
* **Boot Flash Belleği:** 128 Mb (16 MB) Winbond W25Q128JVSIC QSPI Flash[cite: 3].
* **Çift USB Type-C Portu:**
  * *Port 1 (Depolama):* USB2244 üzerinden doğrudan eMMC'ye yüksek hızlı USB 2.0 (480 Mbps) doğrudan erişim.
  * *Port 2 (MCU / Telemetri):* RP2040 Full-Speed USB arayüzü (veri kaydı, programlama, seri haberleşme).
* **Donanımsal Güç ve Sıcaklık Telemetrisi:**
  * 2x **TI INA226** I2C güç/gerilim/akım izleme entegresi (2 mΩ 4-terminalli Kelvin şönt algılama)[cite: 7, 8].
  * 1x **TI TMP119** ±0.1°C yüksek hassasiyetli dijital sıcaklık sensörü[cite: 4].
  * 0.96'' I2C SSD1306 OLED ekran konnektörü ile anlık telemetri gösterimi[cite: 5].
* **Genişleme Portu:** Harici SPI, I2C, UART ve GPIO erişimi sağlayan 12-pin breakout soketi[cite: 1].
* **Üretim Optimizasyonu:** Maliyet düşürmek amacıyla tüm kritik bileşenler tek yüzeye (Single-Sided Assembly) yerleştirilmiştir.

---

## 📐 Donanım Mimarisi ve Yüksek Hızlı Sinyal Bütünlüğü

### 1. USB2244 – eMMC (BGA-153) 8-Bit Yüksek Hızlı Veri Yolu
USB2244 denetleyicisi ile eMMC arasındaki 8-bit paralel veri yolu (MMC/eMMC standardı), yüksek frekanslı sinyal geçişlerinde veri kaymasını (skew) ve faz farklarını önlemek amacıyla sıkı empedans ve uzunluk toleranslarıyla yönlendirilmiştir:
* **Referans Saat:** `MMC_CLK` hattı referans alınarak tüm kontrol (`CMD`) ve veri (`DAT0`–`DAT7`) hatları eşlenmiştir.
* **Hat Empedansı:** Katman 2'deki kesintisiz GND referansı üzerinden 50 Ω tek uçlu (single-ended) mikroşerit hat geometrisi.
* **Tolerans Kriteri:** Saat ve veri hatları arasındaki gecikme farkı (skew) < 50 ps (~±1.0 mm) hedeflenmiştir.

| Sinyal Adı | Pin Adı | Rotalama Türü | Hedef Empedans | Via Sayısı | Uzunluk Eşleme Toleransı |
| :--- | :--- | :--- | :---: | :---: | :---: |
| **`MMC_CLK`** | Clock (Saat) | Mikroşerit / GND Referanslı | 50 Ω | Eşit | Referans Hat |
| **`MMC_CMD`** | Command / Response | Çift Yönlü Kontrol | 50 Ω | Eşit | ±1.0 mm |
| **`MMCDA0`** | Veri Biti 0 | Yüksek Hızlı Veri | 50 Ω | Eşit | ±1.0 mm |
| **`MMCDA1`** | Veri Biti 1 | Yüksek Hızlı Veri | 50 Ω | Eşit | ±1.0 mm |
| **`MMCDA2`** | Veri Biti 2 | Yüksek Hızlı Veri | 50 Ω | Eşit | ±1.0 mm |
| **`MMCDA3`** | Veri Biti 3 | Yüksek Hızlı Veri | 50 Ω | Eşit | ±1.0 mm |
| **`MMCDA4`** | Veri Biti 4 | Yüksek Hızlı Veri | 50 Ω | Eşit | ±1.0 mm |
| **`MMCDA5`** | Veri Biti 5 | Yüksek Hızlı Veri | 50 Ω | Eşit | ±1.0 mm |
| **`MMCDA6`** | Veri Biti 6 | Yüksek Hızlı Veri | 50 Ω | Eşit | ±1.0 mm |
| **`MMCDA7`** | Veri Biti 7 | Yüksek Hızlı Veri | 50 Ω | Eşit | ±1.0 mm |
| **`MMC_RST_N`** | Hardware Reset | Filtrelenmiş Kontrol | 50 Ω | - | Asenkron |

*(Not: Veri yollarının tamamında stubsız routing yapılmış, BGA çıkışlarında dönüş akımı sürekliliği için bitişik GND viaları yerleştirilmiştir).*

### 2. QSPI Flash Yönlendirme ve Gecikme Eşleme (Length Matching)
Flash bellek yolu, 133 MHz saat hızında okuma/yazma kararlılığını garanti altına almak için 50 Ω kontrollü empedans ve katı boy eşleme kurallarıyla yönlendirilmiştir:
* **Tüm Sinyal Hatları:** Eşit parazitik yük için tam olarak **2 via** kullanılarak alt katmana geçirilmiştir.
* **Saat Hattı Referansı (`QSPI_SCLK`):** 24.54 mm
* **Veri Hatları Skew Dengesi:**
  * `QSPI_SD0`: 26.10 mm (ΔL = +1.56 mm)
  * `QSPI_SD1`: 25.76 mm (ΔL = +1.22 mm)
  * `QSPI_SD2`: 27.14 mm (ΔL = +2.60 mm)
  * `QSPI_SD3`: 24.81 mm (ΔL = +0.27 mm)
  * `QSPI_SS`: 22.57 mm (ΔL = -1.97 mm)
* **Stub İzolasyonu:** 1 kΩ (`R504`) BOOTSEL izolasyon direnci ve 10 kΩ (`R505`) pull-up direnci, yüksek hızlı `/CS` hattında açık devre yansıması oluşturmaması için doğrudan Flash entegresinin Pin 1 bacağının 1 mm yakınına konumlandırılmıştır[cite: 3].

### 3. Analog Kristal Osilatör Bölgesi (EMC & Noise Immunity)
* **Rezonatör:** RP2040 için 12 MHz SMD kristal ve 15 pF yük kapasitörleri.
* **Sönümleme:** `XOUT` bacağında 1 kΩ seri akım sınırlama/sönümleme direnci.
* **Gürültü Koruması (Keepout Area):** Kristalin ve yük kapasitörlerinin altındaki tüm katmanlar dijital sinyallerden arındırılmıştır[cite: 2]. `SPI_SCK` ve hızlı GPIO hatları kristal sınırının dışına taşınmış; Layer 2 üzerinde kesintisiz katı GND bakırı ve kristal gövde pinlerinde (Pin 2 ve 4) çift via topraklaması uygulanmıştır[cite: 2].

### 4. eMMC Güç Dağıtım Ağı (PDN - Decoupling Network)
* Tek yüzeyli montaj gereği dekuplaj kapasitörleri BGA-153 paketinin çevresine optimize mesafelerle dizilmiştir:
  * **VDDIM (`C2`):** 4.6 mm hat, 2 via, 100 nF + 1 µF çift filtreleme.
  * **VCC (`E6, F5` & `J10, K9`):** 100 nF, 1 µF ve 2.2 µF kapasitör grupları (6.5–8.0 mm).
  * **VCCQ (`C6` & `M4, N4, P3, P5`):** Çekirdek VCC hattındaki anahtarlama gürültüsünden etkilenmemesi için yüzeyde VCC'den izole edilmiş; ana +3.3V düzleminden yerel 2.2 µF kapasitörlerle beslenmiştir.

### 5. Kelvin Akım Algılama ve I2C Topolojisi
* **4-Wire Kelvin Sensing:** 2 mΩ şönt dirençlerin gerilim algılama pinleri (`IN+` ve `IN-`), yüksek akım taşıyan güç pedlerinden bağımsız olarak alt katmandan diferansiyel çift mantığıyla INA226 entegrelerine taşınmıştır[cite: 7, 9].
* **Adresleme:**
  * `IC702` (INA226 #1): A1 = GND, A0 = GND → **0x40**[cite: 8]
  * `IC703` (INA226 #2): A1 = GND, A0 = +3.3V → **0x41**[cite: 8]
* **I2C Veriyolu:** RP2040'tan çıkan tek veriyolu TMP119 (3.5 cm), OLED (5.3 cm), INA1 (7.0 cm) ve INA2 (8.8 cm) arayüzlerini beslemektedir[cite: 4, 5, 8]. Hat empedansı TMP119 yakınındaki 5 kΩ pull-up dirençleri (`R701`, `R702`) ile sürülmekte ve toplam veriyolu kapasitansı 400 pF standardının oldukça altında kalmaktadır[cite: 4, 6].

---

## 🔌 Bağlantı Portları ve Pin Şeması

### 1. J803 Genişleme Konnektörü (1x12 Pin Header)
Harici çevre birimleri ve geliştirme modülleri için dışarı çıkarılmış arayüzdür[cite: 1]:

| Pin | Sinyal Adı | Tip | Rotalama Detayı | Açıklama |
| :---: | :--- | :--- | :--- | :--- |
| **1** | `+3.3V` | Güç | Çift via ile +3.3V katmanına bağlı | Harici modül beslemesi |
| **2** | `UART_TX` | Çıkış | 50.00 mm (0 via) | Seri konsol / Debug çıkışı |
| **3** | `UART_RX` | Giriş | 49.80 mm (0 via) | Seri konsol giriş |
| **4** | `SPI_SCK` | Çıkış | 42.17 mm (2 via) | Harici SPI Saat hattı |
| **5** | `SPI_MOSI` | Çıkış | 41.52 mm (2 via) | Harici SPI Veri çıkışı |
| **6** | `SPI_MISO` | Giriş | 41.83 mm (2 via) | Harici SPI Veri girişi |
| **7** | `SPI_CS` | Çıkış | 41.92 mm (2 via) | Harici SPI Çip seçimi |
| **8** | `I2C_SDA` | Çift Yönlü | 57.08 mm (2 via) | Harici I2C Veri hattı |
| **9** | `I2C_SCL` | Çıkış | 56.62 mm (2 via) | Harici I2C Saat hattı |
| **10** | `GPIO14` | Giriş/Çıkış | 43.70 mm (2 via) | Genel amaçlı GPIO / Kesme |
| **11** | `GPIO15` | Giriş/Çıkış | 43.87 mm (2 via) | Genel amaçlı GPIO / Kesme |
| **12** | `GND` | Güç | Çift via ile ana GND katmanına bağlı | Ortak sistem toprağı |

*(Not: SPI veri yolu hatları arasındaki maksimum skew yalnızca 0.65 mm'dir; bu sayede yüksek hızlı SPI ekranlar veya harici SD modülleriyle sorunsuz çalışır).*

### 2. Programlama ve Hata Ayıklama (Debug)
* **SWD Portu (`SWCLK`, `SWD`, `GND`):** Canlı hata ayıklama (step-by-step breakpoint debug) ve doğrudan bellek programlama arayüzüdür.
  * `SWCLK` (22.60 mm) ve `SWD` (23.00 mm) hatları tam simetrik ve 0.40 mm toleransla eşlenmiştir.
* **RUN (Donanımsal Reset):** 100 nF kapasitör ve 10 kΩ pull-up direnci ile çipin 3.66 mm dibinde filtrelenmiştir.
* **J502 (`USB_BOOT`):** 1 kΩ direnç üzerinden Flash `/CS` hattını toprağa çekerek RP2040'ı dahili ROM UF2 Mass Storage bootloader moduna geçirir[cite: 3].

---

## 🗂️ Depo Dosya Yapısı

```text
├── hardware/
│   ├── schematics/          # KiCad hiyerarşik şematik dosyaları (.kicad_sch)
│   ├── layout/              # KiCad 4-katman PCB yerleşim dosyaları (.kicad_pcb)
│   ├── libraries/           # Projeye özel kütüphane sembol ve ayak izleri (Footprints)
│   └── outputs/
│       ├── gerbers/         # Üretim için RS-274X Gerber ve Drill dosyaları
│       └── bom/             # Parça listesi (Bill of Materials - .csv)
├── docs/
│   ├── schematics.pdf       # Tek tıkla incelenebilir tam şematik PDF çıktısı
│   ├── 3d_renders/          # Kartın ön/arka 3D görsel renderları
│   └── datasheets/          # Kritik entegre dökümanları (USB2244, INA226, TMP119, W25Q128)
├── .gitignore               # KiCad geçici ve yedek dosyalarını filtreler
└── README.md                # Proje dokümantasyonu
