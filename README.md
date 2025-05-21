<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Multi-Page Stock Tracker</title>
<style>
  body {
    background: #0a0a0a;
    color: #eee;
    font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
    max-width: 900px;
    margin: 1rem auto;
    padding: 1rem;
  }
  h1, h2 {
    color: #0ff;
  }
  nav {
    display: flex;
    gap: 1rem;
    margin-bottom: 1rem;
  }
  nav button {
    background: #111;
    color: #0ff;
    border: none;
    padding: 0.6rem 1rem;
    cursor: pointer;
    border-radius: 6px;
    font-weight: bold;
  }
  nav button.active {
    background: #0ff;
    color: #000;
  }
  section {
    display: none;
  }
  section.active {
    display: block;
  }
  input, button {
    padding: 0.4rem 0.7rem;
    font-size: 1rem;
    margin-right: 0.5rem;
    border-radius: 4px;
    border: none;
  }
  button {
    cursor: pointer;
    background: #0ff;
    color: #000;
  }
  ul {
    list-style: none;
    padding-left: 0;
  }
  li {
    margin-bottom: 0.5rem;
  }
  a {
    color: #0ff;
    text-decoration: none;
  }
  .stock-box {
    background: #111;
    padding: 1rem;
    margin: 0.5rem 0;
    border-radius: 6px;
    position: relative;
  }
  .remove-btn {
    position: absolute;
    right: 10px;
    top: 10px;
    background: red;
    color: white;
    border: none;
    padding: 0.2rem 0.5rem;
    border-radius: 4px;
    cursor: pointer;
  }
  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 1rem;
  }
  th, td {
    border: 1px solid #222;
    padding: 0.5rem;
    text-align: center;
  }
  .alert-set-btn {
    background: #0f0;
    color: #000;
    font-weight: bold;
  }
  .alert-list li {
    margin-bottom: 0.3rem;
  }
</style>
</head>
<body>

<nav>
  <button class="tab-btn active" data-tab="home">Home</button>
  <button class="tab-btn" data-tab="news">News</button>
  <button class="tab-btn" data-tab="prices">Prices</button>
  <button class="tab-btn" data-tab="recommendations">Recommendations</button>
  <button class="tab-btn" data-tab="portfolio">Portfolio</button>
  <button class="tab-btn" data-tab="alerts">Alerts</button>
  <button class="tab-btn" data-tab="about">About</button>
</nav>

<section id="home" class="active">
  <h1>Welcome to Multi-Page Stock Tracker</h1>
  <p>Use the tabs above to track stocks, get news, view recommendations, manage your portfolio, and set alerts.</p>
  <p>Add stocks on the Prices page to start.</p>
</section>

<section id="news">
  <h2>Latest News</h2>
  <ul id="newsList"><li>No stocks added yet. Add some on the Prices page!</li></ul>
</section>

<section id="prices">
  <h2>Track Stocks</h2>
  <input id="stockInput" placeholder="Enter stock symbol (e.g. AAPL)" />
  <button onclick="addStock()">Add Stock</button>
  <div id="stocksContainer"></div>
</section>

<section id="recommendations">
  <h2>Recommendations</h2>
  <p>Popular stocks to consider:</p>
  <ul id="recList"></ul>
</section>

<section id="portfolio">
  <h2>Your Portfolio</h2>
  <p>Track your stock holdings and their value.</p>
  <input id="portfolioStockInput" placeholder="Stock symbol" style="width:100px" />
  <input id="portfolioSharesInput" type="number" min="1" placeholder="Shares" style="width:80px" />
  <button onclick="addPortfolioStock()">Add to Portfolio</button>
  <table>
    <thead>
      <tr><th>Stock</th><th>Shares</th><th>Price</th><th>Value</th><th>Remove</th></tr>
    </thead>
    <tbody id="portfolioTableBody"></tbody>
    <tfoot>
      <tr>
        <th colspan="3" style="text-align:right">Total Value:</th>
        <th id="portfolioTotalValue">$0.00</th>
        <th></th>
      </tr>
    </tfoot>
  </table>
