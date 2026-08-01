# Framework Seçimi

> Kaynak: 2025–2026 açık kaynak framework araştırma brifi.

## Karşılaştırma

| Framework | Dil | En iyi olduğu | Binance | Backtest | Canlı olgunluk | Lisans |
|-----------|-----|---------------|---------|----------|----------------|--------|
| **Freqtrade** | Python | Uçtan uca crypto bot (backtest→canlı) | Spot+Futures+Margin (ccxt) | İyi (bazı lookahead riski) | **Yüksek** (dry-run, WebUI, Telegram, REST) | GPLv3 |
| **Jesse** | Python | Araştırma + temiz DSL | Spot+Futures | **Çok iyi** | Orta–yüksek (canlı lisanslı olabilir) | MIT |
| **Hummingbot** | Python | **Market making / arbitraj** | Spot+Perp, CEX+DEX | Zayıf (MM odaklı) | **Çok yüksek** (prod MM) | Apache-2.0 |
| **NautilusTrader** | **Rust+Python** | Kurumsal, düşük-latency, deterministik | Spot+Futures, güçlü adapter | **En yüksek** (research=live) | Yüksek | LGPL-3.0 |
| **Backtrader** | Python | Klasik event-driven backtest | ccxt (topluluk) | İyi | Düşük (**bakım durgun**) | GPLv3 |
| **vectorbt** | Python (Numba) | **Vektörize hızlı parametre taraması** | Yok (veri getir) | Hız şampiyonu, gerçekçilik düşük | Yok | Apache-2.0 (PRO ticari) |
| **QuantConnect/Lean** | **C#+Python** | Çok-varlık kurumsal | Spot+Futures | Çok iyi | Yüksek | Apache-2.0 |

Kaynaklar: [Freqtrade](https://github.com/freqtrade/freqtrade) · [Jesse](https://github.com/jesse-ai/jesse) · [Hummingbot](https://github.com/hummingbot/hummingbot) · [NautilusTrader](https://github.com/nautechsystems/nautilus_trader) · [vectorbt](https://github.com/polakowo/vectorbt) · [Lean](https://github.com/QuantConnect/Lean)

## Kim neyi en iyi yapıyor?

- **En gerçekçi backtest + research=live:** NautilusTrader (deterministik, order-book, tek motor).
- **En hızlı parametre taraması:** vectorbt (~1000 kombinasyonu tek klasik backtest süresinde) — ama fill gerçekçiliği düşük, triage aracı.
- **Hazır operasyonel crypto bot** (dry-run, protections, stop-loss, WebUI, REST): Freqtrade.
- **Market making / order lifecycle olgunluğu:** Hummingbot.

## Karar: ccxt üzerine custom (framework'e fork değil)

**Bağlam:** spot+futures+margin, **custom ECA veri-DSL**, **web GUI**.

!!! success "Tavsiye"
    **"ccxt üzerinde custom çekirdek, olgun framework'lerin desenlerini ödünç alarak."** Ne sıfırdan her şey, ne de tek framework'e kilit.

    1. **Execution/borsa katmanı = ccxt** (spot+futures+margin unified), kendi `ExecutionHandler` adapter'ın arkasına.
    2. **Mimari şablon = NautilusTrader'ı kopyala** (satın alma değil, desen): MessageBus pub/sub, `DataEngine`/`ExecEngine`/`RiskEngine` ayrımı, adapter arayüzü, deterministik event kuyruğu.
    3. **ECA motoru = custom** — asıl katma değerin. YAML DSL + expression-tree + priority/specificity çakışma çözümü. Bağımsız, test edilebilir çekirdek.
    4. **Backtest research = vectorbt yan araç** (hızlı tarama) + kendi event-driven simülatörün (gerçekçi fill).
    5. **Web GUI = custom** (FastAPI + React), Freqtrade'in FreqUI/REST tasarımını referans al.

!!! warning "Neden Freqtrade/Nautilus üstüne *fork* değil?"
    Custom ECA veri-DSL + custom WebGUI, iki framework'ün opinionated iç modelleriyle çatışır; onları "framework olarak" kullanmak zamanla onlara karşı savaşmaya döner. Bunun yerine **kod/desen kaynağı** olarak madencilik yapın: Freqtrade'in dry-run + exchange soyutlaması + protections; NautilusTrader'ın message bus + risk engine.

    İstisna: hedef **kurumsal/HFT-latency** olsaydı doğrudan NautilusTrader üstüne inşa önerilirdi. Retail spot+futures+margin ECA botu + web GUI için ccxt-tabanlı custom en iyi denge.

**Tek cümle:** ECA motorunu ve GUI'yi custom yaz, execution için ccxt kullan, mimari iskelet için NautilusTrader'ı ve operasyon desenleri için Freqtrade'i referans al.
