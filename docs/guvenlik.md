# Güvenlik & Operasyonel Sağlamlık

Botlar en çok strateji değil, **operasyon hatalarından** patlar. Bu bölüm pazarlık konusu değildir.

## Anahtar & sır güvenliği

- **API anahtarında withdrawal (para çekme) izni ASLA açılmaz.** Sadece "trade" (ve gerekiyorsa "read").
- **IP allowlist** açık (sadece Hetzner sunucu IP'si).
- **Market-data anahtarı ile trade anahtarı ayrı.** Testnet ve production sırları tamamen ayrı store'da.
- Anahtarlar **koda, loglara, AI promptlarına ASLA girmez** (`.env` + secret manager, log maskeleme).

## Emir güvenliği (çift-emir felaketini önle)

- **Timeout / `5XX` = "başarısız" DEĞİLDİR.** Emir borsaya ulaşmış olabilir. Kör tekrar **yasak**.
- Belirsiz durumda emir **User Data Stream veya sorgu ile uzlaştırılır** (reconciliation), sonra karar verilir.
- Her emir **`idempotency_key` / client-order-ID** taşır → aynı emir iki kez işlenmez.
- Emir **niyeti** ile borsa **sonucu** append-only **audit log**'a yazılır.
- Otomatik retry **sadece güvenli/idempotent sorgularda** (emir gönderiminde değil).

Detay: [Binance API — Emir güvenilirliği](mimari/binance-api.md#4-emir-guvenilirligi-timeout-basarisizlik).

## Sağlık & devre kesiciler

- **Veri bayat/eksik/çelişkili → varsayılan eylem `NO_TRADE`** (fail-closed).
- **Kill-switch işlem motorundan bağımsız** çalışır (motor kilitlense bile durdurabilmeli).
- Sistem saati ↔ Binance server-time farkı (**clock drift**) izlenir; eşik aşılırsa dur (`recvWindow` reddini önler).
- WebSocket kopması, `418` (IP ban), `429` (rate-limit) için tanımlı geri-dönüş runbook'u.
- Üçüncü taraf API verisi **güvenilmez girdi** kabul edilir; doğrulanmadan işleme sokulmaz ([OWASP: Unsafe Consumption of APIs](https://owasp.org/API-Security/editions/2023/en/0xaa-unsafe-consumption-of-apis/)).

## GUI güvenliği

- Dashboard **kimlik doğrulamalı** (tek kullanıcı + güçlü parola/2FA), tercihen IP kısıtlı veya VPN/reverse-proxy arkasında.
- Kill-switch ve mod değişimi gibi kritik aksiyonlar ek onay + loglama gerektirir.
- TLS zorunlu (Caddy/Nginx + Let's Encrypt).

## Canlıya geçiş kapıları (gerçek para açılmadan önce)

Gerçek paraya geçiş **otomatik değildir**; şu kapılar geçilmeden açılmaz:

- [ ] Tanımlı minimum paper/testnet süresi tamamlandı
- [ ] Shadow mode'da backtest-canlı sapması kabul edilebilir
- [ ] **Sıfır** açıklanamayan emir, **sıfır** çözülmemiş reconciliation farkı
- [ ] Kesinti + yeniden başlatma testleri geçti
- [ ] Günlük/toplam kayıp limiti testleri geçti
- [ ] Bağımsız güvenlik incelemesi yapıldı
- [ ] **Futures/margin, Spot'un devamı değil — ayrı proje ve ayrı onay kapısıdır**

!!! note "Doğrulama notu"
    Buradaki emir-uzlaştırma, rate-limit ve üçüncü-taraf-API ilkeleri Binance resmî API kuralları ve OWASP API güvenliği pratikleriyle uyumludur. Canlıya geçmeden önce **güncel Binance dokümanından** (rate-limit, filtreler, hata kodları) doğrulama yapılmalı — bu değerler değişebiliyor.
