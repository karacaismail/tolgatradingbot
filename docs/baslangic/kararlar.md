# Alınan Kararlar

Bu sayfa, projenin üzerine kurulduğu kesinleşmiş kararları toplar. İki kaynaktan gelir: **(A)** kullanıcının doğrudan verdiği kararlar, **(B)** iyi bilinen best-practice olduğu için önerilen ve kabul edilen varsayılanlar.

## A. Kullanıcının verdiği kararlar

| # | Konu | Karar |
|---|------|-------|
| 1 | ECA modeli | **Event–Condition–Action** — **tam otomatik, insan onayı olmadan, AI destekli.** Kural motoruna bir AI karar/doğrulama katmanı eklenir. |
| 2 | Yürütme & para | **4 mod seçilebilir:** `testnet` · `paper` · `semi-auto` (onaya sunar) · `full-auto`. Kullanıcı modu GUI'den değiştirir. |
| 3 | Piyasa kapsamı | **Hepsi:** Spot + Futures (kaldıraçlı) + Margin. Fazlama: **Spot (MVP) → Futures → Margin**. |
| 4 | İndikatör/veri | **RSI, MACD ve diğer TA verileri** referans; veriler "eşleşince" (confluence) işlem. |
| 5 | Arayüz | **GUI zorunlu** (web dashboard: canlı grafik, kural editörü, PnL paneli, mod seçici, kill-switch). |
| 6 | Host | **Hetzner** (VPS, 7/24, Docker). |
| 7 | Veri bütçesi | **Saf teknik analiz + ücretsiz veriler**; ücretli veriler opsiyonel, sonraki faz. |
| 8 | Kâr hedefi | Günlük %2 = **hedef/dur-kâr ayarı**, garanti minimum DEĞİL. Bkz. [Getiri Gerçekliği](../risk/backtest.md#getiri-gercekligi). |
| 9 | Gözlem | **Derin loglama + filtreli log viewer + canlı ECharts kâr/zarar monitoring + duygusuz feedback** (erken zarar kesme / uzun vade önerisi). Bkz. [Gözlem](../gozlem/loglama.md). |

## B. Önerilen ve kabul edilen varsayılanlar

| Karar | Ne | Neden |
|-------|-----|-------|
| K1 | **Önce backtest + paper + shadow, sonra testnet, en son gerçek para.** | Test edilmemiş strateji = kesin zarar. Pazarlık konusu değil. |
| K2 | **ECA Kural Motoru + Event Bus.** Kurallar koddan ayrı, **YAML DSL** ile. | Strateji değiştirmek için kod değiştirmeye gerek kalmaz. |
| K3 | **Zorunlu, bağımsız Risk Motoru** (her emir buradan geçer, veto edebilir). | Botlar en çok risk kontrolü yokluğundan patlar. |
| K4 | **API anahtarı: sadece trade izni; para çekme (withdrawal) KAPALI; IP whitelist açık.** | Anahtar sızsa bile para çekilemez. |
| K5 | **Modüler katmanlı mimari** (bağlantı/veri/strateji/risk/yürütme/gözlem ayrık). | Test edilebilir, genişletilebilir. |
| K6 | **Her şey loglanır + trade journal append-only DB.** | Neyin neden yapıldığını görmeden strateji iyileştirilemez. |
| K7 | **Sırlar `.env` + secret store; repoya asla girmez.** | Güvenlik hijyeni. |
| K8 | **Docker + docker-compose.** | 7/24 çalışma ve taşınabilirlik. |
| K9 | **Kill-switch (motordan bağımsız) + günlük maksimum zarar limiti.** | Bot çıldırırsa otomatik durur. |
| K10 | **Yürütme: ccxt üzerine custom çekirdek;** mimari şablon NautilusTrader'dan, operasyon desenleri Freqtrade'den ödünç. | Bkz. [Framework Seçimi](../mimari/framework.md). |

## Teknoloji yığını (özet)

| Katman | Seçim |
|--------|-------|
| Dil | Python 3.12+ |
| Borsa bağlantısı | **ccxt** (birincil) + resmî **binance-connector** (doğrulama) |
| İndikatör | pandas-ta / TA-Lib |
| Backtest | vectorbt (hızlı tarama) + özel event-driven motor (gerçekçi fill) |
| Veritabanı | SQLite (başlangıç) → PostgreSQL + TimescaleDB |
| Kural DSL | YAML |
| GUI backend | FastAPI (REST + WebSocket) |
| GUI frontend | React + Vite + Tailwind; grafikler **ECharts** / TradingView Lightweight Charts |
| AI katmanı | Claude API (filtre/veto/rejim) |
| Bildirim | Telegram + e-posta |
| Paketleme / Host | Docker + docker-compose · **Hetzner VPS** |

Detaylı gerekçeler için [Katmanlı Mimari](../mimari/genel.md) ve [Framework Seçimi](../mimari/framework.md).
