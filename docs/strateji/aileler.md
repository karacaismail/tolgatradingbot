# Strateji Aileleri, Rejim & Konfluans

> Kaynak: strateji araştırma brifi. **Eğitim amaçlıdır.** Hangi ailenin *senin* paritelerinde tuttuğuna [backtest](../risk/backtest.md) ile karar verilir.

## Strateji aileleri

| Aile | Giriş/çıkış | En iyi rejim | Tipik başarısızlık |
|------|-------------|--------------|--------------------|
| **Trend-following** | EMA crossover / Supertrend / Donchian breakout; trailing ATR stop; ADX>25 filtresi | Güçlü tek yönlü trend | Yatayda sürekli whipsaw, çok küçük zarar |
| **Mean-reversion** | RSI<30 veya alt Bollinger'da long; ortaya dönünce çık | Range/yatay, düşük-orta volatilite | Trend başlayınca "bıçak yakalama", tek büyük zarar |
| **Momentum / Breakout** | Konsolidasyon sonrası direnç kırılımı + hacim teyidi; stop kırılım altı | Volatilite genişlemesi, katalizör | Sahte kırılım (fakeout), stop avı |
| **Grid trading** | Aralığı ızgaralara böl; altta al, üstte sat; yön-nötr | Yatay, oynak ama yönsüz | Range kırılınca tek yönde birikim → büyük zarar/likidasyon |
| **DCA** | Sabit aralıkta/düşüşte eşit tutar al; ortalama düşür | Uzun vadeli birikim/boğa | Kalıcı düşüşte "düşen bıçağı" ortalama, sermaye kilitlenir |
| **Funding-rate arbitrajı** (delta-nötr) | Spot AL + perp SHORT (funding+); yön riski ≈ 0, funding topla | Yüksek pozitif funding | Funding tersine döner; short bacak likidasyonu; maliyet > funding |
| **Market making** | İki taraflı limit; spread kazan; envanteri nötr tut | Yüksek hacim, dar spread, düşük vol. | Volatilite patlaması → adverse selection |

### Funding-rate arbitrajı (detay)
Perp funding pozitifse long'lar short'lara öder. **Spot'ta X al + aynı miktar perp short** → net delta ≈ 0; gelir = toplanan funding. Gözlemlenen ~%8–40 APR (garanti değil). **Riskler:** funding tersine dönmesi; delta-nötr ≠ likidasyon-korumalı (short'un likidasyon fiyatı var); işlem maliyeti funding'i aşabilir; borsa riski.
Kaynaklar: [Kraken funding arb](https://www.kraken.com/learn/futures-trading-funding-rate-arbitrage) · [Sharpe.ai](https://www.sharpe.ai/learn/funding-rate-arbitrage)

## Piyasa rejimi tespiti

Rejim sınıflandırması, hangi strateji ailesinin aktif olacağını seçer — botun en kritik üst-katman kararı.

- **ADX ile:** `ADX>25 → trend rejimi` (trend-follow aç); `ADX<20 → range` (mean-reversion/grid aç). Basit, sağlam şalter.
- **ATR / Bollinger Bandwidth ile:** yüksek → yüksek volatilite (pozisyon küçült, stopları genişlet); çok düşük Bandwidth → squeeze/sakinlik (breakout bekle).
- **Hurst üsteli (H):** R/S analizi, 100–200 barlık pencere. **H<0.5 mean-reverting**, H≈0.5 rastgele yürüyüş, **H>0.5 trendli**. H>0.5 → trend-takip, H<0.5 → mean-reversion.
- **İleri:** HMM/Markov rejim modelleri, choppiness index.

Kaynaklar: [Macrosynergy Hurst](https://macrosynergy.com/research/detecting-trends-and-mean-reversion-with-the-hurst-exponent/) · [PyQuantLab rejim filtresi](https://pyquantlab.medium.com/enhancing-trading-strategies-with-a-hurst-based-regime-filter-ac6639be43cf)

## Çoklu zaman dilimi uyumu (Multi-Timeframe Confluence)

İlke: **Yüksek zaman dilimi (HTF) yönü belirler → düşük zaman dilimi (LTF) girişi zamanlar.**

**Örnek (Daily trend + 1h giriş):**

1. **Günlük (HTF filtre):** `close > EMA(200)` VE `ADX>25` VE `+DI>−DI` → makro yön yukarı. Yalnız long'a izin.
2. **1h (LTF tetik):** günlük yukarı iken fiyat EMA(20)'ye pullback + `RSI(14) crosses_above 40` VE `MACD crosses_above Signal` → long.
3. **Stop/hedef:** stop = giriş − 2·ATR(1h); trailing = 1h Supertrend; çıkış = `daily close < EMA200`.

ECA olarak: HTF koşulu bir "gate" flag üretir; LTF sinyali yalnız gate açıkken tetikler. Yaygın oran 3:1/4:1 (Daily→4H→1H).

## Sinyal konfluansı (false-signal azaltma)

Tek indikatör = çok gürültü. **Farklı kategorilerden** indikatörleri AND'lemek yanlış sinyali düşürür (ama sinyal sıklığını da azaltır — aşırıya kaçınca hiç işlem çıkmaz = overfitting riski).

**Long şablonu:**

- Trend filtresi: `close > EMA(200)` (yön izni)
- Momentum teyidi: `MACD > Signal`
- Zamanlama: `RSI(14) crosses_above 40–50`
- Hacim (opsiyonel): `volume > 1.5·SMA_vol(20)`
- Rejim şalteri: yalnız `ADX>25` iken aktif

!!! warning "Aynı ailenin iki indikatörü konfluans değildir"
    RSI + Stochastic ikisi de osilatör → aynı bilgiyi iki kez sayarsın. **Trend + momentum + volatilite/hacim** gibi farklı bilgi eksenlerini birleştir. Her ek koşul örneklem sayısını düşürür; istatistiksel anlamlılık için yeterli işlem kalmalı.

## Neden hiçbir şey her rejimde çalışmaz

Trend-takip trendde kazanır, range'de whipsaw'la ölür; mean-reversion tam tersi. Bir stratejinin edge'i bulunduğu **rejime koşulludur** — bu yüzden rejim tespiti stratejinin kendisi kadar önemli. "Her koşulda kazanan" bir kural bulduysanız, muhtemelen geçmişe **overfit** etmişsinizdir. BTC'de çalışan RSI eşiği düşük likiditeli bir altcoin'de aynı davranmaz — **her strateji her paritede ayrı kalibre + doğrulanmalı.**
