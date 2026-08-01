# AI Karar Katmanı — Yetki Sınırı

## Uzlaştırma: "tam otomatik" vs "AI'ın emir yetkisi yok"

Kullanıcı **"tam otomatik, insan onayı olmadan, AI destekli"** istedi. İyi mühendislik pratiği ise AI'ın körü körüne emir vermesine karşı uyarır. Bu iki ilke **çelişmez**:

!!! success "İki ilke birden doğru"
    - **"Tam otomatik"** = her işlemde insan tıklamaz; **yürütme** otomatiktir.
    - **"AI'ın emir yetkisi yok"** = AI **sıfırdan, sınırsız emir icat etmez**.

    Doğru mimari ikisini birden sağlar:

    ```
    Deterministik ECA çekirdeği  →  aday sinyal
        ↓
    AI katmanı  →  filtrele / veto et / bağlam ekle (yeni sinyal ÜRETMEZ)
        ↓
    Risk motoru  →  son söz; hiçbir limiti aştırmaz
        ↓
    Emir  →  otomatik gider
    ```

    Bot insan onayı olmadan çalışır (kullanıcının istediği); ama AI "denetleyen akıl"dır, "körü körüne para basan" değil.

## AI rolleri (kapsam D sorusunda netleşir)

| Rol | Ne yapar | Risk seviyesi |
|-----|----------|---------------|
| **Sinyal doğrulama / veto** | ECA "al" dedi; AI bağlama bakıp "bu setup zayıf, atla" diyebilir (sadece filtreler) | **Düşük — güvenli başlangıç** |
| **Rejim tespiti** | Piyasa trend mi yatay mı? Hangi strateji ailesi aktif olsun | Düşük–orta |
| **Parametre önerisi** | Volatiliteye göre SL/TP veya boyut önerir (risk katmanı onaylar) | Orta |
| **Anomali/haber uyarısı** | Ani hareket/haber algılayıp pozisyonları koruma moduna alır | Orta |

Önerilen başlangıç: **yalnızca veto/filtre** rolü. Diğer roller backtest + shadow ile kanıtlandıkça açılır.

## İlkeler

- **AI çıktısı da risk motorundan geçer.** AI hiçbir sınırı aşamaz (max kaldıraç, max zarar, max pozisyon).
- **Her AI kararı loglanır** — girdi bağlamı, çıktı, gerekçe. Denetlenebilirlik şart (bkz. [Loglama](../gozlem/loglama.md)).
- **AI çıktısı ECA'da bir koşul olarak temsil edilir:** `{ai_confidence: {gt: 0.7}}`. Böylece deterministik katman AI'ı "gate"ler.
- **Fail-safe:** AI katmanı yanıt vermezse/hata verirse, sistem AI'sız deterministik ECA + risk motoruyla çalışmaya devam eder (AI kritik yol değil, danışman).
- **Maliyet kontrolü:** AI çağrıları önbelleklenebilir/örneklenebilir; her tick'te değil, karar anlarında çağrılır.

## Model

Öneri: **Claude API** — maliyet kontrollü, güçlü muhakeme, her karar loglanabilir. Alternatif: yerel ML modeli (ör. rejim sınıflandırma için hafif bir model), ama başlangıç için LLM tabanlı danışman daha esnek.
