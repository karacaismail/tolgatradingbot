# Veri Kaynakları

> Kaynak: 4 ayrı veri araştırma brifi (piyasa/türev · on-chain · haber/makro · sentiment/X). **Fiyatlar sık değişir** — her rakam satın almadan önce canlı sayfadan doğrulanmalı (⚠️).

## Veri kaynağı hiyerarşisi (güven sırası)

Alt katman üst katmanı **tetiklemez, sadece zenginleştirir**:

- **Seviye 0 — Emir/hesap gerçeği (tek "kesin" veri):** Binance matching-engine, User Data Stream, açık emirler, fill'ler, bakiye/pozisyon, `exchangeInfo`. Bkz. [Binance API](../mimari/binance-api.md).
- **Seviye 1 — Kurumsal birincil:** Binance duyuru/changelog, Fed/FOMC, SEC, CFTC, BLS/BEA, varlığın vakfı/protokol yönetişimi.
- **Seviye 2 — Lisanslı piyasa/on-chain:** Kaiko, Coin Metrics, Glassnode, CryptoQuant… (timestamp/gecikme, düzeltme politikası, lisansla değerlendir).
- **Seviye 3 — X/Twitter & sosyal:** önce kurumlar, sonra doğrulanmış analistler. **Tek başına asla emir tetiklemez** (ECA-007).

## Önerilen ücretsiz-öncelikli başlangıç yığını

Kullanıcı bütçesi "ücretsiz-öncelikli" olduğu için MVP tamamen ücretsiz kaynaklarla kurulabilir:

| İhtiyaç | Ücretsiz kaynak | Not |
|---------|-----------------|-----|
| Fiyat/mum/emir | **Binance API** | Birincil, zaten gerekli |
| Türev (funding/OI/likidasyon) | Coinglass (sınırlı) / Binance fapi | Coinglass tam erişim ücretli |
| Piyasa geneli sentiment | **alternative.me Fear & Greed** (keysiz) | `GET api.alternative.me/fng/` |
| Sosyal sentiment | **LunarCrush Free** | Galaxy Score, AltRank, sosyal hacim |
| Makro/rejim | **FRED** (ücretsiz, 120 req/dk) | rates, yield curve, DXY, VIX |
| Backtest makro | **ALFRED** (ücretsiz, vintage) | look-ahead'i önler — kritik |
| Olay karartması | **FRED release dates** + hardcoded FOMC/NFP/CPI | ücretli takvim gerekmez |
| Haber | **Cointelegraph/CoinDesk RSS** + CryptoPanic Free | 1–5 dk polling |
| Pozisyonlanma | **CFTC COT (TFF)** ücretsiz | CME BTC/ETH leveraged funds |
| Kurumsal filing | **SEC EDGAR** ücretsiz | ETF akışları, MSTR/COIN 8-K |

## Ücretli yükseltme yolu (opsiyonel, sonraki faz)

### Piyasa & türev veri
| Sağlayıcı | En ucuz paid | En iyi olduğu | ⚠️ |
|-----------|--------------|---------------|-----|
| **CoinGecko** | ~$35/ay (Basic) | Ucuz spot/referans fiyat; free tier kullanılabilir | plan adları değişti |
| **Coinglass** | ~$29/ay (Hobbyist); ticari $299 | Funding/OI/likidasyon (en iyi değer) | free tier yok |
| **CoinMarketCap** | ~$29/ay (Hobbyist) | Referans/listeleme; kredi sistemi | değişken kredi |
| **Kaiko** | ~$9.5k–55k/yıl | Kurumsal/uyumluluk/indeksler | quote-based |

### On-chain
| Sağlayıcı | En ucuz paid | En iyi sinyal | ⚠️ |
|-----------|--------------|---------------|-----|
| **Glassnode** | ~$49/ay (annual); gerçek API ≈ Pro $999/ay | En derin SOPR/MVRV/NUPL, exchange flow | free tier fiilen yok |
| **CryptoQuant** | ~$29–39/ay | Exchange netflow, whale ratio, miner/stablecoin | iki fiyat yapısı live |
| **Santiment** | ~$49/ay (Pro) | On-chain + **sosyal** | Free/Pro'da **30 gün API gecikmesi**; gerçek zaman = Max $249 |
| **Nansen** | $49/ay (Pro) | Smart-money / whale cüzdan | kredi PAYG $10/1k |
| **Dune** | $75/ay (Analyst) | Özel SQL sinyalleri (DIY) | kredi tabanlı |
| **Arkham** | dashboard ücretsiz | Entity labeling / fund-flow | API custom/quote |

