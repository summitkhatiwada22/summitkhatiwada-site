---
layout: page
title: tracker
permalink: /tracker/
nav: true
nav_order: 2
description: Live data from the AI Compute Tracker — GPU pricing, hyperscaler capex, and AI funding, updated daily.
_styles: |
  .post-title { display: none; }
  .viz-root {
    color-scheme: light;
    --surface-1: #fcfcfb;
    --text-primary: #0b0b0b;
    --text-secondary: #52514e;
    --text-muted: #898781;
    --grid-line: #e1e0d9;
    --series-1: #2a78d6;
    --series-2: #eb6834;
  }
  @media (prefers-color-scheme: dark) {
    :root:where(:not([data-theme="light"])) .viz-root {
      color-scheme: dark;
      --surface-1: #1a1a19;
      --text-primary: #ffffff;
      --text-secondary: #c3c2b7;
      --text-muted: #898781;
      --grid-line: #2c2c2a;
      --series-1: #3987e5;
      --series-2: #d95926;
    }
  }
  :root[data-theme="dark"] .viz-root {
    color-scheme: dark;
    --surface-1: #1a1a19;
    --text-primary: #ffffff;
    --text-secondary: #c3c2b7;
    --text-muted: #898781;
    --grid-line: #2c2c2a;
    --series-1: #3987e5;
    --series-2: #d95926;
  }
  .viz-root { margin-top: 1rem; }
  .viz-section { margin-bottom: 3rem; }
  .viz-section h2 { font-size: 1.3rem; margin-bottom: 0.25rem; }
  .viz-note { color: var(--text-secondary); font-size: 1rem; margin-bottom: 1rem; }
  .viz-card {
    background: var(--surface-1);
    border-radius: 8px;
    padding: 1rem;
  }
  .viz-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
    gap: 1rem;
  }
  .viz-status { color: var(--text-muted); font-size: 1rem; }
  .viz-chart-wrap { position: relative; height: 300px; }
  .viz-mini-chart-wrap { position: relative; height: 150px; margin-top: 0.5rem; }
  .viz-live {
    display: inline-flex;
    align-items: center;
    gap: 0.4rem;
    font-size: 1rem;
    color: var(--text-secondary);
    margin-bottom: 0.75rem;
  }
  .viz-live-dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background: #0ca30c;
    animation: viz-pulse 1.8s infinite;
  }
  @keyframes viz-pulse {
    0% { box-shadow: 0 0 0 0 rgba(12, 163, 12, 0.5); }
    70% { box-shadow: 0 0 0 6px rgba(12, 163, 12, 0); }
    100% { box-shadow: 0 0 0 0 rgba(12, 163, 12, 0); }
  }
---

<div class="viz-root">

<p class="viz-note">
  Data from the <a href="https://github.com/summitkhatiwada22/ai-compute-tracker" target="_blank" rel="noopener">AI Compute Tracker</a>,
  updated daily. See the <a href="https://github.com/summitkhatiwada22/ai-compute-tracker#methodology-notes" target="_blank" rel="noopener">methodology notes</a>
  for what these numbers do and don't mean.
</p>

<div class="viz-section">
  <h2>GPU pricing — marketplace vs. neocloud</h2>
  <div class="viz-live"><span class="viz-live-dot"></span> Live — reflects the tracker's latest automated run</div>
  <p class="viz-note">Median on-demand $/hr, restricted to GPU models listed on <em>both</em> tiers that day (so this is the same hardware priced two ways, not two different catalogs blended together). Marketplace (Vast.ai) is the observable retail floor; neocloud (Lambda Cloud) is published mid-tier list pricing. Neither reflects undisclosed hyperscaler-negotiated bulk pricing.</p>
  <div class="viz-card">
    <div class="viz-chart-wrap"><canvas id="gpu-pricing-chart"></canvas></div>
  </div>
  <p class="viz-status" id="gpu-pricing-status">Loading…</p>
</div>

<div class="viz-section">
  <h2>Hyperscaler capex</h2>
  <div class="viz-live"><span class="viz-live-dot"></span> Live — reflects the tracker's latest automated run</div>
  <p class="viz-note">Total company-wide capex per quarter, from SEC EDGAR structured filing data — not an AI-specific breakout (no such breakout exists in structured filings).</p>
  <div class="viz-grid" id="capex-grid"></div>
  <p class="viz-status" id="capex-status">Loading…</p>
</div>

<div class="viz-section">
  <h2>AI sector funding</h2>
  <div class="viz-live"><span class="viz-live-dot"></span> Live — reflects the tracker's latest automated run</div>
  <p class="viz-note">Latest Dealroom market-map snapshot, ranked by tracked company count. <code>sample_funding_usd</code> is a capped-sample lower bound, not a true sector total — see methodology.</p>
  <div class="viz-card">
    <div class="viz-chart-wrap"><canvas id="funding-chart"></canvas></div>
  </div>
  <p class="viz-status" id="funding-status">Loading…</p>
</div>

