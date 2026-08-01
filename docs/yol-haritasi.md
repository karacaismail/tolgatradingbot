# Yol Haritası

Fazlar sırayla ilerler; her faz bir öncekinin üstüne inşa edilir. Gerçek para en sona bırakılır.

| Faz | Kapsam | Çıktı |
|-----|--------|-------|
| **Faz 0** | Repo iskeleti, `.env`, config, güvenli API anahtarı (trade-only, IP whitelist) | Çalışan iskelet |
| **Faz 1** | Bağlantı + veri katmanı (**Spot** canlı fiyat/mum, RSI+MACD+ATR) | Ekrana canlı veri |
| **Faz 2** | ECA kural motoru + YAML DSL (1–2 basit strateji, multi-timeframe) | Kural → sinyal |
| **Faz 3** | Backtest motoru + bütünlük denetimi (look-ahead/survivorship/vintage, walk-forward) | Güvenilir strateji karşılaştırması |
| **Faz 3.5** | **Shadow mode** (canlı veri, emir önerisi üret ama gönderme, gerçekle karşılaştır) | Canlı-benzeri doğrulama (risksiz) |
| **Faz 4** | Risk yönetimi (SL/TP, günlük zarar limiti, kill-switch) + **paper/testnet** | Sanal işlemler |
| **Faz 5** | Yürütme + trade journal + Telegram bildirim + **4 mod anahtarı** | Testnet'te tam döngü |
| **Faz 6** | **Web GUI:** canlı ECharts monitoring, filtreli log viewer, kural editörü, PnL paneli, mod seçici | Görsel yönetim |
| **Faz 7** | **Feedback sistemi** (erken zarar kesme + sürekli iyileştirme önerileri) | Veri-odaklı özdenetim |
| **Faz 8** | **AI karar katmanı** (önce "veto/filtre" rolüyle, risk katmanı altında) | AI destekli sinyal |
| **Faz 9** | **Hetzner deployment** (Docker, 7/24, otomatik restart, izleme) | Canlı altyapı |
| **Faz 10** | **Futures** (kaldıraç sınırlı) → sonra **Margin** desteği | Genişletilmiş piyasa |
| **Faz 11** | Küçük gerçek para → izleme → kademeli ölçek | Canlı (dikkatli) |
| **Faz 12** | (Opsiyonel) Ücretli sentiment/on-chain veri entegrasyonu | Sinyal zenginleştirme |

## Sıradaki somut adımlar

1. **Bölüm 1.1'deki açık soruları** cevapla (sermaye, max kaldıraç, parite evreni, AI kapsamı). Bkz. [Açık Sorular](baslangic/acik-sorular.md).
2. **Repo iskeletini** kur (Faz 0): klasör yapısı, `config.yaml`, örnek `rules/*.yaml`, `.env.example`, `docker-compose`, boş modüller + testler.
3. **`eca-rule-authoring` + `strategy-backtest`** skill'lerini oluştur (çekirdeği hızlandırır). Bkz. [Claude Skills](skills.md).

> Bu döküman canlı bir belgedir; kararlar değiştikçe güncellenir.
