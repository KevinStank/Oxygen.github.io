<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>OxyPulse — Wearable Dashboard</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
  <style>
    :root {
      --bg: #0d1117;
      --surface: #161b22;
      --surface2: #1c2330;
      --border: #30363d;
      --text: #e6edf3;
      --muted: #7d8590;
      --accent: #58a6ff;
      --green: #3fb950;
      --yellow: #d29922;
      --red: #f85149;
      --purple: #bc8cff;
      --cyan: #39d5d3;
    }

    * { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      background: var(--bg);
      color: var(--text);
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      min-height: 100vh;
      padding: 24px;
    }

    header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-bottom: 28px;
    }

    .brand { display: flex; align-items: center; gap: 10px; }

    .brand-icon {
      width: 36px; height: 36px;
      background: linear-gradient(135deg, var(--cyan), var(--accent));
      border-radius: 10px;
      display: flex; align-items: center; justify-content: center;
      font-size: 18px;
    }

    .brand h1 { font-size: 1.4rem; font-weight: 700; letter-spacing: -0.3px; }
    .brand span { font-size: 0.75rem; color: var(--muted); display: block; margin-top: 1px; }

    .status-pill {
      display: flex; align-items: center; gap: 7px;
      background: var(--surface); border: 1px solid var(--border);
      border-radius: 20px; padding: 6px 14px;
      font-size: 0.8rem; color: var(--muted);
    }

    .dot {
      width: 8px; height: 8px; border-radius: 50%;
      background: var(--green);
      animation: pulse-dot 2s infinite;
    }

    @keyframes pulse-dot {
      0%, 100% { opacity: 1; }
      50% { opacity: 0.3; }
    }

    .grid { display: grid; gap: 16px; }
    .row-4 { grid-template-columns: repeat(4, 1fr); }
    .row-2 { grid-template-columns: 2fr 1fr; }
    .row-3 { grid-template-columns: 1fr 1fr 1fr; }

    @media (max-width: 900px) {
      .row-4 { grid-template-columns: repeat(2, 1fr); }
      .row-2 { grid-template-columns: 1fr; }
      .row-3 { grid-template-columns: 1fr; }
    }
    @media (max-width: 500px) {
      .row-4 { grid-template-columns: 1fr 1fr; }
    }

    .card {
      background: var(--surface); border: 1px solid var(--border);
      border-radius: 12px; padding: 20px;
    }

    .card-label {
      font-size: 0.72rem; text-transform: uppercase;
      letter-spacing: 0.8px; color: var(--muted); margin-bottom: 8px;
    }

    .metric-value {
      font-size: 2.6rem; font-weight: 700;
      line-height: 1; letter-spacing: -1px;
    }

    .metric-unit { font-size: 0.85rem; color: var(--muted); margin-left: 4px; font-weight: 400; }
    .metric-sub { font-size: 0.78rem; color: var(--muted); margin-top: 6px; }

    .trend-up { color: var(--green); }
    .trend-down { color: var(--red); }
    .trend-neutral { color: var(--muted); }

    .spo2-value { color: var(--cyan); }
    .hr-value { color: var(--red); }
    .rr-value { color: var(--purple); }
    .hrv-value { color: var(--green); }

    .chart-card {
      background: var(--surface); border: 1px solid var(--border);
      border-radius: 12px; padding: 20px;
    }

    .chart-header {
      display: flex; justify-content: space-between;
      align-items: flex-start; margin-bottom: 16px;
    }

    .chart-title { font-size: 0.9rem; font-weight: 600; }
    .chart-subtitle { font-size: 0.72rem; color: var(--muted); margin-top: 2px; }

    .badge {
      font-size: 0.7rem; padding: 3px 9px;
      border-radius: 20px; font-weight: 500;
    }

    .badge-green  { background: rgba(63,185,80,0.15);  color: var(--green); }
    .badge-yellow { background: rgba(210,153,34,0.15); color: var(--yellow); }
    .badge-blue   { background: rgba(88,166,255,0.15); color: var(--accent); }

    .canvas-wrap { position: relative; height: 180px; }

    .divider { height: 1px; background: var(--border); margin: 16px 0; }

    .stat-row { display: flex; justify-content: space-between; font-size: 0.8rem; }
    .stat-row .label { color: var(--muted); }
    .stat-row .val { font-weight: 600; }

    .progress-ring-wrap {
      display: flex; justify-content: center; align-items: center; padding: 10px 0;
    }

    .ring-container { position: relative; width: 140px; height: 140px; }
    .ring-container svg { transform: rotate(-90deg); }

    .ring-center {
      position: absolute; top: 50%; left: 50%;
      transform: translate(-50%, -50%); text-align: center;
    }

    .ring-pct { font-size: 2rem; font-weight: 700; color: var(--cyan); line-height: 1; }
    .ring-label { font-size: 0.65rem; color: var(--muted); text-transform: uppercase; letter-spacing: 0.5px; }

    .sleep-bar-container { margin-top: 8px; }

    .sleep-row { display: flex; align-items: center; gap: 10px; margin-bottom: 8px; }

    .sleep-label { width: 60px; font-size: 0.72rem; color: var(--muted); flex-shrink: 0; }

    .sleep-bar-bg {
      flex: 1; height: 10px; background: var(--surface2);
      border-radius: 6px; overflow: hidden;
    }

    .sleep-bar-fill { height: 100%; border-radius: 6px; }
    .sleep-pct { width: 36px; font-size: 0.72rem; color: var(--muted); text-align: right; }

    .insight-list { list-style: none; display: flex; flex-direction: column; gap: 12px; margin-top: 4px; }

    .insight-item { display: flex; gap: 12px; align-items: flex-start; }

    .insight-icon { font-size: 1.2rem; flex-shrink: 0; margin-top: 1px; }

    .insight-text strong { display: block; font-size: 0.85rem; font-weight: 600; }
    .insight-text p { font-size: 0.78rem; color: var(--muted); margin-top: 2px; line-height: 1.5; }

    .timestamp { font-size: 0.7rem; color: var(--muted); text-align: right; margin-top: 20px; }

    footer { margin-top: 12px; text-align: center; font-size: 0.72rem; color: var(--muted); }
  </style>