</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js@4/dist/chart.umd.min.js"></script>
<script>
(function () {
  const BASE = "https://raw.githubusercontent.com/summitkhatiwada22/ai-compute-tracker/main/data/public/";

  function themeVars() {
    const root = getComputedStyle(document.querySelector(".viz-root"));
    return {
      text: root.getPropertyValue("--text-secondary").trim(),
      grid: root.getPropertyValue("--grid-line").trim(),
      series1: root.getPropertyValue("--series-1").trim(),
      series2: root.getPropertyValue("--series-2").trim(),
    };
  }

  function commonScales(vars) {
    return {
      x: { grid: { color: vars.grid }, ticks: { color: vars.text } },
      y: { grid: { color: vars.grid }, ticks: { color: vars.text }, beginAtZero: true },
    };
  }

  // --- GPU pricing: marketplace vs neocloud -------------------------------
  fetch(BASE + "gpu_pricing_overall.json")
    .then((r) => r.json())
    .then((rows) => {
      const vars = themeVars();
      const dates = [...new Set(rows.map((r) => r.date))].sort();
      const byTier = (tier) =>
        dates.map((d) => {
          const row = rows.find((r) => r.date === d && r.tier === tier);
          return row ? row.median_price_usd_per_hr : null;
        });

      new Chart(document.getElementById("gpu-pricing-chart"), {
        type: "line",
        data: {
          labels: dates,
          datasets: [
            {
              label: "Marketplace (Vast.ai)",
              data: byTier("marketplace"),
              borderColor: vars.series1,
              backgroundColor: vars.series1,
              borderWidth: 2,
              pointRadius: 0,
              tension: 0.15,
            },
            {
              label: "Neocloud (Lambda Cloud)",
              data: byTier("neocloud"),
              borderColor: vars.series2,
              backgroundColor: vars.series2,
              borderWidth: 2,
              pointRadius: 0,
              tension: 0.15,
            },
          ],
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          interaction: { mode: "index", intersect: false },
          plugins: { legend: { labels: { color: vars.text } } },
          scales: commonScales(vars),
        },
      });
      document.getElementById("gpu-pricing-status").textContent =
        `${rows.length} daily data point(s) across ${dates.length} day(s).`;
    })
    .catch(() => {
      document.getElementById("gpu-pricing-status").textContent =
        "Couldn't load GPU pricing data right now.";
    });

  // --- Capex: one small chart per company ---------------------------------
  fetch(BASE + "hyperscaler_financials.json")
    .then((r) => r.json())
    .then((rows) => {
      const vars = themeVars();
      const capex = rows.filter((r) => r.metric === "capex");
      const companies = [...new Set(capex.map((r) => r.company))];
      const grid = document.getElementById("capex-grid");

      companies.forEach((company) => {
        const companyRows = capex
          .filter((r) => r.company === company)
          .sort((a, b) => (a.period_end > b.period_end ? 1 : -1));

        const card = document.createElement("div");
        card.className = "viz-card";
        card.innerHTML = `<strong style="color:${vars.text}">${company}</strong><div class="viz-mini-chart-wrap"><canvas></canvas></div>`;
        grid.appendChild(card);

        new Chart(card.querySelector("canvas"), {
          type: "bar",
          data: {
            labels: companyRows.map((r) => r.period_end),
            datasets: [
              {
                data: companyRows.map((r) => r.value_usd),
                backgroundColor: vars.series1,
                borderRadius: 4,
              },
            ],
          },
          options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: { legend: { display: false } },
            scales: {
              x: { display: false },
              y: { grid: { color: vars.grid }, ticks: { color: vars.text, callback: (v) => "$" + (v / 1e9).toFixed(1) + "B" } },
            },
          },
        });
      });

      document.getElementById("capex-status").textContent =
        `${companies.length} companies, ${capex.length} disclosed quarter(s) total.`;
    })
    .catch(() => {
      document.getElementById("capex-status").textContent =
        "Couldn't load capex data right now.";
    });

  // --- Funding: latest snapshot bar chart ----------------------------------
  fetch(BASE + "funding_latest.json")
    .then((r) => r.json())
    .then((rows) => {
      const vars = themeVars();
      const top = rows.slice(0, 12);

      new Chart(document.getElementById("funding-chart"), {
        type: "bar",
        data: {
          labels: top.map((r) => r.market_map_title),
          datasets: [
            {
              label: "Tracked companies",
              data: top.map((r) => r.total_companies),
              backgroundColor: vars.series1,
              borderRadius: 4,
            },
          ],
        },
        options: {
          indexAxis: "y",
          responsive: true,
          maintainAspectRatio: false,
          plugins: { legend: { display: false } },
          scales: {
            x: { title: { display: true, text: "Companies tracked", color: vars.text }, grid: { color: vars.grid }, ticks: { color: vars.text }, beginAtZero: true },
            y: { grid: { display: false }, ticks: { color: vars.text } },
          },
        },
      });
      const latest = rows[0] ? rows[0].collected_at : "unknown";
      document.getElementById("funding-status").textContent =
        `${rows.length} market map(s), snapshot from ${latest}.`;
    })
    .catch(() => {
      document.getElementById("funding-status").textContent =
        "Couldn't load funding data right now.";
    });
})();
</script>