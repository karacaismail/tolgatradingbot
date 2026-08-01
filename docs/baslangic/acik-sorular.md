# Açık Sorular

Bunlar akışı bloklamaz ama kod ilerledikçe netleşmesi gereken kararlardır. Cevaplandıkça bu sayfa güncellenir.

## Erken fazda gerekli

| # | Soru | Neden önemli | Öneri |
|---|------|--------------|-------|
| A | **İlk sermaye ve işlem başına risk %'si?** | Pozisyon boyutlama formülü için. | Örn. 1.000 USDT, işlem başına %0.5–1 risk. |
| B | **Futures'ta izin verilen maksimum kaldıraç?** | Risk devre kesici sınırı. | Başlangıçta **≤ 3x**. |
| C | **Parite evreni?** (BTC/ETH mi, top-20 mi, tüm USDT pariteleri tarama mı) | Tarama motoru ve veri hacmi. | MVP: **BTC/USDT + ETH/USDT**. |
| D | **AI katmanının kapsamı?** (veto/filtre, rejim tespiti, parametre önerisi) ve hangi model/bütçe? | AI katmanının yetki sınırı. | Önce sadece **veto/filtre** (en güvenli). Bkz. [AI Katmanı](../eca/ai.md). |
| E | **Coğrafya / regülasyon:** hangi ülke, hangi Binance hesabı (Global / TR)? | Uyumluluk ve API erişimi. TR'de MASAK, kripto düzenlemeleri; Binance ülke kısıtları değişebiliyor. | Kullanıcı netleştirmeli. |
| F | **GUI'yi kim kullanacak** (tek kullanıcı / çok kullanıcı)? Kimlik doğrulama gerekli mi? | Dashboard güvenliği. | Tek kullanıcı + güçlü auth + IP kısıtı. |

## İleri fazda gerekli

| # | Soru | Bağlam |
|---|------|--------|
| G | Ücretli veri kaynaklarına ne zaman geçilecek? | Bkz. [Veri Kaynakları](../veri/kaynaklar.md) — ücretsiz yığın + yükseltme yolu. |
| H | Hangi strateji aileleri önceliklendirilecek? | Backtest sonuçlarına göre veri-odaklı seçilecek. Bkz. [Strateji Aileleri](../strateji/aileler.md). |
| I | Vergi/işlem raporlaması hangi ülke kurallarına göre? | Trade journal bu raporları besleyebilir. |
| J | Çoklu hesap / başkalarının parasını yönetme ihtimali? | Bu, regülasyon açısından bambaşka bir yükümlülük (lisans vb.) getirir. |

## Regülasyon notu

Türkiye'de kripto varlık hizmet sağlayıcıları MASAK ve SPK düzenlemelerine tabidir; kişisel kullanım için bir botun hukuki durumu, başkalarının fonlarını yönetmekten farklıdır. Binance'in ülkeye özgü ürün/erişim kısıtları da zamanla değişir. **Bu bir hukuki tavsiye değildir** — netlik için bir hukuk uzmanına danışılmalıdır; bot yalnızca kaynak toplayıp belirsizlikleri işaretler, hukuki karar vermez.
