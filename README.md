<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Trading Platform</title>
<script src="https://cdn.jsdelivr.net/npm/lightweight-charts@3.8.0/dist/lightweight-charts.standalone.production.js"></script>
<script src="https://cdn.jsdelivr.net/npm/technicalindicators@3.0.0/dist/browser.js"></script>
<style>
  body { font-family: Arial; margin: 0; background:#1b1b1b; color:white; }
  #chart { width: 100%; height: 500px; }
  #watchlist { display:flex; margin:10px; gap:10px; }
  .pair { padding:5px 10px; background:#333; cursor:pointer; border-radius:5px; }
</style>
</head>
<body>
<h2>TradingView Clone with Signal Bot</h2>
<div id="watchlist"></div>
<div id="chart"></div>
<div id="signal"></div>

<script>
const watchlist = ["BTCUSDT","ETHUSDT","EURUSD","GBPUSD"];
let selectedPair = watchlist[0];

const watchlistDiv = document.getElementById("watchlist");
watchlist.forEach(pair => {
    const btn = document.createElement("div");
    btn.className = "pair";
    btn.textContent = pair;
    btn.onclick = () => {
        selectedPair = pair;
        fetchData(pair);
    };
    watchlistDiv.appendChild(btn);
});

const chart = LightweightCharts.createChart(document.getElementById('chart'), {
    layout: { backgroundColor: '#1b1b1b', textColor: 'white' },
    grid: { vertLines: { color: '#444' }, horzLines: { color: '#444' } }
});
const candleSeries = chart.addCandlestickSeries();

// Simple Signal Bot: Moving Average Crossover
const SMA = technicalindicators.SMA;

async function fetchData(pair){
    // Example with Binance API (spot data)
    const res = await fetch(`https://api.binance.com/api/v3/klines?symbol=${pair}&interval=15m&limit=200`);
    const data = await res.json();
    const candles = data.map(d => ({
        time: d[0]/1000,
        open: parseFloat(d[1]),
        high: parseFloat(d[2]),
        low: parseFloat(d[3]),
        close: parseFloat(d[4])
    }));
    candleSeries.setData(candles);

    // Bot signals: MA crossover
    const closePrices = candles.map(c => c.close);
    const smaFast = SMA.calculate({ period: 5, values: closePrices });
    const smaSlow = SMA.calculate({ period: 20, values: closePrices });
    const lastFast = smaFast[smaFast.length-1];
    const lastSlow = smaSlow[smaSlow.length-1];
    const signalDiv = document.getElementById("signal");
    if(lastFast > lastSlow){
        signalDiv.textContent = `${pair}: BUY Signal`;
        signalDiv.style.color = "lime";
    } else if(lastFast < lastSlow){
        signalDiv.textContent = `${pair}: SELL Signal`;
        signalDiv.style.color = "red";
    } else {
        signalDiv.textContent = `${pair}: HOLD`;
        signalDiv.style.color = "white";
    }
}

// Initial load
fetchData(selectedPair);

// Auto-refresh every minute
setInterval(()=>fetchData(selectedPair), 60000);
</script>