</section>

<section id="alerts">
  <h2>Price Alerts</h2>
  <p>Set alerts for when a stock hits a target price.</p>
  <input id="alertStockInput" placeholder="Stock symbol" style="width:100px" />
  <input id="alertPriceInput" type="number" min="0" placeholder="Target price" style="width:100px" />
  <button onclick="addAlert()">Set Alert</button>
  <ul id="alertsList" class="alert-list"></ul>
</section>

<section id="about">
  <h2>About</h2>
  <p>This stock tracker uses <a href="https://twelvedata.com" target="_blank">Twelve Data API</a> for stock prices and <a href="https://newsapi.org" target="_blank">NewsAPI</a> for stock news.</p>
  <p>Created as a zero-setup multi-page app you can open anywhere. Just add stocks, track your portfolio, set alerts, and stay updated.</p>
</section>

<script>
  // Your API keys
  const TWELVE_API_KEY = "5ef5daf5f4454f209b8a74ecd5907d92";
  const NEWS_API_KEY = "fb972affa3a5480799e8ebebadc275ff";

  // Application state
  const stocks = JSON.parse(localStorage.getItem('stocks') || '[]');
  const portfolio = JSON.parse(localStorage.getItem('portfolio') || '[]');
  const alerts = JSON.parse(localStorage.getItem('alerts') || '[]');
  let prices = {};

  // Tab navigation
  const tabs = document.querySelectorAll('.tab-btn');
  const sections = document.querySelectorAll('section');

  tabs.forEach(tab => {
    tab.addEventListener('click', () => {
      tabs.forEach(t => t.classList.remove('active'));
      sections.forEach(s => s.classList.remove('active'));
      tab.classList.add('active');
      document.getElementById(tab.dataset.tab).classList.add('active');
      if(tab.dataset.tab === 'news') fetchNews();
      if(tab.dataset.tab === 'prices') renderStocks();
      if(tab.dataset.tab === 'recommendations') renderRecommendations();
      if(tab.dataset.tab === 'portfolio') renderPortfolio();
      if(tab.dataset.tab === 'alerts') renderAlerts();
    });
  });

  // Stocks Page
  const stocksContainer = document.getElementById("stocksContainer");

  function addStock() {
    const input = document.getElementById("stockInput");
    const sym = input.value.trim().toUpperCase();
    if (!sym) return alert("Please enter a stock symbol.");
    if (stocks.includes(sym)) return alert("Stock already added.");
    stocks.push(sym);
    saveStocks();
    input.value = "";
    fetchPrices().then(renderStocks);
  }

  function removeStock(sym) {
    const idx = stocks.indexOf(sym);
    if (idx > -1) {
      stocks.splice(idx, 1);
      saveStocks();
      delete prices[sym];
      renderStocks();
    }
  }

  function saveStocks() {
    localStorage.setItem('stocks', JSON.stringify(stocks));
  }

  function renderStocks() {
    stocksContainer.innerHTML = "";
    if(stocks.length === 0) {
      stocksContainer.innerHTML = "<p>No stocks tracked yet. Add some above.</p>";
      return;
    }
    for (const sym of stocks) {
      const box = document.createElement("div");
      box.className = "stock-box";
      box.innerHTML = `
        <button class="remove-btn" onclick="removeStock('${sym}')">X</button>
        <h3>${sym}</h3>
        <p>Current Price: <strong>${prices[sym] ? "$" + prices[sym] : "Loading..."}</strong></p>
      `;
      stocksContainer.appendChild(box);
    }
  }

  async function fetchPrices() {
    if (stocks.length === 0) return;
    try {
      const symbolStr = stocks.join(",");
      const url = `https://api.twelvedata.com/price?symbol=${symbolStr}&apikey=${TWELVE_API_KEY}`;
      const res = await fetch(url);
      const data = await res.json();

      if (stocks.length === 1) {
        prices[stocks[0]] = data.price || "N/A";
      } else {
        for (const sym of stocks) {
          prices[sym] = data[sym]?.price || "N/A";
        }
      }
    } catch (e) {
      console.error("Price fetch error:", e);
    }
  }

  // News Page
  const newsList = document.getElementById("newsList");

  async function fetchNews() {
    newsList.innerHTML = "<li>Loading news...</li>";
    if (stocks.length === 0) {
      newsList.innerHTML = "<li>No stocks tracked. Add some to see news.</li>";
      return;
    }
    const query = stocks.join(" OR ");
    try {
      const url = `https://newsapi.org/v2/everything?q=${encodeURIComponent(query)}&sortBy=publishedAt&pageSize=7&apiKey=${NEWS_API_KEY}`;
      const res = await fetch(url);
      const data = await res.json();

      if (!data.articles || data.articles.length === 0) {
        newsList.innerHTML = "<li>No news found.</li>";
        return;
      }

      newsList.innerHTML = "";
      for (const article of data.articles) {
        const li = document.createElement("li");
        li.innerHTML = `<a href="${article.url}" target="_blank" rel="noopener">${article.title}</a> <br/><small>${new Date(article.publishedAt).toLocaleString()}</small>`;
        newsList.appendChild(li);
      }
    } catch (e) {
      console.error("News fetch error:", e);
      newsList.innerHTML = "<li>Error loading news.</li>";
    }
  }

  // Recommendations Page
  function renderRecommendations() {
    const recList = document.getElementById("recList");
    // Basic popular stocks for demo
    const popularStocks = ["AAPL", "TSLA", "MSFT", "GOOGL", "AMZN", "NVDA", "META"];
    recList.innerHTML = "";
    popularStocks.forEach(sym => {
      const li = document.createElement("li");
      li.innerHTML = `<strong>${sym}</strong> - <button onclick="addRecStock('${sym}')">Add to Track</button>`;
      recList.appendChild(li);
    });
  }
  function addRecStock(sym) {
    if(!stocks.includes(sym)) {
      stocks.push(sym);
      saveStocks();
      fetchPrices().then(renderStocks);
      alert(sym + " added to your tracked stocks.");
    } else {
      alert(sym + " is already in your tracked stocks.");
    }
  }

  // Portfolio Page
  const portfolioTableBody = document.getElementById("portfolioTableBody");
  const portfolioTotalValueEl = document.getElementById("portfolioTotalValue");

  function addPortfolioStock() {
    const symInput = document.getElementById("portfolioStockInput");
    const sharesInput = document.getElementById("portfolioSharesInput");
    const sym = symInput.value.trim().toUpperCase();
    const shares = parseInt(sharesInput.value);
    if (!sym) return alert("Enter stock symbol.");
    if (!shares || shares <= 0) return alert("Enter a valid number of shares.");

    const existing = portfolio.find(p => p.symbol === sym);
    if(existing) {
      existing.shares += shares;
    } else {
      portfolio.push({ symbol: sym, shares });
    }
    savePortfolio();
    symInput.value = "";
    sharesInput.value = "";
    renderPortfolio();
  }

  function removePortfolioStock(sym) {
    const idx = portfolio.findIndex(p => p.symbol === sym);
    if (idx > -1) {
      portfolio.splice(idx, 1);
      savePortfolio();
      renderPortfolio();
    }
  }

  function savePortfolio() {
    localStorage.setItem('portfolio', JSON.stringify(portfolio));
  }

  async function renderPortfolio() {
    portfolioTableBody.innerHTML = "";
    if(portfolio.length === 0) {
      portfolioTableBody.innerHTML = `<tr><td colspan="5">No stocks in portfolio.</td></tr>`;
      portfolioTotalValueEl.textContent = "$0.00";
      return;
    }

    // Fetch prices for portfolio stocks
    const portfolioSymbols = portfolio.map(p => p.symbol);
    if(portfolioSymbols.length > 0) {
      try {
        const url = `https://api.twelvedata.com/price?symbol=${portfolioSymbols.join(",")}&apikey=${TWELVE_API_KEY}`;
        const res = await fetch(url);
        const data = await res.json();

        let totalValue = 0;
        for(const p of portfolio) {
          const price = data[p.symbol]?.price || (portfolioSymbols.length === 1 ? data.price : "N/A");
          const value = price !== "N/A" ? price * p.shares : 0;
          totalValue += value;

          const row = document.createElement("tr");
          row.innerHTML = `
            <td>${p.symbol}</td>
            <td>${p.shares}</td>
            <td>${price !== "N/A" ? "$" + price : "N/A"}</td>
            <td>${value ? "$" + value.toFixed(2) : "N/A"}</td>
            <td><button onclick="removePortfolioStock('${p.symbol}')">X</button></td>
          `;
          portfolioTableBody.appendChild(row);
        }
        portfolioTotalValueEl.textContent = "$" + totalValue.toFixed(2);
      } catch(e) {
        console.error("Portfolio price fetch error:", e);
      }
    }
  }

  // Alerts Page
  const alertsList = document.getElementById("alertsList");

  function addAlert() {
    const stockSym = document.getElementById("alertStockInput").value.trim().toUpperCase();
    const targetPrice = parseFloat(document.getElementById("alertPriceInput").value);
    if (!stockSym) return alert("Enter stock symbol.");
    if (!targetPrice || targetPrice <= 0) return alert("Enter valid target price.");

    alerts.push({ symbol: stockSym, price: targetPrice });
    saveAlerts();
    document.getElementById("alertStockInput").value = "";
    document.getElementById("alertPriceInput").value = "";
    renderAlerts();
  }

  function removeAlert(index) {
    alerts.splice(index,1);
    saveAlerts();
    renderAlerts();
  }

  function saveAlerts() {
    localStorage.setItem('alerts', JSON.stringify(alerts));
  }

  async function renderAlerts() {
    alertsList.innerHTML = "";
    if(alerts.length === 0) {
      alertsList.innerHTML = "<li>No alerts set.</li>";
      return;
    }

    // Check current prices against alerts
    const alertSymbols = [...new Set(alerts.map(a => a.symbol))];
    let currentPrices = {};
    try {
      const url = `https://api.twelvedata.com/price?symbol=${alertSymbols.join(",")}&apikey=${TWELVE_API_KEY}`;
      const res = await fetch(url);
      const data = await res.json();

      if(alertSymbols.length === 1) {
        currentPrices[alertSymbols[0]] = data.price || null;
      } else {
        for (const sym of alertSymbols) {
          currentPrices[sym] = data[sym]?.price || null;
        }
      }
    } catch(e) {
      console.error("Alert price fetch error:", e);
    }

    alerts.forEach((alert, i) => {
      const currentPrice = currentPrices[alert.symbol];
      let status = "Waiting...";
      if(currentPrice) {
        if(currentPrice >= alert.price) status = `✅ Target met! Current: $${currentPrice}`;
        else status = `Current: $${currentPrice}`;
      } else {
        status = "Price unavailable";
      }

      const li = document.createElement("li");
      li.innerHTML = `
        <strong>${alert.symbol}</strong> at $${alert.price} — ${status} 
        <button onclick="removeAlert(${i})" style="margin-left:10px;background:red;color:#fff;border:none;padding:0.2rem 0.5rem;border-radius:4px;cursor:pointer;">X</button>
      `;
      alertsList.appendChild(li);
    });
  }

  // Initial data load
  fetchPrices().then(() => {
    renderStocks();
    renderPortfolio();
  });

</script>

</body>
</html>
