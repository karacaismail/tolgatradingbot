# Tolga Trading Bot — Dokümantasyon

Binance tabanlı, **ECA (Event-Condition-Action)** kural motoru üzerine kurulu, çok modlu (backtest / paper / semi-auto / full-auto) bir trading bot için **gereksinim analizi ve mimari dokümantasyon sistemi**.

> ⚠️ **Sorumluluk reddi:** Bu depo bir yazılım mühendisliği ve eğitim kaynağıdır. **Yatırım tavsiyesi değildir** ve hiçbir kâr garantisi vermez. Kripto işlemleri (özellikle kaldıraçlı) yüksek risklidir. Botu gerçek para ile çalıştırmadan önce mutlaka backtest → paper → testnet aşamalarından geçirin.

## Bu depo nedir?

İki taslağın (Claude + ChatGPT/Codex) birleştirilmesi ve **7 ayrı güncel web araştırması** (Binance API, açık kaynak framework'ler, strateji/indikatör, risk/backtest, piyasa/on-chain/haber-makro/sentiment veri kaynakları) ile zenginleştirilmesiyle oluşturulmuş, kaynaklı ve çok sayfalı bir dokümantasyon sitesidir.

## Yerelde çalıştırma

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install mkdocs-material pymdown-extensions
mkdocs serve
```

Tarayıcıda: <http://127.0.0.1:8000>

## Yayın

`main` dalına her push'ta GitHub Actions (`.github/workflows/deploy.yml`) siteyi otomatik derleyip **GitHub Pages**'e yayınlar.

Canlı site: <https://karacaismail.github.io/tolgatradingbot/>

## İçindekiler (özet)

- **Başlangıç** — alınan kararlar, açık sorular
- **Mimari** — katmanlı mimari, Binance API entegrasyonu, framework seçimi
- **ECA Kural Motoru** — anatomi, karar akışı, şema, örnek kurallar, AI katmanı
- **Strateji** — indikatörler, strateji aileleri, rejim tespiti
- **Risk & Backtest** — pozisyon boyutlama, risk motoru, backtest bütünlüğü
- **Veri Kaynakları** — ücretsiz-öncelikli veri yığını + ücretli yükseltme yolu
- **Gözlem & Kontrol** — loglama, log viewer, canlı ECharts monitoring, feedback sistemi
- **Güvenlik & Operasyon**, **Yol Haritası**, **Claude Skills**, **Kaynakça**
