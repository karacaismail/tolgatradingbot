# Binance API Entegrasyonu

> Kaynak: 2025–2026 araştırma brifi. Sık değişen alanlar ⚠️ ile işaretlidir; canlıya almadan önce [resmî dokümandan](https://developers.binance.com/en) doğrulayın.

## 1. Ürün API yüzeyleri — AYRI yüzeylerdir

Binance her ürünü **farklı base host + farklı path prefix + kısmen farklı semantik** ile sunar. "Tek Binance client" yanılgısına düşmeyin — ortak çekirdek + ürün başına adapter şart.

| Ürün | REST base | Prefix | Not |
|------|-----------|--------|-----|
| **Spot** | `api.binance.com` (yük dengeli `api1..api4`; sadece market-data: `data-api.binance.vision`) | `/api/v3` | Referans yüzey. |
| **USDⓈ-M Futures** | `fapi.binance.com` | `/fapi/v1`, `/fapi/v2` | USDT/USDC teminatlı. Pozisyon modu, kaldıraç, funding. |
| **COIN-M Futures** | `dapi.binance.com` | `/dapi/v1` | Coin teminatlı. |
| **Margin** | `api.binance.com` (spot host) | `/sapi/v1` | Cross & Isolated; farklı endpoint ailesi + farklı ağırlık. |
| **Options (EAPI)** | `eapi.binance.com` | `/eapi/v1` | Avrupa-tipi opsiyon. |

**Mimari sonuç:** Ortak çekirdek (HMAC-SHA256/Ed25519 imzalama, `timestamp`+`recvWindow`, rate-limit muhasebesi, retry/reconciliation, WS yaşam döngüsü, `exchangeInfo` filtre motoru) **+** ürün başına adapter (base URL, path, emir şeması, filtre çözümü, stream isimleri).

Kaynaklar: [Spot REST API](https://developers.binance.com/docs/binance-spot-api-docs/rest-api) · [USDⓈ-M General Info](https://developers.binance.com/docs/derivatives/usds-margined-futures/general-info) · [Developer Center](https://developers.binance.com/en)

## 2. Testnet / Demo Trading

| Ortam | REST | WebSocket |
|-------|------|-----------|
| **Spot Testnet** | `testnet.binance.vision/api` | `wss://testnet.binance.vision/ws` |
| **Futures Demo** ⚠️ | Klasik `testnet.binancefuture.com`; yeni `demo-fapi.binance.com` | `wss://demo-fstream.binance.com` |

!!! warning "Kritik uyarılar"
    - **Prod key ≠ testnet key.** Testnet key'i prod host'ta `Invalid Api-Key ID` verir (yaygın hata: futures testnet çağrısının canlı spot host'a yönlenmesi).
    - ⚠️ Binance futures testnet'i "**Demo Trading**" adıyla yeniden markaladı ve host adlarını değiştirmeye başladı. **Base URL'i config'ten okuyun, koda gömmeyin.**
    - Testnet likiditesi gerçekçi değildir; filtre/reconciliation mantığını doğrular ama slippage/latency testine uygun değildir.

Kaynaklar: [Spot Testnet](https://developers.binance.com/docs/binance-spot-api-docs/testnet) · [Derivatives Quick Start](https://developers.binance.com/docs/derivatives/quick-start)

## 3. Rate limit — weight, 429 vs 418

- **Weight sistemi:** Her endpoint'in ağırlığı var; limit **IP başına**.
- **Header'lar (canlı muhasebe):** `X-MBX-USED-WEIGHT-1M` (son 1 dk IP ağırlığı), `X-MBX-ORDER-COUNT-*` (hesap emir sayacı). Her yanıtta gelir.
- **429 Too Many Requests** — limit aşıldı; `Retry-After` kadar **geri çekil**. Devam edersen ban.
- **418 (IP BAN)** — 429 sonrası ısrar edersen. `Retry-After` = ban süresi; ⚠️ tekrar edende **2 dk → 3 gün** ölçeklenir.
- **Limitleri `exchangeInfo`'dan dinamik oku:** `rateLimits[]` dizisi `REQUEST_WEIGHT`/`ORDERS`/`RAW_REQUESTS` tiplerini verir. ⚠️ Sabit kodlama.

!!! tip "Somut rehber"
    1. Merkezî **token-bucket weight limiter**, `exchangeInfo.rateLimits`'ten init; her yanıtta `X-MBX-USED-WEIGHT` ile **gerçek** değere senkronla (client sayacı drift eder).
    2. Emirler için ayrı `ORDERS` sayacı (10s + 1m pencereleri).
    3. `429` → `Retry-After` bekle + limiter'ı doygun işaretle, **asla hemen retry etme**. `418` → IP'yi durdur, alarm.
    4. Limitin **~%80'inde** proaktif throttle.

Kaynak: [Spot REST Limits](https://developers.binance.com/docs/binance-spot-api-docs/rest-api/limits)

## 4. Emir güvenilirliği — timeout ≠ başarısızlık ⭐ { #4-emir-guvenilirligi-timeout-basarisizlik }

!!! danger "En kritik operasyon kuralı"
    Emir POST'unda **timeout / 5XX / "Unknown error"** yanıtı, isteğin borsaya **ulaşıp yürütülmüş olabileceği** anlamına gelir. Durum **UNKNOWN**'dır — başarı da olabilir. **Bunu başarısızlık sayıp körü körüne tekrar göndermek → çift pozisyon.**

**Doğru desen:**

1. **Idempotency — `newClientOrderId`:** Her emre göndermeden önce benzersiz `newClientOrderId` atayın ve **kalıcı saklayın**. Aynı id ile yeni emir, öncekisi dolmadıkça reddedilir → doğal deduplikasyon.
2. **Reconciliation (kör retry değil, sorgula):** Belirsiz yanıtta emri kendi `origClientOrderId`'nizle sorgulayın (Spot `GET /api/v3/order`, Futures `GET /fapi/v1/order`). Bulunursa → başarı; bulunmazsa → güvenle tekrar gönderilebilir.
3. **User Data Stream birincil kaynak:** Reconciliation'ın asıl kaynağı push tabanlı order-update event'leridir; REST sorgusu doğrulama/fallback.
4. **recvWindow:** İstekleri `recvWindow` (≤ 5000ms) + senkron `timestamp` ile gönderin; saat kayması "timestamp outside recvWindow" reddi verir → **NTP zorunlu**.

Kaynak: [USDⓈ-M General Info](https://developers.binance.com/docs/derivatives/usds-margined-futures/general-info)

## 5. WebSocket akışları

**Market data (public, auth yok):** `wss://stream.binance.com:9443/ws/<stream>` — ör. `btcusdt@kline_1m`, `btcusdt@depth`. Futures: `fstream.binance.com` / `dstream.binance.com`.

**User Data Stream (özel; hesap/emir/bakiye) — listenKey yaşam döngüsü:**

1. **Oluştur:** `POST /api/v3/userDataStream` → `listenKey`.
2. **Bağlan:** `wss://stream.binance.com:9443/ws/<listenKey>`.
3. **Geçerlilik 60 dk** — **~30 dakikada bir** `PUT .../userDataStream` (keepalive).
4. **Kapat:** `DELETE .../userDataStream`.

!!! warning "Dayanıklılık"
    - ⚠️ WS bağlantısı **24 saatte bir** zorla kapatılır; ping'e zamanında pong dönmezseniz düşürülürsünüz.
    - Kopmada: exponential backoff reconnect → yeni listenKey → **REST snapshot ile resync** (açık emir + bakiye + pozisyon), çünkü kopukken event kaçmış olabilir.
    - Depth stream'de local order book'u `U`/`u` sıralamasıyla senkronla; boşlukta snapshot'tan yeniden kur.

Kaynak: [User Data Stream](https://developers.binance.com/docs/binance-spot-api-docs/user-data-stream)

## 6. exchangeInfo sembol filtreleri

Emir göndermeden önce sembol `filters[]`'ı çözün; uymayan emir **yürütülmeden reddedilir**.

| Filtre | Kural |
|--------|-------|
| **LOT_SIZE** | `qty ≥ minQty`, `≤ maxQty`, `(qty − minQty) % stepSize == 0` |
| **PRICE_FILTER** | `price ≥ minPrice`, `≤ maxPrice`, `(price − minPrice) % tickSize == 0` |
| **MIN_NOTIONAL / NOTIONAL** | `price * qty ≥ minNotional` |
| **MARKET_LOT_SIZE** | LOT_SIZE ile aynı, sadece market emirleri |
| **PERCENT_PRICE_BY_SIDE** | fiyat, ortalamanın belirli çarpanı içinde |
| **MAX_NUM_ORDERS / _ALGO_ORDERS** | açık/algo emir tavanları |

!!! tip "Mimari sonuç"
    - `stepSize`/`tickSize` için **float değil `Decimal`** kullanın; doğru basamağa **floor** yuvarlayın. Yanlış yuvarlama en yaygın emir-red sebebidir.
    - Filtreleri açılışta yükleyin, ⚠️ **periyodik yenileyin** (bildirimsiz değişebilir). Ürün başına ayrı filtre seti.
    - Pipeline: `qty/price` → step/tick normalizasyonu → minNotional kontrolü → gönder.

Kaynak: [Filters](https://developers.binance.com/docs/binance-spot-api-docs/filters)

## 7. Python kütüphaneleri

| Kriter | **ccxt** | **binance-connector** (resmî) | **python-binance** |
|--------|----------|-------------------------------|--------------------|
| Bakım | Çok aktif | Resmî, spec'e sadık | Topluluk, popüler |
| Kapsam | 100+ borsa unified | Sadece Binance | Binance + async |
| Yeni endpoint | Hızlı | En güncel | Değişken |
| Downside | Soyutlama Binance'e özgü davranışı gizleyebilir | Binance dışına taşınmaz | ⚠️ Bakım/parite garantisi zayıf |

**Öneri (spot+futures+margin):**

- **Birincil: ccxt** — tek unified arayüz, çoklu-ürün (`defaultType`), en hızlı yama.
- **Yanında: resmî binance-connector** — order reliability, rate-limit header, listenKey, yeni endpoint'lerde **ground truth**.
- **Kritik:** weight limiter, reconciliation ve filtre motorunu **kütüphaneye devretmeyin, kendi çekirdeğinizde tutun** — kütüphaneler bunları eksik/farklı yapıyor.

Kaynaklar: [ccxt](https://github.com/ccxt/ccxt) · [ccxt binance docs](https://docs.ccxt.com/exchanges/binance) · [python-binance](https://github.com/sammchardy/python-binance)

## Özet mimari kararlar

1. Core + per-product adapter (spot/fapi/dapi/sapi/eapi).
2. Base URL'ler config'ten (prod ↔ testnet/demo geçişi + ⚠️ demo rebrand).
3. Merkezî weight+order limiter; 429=back off, 418=IP durdur.
4. `newClientOrderId` + reconciliation; timeout/5XX asla "başarısız" değil.
5. User Data Stream event-driven state; 30 dk keepalive, kopmada REST resync.
6. `exchangeInfo` filtre motoru (Decimal, floor-to-step/tick), periyodik yenileme.
7. ccxt birincil + resmî connector doğrulama; reliability/limit/filtre çekirdeği kendinizde.
