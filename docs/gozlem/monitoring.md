# Canlı Monitoring (ECharts)

> Kullanıcı gereksinimi: *"Kâr ve zararları anlık görüntüleyebileceğimiz, ECharts monitoring live ekranlar olmalı."*

Web dashboard, botun durumunu **gerçek zamanlı** gösterir. Grafikler [Apache ECharts](https://echarts.apache.org/) ile çizilir; veriler FastAPI backend'inden **WebSocket** üzerinden canlı akar.

## Panel seti

| Panel | Görselleştirme | Veri |
|-------|----------------|------|
| **Equity eğrisi** | Çizgi + alan | Toplam sermaye zaman serisi (gerçekleşmiş + gerçekleşmemiş) |
| **Günlük PnL** | Bar (yeşil/kırmızı) | Gün bazında kâr/zarar; günlük hedef (+%2) ve limit çizgileri |
| **Drawdown** | Ters alan / gauge | Tepe-dip düşüş %; circuit-breaker eşiği işaretli |
| **Açık pozisyonlar** | Tablo | Sembol, yön, giriş, anlık PnL, SL/TP, kaldıraç |
| **Strateji/kural bazlı PnL** | Yatay bar | Hangi `rule_id` ne kazandırdı/kaybettirdi |
| **Win-rate & profit factor** | Stat kartları | Metrikler (bkz. [Risk](../risk/yonetim.md)) |
| **Risk göstergesi** | Gauge | Günlük zarar / limit oranı, kill-switch durumu |

## Canlı veri akışı

```mermaid
flowchart LR
    BOT[Bot çekirdeği] -->|olay/fill| BUS[Event Bus]
    BUS --> API[FastAPI]
    API -->|WebSocket| UI[React + ECharts Dashboard]
    DB[(Trade Journal / TimescaleDB)] --> API
```

- **Anlık:** fill, pozisyon değişimi, PnL güncellemesi WebSocket ile push edilir → grafik `setOption` ile güncellenir.
- **Tarihsel:** sayfa açılışında REST'ten equity/PnL geçmişi çekilir.
- **Mod bilgisi:** her panelde aktif mod (testnet/paper/semi-auto/full-auto) rozeti; canlı modda kırmızı vurgu.

## Canlı demo (örnek veri)

Aşağıdaki grafikler **örnek verilerle** çalışan bir önizlemedir (gerçek bot verisi değildir); dashboard'un görünümünü temsil eder. Değerler birkaç saniyede bir simüle güncellenir.

<div id="tb-stats" style="display:flex;gap:12px;flex-wrap:wrap;margin:1rem 0;">
  <div style="flex:1;min-width:120px;padding:12px;border-radius:8px;background:rgba(255,193,7,.12);">
    <div style="font-size:.8rem;opacity:.7;">Toplam PnL</div>
    <div id="tb-pnl" style="font-size:1.4rem;font-weight:700;">+—</div>
  </div>
  <div style="flex:1;min-width:120px;padding:12px;border-radius:8px;background:rgba(76,175,80,.12);">
    <div style="font-size:.8rem;opacity:.7;">Win-rate</div>
    <div id="tb-wr" style="font-size:1.4rem;font-weight:700;">—</div>
  </div>
  <div style="flex:1;min-width:120px;padding:12px;border-radius:8px;background:rgba(244,67,54,.12);">
    <div style="font-size:.8rem;opacity:.7;">Max Drawdown</div>
    <div id="tb-dd" style="font-size:1.4rem;font-weight:700;">—</div>
  </div>
  <div style="flex:1;min-width:120px;padding:12px;border-radius:8px;background:rgba(33,150,243,.12);">
    <div style="font-size:.8rem;opacity:.7;">Açık Pozisyon</div>
    <div id="tb-op" style="font-size:1.4rem;font-weight:700;">3</div>
  </div>
</div>

<div id="tb-equity" style="width:100%;height:300px;"></div>
<div id="tb-pnlbar" style="width:100%;height:260px;margin-top:1rem;"></div>

<script src="https://cdn.jsdelivr.net/npm/echarts@5/dist/echarts.min.js"></script>
<script>
(function () {
  function build() {
    if (!window.echarts) { setTimeout(build, 200); return; }
    var eqEl = document.getElementById('tb-equity');
    var barEl = document.getElementById('tb-pnlbar');
    if (!eqEl || !barEl) return;
    if (eqEl.dataset.init === '1') return; // idempotent
    eqEl.dataset.init = '1';

    var dark = document.body.getAttribute('data-md-color-scheme') === 'slate';
    var txt = dark ? '#cfd3dc' : '#333';

    // --- örnek equity serisi ---
    var days = 60, base = 10000, eq = [], cats = [], seed = 42;
    function rnd(){ seed = (seed*9301+49297)%233280; return seed/233280; }
    var v = base, peak = base, maxdd = 0;
    for (var i=0;i<days;i++){
      var chg = (rnd()-0.45)*0.02; // hafif pozitif drift
      v = v*(1+chg);
      eq.push(+v.toFixed(2));
      peak = Math.max(peak, v);
      maxdd = Math.max(maxdd, (peak-v)/peak);
      var d = new Date(2026,5,1); d.setDate(d.getDate()+i);
      cats.push((d.getMonth()+1)+'/'+d.getDate());
    }
    var pnl = eq.map(function(x,i){ return i===0?0:+(eq[i]-eq[i-1]).toFixed(2); });

    var eqChart = echarts.init(eqEl, null, {renderer:'canvas'});
    eqChart.setOption({
      textStyle:{color:txt},
      tooltip:{trigger:'axis'},
      grid:{left:60,right:20,top:30,bottom:40},
      title:{text:'Equity Eğrisi (örnek)',textStyle:{color:txt,fontSize:13}},
      xAxis:{type:'category',data:cats,axisLabel:{color:txt}},
      yAxis:{type:'value',scale:true,axisLabel:{color:txt}},
      series:[{
        type:'line',data:eq,smooth:true,showSymbol:false,
        lineStyle:{width:2,color:'#ffb300'},
        areaStyle:{color:'rgba(255,179,0,.15)'}
      }]
    });

    var barChart = echarts.init(barEl, null, {renderer:'canvas'});
    barChart.setOption({
      textStyle:{color:txt},
      tooltip:{trigger:'axis'},
      grid:{left:60,right:20,top:30,bottom:40},
      title:{text:'Günlük PnL (örnek)',textStyle:{color:txt,fontSize:13}},
      xAxis:{type:'category',data:cats,axisLabel:{color:txt}},
      yAxis:{type:'value',axisLabel:{color:txt}},
      series:[{
        type:'bar',data:pnl,
        itemStyle:{color:function(p){return p.value>=0?'#4caf50':'#f44336';}}
      }]
    });

    // stat kartları
    var totalPnl = +(eq[eq.length-1]-base).toFixed(2);
    var wins = pnl.filter(function(x){return x>0;}).length;
    var wr = Math.round(wins/(pnl.length-1)*100);
    var pnlEl=document.getElementById('tb-pnl'), wrEl=document.getElementById('tb-wr'), ddEl=document.getElementById('tb-dd');
    if(pnlEl){ pnlEl.textContent=(totalPnl>=0?'+':'')+totalPnl+' USDT'; pnlEl.style.color=totalPnl>=0?'#4caf50':'#f44336'; }
    if(wrEl) wrEl.textContent='%'+wr;
    if(ddEl) ddEl.textContent='%'+(maxdd*100).toFixed(1);

    // simüle "canlı" güncelleme
    if (window.__tbTimer) clearInterval(window.__tbTimer);
    window.__tbTimer = setInterval(function(){
      if(!document.body.contains(eqEl)){ clearInterval(window.__tbTimer); return; }
      var last = eq[eq.length-1];
      var nv = +(last*(1+(rnd()-0.45)*0.02)).toFixed(2);
      eq.push(nv); eq.shift();
      var np = +(nv-last).toFixed(2); pnl.push(np); pnl.shift();
      eqChart.setOption({series:[{data:eq}]});
      barChart.setOption({series:[{data:pnl}]});
      var tp=+(eq[eq.length-1]-base).toFixed(2);
      if(pnlEl){ pnlEl.textContent=(tp>=0?'+':'')+tp+' USDT'; pnlEl.style.color=tp>=0?'#4caf50':'#f44336'; }
    }, 2500);

    window.addEventListener('resize', function(){ eqChart.resize(); barChart.resize(); });
  }
  if (typeof document$ !== 'undefined' && document$.subscribe) { document$.subscribe(build); }
  else { document.addEventListener('DOMContentLoaded', build); build(); }
})();
</script>

!!! note "Bu bir mockup'tır"
    Yukarıdaki grafikler dokümantasyonu somutlaştırmak için örnek/simüle veriyle çalışır. Gerçek dashboard, aynı ECharts bileşenlerini FastAPI WebSocket'ten gelen canlı bot verisiyle besleyecektir. Grafikler görünmüyorsa sayfayı yenileyin (CDN yüklemesi).
