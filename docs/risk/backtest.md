# Backtest Bütünlüğü & Getiri Gerçekliği

> "Kağıt üstünde harika, canlıda çöp" tuzağından kaçınmanın yolu. Kaynak: risk & backtest araştırma brifi.

## Backtest tuzakları (her biri + nasıl kaçınılır)

1. **Look-ahead bias (ileriye bakma):** Test anında bilinmeyen veriyi kullanmak (mumun high/low'unu kapanmadan kullanmak; revize veriyi anlık saymak). **Çözüm:** her sinyali en az bir periyot **gecikmeli** hesapla; sadece `t` anında mevcut veriyi kullan.
2. **Survivorship bias:** Sadece bugün var olan varlıkları test etmek; delist/iflas coinleri dışlamak performansı şişirir. **Çözüm:** delist sembolleri de içeren **point-in-time evren**.
3. **Revize makro veri (vintage / point-in-time):** FRED güncel (revize) değerler sunar → look-ahead. **Çözüm:** **ALFRED** ile o tarihte fiilen yayınlanmış "vintage" değeri kullan (`realtime_start`/`realtime_end`). Bkz. [Veri Kaynakları](../veri/kaynaklar.md).
4. **Data snooping / multiple testing (p-hacking):** Yüzlerce parametre deneyip en iyisini seçmek. **Çözüm:** deneme sayısını kaydet, **DSR / Bonferroni** düzelt, hipotezi veriden önce kur.
5. **Overfitting (curve fitting):** Modeli geçmiş gürültüye uydurmak. **Çözüm:** az parametre, OOS + walk-forward zorunlu, parametre kararlılığına bak.
6. **Ücret/spread/slippage/funding ihmali:** Sıfır maliyet HFT'yi sahte kârlı gösterir. **Binance USDⓈ-M:** ~%0.02 maker / **%0.05 taker**; funding ~%0.01/8s. Round-trip "all-in" genelde **%0.15–0.30**. **Çözüm:** hepsini modelle; **ücretleri 2× ile test et** — 2×'te kârlıysa gerçekte de kârlı olma olasılığı yüksek.
7. **Gerçekçi olmayan fill'ler:** Her emrin tam fiyattan anında dolduğunu varsaymak. **Çözüm:** order-book derinliğine göre fill olasılığı + **kısmi doluş** modelle; stop'ları market emri olarak.

Kaynaklar: [QuantStart Backtesting](https://www.quantstart.com/articles/Successful-Backtesting-of-Algorithmic-Trading-Strategies-Part-I/) · [ALFRED vintage](https://fredblog.stlouisfed.org/2021/04/alfred-at-15-archiving-fred-data-since-2006/) · [López de Prado — Backtest Overfitting](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=2460551) · [StratBase realistic fees](https://stratbase.ai/en/blog/backtest-realistic-fees)

## Sağlam validasyon

- **Out-of-sample (OOS):** parametreleri in-sample'da bul, dokunulmamış OOS'ta doğrula. IS'de mükemmel + OOS'ta çöken = overfit.
- **Walk-forward analizi (WFA):** kayan pencereler; her pencerenin IS kısmında optimize, hemen ardından OOS'ta **kör** test, ileri kaydır. Gerçek canlı yeniden-optimizasyonu taklit eder. Train/validation arası **embargo** (ör. 30 gün) ile sızıntıyı engelle.
- **Parametre kararlılığı:** optimal parametreler pencereden pencereye erratik değişiyorsa edge gürültüye bağlıdır. Sağlam strateji parametre yüzeyinde **geniş, düz plato** üzerinde durur (keskin tek tepe = overfit).
- **Monte Carlo:** işlem sırasını/getirileri resample ederek binlerce alternatif eşküm; drawdown/iflas olasılığının **dağılımını** çıkar.
- **CPCV (Combinatorial Purged Cross-Validation):** López de Prado'nun purging+embargo ile sızıntısız çapraz-doğrulaması; DSR ile gold-standart.

Kaynaklar: [QuantInsti WFO](https://blog.quantinsti.com/walk-forward-optimization-introduction/) · [BuildAlpha robustness](https://www.buildalpha.com/robustness-testing-guide/)

## Getiri Gerçekliği {#getiri-gercekligi}

!!! danger "\"Günde %X garanti getiri\" matematiksel olarak makul değildir"
    - **Bileşik patlama:** günde sadece **%1** garanti getiri → yılda (1.01)^365 ≈ **37.8×**. Günde **%2** → (1.02)^365 ≈ **~1.377×** (bin kattan fazla). Bu oranlar sürdürülse birkaç yılda dünyanın tüm sermayesini geçerdi — tek başına iddiayı çürütür.
    - **Risk-getiri ayrılmaz:** "garanti + yüksek getiri" ancak (a) risk gizleniyorsa (nadir ama yıkıcı kuyruk riski), (b) geçmiş overfit ediliyorsa, veya (c) Ponzi ise mümkündür.
    - **Kapasite:** gerçek edge bile sermaye büyüdükçe slippage/piyasa etkisiyle aşınır.
    - **Getiri bir dağılımdır:** gerçek stratejide günlük getiri kayıp günleri de içerir; "her gün +%X" bir sabit değil, olsa olsa **ortalama** ve daima drawdown eşliğinde.

**Sonuç:** Bot için hedef "günlük sabit getiri" değil, **pozitif beklenti + kontrollü drawdown + hayatta kalma** olmalıdır. Kullanıcının "günlük %2" isteği botta bir **dur-kâr hedefi** olarak yaşar ("+%2'ye ulaşınca günü kapat"), garanti minimum olarak değil. Sermayeyi koruyan sistem, agresif vaat eden ama iflas riski taşıyandan uzun vadede üstündür.

Kaynaklar: [López de Prado — 10 Reasons ML Funds Fail](https://www.garp.org/hubfs/Whitepapers/a1Z1W0000054x6lUAA.pdf) · [Deflated Sharpe PDF](https://www.davidhbailey.com/dhbpapers/deflated-sharpe.pdf)
