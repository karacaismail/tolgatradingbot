# Örnek ECA Kuralları

!!! warning "Bunlar hipotezdir, reçete değil"
    Aşağıdaki kurallar **backtest edilecek hipotezlerdir**, "kesin kazandırır" reçeteleri değil. Eşik değerleri başlangıç noktasıdır; her parite × zaman dilimi × rejim için ayrı doğrulanmalıdır. Bkz. [Backtest Bütünlüğü](../risk/backtest.md).

## Koruyucu / operasyonel kurallar (yüksek öncelik)

### ECA-001 — Bayat/eksik veri (en yüksek öncelik)
- **Event:** yeni mum kapanışı
- **Condition:** mum / order book / hesap verisinden biri beklenenden eski
- **Action:** yeni pozisyon açma; açık emirleri incele; uyarı üret → **NO_TRADE**
- *Her alış kuralından yüksek önceliklidir.*

### ECA-006 — Günlük kayıp limiti (devre kesici)
- **Event:** her gerçekleşen işlem / bakiye güncellemesi
- **Condition:** günlük gerçekleşmiş + gerçekleşmemiş zarar tanımlı limiti geçti
- **Action:** yeni girişleri engelle, gerekirse açık emirleri iptal, **insan onayı olmadan yeniden başlatma**
- *Sayısal limit kullanıcı risk profili belirlenmeden kesinleşmez.*

### ECA-008 — Makro olay karartması
- **Event:** CPI / NFP / FOMC / PCE yaklaşıyor
- **Condition:** olay için tanımlı karartma penceresi aktif
- **Action:** yeni pozisyonu engelle veya yalnız azaltıcı emirlere izin ver
- *FOMC tarihleri resmi takvimden; bkz. [Veri Kaynakları](../veri/kaynaklar.md).*

### ECA-009 — Çapraz piyasa riski
- **Event:** BTC volatilitesi / stablecoin fiyatı / funding / genel likidite eşik değiştirdi
- **Condition:** altcoin pozisyonu/emri var VE BTC risk-off'a geçti veya stablecoin depeg alarmı
- **Action:** altcoin girişlerini durdur; mevcut riskleri yeniden hesapla

## Strateji (giriş) kuralları

### ECA-002 — Trend yönünde pullback
- **Event:** 1h mum kapandı
- **Condition:** 4h `EMA50 > EMA200` (trend ↑) VE 1h fiyat EMA50 üstüne yeniden kapandı VE hacim yeterli VE spread/slippage sınırda VE haber karartması yok VE portföy riski izin veriyor
- **Action:** doğrudan alış değil, **risk motoruna alış önerisi**

### ECA-003 — Kırılım (breakout)
- **Event:** fiyat son 20 mum tepesi üstünde **kapandı** (iğne değil)
- **Condition:** hacim artışı doğrulandı VE order book aşırı dengesiz değil VE cooldown aktif değil
- **Action:** limit emir önerisi; dolmazsa `expiry`'de iptal

### ECA-004 — Range / mean-reversion
- **Event:** fiyatın istatistiksel sapması eşiği geçti
- **Condition:** rejim sınıflandırıcı = yatay VE güçlü trend/haber şoku yok VE likidite yeterli VE spread normal
- **Action:** küçük pozisyon öner; ortalamaya dönüşte kapat
- *Trend rejiminde bu kural çalışamaz.*

## Çıkış kuralı

### ECA-005 — Koruyucu çıkış { #eca-005-koruyucu-cikis }
- **Event:** fiyat volatiliteye göre hesaplanan risk sınırına ulaştı
- **Condition:** pozisyon gerçekten var VE reconciliation tamam VE çıkış emri henüz gönderilmedi
- **Action:** pozisyon azalt/kapat
- *Stop, boşluk/slippage nedeniyle kaybı kesin sınırlamaz — bkz. [Risk](../risk/yonetim.md).*

## Sentiment kuralı (sınırlı yetki)

### ECA-007 — Sosyal medya olayı
- **Event:** izlenen hesaplarda olağandışı yoğunluk
- **Condition:** izinli+doğrulanmış kaynak VE bağlantı birincil kaynağa gidiyor VE en az bir bağımsız kaynakla teyit VE eski haber tekrarı değil VE piyasa verisi olayı destekliyor
- **Action:** **sadece güven puanı / risk uyarısı üret**
- **Yasak:** tek başına emir oluşturmak. Bkz. [Veri hiyerarşisi](../veri/kaynaklar.md).
