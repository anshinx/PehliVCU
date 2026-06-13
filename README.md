# Pehlivan VCU (Vehicle Control Unit) - REVO.1 🏎️⚡

Trakya Üniversitesi Pehlivan Team ve Tas-Pro (Tasarım ve Proje Topluluğu) bünyesinde Shell Eco-marathon yarış aracı için özel olarak geliştirilmiş, endüstriyel standartlarda **Araç Kontrol Ünitesi (VCU)** donanım ve gömülü yazılım projesidir.

Bu kart; verimlilik, hafiflik ve kurşungeçirmez gürültü bağışıklığı (EMI/EMC) felsefesiyle, geleneksel ağır komponentlerin (lineer regülatörler, taş dirençler vb.) yerini tamamen akıllı yazılımsal ve anahtarlamalı çözümlere bırakması amacıyla sıfırdan tasarlanmıştır.

---

## 🛠️ Teknik Özellikler & Donanım Mimarisi

* **Ana İşlemci:** STM32G431 (ARM Cortex-M4 @170MHz, Matematik Hızlandırıcı CORDIC & FMAC).
* **Güç Girişi:** 12V - 14.4V Yarış Aküsü (Doğrudan besleme).
* **Periferi Beslemeleri:** * Yeni nesil yüksek verimli anahtarlamalı güç katı.
  * Nextion HMI, LoRa ve SD Kart modülleri için `100nF` seramik ve `10µF` kutuplu elektrolitik filtre kombinasyonlu izole besleme hatları.
* **Güç Çıkışları (Sürücü Katı):** `BTS432` Akıllı Güç Anahtarları (Far, Sinyal ve Fren Lambası kontrolü için).
* **Haberleşme & Telemetri:** * Doğrudan işlemciye bağlı LoRa Modülü (Kablosuz Telemetri).
  * Nextion HMI Ekran Arayüzü (UART).
  * FDCAN (Geriye dönük klasik CAN-Bus uyumlu, pitte maksimum esneklik).
* **Veri Günlüğü (Data Logging):** SD Kart Modülü üzerinden SPI haberleşmeli, anlık veri kaybını engelleyen "Kara Kutu" loglama altyapısı.
* **Zaman Senkronizasyonu:** CR2032/Düğme Pil destekli dahili RTC (Real Time Clock) mimarisi.

---

## ⚡ Donanım Tasarım Radikalleri (Neden REVO.1?)

REVO.1 revizyonu, pitteki kriz anlarını ve araç içi verimliliği maksimuma çıkarmak için şu kritik donanımsal kararlarla şekillendirilmiştir:

1. **Sıfır Taş Direnç / Sıfır Isı:** Şerit LED'lerin akım sınırlaması için kaba ve el yakan taş dirençler kullanılmamıştır. Akım kontrolü, STM32'nin Timer birimleri üzerinden **yazılımsal PWM (Sinyal Genişlik Modülasyonu)** ile çözülmüştür. Akü 14.4V'a vurduğunda lamba hatları `%83 Duty Cycle` ile kilitlenerek ortalama akım ideal 12V seviyesine çekilir.
2. **Yüksek Titreşim Dayanımı:** Kart üzerindeki 16 MHz harici kristal katında, yarış esnasındaki vibrasyondan bacakların yorulup kopmasını engellemek amacıyla bacaklı kılıflar yerine **Raltron 3225 4-SMD (RH100-16.000-18-2020-TR)** seramik kılıf tercih edilmiştir. Kristal yük kapasitesi ($C_L = 18\text{pF}$) hesaba katılarak hatlar `26pF` (veya `22pF`/`27pF`) kondansatörlerle stabilize edilmiştir.
3. **Parazit Bağışıklığı & Reset Koruması:** Araç içi motor sürücülerin ve yüksek akım hatlarının yaratacağı manyetik gürültülerden (EMI) korunmak için `NRST` (Reset) pini harici **10 kΩ dirençle `+3V3` hattına asılmış (Pull-Up)** ve `100nF` kondansatör ile şaseye filtrelenmiştir.
4. **Çift Katmanlı Bakır Havuzu:** Gürültü bağışıklığı için Bottom Layer tamamen kesintisiz `GND` bakır dolgusuyla kaplanmış, Top Layer ise STM32 etrafında `3V3` güç adalarıyla zırhlandırılmıştır.

---

## 🛰️ Gecikme (Latency) & Telemetri Lojiği

Kart üzerindeki pil destekli RTC donanımı, pit duvarı ile araç arasındaki LoRa veri transferinin gecikme süresini (latency) milisaniyelik çözünürlükle hesaplamak için kullanılır:
* Araç çalıştırılmadan önce bilgisayardan (veya Nextion üzerinden) Unix Epoch Time sinyaliyle RTC eşitlenir.
* VCU, LoRa paketini fırlatırken Unix zaman damgasının arkasına `HAL_GetTick()` değerinin son 3 basamağını ekler (`[UnixTimestamp].[Milisaniye]`).
* Pit tarafındaki alıcı yazılım, paketin varış zamanı ile paket içindeki zaman damgasını çıkararak hattaki milisaniyelik gecikmeyi anlık olarak raporlar.

---
