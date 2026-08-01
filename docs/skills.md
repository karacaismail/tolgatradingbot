# Oluşturulacak Claude Skills

Skill'ler iki gruba ayrılır: **(A)** geliştirme sırasında işini kolaylaştıranlar, **(B)** botun/asistanın çalışırken kullandığı operasyonel skill'ler.

!!! danger "Güvenlik ilkesi"
    Hiçbir skill Binance anahtarına, gerçek bakiyeye veya emir endpoint'ine erişmemeli. Skill çıktıları yalnız **öneri, araştırma paketi veya doğrulanabilir JSON** olmalı — otomatik icra değil.

## A. Geliştirme (dev-time) skill'leri

| Skill | Ne yapar |
|-------|----------|
| `eca-rule-authoring` | YAML ECA kuralı yaz + şema doğrula + "bu kural ne yapar?" açıkla. Eksik/yoruma açık alan varsa reddet. |
| `strategy-backtest` | Verilen strateji + parite + tarih aralığı için backtest çalıştır, metrik raporu (Sharpe/DD/win-rate/PF) üret. |
| `backtest-integrity-auditor` | Look-ahead / survivorship / vintage-veri / sızıntı / overfitting / maliyet eksikliği avla (bkz. [Backtest](risk/backtest.md)). |
| `risk-policy-auditor` | Stratejiden bağımsız risk kontratını denetle (günlük kayıp, drawdown, korelasyon, kaldıraç, kill-switch). |
| `indicator-reference` | TA indikatörleri sözlüğü (formül + kullanım + tipik eşik). Bkz. [İndikatörler](strateji/indikatorler.md). |
| `binance-api-helper` | Binance endpoint, rate-limit, hata kodları, örnek çağrı üretimi. |
| `binance-api-contract-radar` | Binance doküman/changelog değişimlerini izle (endpoint, filtre, rate-limit, hata); kod üretmez. |
| `source-registry-curator` | Sürümlü kaynak sicilini yönet; takipçi sayısını güvenilirlik saymayı reddet. Bkz. [Veri](veri/kaynaklar.md). |

## B. Operasyonel (runtime) skill'ler

| Skill | Ne yapar |
|-------|----------|
| `market-scan` | Parite evrenini tara, ECA koşullarına uyan "setup"ları listele. |
| `trade-journal-analyze` | İşlem günlüğü + logları analiz et: hangi kural/rejim kazandırdı/kaybettirdi, slippage dağılımı, öneriler. [Feedback](gozlem/feedback.md) motorunun beyni. |
| `daily-report` | Günlük PnL, açık pozisyon, risk durumu raporu (Telegram/e-posta). |
| `ops-healthcheck` | Bot ayakta mı, WS kopmuş mu, API kotası, clock drift, hata oranı. |
| `execution-reconciliation-auditor` | İstenen ↔ gönderilen ↔ kabul ↔ fill ↔ bakiye farklarını denetle; belirsiz emirde tekrar-gönderimi engelle. |
| `social-event-verifier` | Sosyal mesajı birincil kaynak + bağımsız kanıtla doğrula; eski haber / taklit / manipülasyonu işaretle (ECA-007). |
| `incident-kill-switch-runbook` | WS kopması, API banı, clock drift, çift emir, stablecoin depeg için durdurma/geri-dönüş yönergesi. |

## Öncelik

Başlangıç için **`eca-rule-authoring`** ve **`strategy-backtest`** — çekirdeği en çok hızlandıran ikili. Skill'leri sistemdeki `skill-creator` ile üretmek önerilir.
