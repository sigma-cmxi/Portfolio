# Portfolio Tracker

A personal, single-file portfolio tracker (dark mode). It tracks **three funds** in one view,
converts everything with a **live USD→EGP rate**, and shows invested / value / profit-loss
against my target allocation, with allocation, rebalancing and history charts.

| Fund | Ticker | Target | Currency |
|------|--------|-------:|----------|
| S&P World ex-US Sharia   | `SPWO` | 50% | USD |
| AZ Sharia Opportunities | `ASO`  | 35% | EGP |
| AZ Gold                 | `AZG`  | 15% | EGP |

- Files: only **`index.html`** (the whole app + the data) and this **`README.md`**.
- **EGP amounts are shown as a plain number; USD amounts keep the `$` symbol.** Switch the
  whole display between EGP and USD with the toggle in the header.
- I update the numbers by asking **Claude**; Claude edits the data and pushes to GitHub; **GitHub Pages** redeploys.

> ⚠️ **Privacy:** GitHub Pages on a **free** plan only serves **public** repos. That means
> anyone with the URL — and anyone browsing the repo on github.com — can see these numbers.
> If the amounts are sensitive, either keep `totalInvested`/values as small test figures,
> or move to a host that serves private repos (Cloudflare Pages, Netlify, or GitHub Pro Pages).
> The page already sends `noindex, nofollow` so search engines skip it, but the repo source is still public.

---

## What's on each screen

- **Dashboard** — total invested, current value, profit/loss (with an **annualized
  money-weighted return / XIRR** once there are 2+ months), and the change since last month;
  an allocation donut vs target; a **rebalance** panel (how much to add/trim per fund to hit
  target); a card per fund; and three time charts (invested vs value, monthly P/L, composition).
  Hover any time chart to see exact values for that month.
- **Funds** — a table for the latest month: invested, value, P/L and weight vs target per fund.
- **History** — one row per month: invested, value, P/L, return %, and the Δ in value vs the
  previous month.

---

## For Claude — how to update the data

All numbers live in **one `DATA` object** near the top of the `<script>` in `index.html`.

`DATA.funds` defines the three funds **once** (ticker, name, currency, target %, colour). The
target percentages must sum to **100**. You normally don't touch this.

Each **month** object has just four numbers:

| field | meaning |
|-------|---------|
| `month` | `"YYYY-MM"` |
| `totalInvested` | total cash I've put in, in **EGP**. Split **automatically** by each fund's target % |
| `values.spwo` | current value of S&P World ex-US Sharia (**USD**) |
| `values.aso`  | current market value of AZ Sharia Opportunities (**EGP**) |
| `values.azg`  | current market value of AZ Gold (**EGP**) |

The app derives everything else: invested per fund = `totalInvested × target%`,
profit/loss = value − invested, weights, rebalancing, and EGP↔USD conversion.

> `DATA.months` currently starts **empty** (the app shows "No data yet"). For the **first**
> month, uncomment the template object inside `DATA.months` and fill it in. After that, follow
> the normal flow below.

**Normal monthly update:**
1. Copy the **last** object in `DATA.months` (or the template, for the first month).
2. Change `month` to the new `"YYYY-MM"`.
3. Set the new `totalInvested` (EGP).
4. Set the three `values`:
   - **SPWO** → `values.spwo` (USD)
   - **ASO**  → `values.aso` (EGP "Market value")
   - **AZG**  → `values.azg` (EGP "Market value")
5. Commit + push (below).

> The USD→EGP rate is fetched live (open.er-api.com + fallbacks) and cached locally; it
> auto-refreshes when the tab regains focus (if not current) and on a slow 6-hour heartbeat.
> Click the rate pill to refresh on demand. `DATA.fallbackRate` is used only when offline.

---

## Push to GitHub → auto-deploy (GitHub Pages)

After editing the data:

```bash
git add -A
git commit -m "Update portfolio: <month>"
git push
```

GitHub Pages rebuilds the site on every push to the deployed branch (usually `main`),
so within a minute the same URL shows the latest data on any device.

### One-time deploy setup (GitHub Pages, free)
1. Repo on GitHub: `sigma-cmxi/Portfolio` — must be **public** for free Pages (see the privacy note above).
2. Repo **Settings → Pages**.
3. **Source:** *Deploy from a branch* · **Branch:** `main` · **Folder:** `/ (root)` → **Save**.
4. Wait ~1 min, then open the URL shown there: `https://sigma-cmxi.github.io/Portfolio/`.
5. Done — future `git push`es to `main` redeploy automatically.

> No build step is needed: `index.html` is served as-is. There's nothing to configure for a build command.
