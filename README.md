# GopexFund — System Documentation

> Africa's Social Investing Platform · Built for Nigerian & African investors
> Version 1.0 · May 2026

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Tech Stack](#2-tech-stack)
3. [File Structure](#3-file-structure)
4. [Pages & Features](#4-pages--features)
5. [Database Schema](#5-database-schema)
6. [Authentication System](#6-authentication-system)
7. [Live Stock Price System](#7-live-stock-price-system)
8. [Social Feed System](#8-social-feed-system)
9. [Portfolio System](#9-portfolio-system)
10. [Supabase Configuration](#10-supabase-configuration)
11. [How to Deploy](#11-how-to-deploy)
12. [Environment Variables](#12-environment-variables)
13. [Known Limitations & Roadmap](#13-known-limitations--roadmap)

---

## 1. Project Overview

GopexFund is a social wealth-building platform for African investors. It combines:

- **Live NGX market data** — 124+ Nigerian Exchange stocks updated every 10 minutes automatically
- **Social investing feed** — Posts, likes, comments, follows with real-time updates
- **Portfolio tracker** — Holdings management with live P&L calculations
- **Learn & Earn** — Investing lessons with quizzes and a points/leaderboard system
- **Privacy-first design** — Portfolio ₦ values are always private; only % gains can be shared publicly

The entire app is built with **vanilla HTML, CSS, and JavaScript** — no frameworks, no build tools. The backend is powered entirely by **Supabase** (PostgreSQL + Auth + Real-time + pg_cron).

---

## 2. Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Vanilla HTML5, CSS3, JavaScript (ES Modules) |
| Fonts | Plus Jakarta Sans (headings), Inter (body) — Google Fonts |
| Charts | Chart.js 4.4.1 (CDN) |
| Backend / Database | Supabase (PostgreSQL) |
| Authentication | Supabase Auth (email + password) |
| Real-time | Supabase Realtime (postgres_changes) |
| Scheduled Jobs | pg_cron (built into Supabase) |
| HTTP Fetching | PostgreSQL http extension (for price scraping) |
| Stock Data Source | afx.kwayisi.org (NGX live prices, scraped via pg_cron) |
| Hosting | Any static file host (Vercel, Netlify, GitHub Pages, Cloudflare Pages) |

**No Node.js. No npm. No build step. Open any `.html` file in a browser and it works.**

---

## 3. File Structure

```
gopexfund/
│
├── landing.html          ← Public homepage (sign in / sign up)
├── home.html             ← Social feed (protected)
├── market.html           ← NGX market data (protected)
├── portfolio.html        ← Portfolio tracker (protected)
├── profile.html          ← User profile (protected)
├── learn.html            ← Learn & Earn (protected)
├── about.html            ← About page (protected)
│
├── load-all-ngx-stocks.sql   ← One-time SQL to seed 124 NGX stocks
├── DEPLOY-GUIDE.txt          ← Edge Function deployment guide (optional)
├── refresh-prices.ts         ← Edge Function code (optional alternative)
├── get-prices.ts             ← Edge Function code (optional alternative)
│
└── README.md                 ← This file
```

### Page Access Rules

| Page | Public? | Requires Login? |
|------|---------|----------------|
| `landing.html` | ✅ Yes | No |
| `home.html` | ❌ No | Yes |
| `market.html` | ❌ No | Yes |
| `portfolio.html` | ❌ No | Yes |
| `profile.html` | ❌ No | Yes |
| `learn.html` | ❌ No | Yes |
| `about.html` | ❌ No | Yes |

---

## 4. Pages & Features

### `landing.html` — Public Homepage
The first page every visitor sees. Fully public.

**Sections:**
- **Navbar** — Logo, Features / About / Contact links, Sign In + Get Started buttons
- **Hero** — Animated colorful blob background, headline, trust stats, Sign In / Sign Up form card
- **Features** — 6 pastel feature cards explaining the platform
- **How It Works** — 3-step process with colored icons
- **About Us** — Mission, Vision, Values cards + stats banner
- **Contact** — Email form with confirmation
- **Footer** — Links and copyright

**Auth Flow:**
- Sign Up → creates Supabase Auth account + inserts into `profiles` table → redirects to `home.html`
- Sign In → authenticates with Supabase → redirects to `home.html`
- If already logged in when landing.html loads → auto-redirects to `home.html`

---

### `home.html` — Social Feed
The main social experience after login.

**Features:**
- **Left sidebar** — Portfolio balance, navigation links, wealth goals widget
- **Center feed** — Tabs: For You / Following / News
- **Compose box** — Write posts, tag NGX stock tickers, share % portfolio gain/loss
- **Post cards** — Like, comment, follow, emoji reactions, share
- **Ticker chips** — Each tagged stock shows live price + "I hold X% of portfolio" or "Not in my portfolio"
- **Portfolio performance badge** — Optional: shows author's % gain (never ₦ value)
- **Right sidebar** — Suggested investors, trending hashtags

**Real-time:** New posts appear live via Supabase Realtime without page refresh.

---

### `market.html` — NGX Market
Live Nigerian Exchange market data.

**Features:**
- **Scrolling ticker tape** — All stocks scrolling across the top
- **Overview cards** — ASI index, Market Cap, Volume, Breadth, USD/NGN rate
- **Fear & Greed gauge** — Visual semicircle gauge (0–100)
- **Market Breadth panel** — Gainers/losers count, sector indices
- **Top Movers** — Top 7 gainers and top 6 losers with % change badges
- **Sector browser** — 10 sectors, click any to expand full stock table
- **Stock detail panel** — Slide-in panel with 5-period chart, 8 stats, description, watchlist button
- **Market Intelligence** — Volume leaders, news, upcoming dividends

**Prices update automatically** — Every time the page loads, it fetches fresh prices from the `stock_prices` table. The table itself is updated by pg_cron every 10 minutes.

---

### `portfolio.html` — Portfolio Tracker
Personal portfolio management.

**Features:**
- **Hero banner** — Total portfolio value, P&L chip, period toggle (1D/1W/1M/All), mini stats
- **Holdings table** — Add/remove stocks, shows live price, 1D change, market value, P&L%
- **Add holding form** — Inline form: ticker, company name, shares, buy price
- **Allocation donut chart** — Chart.js pie showing % allocation per stock
- **Performance chart** — Line chart over 1M/3M/6M/1Y/All periods
- **Wealth Goals** — 3 default goals (Japa Fund, First Car, House Fund) with progress bars
- **Brokerage Connect** — 6 Nigerian brokerages shown as **Coming Soon** (Bamboo, Trove, Chaka, Meristem, Afrinvest, Risevest)
- **Verification badge** — Shows "Unverified — Broker sync coming soon" for all users currently

**Data persistence:** Holdings saved to `localStorage` AND synced to Supabase `holdings` table on login.

> ⚠️ **Note:** Brokerage API integration is not yet live. All 6 broker connect buttons show "Coming Soon". This will be enabled in a future release when broker API agreements are in place.

---

### `profile.html` — User Profile
Public-facing user profile with private editing.

**Features:**
- Cover banner (gradient), avatar with initials, verification badge
- Stats row: Portfolio %, Total Return, Followers, Points, Streak
- Tabs: Posts / Portfolio snapshot / Activity
- About sidebar + Badges sidebar
- Edit Profile modal (name + bio → saves to Supabase `profiles` table)
- Logout button (signs out of Supabase, clears session)

---

### `learn.html` — Learn & Earn
Gamified investing education.

**Features:**
- **Hero strip** — Points balance, 5-day streak, today's goal progress bar
- **Lesson cards** — Expandable with inline quiz questions
- **Quiz system** — Correct answer adds +100 points to balance AND updates leaderboard live
- **Right sidebar** — Leaderboard (5 users, "You" highlighted) + badges grid
- **Badges** — 8 badges, 3 earned by default (Streak, Learner, Saver)

---

### `about.html` — About Page
Company information. Requires login (protected page).

---

## 5. Database Schema

All tables are in the `public` schema in Supabase PostgreSQL.

### `profiles`
Extends Supabase `auth.users`. Created automatically on signup via trigger.

```sql
id                  uuid (PK, references auth.users)
name                text
first_name          text
initials            text
bio                 text
location            text
points              integer (default 0)
streak              integer (default 0)
is_verified         boolean (default false)
share_performance   boolean (default false)
portfolio_pct_gain  numeric
portfolio_pct_period text
followers_count     integer (default 0)
following_count     integer (default 0)
avatar_url          text
joined_at           timestamptz
```

### `holdings`
Portfolio holdings per user.

```sql
id            uuid (PK)
user_id       uuid (FK → profiles)
ticker        text
company_name  text
shares        numeric
buy_price     numeric
created_at    timestamptz
```

### `goals`
Wealth savings goals.

```sql
id                    uuid (PK)
user_id               uuid (FK → profiles)
name                  text
icon                  text
target_amount         numeric
saved_amount          numeric
monthly_contribution  numeric
created_at            timestamptz
```

### `posts`
Social feed posts.

```sql
id                uuid (PK)
user_id           uuid (FK → profiles)
body              text
tickers           text[]
ticker_holdings   jsonb   ← [{ticker, i_hold, my_pct}]
portfolio_pct_gain numeric ← optional, user opt-in only
portfolio_period  text
likes_count       integer (auto-updated by trigger)
comments_count    integer (auto-updated by trigger)
image_url         text
tags              text[]
created_at        timestamptz
```

### `comments`
Comments on posts.

```sql
id          uuid (PK)
post_id     uuid (FK → posts)
user_id     uuid (FK → profiles)
body        text
likes_count integer
created_at  timestamptz
```

### `likes`
Post likes — one per user per post (composite PK prevents duplicates).

```sql
post_id     uuid (FK → posts)
user_id     uuid (FK → profiles)
created_at  timestamptz
PRIMARY KEY (post_id, user_id)
```

### `follows`
User follow relationships.

```sql
follower_id   uuid (FK → profiles)
following_id  uuid (FK → profiles)
PRIMARY KEY (follower_id, following_id)
```

### `brokerage_connections`
Tracks which brokerages a user has connected.

```sql
id            uuid (PK)
user_id       uuid (FK → profiles)
broker_name   text
connected_at  timestamptz
is_active     boolean
```

### `stock_prices`
NGX live prices cache — updated every 10 minutes by pg_cron.

```sql
ticker          text (PK)
company_name    text
price           numeric
prev_close      numeric
change_pct      numeric
change_amt      numeric
day_high        numeric
day_low         numeric
volume          bigint
market_cap      text
week_52_high    numeric
week_52_low     numeric
last_updated    timestamptz
is_market_open  boolean
```

---

## 6. Authentication System

**Provider:** Supabase Auth (email + password)

### Sign Up Flow
```
User fills form (first name, last name, email, password)
    ↓
supabase.auth.signUp({ email, password })
    ↓
Supabase creates auth.users record
    ↓
handle_new_user() trigger fires → inserts into public.profiles
    ↓
Frontend upserts profile with name/initials
    ↓
sessionStorage.setItem('gopex_user', {...})
    ↓
Redirect to home.html
```

### Sign In Flow
```
User fills form (email, password)
    ↓
supabase.auth.signInWithPassword({ email, password })
    ↓
Fetch profile from public.profiles
    ↓
sessionStorage.setItem('gopex_user', {...})
    ↓
Redirect to home.html
```

### Auth Guard (on every protected page)
```javascript
// Runs immediately on page load
const raw = sessionStorage.getItem('gopex_user');
const tok = localStorage.getItem('sb-lzlitanvlkycrkkpwczm-auth-token');
const hasSession = raw || (tok && JSON.parse(tok)?.access_token);
if (!hasSession) { window.location.replace('landing.html'); }
```

### Logout
```javascript
supabase.auth.signOut()  // invalidates Supabase session
sessionStorage.clear()   // clears local user data
window.location.href = 'landing.html'
```

---

## 7. Live Stock Price System

### How It Works

```
afx.kwayisi.org/ngx/{TICKER}.html
        ↓  scraped by
fetch_stock_price(ticker)   ← PostgreSQL function using http extension
        ↓  called by
fetch_all_ngx_prices()      ← loops all 25+ tickers
        ↓  scheduled by
pg_cron: '*/10 * * * *'    ← every 10 minutes, Mon-Fri only
        ↓  results stored in
public.stock_prices table
        ↓  read by
market.html, home.html, portfolio.html ← via Supabase REST API
```

### Tickers Tracked (25 core + 99 seeded)
Core auto-refreshed: MTNN, DANGCEM, GTCO, ZENITHBANK, BUAFOODS, ACCESSCORP, AIRTELAFRI, SEPLAT, GEREGU, FIRSTHOLDCO, BUACEMENT, TRANSCORP, NGXGROUP, FIDSON, BERGER, DANGSUGAR, NESTLE, PRESCO, UNILEVER, FIDELITYBK, UBA, WEMABANK, STANBIC, OKOMUOIL, FCMB, STERLINGNG + more

All 124 stocks seeded from May 15, 2026 NGX closing data.

### Market Hours
Scraper only runs during market hours (pg_cron still fires but function returns early):
- **Days:** Monday – Friday
- **Hours:** 9:30am – 4:00pm WAT (UTC+1)

### Frontend Price Fetch
```javascript
// Reads directly from Supabase REST — no Edge Function needed
fetch('https://[project].supabase.co/rest/v1/stock_prices?select=...', {
  headers: { 'apikey': ANON_KEY, 'Authorization': 'Bearer ' + ANON_KEY }
})
```

### Adding More Tickers to Auto-Refresh
Edit the `tickers` array inside the `fetch_all_ngx_prices()` SQL function in Supabase:
```sql
tickers text[] := array[
  'MTNN', 'DANGCEM', ... 'YOUR_NEW_TICKER'
];
```

---

## 8. Social Feed System

### Post Creation
```javascript
// Saved to Supabase posts table with:
{
  user_id,
  body,
  ticker_holdings: [{ticker, i_hold, my_pct}],  // % only, never ₦
  portfolio_pct_gain,  // null unless user opts in
  portfolio_period
}
```

### Privacy Rules
| Data | Visible to others? |
|------|-------------------|
| Post text | ✅ Always |
| Tagged tickers + prices | ✅ Always |
| Whether author holds the stock | ✅ Always |
| Author's % of portfolio in that stock | ✅ Always (if tagged) |
| Author's % portfolio gain | ✅ Only if they toggle it on |
| Author's ₦ portfolio value | ❌ Never — not stored in posts |

### Real-time Updates
```javascript
supabase.channel('social')
  .on('postgres_changes', { event: 'INSERT', table: 'posts' }, payload => {
    prependPost(payload.new)  // new posts appear instantly
  })
  .on('postgres_changes', { event: 'UPDATE', table: 'posts' }, payload => {
    updatePostLikes(payload.new)  // like counts update in real-time
  })
  .subscribe()
```

### Database Triggers (auto-maintain counts)
- `on_like_change` → updates `posts.likes_count` on INSERT/DELETE from `likes`
- `on_comment_change` → updates `posts.comments_count` on INSERT/DELETE from `comments`
- `on_follow_change` → updates `profiles.followers_count` and `following_count`
- `on_auth_user_created` → auto-creates `profiles` row on new signup

---

## 9. Portfolio System

### Holdings Calculation
```javascript
// For each holding:
const currentPrice = MARKET_PRICES[ticker]?.price || buyPrice
const marketValue  = currentPrice * shares
const costBasis    = buyPrice * shares
const plPct        = ((currentPrice - buyPrice) / buyPrice) * 100
```

### Portfolio % Gain (for social sharing)
```javascript
// Calculated at login — stored in profiles.portfolio_pct_gain
let costBasis = 0, marketVal = 0
holdings.forEach(h => {
  costBasis += h.shares * h.buy_price
  marketVal += h.shares * (MARKET_PRICES[h.ticker]?.price || h.buy_price)
})
const pctGain = ((marketVal - costBasis) / costBasis) * 100
// ← This is the ONLY portfolio number ever shared publicly
```

### Brokerage Integration (Coming Soon)
Currently **disabled**. All 6 brokerage buttons show "Coming Soon":
- Bamboo (bamboo.africa)
- Trove Finance (troveapp.co)
- Chaka (chaka.com)
- MERITRADE / Meristem (meristemng.com)
- Afrinvest (afrinvest.com)
- Risevest (risevest.com)

When enabled, connecting a broker will:
1. Save to `brokerage_connections` table
2. Set `profiles.is_verified = true`
3. Show ✓ Verified badge on profile and posts

---

## 10. Supabase Configuration

### Project Details
| Setting | Value |
|---------|-------|
| Project Name | GopexFund |
| Project ID | lzlitanvlkycrkkpwczm |
| Region | East US (North Virginia) |
| URL | https://lzlitanvlkycrkkpwczm.supabase.co |
| Compute | Nano (free tier) |

### API Keys
| Key | Usage |
|-----|-------|
| `anon` / `publishable` | Used in frontend HTML files — safe to expose |
| `service_role` / `secret` | Never used in frontend — backend only |

### Row Level Security Policies
Every table has RLS enabled. Key policies:

| Table | Policy |
|-------|--------|
| `profiles` | Anyone can read; users can only update their own |
| `holdings` | Users can only read/write their own |
| `goals` | Users can only read/write their own |
| `posts` | Anyone can read; users can only insert their own |
| `comments` | Anyone can read; users can only insert their own |
| `likes` | Anyone can read; users manage their own |
| `follows` | Anyone can read; users manage their own follows |
| `brokerage_connections` | Users can only access their own |
| `stock_prices` | Anyone can read (public market data) |

### Extensions Required
```sql
create extension if not exists pg_cron;   -- for scheduled jobs
create extension if not exists http;       -- for HTTP requests in SQL
```

### Scheduled Jobs (pg_cron)
```sql
-- Runs every 10 minutes
select cron.schedule(
  'refresh-ngx-prices',
  '*/10 * * * *',
  'select fetch_all_ngx_prices()'
);

-- View scheduled jobs
select * from cron.job;

-- View recent job runs
select * from cron.job_run_details order by start_time desc limit 10;
```

---

## 11. How to Deploy

GopexFund is a **static website** — just HTML files. No server needed.

### Option 1 — Vercel (Recommended, Free)
1. Create account at vercel.com
2. Install Vercel CLI: `npm install -g vercel`
3. In your project folder: `vercel`
4. Follow prompts — your site goes live at `gopexfund.vercel.app`
5. Add custom domain in Vercel dashboard

### Option 2 — Netlify (Free)
1. Go to netlify.com → "Add new site" → "Deploy manually"
2. Drag and drop your project folder
3. Site goes live instantly at a `.netlify.app` URL

### Option 3 — GitHub Pages (Free)
1. Push all files to a GitHub repository
2. Go to repo Settings → Pages → Source: `main` branch
3. Site goes live at `username.github.io/repo-name`

### Option 4 — Cloudflare Pages (Free, fastest in Africa)
1. Go to pages.cloudflare.com
2. Connect GitHub repo or upload directly
3. Cloudflare has edge nodes in Africa — fastest for Nigerian users

### After Deploying
Update your Supabase Auth settings:
1. Go to Supabase dashboard → Authentication → URL Configuration
2. Add your live URL to **Site URL** and **Redirect URLs**
3. Example: `https://gopexfund.vercel.app`

---

## 12. Environment Variables

All Supabase credentials are currently **hardcoded** in the HTML files. This is acceptable for the `anon` key (it's public by design and protected by RLS), but for production you may want to externalise them.

### Current Credentials (in all HTML files)
```javascript
const SUPABASE_URL  = 'https://lzlitanvlkycrkkpwczm.supabase.co'
const SUPABASE_ANON = 'eyJhbGciOiJIUzI1NiIs...' // anon/publishable key
```

### ⚠️ Never put in frontend code
```
service_role key  ← has full database access, bypasses RLS
```

---

## 13. Known Limitations & Roadmap

### Current Limitations

| Item | Status | Notes |
|------|--------|-------|
| Brokerage sync | 🔒 Coming Soon | Waiting for broker API agreements |
| Email verification | ⚠️ Not enforced | Users can sign up without verifying email |
| Password reset | ⚠️ UI only | Supabase handles it but no UI page built |
| Push notifications | ❌ Not built | Planned for future |
| Mobile app | ❌ Not built | Currently web-only |
| US stocks | ❌ Not built | Only NGX stocks currently |
| Real-time prices during market hours | ⚠️ 10-min delay | afx.kwayisi.org is end-of-session data |
| Profile photos | ❌ Not built | Using initials avatars |

### Roadmap (Future Features)

**Phase 2 — Brokerage Integration**
- Connect Bamboo, Trove, Chaka APIs
- Auto-sync holdings
- Verified portfolio badges go live
- Real-time holdings from broker

**Phase 3 — Mobile App**
- React Native app (iOS + Android)
- Push notifications for likes, comments, follows, price alerts

**Phase 4 — Advanced Market Data**
- Upgrade to NGX official API (real-time, ₦625K/year)
- US stocks via Alpha Vantage
- Crypto prices via CoinGecko
- Historical price charts

**Phase 5 — Monetisation**
- GopexFund Pro subscription
- Promoted posts for brokerages
- Premium market analytics

---

## Quick Reference

### Supabase SQL to check system health
```sql
-- Check stock prices are updating
SELECT ticker, price, change_pct, last_updated
FROM public.stock_prices
ORDER BY last_updated DESC
LIMIT 5;

-- Check cron job is running
SELECT jobname, schedule, last_run
FROM cron.job;

-- Check recent users
SELECT id, email, created_at
FROM auth.users
ORDER BY created_at DESC
LIMIT 10;

-- Check post count
SELECT COUNT(*) as total_posts FROM public.posts;

-- Check total holdings across all users
SELECT COUNT(*) as total_holdings FROM public.holdings;
```

### Manually trigger a price refresh
```sql
SELECT fetch_all_ngx_prices();
```

### Add a new stock ticker to auto-refresh list
Edit the `fetch_all_ngx_prices()` function in Supabase SQL Editor and add the ticker to the array.

### Reset a user's password (admin)
Go to Supabase Dashboard → Authentication → Users → find user → Send password reset email.

---

*Built with ❤️ for African investors · GopexFund 2026*