</head>
<body>

<header>
  <div class="brand">
    <div class="brand-icon">💧</div>
    <div>
      <h1>OxyPulse</h1>
      <span>Wearable Health Dashboard</span>
    </div>
  </div>
  <div class="status-pill">
    <div class="dot"></div>
    Live &middot; <span id="sync-label">Synced just now</span>
  </div>
</header>

<!-- Top metric cards -->
<div class="grid row-4" style="margin-bottom:16px;">
  <div class="card">
    <div class="card-label">Blood Oxygen (SpO2)</div>
    <div class="metric-value spo2-value" id="spo2">98<span class="metric-unit">%</span></div>
    <div class="metric-sub trend-up">▲ +1% from yesterday</div>
  </div>
  <div class="card">
    <div class="card-label">Heart Rate</div>
    <div class="metric-value hr-value" id="hr">72<span class="metric-unit">bpm</span></div>
    <div class="metric-sub trend-neutral">— Resting range</div>
  </div>
  <div class="card">
    <div class="card-label">Respiratory Rate</div>
    <div class="metric-value rr-value" id="rr">16<span class="metric-unit">br/min</span></div>
    <div class="metric-sub trend-up">▲ Normal</div>
  </div>
  <div class="card">
    <div class="card-label">HRV Score</div>
    <div class="metric-value hrv-value" id="hrv">58<span class="metric-unit">ms</span></div>
    <div class="metric-sub trend-down">▼ −4ms this week</div>
  </div>
</div>

