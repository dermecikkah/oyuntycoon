# İstanbul Raylı Sistem Simülasyonu
## Ekonomik Dengeleme, Görev Tasarımı ve Teknik Optimizasyon Mimari Raporu

## 1) Amaç ve Kapsam
Bu rapor, İstanbul raylı sistem temalı yönetim simülasyonunda **sürdürülebilir bir oyun döngüsü** kurmak için ekonomik sistem, görev mimarisi ve teknik performans katmanlarını birlikte yeniden tasarlar. Hedef; oyuncuyu zorluk-beceri dengesinin korunduğu “akış” kanalında tutmak, hızlı doygunluğu ve “sınırsız para” etkisini ortadan kaldırmaktır.

---

## 2) Ekonomi Restorasyonu: Musluk–Lavabo Dengesi

### 2.1 Sorun Tanımı
Mevcut yapıdaki dengesizlik, gelir akışlarının (musluklar) gider/para yok etme mekanizmalarından (lavabolar) güçlü olmasından kaynaklanır. Sonuç: erken dönemde anlamlı, geç dönemde ise trivial kararlar.

### 2.2 Çözüm İlkeleri
- **Sert lavabolar**: bakım, vergi, tamir, operasyonel aşınma.
- **Dinamik fiyatlandırma**: oyuncu serveti ve para arzına duyarlı maliyet artışı.
- **Gelir kompozisyon sınırı**: kira gelirinin bilet gelirini ezmemesi.

### 2.3 Üstel Maliyet Ölçeklendirme
Tüm altyapı, araç ve yükseltme maliyetlerinde:

\[
Cost_n = BaseCost \times GrowthRate^{(n-1)}
\]

Önerilen aralık: `GrowthRate = 1.07–1.15`.

### 2.4 Servet/Seviye Tabanlı Ekonomi Parametreleri
| Parametre | Erken Oyun | Geç Oyun | Etki |
|---|---:|---:|---|
| Temel Maliyet Katsayısı | 1.07 | 1.25 | Servetle hızlanan yatırım maliyeti |
| Bakım Oranı | %2 | %10 | Yaş/kullanım yoğunluğu etkisi |
| Kademeli Vergi | %5 | %35 | Yüksek gelirde artan kamu kesintisi |
| Enflasyon Katsayısı | 1.0 | 10.0+ | Para arzı arttıkça satın alma gücü düşüşü |

### 2.5 Satın Alma Gücü Endeksi (PPI)
Piyasadaki toplam nakit (M) büyüdükçe fiyat düzeyi otomatik artar:

\[
PPI = 1 + \alpha \cdot \log_{10}(1 + M / M_0)
\]

Yeni satın alım fiyatı:

\[
DynamicPrice = BasePrice \times PPI
\]

Bu yapı, büyük servette dahi yeni yatırımı anlamlı kılar.

---

## 3) İstasyon Odaklı Tasarım ve Kira Mekaniği

### 3.1 Seviye Bazlı İstasyon Ekonomisi
- **Seviye 1 – Basit Durak**: otomat ağırlıklı, düşük kira.
- **Seviye 2 – Yerel İstasyon**: küçük perakende, orta gelir.
- **Seviye 3 – Aktarma Merkezi**: zincir mağaza/yiyecek, net ROI.
- **Seviye 4 – Ticari Hub**: yüksek slot kapasitesi, güçlü pasif akış.

### 3.2 Kira Geliri Formülü
\[
Rent_{monthly} = (BaseValue \times StationLevel) \times \left(1 + \frac{PassCount}{10000}\right) \times Satisfaction
\]

- `Satisfaction` aralığı: `0.5–1.2`.
- Memnuniyet; gecikme, yoğunluk, bekleme sürelerinden etkilenir.

### 3.3 Örnek İstasyon ROI Tablosu
| İstasyon Tipi | Yükseltme Maliyeti | Slot | Aylık Kira | ROI (oyun günü) |
|---|---:|---:|---:|---:|
| Mahalle Durağı | 50.000 | 2 | 2.500 | 20 |
| Transfer Merkezi | 500.000 | 8 | 35.000 | 14 |
| Mega Hub (Yenikapı) | 2.500.000 | 25 | 250.000 | 10 |

### 3.4 Gelir Kompozisyon Kuralı
- Bilet Geliri: **%70**
- Kira Geliri: **%20**
- Reklam/Kampanya: **%10**

Kira payı üst sınırı: **bilet gelirinin %40’ı**.

