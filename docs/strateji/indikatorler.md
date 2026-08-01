# Teknik İndikatörler

> Kaynak: strateji/indikatör araştırma brifi. **Eğitim amaçlıdır, yatırım tavsiyesi değildir.** Her eşik bir başlangıç hipotezidir.

İndikatörler ECA motorunda "Condition" (koşul) blokları olarak formüle edilir. Aşağıdaki her indikatör için: formül, tipik parametre, ECA kullanımı ve tipik tuzak.

## Momentum / osilatör

### RSI — Relative Strength Index
- **Formül:** `RS = OrtKazanç(N)/OrtKayıp(N)` → `RSI = 100 − 100/(1+RS)` (Wilder yumuşatması).
- **Parametre:** N=14 (scalp 7–9, yavaş 21).
- **Eşik:** >70 aşırı alım, <30 aşırı satım; kripto/güçlü trendde 80/20 daha az yanlış. Merkez 50: >50 boğa, <50 ayı eğilimi.
- **ECA:** mean-reversion `RSI(14) crosses_above 30 → long`; trend teyidi `RSI(14) > 50 AND trend_up → allow_long`.
- **Tuzak:** güçlü trendde haftalarca >70 kalır — tek başına ters işlem intihar. **Divergence** (fiyat yeni tepe, RSI değil) tükeniş sinyali.
- Kaynak: [Fidelity RSI](https://www.fidelity.com/learning-center/trading-investing/technical-analysis/technical-indicator-guide/RSI)

### MACD
- **Formül:** `MACD = EMA(12) − EMA(26)`; `Signal = EMA(9,MACD)`; `Histogram = MACD − Signal`.
- **Parametre:** (12,26,9); hızlı (5,35,5).
- **ECA:** boğa `MACD crosses_above Signal`; sıfır çizgisi (MACD>0) trend teyidi; histogram yön değişimi erken sinyal.
- **Tuzak:** range'de sürekli whipsaw; gecikmeli. En iyi bir trend filtresiyle (EMA/ADX).
- Kaynak: [StockCharts MACD](https://chartschool.stockcharts.com/table-of-contents/technical-indicators-and-overlays/technical-indicators/macd-histogram)

## Trend / ortalama

### EMA / SMA ve crossover
- **EMA:** `EMA_t = P_t·k + EMA_{t−1}·(1−k)`, `k=2/(N+1)` — son fiyata daha duyarlı.
- **Parametre:** kısa 9/20/21, orta 50, uzun 100/200. Golden/Death Cross = 50 EMA'nın 200 EMA'yı kesmesi.
- **ECA:** `EMA(9) crosses_above EMA(21) → long`; `close > EMA200` makro trend filtresi.
- **Tuzak:** range'de sürekli yanlış tetik. EMA hızlı ama gürültülü, SMA yumuşak ama geç.

### ADX — Average Directional Index (trend GÜCÜ, yön değil)
- **Eşik:** <20 trend yok/range; >25 güçlü trend; >40 çok güçlü. +DI>−DI yukarı yön.
- **ECA (rejim şalteri!):** `IF ADX>25 THEN trend-takip ELSE mean-reversion`. Trend botlarında en değerli filtre.
- **Tuzak:** gecikmeli, yön vermez — açma/kapama şalteri, giriş sinyali değil.
- Kaynak: [TradingView ADX](https://www.tradingview.com/support/solutions/43000589099-average-directional-index-adx/)

### Supertrend — ATR tabanlı trend/stop
- **Parametre:** ATR 10, çarpan 3. Küçük çarpan → sıkı stop, çok flip.
- **ECA:** `Supertrend flips green → long / stop = Supertrend çizgisi`. Trailing stop olarak da.
- **Tuzak:** yatayda sürekli renk değiştirir (whipsaw); trendde mükemmel.
- Kaynak: [TradingView Supertrend](https://www.tradingview.com/support/solutions/43000634738-supertrend/)

## Volatilite / risk

### ATR — Average True Range
- **Formül:** `TR = max(H−L, |H−C_prev|, |L−C_prev|)`; `ATR = RMA(TR,14)`. Yön vermez, sadece volatilite.
- **ECA kullanımı (risk):**
  - Stop: `stop = giriş − k·ATR`, k=1.5–3.
  - **Volatilite-ayarlı boyut:** `size = (hesap·risk%)/(k·ATR)` — volatilite artınca pozisyon otomatik küçülür.
  - Trailing: Chandelier Exit = `en_yüksek − 3·ATR`.

### Bollinger Bands
- **Formül:** `Orta = SMA(20)`, `Üst/Alt = SMA(20) ± 2σ(20)`; `%B`, `Bandwidth`.
- **ECA:** mean-reversion (alt banda değince long); breakout ("Squeeze": Bandwidth dibe inince sıkışma, taşan kapanış patlama).
- **Tuzak:** trendde fiyat üst banda "yapışır" (band walking); squeeze breakout yönü belirsiz.
- Kaynak: [StockCharts Bollinger](https://chartschool.stockcharts.com/table-of-contents/technical-indicators-and-overlays/technical-indicators/bollinger-bands)

## Fiyat/hacim referansı

### VWAP
- **Formül:** `Σ(TypicalPrice·Hacim)/Σ(Hacim)`, gün başı sıfırlanır (intraday).
- **ECA:** fiyat > VWAP boğa; pullback VWAP'a dönünce trend yönünde giriş.
- **Tuzak:** intraday araç; günlük üstünde anchored-VWAP. Düşük hacimde gürültülü.

### Donchian Channels
- **Formül:** `Üst = en_yüksek(N)`, `Alt = en_düşük(N)`; N=20 (Turtle: 20 giriş / 10 çıkış).
- **ECA:** `close > Donchian_Üst(20) → breakout_long`.
- **Tuzak:** range'de her iki uçta sahte kırılım.

### OBV / Hacim
- **ECA:** breakout teyidi `close > direnç AND volume > 1.5·ort_hacim(20) → geçerli kırılım`. OBV divergence zayıflık.
- **Tuzak:** hacim borsaya özgü; spot vs perp hacmi karıştırma.

Kütüphaneler: [pandas-ta](https://github.com/twrobel/pandas-ta) · [TA-Lib](https://ta-lib.org/)