<!-- Live chart + score ring -->
<div class="grid row-2" style="margin-bottom:16px;">
  <div class="chart-card">
    <div class="chart-header">
      <div>
        <div class="chart-title">Oxygen & Heart Rate — Live (last 60s)</div>
        <div class="chart-subtitle">Updates every 2 seconds</div>
      </div>
      <span class="badge badge-green">Normal</span>
    </div>
    <div class="canvas-wrap">
      <canvas id="liveChart"></canvas>
    </div>
  </div>

  <div class="chart-card">
    <div class="chart-title" style="margin-bottom:12px;">Today's O2 Health Score</div>
    <div class="progress-ring-wrap">
      <div class="ring-container">
        <svg width="140" height="140" viewBox="0 0 140 140">
          <circle cx="70" cy="70" r="58" fill="none" stroke="#1c2330" stroke-width="14"/>
          <circle cx="70" cy="70" r="58" fill="none" stroke="url(#ringGrad)" stroke-width="14"
            stroke-dasharray="364.4" stroke-dashoffset="54.66" stroke-linecap="round"/>
          <defs>
            <linearGradient id="ringGrad" x1="0%" y1="0%" x2="100%" y2="0%">
              <stop offset="0%" stop-color="#39d5d3"/>
              <stop offset="100%" stop-color="#58a6ff"/>
            </linearGradient>
          </defs>
        </svg>
        <div class="ring-center">
          <div class="ring-pct">85</div>
          <div class="ring-label">out of 100</div>
        </div>
      </div>
    </div>
    <div class="divider"></div>
    <div class="stat-row"><span class="label">Avg SpO2 today</span><span class="val spo2-value">97.4%</span></div>
    <div style="height:6px;"></div>
    <div class="stat-row"><span class="label">Min SpO2 (sleep)</span><span class="val" style="color:var(--yellow)">93%</span></div>
    <div style="height:6px;"></div>
    <div class="stat-row"><span class="label">Time below 95%</span><span class="val">3 min</span></div>
    <div style="height:6px;"></div>
    <div class="stat-row"><span class="label">Resting HR avg</span><span class="val hr-value">64 bpm</span></div>
  </div>
</div>

<!-- 24h trend / sleep / insights -->
<div class="grid row-3" style="margin-bottom:16px;">

  <div class="chart-card">
    <div class="chart-header">
      <div>
        <div class="chart-title">24-Hour SpO2 Trend</div>
        <div class="chart-subtitle">Hourly average · today</div>
      </div>
      <span class="badge badge-blue">Tracked</span>
    </div>
    <div class="canvas-wrap">
      <canvas id="trend24h"></canvas>
    </div>
  </div>

  <div class="chart-card">
    <div class="chart-header">
      <div>
        <div class="chart-title">Sleep O2 Quality</div>
        <div class="chart-subtitle">Last night · 7h 22min</div>
      </div>
      <span class="badge badge-yellow">Fair</span>
    </div>
    <div style="height:8px;"></div>
    <div class="sleep-bar-container">
      <div class="sleep-row">
        <div class="sleep-label">≥ 97%</div>
        <div class="sleep-bar-bg"><div class="sleep-bar-fill" style="width:61%;background:var(--cyan);"></div></div>
        <div class="sleep-pct">61%</div>
      </div>
      <div class="sleep-row">
        <div class="sleep-label">95–96%</div>
        <div class="sleep-bar-bg"><div class="sleep-bar-fill" style="width:27%;background:var(--green);"></div></div>
        <div class="sleep-pct">27%</div>
      </div>
      <div class="sleep-row">
        <div class="sleep-label">93–94%</div>
        <div class="sleep-bar-bg"><div class="sleep-bar-fill" style="width:9%;background:var(--yellow);"></div></div>
        <div class="sleep-pct">9%</div>
      </div>
      <div class="sleep-row">
        <div class="sleep-label">&lt; 93%</div>
        <div class="sleep-bar-bg"><div class="sleep-bar-fill" style="width:3%;background:var(--red);"></div></div>
        <div class="sleep-pct">3%</div>
      </div>
    </div>
    <div class="divider"></div>
    <div class="stat-row"><span class="label">Dip events (&lt;94%)</span><span class="val">2</span></div>
    <div style="height:6px;"></div>
    <div class="stat-row"><span class="label">Longest dip</span><span class="val" style="color:var(--yellow)">4 min 10s</span></div>
    <div style="height:6px;"></div>
    <div class="stat-row"><span class="label">Avg nightly SpO2</span><span class="val">96.2%</span></div>
  </div>

  <div class="chart-card">
    <div class="chart-title" style="margin-bottom:14px;">Health Insights</div>
    <ul class="insight-list">
      <li class="insight-item">
        <div class="insight-icon">✅</div>
        <div class="insight-text">
          <strong>Daytime O2 Excellent</strong>
          <p>Your waking SpO2 averaged 98.1% — well above the 95% threshold.</p>
        </div>
      </li>
      <li class="insight-item">
        <div class="insight-icon">⚠️</div>
        <div class="insight-text">
          <strong>Two Sleep Dips Detected</strong>
          <p>SpO2 dropped below 94% twice during deep sleep. Try adjusting sleep position.</p>
        </div>
      </li>
      <li class="insight-item">
        <div class="insight-icon">💙</div>
        <div class="insight-text">
          <strong>HRV Trending Down</strong>
          <p>7-day declining HRV may indicate accumulated stress. Prioritize sleep tonight.</p>
        </div>
      </li>
      <li class="insight-item">
        <div class="insight-icon">🏃</div>
        <div class="insight-text">
          <strong>Post-Exercise Recovery</strong>
          <p>O2 returned to baseline within 90s of today's workout — excellent fitness marker.</p>
        </div>
      </li>
    </ul>
  </div>

