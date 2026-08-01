# Risk Yönetimi

> Kaynak: risk & backtest araştırma brifi. **Eğitim/mühendislik amaçlıdır; yatırım tavsiyesi değildir.** Tüm parametreler örnektir, kendi verinizle valide edin.

Pozisyon boyutlandırma "ne kadar açacağım" sorusudur ve uzun vadeli sermaye büyümesini sinyal kalitesinden daha çok etkiler.

## 1. Pozisyon boyutlandırma

### 1.1 Sabit kesirli (% risk per trade)
Her işlemde sermayenin sabit bir yüzdesini **riske** atarsınız.

```
Riske edilen tutar R$ = Sermaye × risk%          (tipik %0.5–2)
Stop mesafesi = |Giriş − Stop|
Pozisyon (adet) = R$ / Stop mesafesi
```

**Örnek:** Sermaye 10.000, risk %1 → R$=100. Giriş 60.000, Stop 58.500 → mesafe 1.500 → pozisyon 100/1.500 = **0.0667 BTC**. Stop tetiklenirse kayıp ≈ %1.

- **Avantaj:** kayıp serilerinde pozisyon otomatik küçülür (anti-martingale).
- Kaynak: [backtestbase — risk per trade](https://www.backtestbase.com/education/how-much-risk-per-trade)

### 1.2 Volatilite / ATR tabanlı
Sabit yüzde riski korurken stop mesafesini ATR'ye bağlarsınız.

```
Stop mesafesi = k × ATR              (k tipik 1.5–3)
Pozisyon = (Sermaye × risk%) / (k × ATR)
```

**Örnek:** R$=100, ATR=800, k=2 → mesafe 1.600 → 0.0625 BTC. ATR 1.200'e çıkarsa pozisyon 0.0417 BTC'ye düşer, dolar riski ~100 sabit.
Kaynaklar: [QuantifiedStrategies](https://www.quantifiedstrategies.com/volatility-based-position-sizing/) · [LuxAlgo](https://www.luxalgo.com/blog/5-position-sizing-methods-for-high-volatility-trades/)

### 1.3 Kelly kriteri ve neden fractional Kelly
```
f* = W − (1−W)/R        (W = kazanma oranı, R = ort.kazanç/ort.kayıp)
```
**Örnek:** W=0.55, R=2 → f* = 0.325 (tam Kelly %32.5, kripto için aşırı agresif).

!!! danger "Asla tam Kelly kullanmayın"
    Kelly, W ve R'nin kesin bilindiğini varsayar; gerçekte bunlar hatalı tahmindir → tam Kelly **aşırı bahis ve iflas**. Yarım Kelly, optimal büyümenin ~%75'ini korurken volatiliteyi ~yarıya indirir. Profesyoneller **yarım/çeyrek Kelly** kullanır.

!!! tip "Bot tasarım notu"
    Pozisyon boyutu = `min(fixed-fractional, ATR-based, fractional-Kelly cap)` — en muhafazakâr sonucu al. Kelly'yi asla üst-sınır (cap) olmadan kullanma.

Kaynaklar: [Coriva Kelly](https://coriva.eu.org/en/kelly-criterion-position-sizing/) · [Astute Investor's Calculus](https://astuteinvestorscalculus.com/kelly-criterion-position-sizing/)

## 2. Stop-loss / Take-profit / Trailing

- **ATR stop:** `Giriş ∓ k×ATR`; k=1.5 sıkı, 3 geniş. Volatiliteyle uyumlu, gürültü stop'unu azaltır.
- **R:R:** `R:R = TP mesafesi / SL mesafesi`. Pozitif beklenti için `W > 1/(1+R:R)` (R:R=2 → breakeven %33.3).
- **Trailing:** fiyat lehe gittikçe stop takip eder (Chandelier `en_yüksek − k×ATR`); whipsaw'da erken çıkabilir.

!!! danger "Stop kayıp seviyesini GARANTİ ETMEZ"
    Stop yalnız **tetiklenme** fiyatını tanımlar, **gerçekleşme (fill)** fiyatını değil.
    - **Gap:** likidasyon kaskadı/haber şokunda fiyat stop'u atlayarak çok kötü seviyede dolar.
    - **Slippage:** stop genelde market emrine döner; ince order book'ta beklenenden kötü dolar.
    - **Stop-limit riski:** fiyat limitin ötesine geçerse emir hiç dolmaz, pozisyon açık kalır.
    - **Sonuç:** riske edilen %1 bir **beklenti**dir, garanti değil; risk motoru worst-case slippage'ı modellemeli.

Kaynak: [LuxAlgo ATR stops](https://www.luxalgo.com/blog/5-atr-stop-loss-strategies-for-risk-control/)

## 3. Portföy riski & bağımsız risk motoru

Kritik ilke: **Risk motoru, stratejiden bağımsız çalışan ve HER emri VETO edebilen ayrı bir katmandır.**

| Limit | Tipik | Amaç |
|-------|-------|------|
| **Max günlük kayıp** | %2–5 | Eşikte tüm yeni işlemler durur (kill-switch) |
| **Max drawdown circuit breaker** | tepe-dip %10–20 | Eşikte botu durdur/küçült |
| **Max açık pozisyon** | 3–5 | Aşırı yayılma + korelasyon şoku |
| **Korelasyon limiti** | \|ρ\| < 0.7 | Yüksek korele varlıklarda eşzamanlı büyük pozisyonu engelle |
| **Kaldıraç tavanı** | ≤ 3–5x | Toplam nominal / sermaye |
| **Konsantrasyon** | tek varlık ≤ %X | Tek pozisyonda toplanmayı önle |

!!! warning "Korelasyon stres anında en kritik korumadır"
    Kripto düşüşlerinde korelasyonlar **1'e yaklaşır** (her şey birlikte düşer); ρ=1 ise iki pozisyon tek büyük pozisyona eşdeğerdir.

**Risk motoru iskeleti:**

```python
def risk_engine.validate(order, portfolio_state) -> Decision:
    if daily_pnl <= -max_daily_loss:            return VETO   # gün kapandı
    if current_drawdown >= max_dd:              return VETO   # circuit breaker
    if open_positions >= max_positions:         return VETO
    if abs(corr(order.symbol, existing)) > 0.7: return VETO / SHRINK
    if total_notional + order.notional > lev_cap*equity: return SHRINK
    if order.risk_pct > per_trade_cap:          return SHRINK
    return ACCEPT
```

- **Stateful & idempotent:** günlük PnL/drawdown/pozisyonlar **gerçek borsa durumundan** (reconciliation) beslenir, strateji hafızasından değil.
- **Fail-safe:** belirsizlikte varsayılan karar **VETO** (fail-closed).
- **Kill-switch:** eşik aşılınca tüm açık emirleri iptal + yeni emirleri blokla.
- **Bağımsız test:** risk motoru saf fonksiyon olarak birim testine tabi; strateji koduyla iç içe geçmez.

## 4. Performans metrikleri

| Metrik | Formül | "İyi" | Sınır |
|--------|--------|-------|-------|
| **Sharpe** | (R−Rf)/σ | >1 iyi, >2 çok iyi | Normal dağılım varsayar; fat-tail'i yoksayar; çok denemede şişer |
| **Sortino** | (R−Rf)/σ_downside | Sharpe'tan yüksek | Sadece aşağı volatiliteyi cezalandırır (daha anlamlı) |
| **Calmar** | Yıllık getiri / MaxDD | >3 güçlü | Tek en kötü olaya bağlı |
| **Max Drawdown** | max tepe-dip % | küçük iyi | Sadece geçmişin en kötüsü |
| **Profit Factor** | Σkazanç / \|Σkayıp\| | >1.5 sağlam | Az sayıda büyük kazançtan şişebilir |
| **Win Rate** | kazanan/toplam | tek başına anlamsız | Yüksek win + büyük kayıp = negatif beklenti |
| **Expectancy** | (W·ort.kazanç) − (L·ort.kayıp) | >0 zorunlu | Dağılımı gizler |

!!! note "Deflated Sharpe Ratio (DSR)"
    Çok sayıda strateji/parametre denendiğinde Sharpe **seçilim yanlılığı** için düzeltilmelidir. Tek yüksek Sharpe, yüzlerce varyanttan seçildiyse neredeyse anlamsızdır. Bailey & López de Prado. Kaynak: [SSRN DSR](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=2460551)

Devamı → [Backtest & Getiri Gerçekliği](backtest.md).