### Sentiment / sosyal
| Sağlayıcı | Fiyat | Not |
|-----------|-------|-----|
| **alternative.me F&G** | Ücretsiz, keysiz | market-wide baseline |
| **CoinMarketCap F&G** | Ücretsiz keysiz (⚠️ 2026 yeni) | market-wide |
| **LunarCrush** | Free → ~$24 → ~$240/ay; kredi API ~$30/ay | Galaxy Score, sosyal hacim, sentiment |
| **Santiment** | Free/Pro'da 30g gecikme; gerçek zaman Max $249/ay | on-chain + sosyal |
| **The TIE** | Kurumsal, fiyat gizli | hedge fund seviyesi, atla |

### X / Twitter API — neden aggregator kullan
!!! warning "X API artık pahalı ve model değişti (2026)"
    X, 2026'da sabit tier'lardan **pay-per-use**'a geçti. Legacy: Basic **$200/ay** (sadece 15k okuma/ay — yetersiz), Pro **$5.000/ay**, Enterprise **$42.000+/ay**. Pay-per-use'da 3. taraf gönderi okuma ~$0.005/istek. Küçük bir bot için **doğrudan X API yerine LunarCrush/Santiment gibi aggregator** kullanın — X'i (+ Reddit, Telegram) zaten toplarlar, bot/spam temizlerler, bitmiş sinyali (Galaxy Score, sosyal hacim) onlarca dolara verirler. Ayrıca ToS/rate-limit yükünü de üstlenirler.

## Makro & backtest bütünlüğü (kritik)

- **FRED** (ücretsiz, key ile 120 req/dk): 800k+ seri — rates (DFF, DGS10, DGS2), yield curve (T10Y2Y), enflasyon (CPIAUCSL, PCEPI), istihdam (UNRATE, PAYEMS), M2, finansal stres (NFCI), DXY (DTWEXBGS), VIX (VIXCLS). **Rejim tespiti** için makro omurga.
- **ALFRED** (ücretsiz, vintage): FRED **güncel/revize** değer verir; makro seriler aylarca revize edilir. Backtest'te bugünün revize değerini geçmiş bir tarih için okumak **look-ahead bias**'tır. ALFRED "o an ne biliniyordu"yu verir (`realtime_start/end`, `vintage_dates`). **Makro-sürücülü her strateji ALFRED vintage'ı ile backtest edilmeli.** Arşiv ~2006'dan. Python: `fredapi`.
- **FOMC (2026, 8 toplantı):** 27–28 Oca, 17–18 Mar, 28–29 Nis, 16–17 Haz, 28–29 Tem, 15–16 Eyl, 27–28 Eki, 8–9 Ara. Karar 14:00 ET. Bota **karartma penceresi** olarak hardcode + yıllık yenile.
- **SEC EDGAR** (ücretsiz, **10 req/s**, User-Agent zorunlu): 8-K/S-1/13F/N-1A — ETF akışları, hazine-şirketi BTC pozisyonları.
- **CFTC COT** (ücretsiz, Socrata API): CME BTC/ETH futures **TFF** raporunda; **Leveraged Funds** net pozisyonu = hedge fund kalabalıklaşma göstergesi. ⚠️ Salı verisi Cuma 15:30 ET yayınlanır → **~3 gün gecikme** (look-ahead yok).

Kaynaklar: [FRED API](https://fred.stlouisfed.org/docs/api/fred/) · [ALFRED](https://alfred.stlouisfed.org/) · [FOMC takvimi](https://www.federalreserve.gov/monetarypolicy/fomccalendars.htm) · [SEC EDGAR erişim](https://www.sec.gov/search-filings/edgar-search-assistance/accessing-edgar-data) · [CFTC COT kılavuzu](https://publicreporting.cftc.gov/stories/s/User-s-Guide/p2fg-u73y/) · [alternative.me F&G](https://api.alternative.me/fng/) · [LunarCrush API](https://lunarcrush.com/developers/api/overview)

## Sürümlü kaynak sicili (sabit influencer listesi YERİNE)

Takipçi sayısı güvenilirlik değildir. Her kaynak sicile şu şemayla işlenir ve **rolü sınırlanır**:

```yaml
source_id:
platform:            # x / rss / api
account:
entity:              # kurum/kişi
role:                # exchange | regulator | foundation | analyst
official_url:
identity_verified_at:
reliability_score:
manipulation_risk:
allowed_for: [research, risk_alert]   # trade_trigger ASLA
direct_trade_trigger: false
```

!!! danger "Kaynak değerlendirme ilkeleri"
    - ✅ **Kurumsal araştırma** (Glassnode, Messari, Kaiko, The Block) — veri paylaşır, satış yapmaz.
    - ⛔ İsimsiz, "%1000 kazandırdım" ekran görüntüsü paylaşan, ücretli grup satan, "hemen al" aciliyeti yaratan hesaplar.
    - **Kural:** bir sinyali bota sokmadan önce **geriye dönük test et** — "bu kaynak 6 ay sinyal verseydi PnL ne olurdu?" Doğrulanamayan kaynağı otomatik işleme bağlama.