---

## 4) Üç Hat Stratejisi ve Fırsat Maliyeti

### Marmaray
- Çok yüksek kapasite, çok yüksek CAPEX/OPEX.
- Uzun mesafe bilet geliri güçlü, enerji maliyeti yüksek.

### Metro
- Dengeli gelir-gider, yüksek frekans.
- İstasyon ticarileşmesinde ana omurga.

### Tramvay
- Düşük inşa maliyeti, turizm primi potansiyeli.
- Trafik etkileşimi nedeniyle bekleme duyarlılığı yüksek.

### Yenikapı Mega Hub
- Çoklu hat kesişimi, yüksek tahliye kapasitesi.
- Arkeoloji eventi: prestij çarpanı + inşaat süresi/maliyeti artışı.

---

## 5) 60 Görevlik Kronolojik Görev Sistemi

### Faz 1 (1–20): Temeller ve Nostalji
1988–2000 dönemini kapsar; temel operasyon, ilk hatlar ve kurumsal çekirdek.

### Faz 2 (21–40): Genişleme ve Entegrasyon
2000–2015; havalimanı bağlantıları, köprü/boğaz etkisi, Marmaray başlangıcı, Yenikapı arkeoloji yönetimi.

### Faz 3 (41–60): Sürücüsüz ve 2030 Vizyonu
Otomasyon, yüksek hızlı havaalanı erişimi, mega tünel simülasyonu, 700 km ağ ve karbon hedefleri.

**Bitiş koşulu:** 60 görev tamamlandığında “İstanbul Ulaşım Mimarı” unvanı + final gazete ekranı.

---

## 6) Teknik Mimari Onarım Planı

### 6.1 Frame Rate Bağımsız Simülasyon
Her zaman-bazlı akış `deltaTime` ile güncellenmeli:

```js
function update(timestamp) {
  const deltaTime = (timestamp - lastTime) / 1000;
  money += incomePerSecond * deltaTime;
  simulate(deltaTime);
  render();
  lastTime = timestamp;
  requestAnimationFrame(update);
}
```

### 6.2 Yolcu Pathfinding Optimizasyonu
- A* hesapları **spawn** anında veya ağ değiştiğinde tetiklenmeli.
- Global ağ sürümü (`networkVersion`) ile rota geçerliliği kontrolü.
- Hedef eşiği artırılarak “juggling” azaltılmalı.

### 6.3 Object Pooling
- `new/delete` yerine yolcu/araç havuzları.
- GC baskısı düşer, orta-geç oyun stutter azalır.

---

## 7) Gelişmiş Maddiyat Stratejileri

### 7.1 Servet Bazlı İşletme Giderleri
\[
Maintenance = BaseMaintenance \times \log_{10}(TotalMoney)
\]

Bu “başarı vergisi” kâr marjını baskılar, oyunu kırmaz.

### 7.2 Offline Gelir Sınırı
- İlk 4 saat: normal birikim.
- Sonrası: denetimsizlik nedeniyle %90 kısıtlama.

Amaç: günlük geri dönüş davranışını teşvik etmek.

---

## 8) Uygulama Yol Haritası (Teknik)
1. **Ekonomi çekirdeği**: PPI, üstel maliyet, kademeli vergi.
2. **İstasyon sistemi**: seviye/slot/kira/ROI katmanı.
3. **Görev motoru**: 60 görev, tekil kilit açma/bitirme koşulları.
4. **Performans**: delta-time, path cache, object pool.
5. **Dengeleme turu**: telemetriye dayalı oran ayarı.

---

## 9) Başarı Ölçütleri (KPI)
- Erken oyun terk oranı düşüşü.
- Geç oyun “çok kolay” geri bildirimi düşüşü.
- Ortalama görev tamamlama süresi artışı (hedefli).
- FPS stabilitesi ve GC spike sayısında düşüş.
- Gelir dağılımının hedef banda yaklaşması (70/20/10).

---

## 10) Sonuç
Önerilen yeniden yapılandırma, oyunun öz deneyimini korurken ekonomik dengesizlikleri kapatır, görevleri tarihsel anlamla güçlendirir ve teknik altyapıyı donanımdan bağımsız hale getirir. Bu sayede oyuncu, “sınırsız para” etkisinde kaybolmak yerine sürekli stratejik karar üretmek zorunda kalır; simülasyon da daha uzun ömürlü bir tycoon deneyimine dönüşür.