</div>

<!-- 7-day bar chart -->
<div class="chart-card" style="margin-bottom:16px;">
  <div class="chart-header">
    <div>
      <div class="chart-title">7-Day SpO2 Average</div>
      <div class="chart-subtitle">Daily average blood oxygen — this week</div>
    </div>
    <span class="badge badge-green">Healthy Range</span>
  </div>
  <div class="canvas-wrap" style="height:140px;">
    <canvas id="weekChart"></canvas>
  </div>
</div>

<div class="timestamp" id="timestamp"></div>
<footer>OxyPulse Wearable Dashboard &middot; Simulated data for demonstration &middot; Not a medical device</footer>

<script>
  // ── Helpers ───────────────────────────────────────────────────────────────
  function rand(min, max) { return Math.round(Math.random() * (max - min) + min); }
  function clamp(v, min, max) { return Math.min(max, Math.max(min, v)); }

  let syncSeconds = 0;
  function updateTimestamp() {
    syncSeconds += 2;
    const s = syncSeconds;
    document.getElementById('sync-label').textContent =
      s < 10 ? 'Synced just now' : 'Synced ' + s + 's ago';
    document.getElementById('timestamp').textContent =
      'Last updated: ' + new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit', second: '2-digit' });
  }

  // ── Live state ────────────────────────────────────────────────────────────
  let spo2 = 98, hr = 72, rr = 16, hrv = 58;
  const MAX = 30;
  const spo2Data = Array.from({ length: MAX }, () => rand(96, 99));
  const hrData   = Array.from({ length: MAX }, () => rand(68, 78));

  // ── Live chart ────────────────────────────────────────────────────────────
  const liveChart = new Chart(document.getElementById('liveChart'), {
    type: 'line',
    data: {
      labels: Array(MAX).fill(''),
      datasets: [
        {
          label: 'SpO2 (%)',
          data: spo2Data,
          borderColor: '#39d5d3',
          backgroundColor: 'rgba(57,213,211,0.08)',
          borderWidth: 2, pointRadius: 0, tension: 0.4,
          yAxisID: 'ySpo2', fill: true,
        },
        {
          label: 'Heart Rate (bpm)',
          data: hrData,
          borderColor: '#f85149',
          backgroundColor: 'rgba(248,81,73,0.06)',
          borderWidth: 2, pointRadius: 0, tension: 0.4,
          yAxisID: 'yHr', fill: true,
        }
      ]
    },
    options: {
      animation: false, responsive: true, maintainAspectRatio: false,
      interaction: { mode: 'index', intersect: false },
      plugins: {
        legend: { labels: { color: '#7d8590', font: { size: 11 }, boxWidth: 12 } },
        tooltip: { backgroundColor: '#1c2330', borderColor: '#30363d', borderWidth: 1, titleColor: '#e6edf3', bodyColor: '#7d8590' }
      },
      scales: {
        x: { display: false },
        ySpo2: {
          position: 'left', min: 90, max: 101,
          grid: { color: '#1c2330' },
          ticks: { color: '#7d8590', font: { size: 10 }, callback: v => v + '%' }
        },
        yHr: {
          position: 'right', min: 50, max: 120,
          grid: { drawOnChartArea: false },
          ticks: { color: '#7d8590', font: { size: 10 }, callback: v => v + ' bpm' }
        }
      }
    }
  });

  // ── 24h trend ─────────────────────────────────────────────────────────────
  const now = new Date();
  const hours24 = Array.from({ length: 24 }, (_, i) => {
    const h = (now.getHours() - 23 + i + 24) % 24;
    return h + ':00';
  });
  // Sleep dip in early hours, healthy during day
  const spo2_24h = [93,94,94,93,95,96,97,97,98,98,98,99,98,98,97,98,99,98,97,98,98,97,98,98];

  new Chart(document.getElementById('trend24h'), {
    type: 'line',
    data: {
      labels: hours24,
      datasets: [{
        label: 'Avg SpO2',
        data: spo2_24h,
        borderColor: '#58a6ff',
        backgroundColor: 'rgba(88,166,255,0.1)',
        borderWidth: 2, pointRadius: 2, pointBackgroundColor: '#58a6ff',
        tension: 0.4, fill: true,
      }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: {
        legend: { display: false },
        tooltip: {
          backgroundColor: '#1c2330', borderColor: '#30363d', borderWidth: 1,
          titleColor: '#e6edf3', bodyColor: '#7d8590',
          callbacks: { label: ctx => 'SpO2: ' + ctx.raw + '%' }
        }
      },
      scales: {
        x: { ticks: { color: '#7d8590', font: { size: 9 }, maxRotation: 0, autoSkip: true, maxTicksLimit: 8 }, grid: { color: '#1c2330' } },
        y: { min: 89, max: 101, ticks: { color: '#7d8590', font: { size: 10 }, callback: v => v + '%' }, grid: { color: '#1c2330' } }
      }
    }
  });

  // ── Weekly bar chart ──────────────────────────────────────────────────────
  const weekAvgs = [97.2, 97.8, 96.9, 98.1, 97.5, 97.4, 98.0];
  new Chart(document.getElementById('weekChart'), {
    type: 'bar',
    data: {
      labels: ['Mon','Tue','Wed','Thu','Fri','Sat','Sun'],
      datasets: [{
        label: 'Avg SpO2',
        data: weekAvgs,
        backgroundColor: weekAvgs.map(v =>
          v >= 97.5 ? 'rgba(57,213,211,0.8)' : v >= 96 ? 'rgba(88,166,255,0.8)' : 'rgba(210,153,34,0.8)'
        ),
        borderRadius: 6, borderSkipped: false,
      }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: {
        legend: { display: false },
        tooltip: {
          backgroundColor: '#1c2330', borderColor: '#30363d', borderWidth: 1,
          titleColor: '#e6edf3', bodyColor: '#7d8590',
          callbacks: { label: ctx => 'SpO2: ' + ctx.raw + '%' }
        }
      },
      scales: {
        x: { ticks: { color: '#7d8590', font: { size: 11 } }, grid: { display: false } },
        y: { min: 93, max: 100, ticks: { color: '#7d8590', font: { size: 10 }, callback: v => v + '%' }, grid: { color: '#1c2330' } }
      }
    }
  });

  // ── Live update loop ──────────────────────────────────────────────────────
  function tick() {
    spo2 = clamp(spo2 + rand(-1, 1), 95, 100);
    hr   = clamp(hr   + rand(-3, 3), 58, 95);
    rr   = clamp(rr   + rand(-1, 1), 12, 20);
    hrv  = clamp(hrv  + rand(-2, 2), 30, 90);

    document.getElementById('spo2').innerHTML = spo2 + '<span class="metric-unit">%</span>';
    document.getElementById('hr').innerHTML   = hr   + '<span class="metric-unit">bpm</span>';
    document.getElementById('rr').innerHTML   = rr   + '<span class="metric-unit">br/min</span>';
    document.getElementById('hrv').innerHTML  = hrv  + '<span class="metric-unit">ms</span>';

    spo2Data.push(spo2); hrData.push(hr);
    if (spo2Data.length > MAX) { spo2Data.shift(); hrData.shift(); }
    liveChart.update();
    updateTimestamp();
  }

  updateTimestamp();
  setInterval(tick, 2000);
</script>
</body>
</html>
