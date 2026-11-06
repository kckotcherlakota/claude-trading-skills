# Trading Rulebook (single source of truth)

Every trading session reads this file first and follows it. Rules here come from the user;
when the user gives a new rule, update this file (and the Change log) before acting on it.
Account number and report email live in the routine prompts, not here (public repo).

## 1. Prime duty
- PROTECT CAPITAL — highest priority.
- Be aggressive in FINDING profits; once money is in a trade, be 100% cautious.
- "Small drops become a big river": stack small protected wins and let the account compound.
- Long-term goal from the user: $1,000/week. Never break a risk rule to chase it.
- Pre-authorized: trade without waiting for approval when a setup qualifies. A bad trade is worse than no trade.
- Day-trade guard (user 2026-11-06: must NOT be flagged PDT). The API shows no PDT flag or day-trade
  counter, and the account is limited_margin under $25k, so assume the old rule may still apply:
  max 3 same-day round trips (open+close same day) in any rolling 5 business days. Count conservatively.
  Log: 11-05 SPY put (1); 11-06 QQQ call (1); 11-06 RIOT call (1 buy, 2 sells = 1, maybe 2).
  At the cap: no new 0DTE trades; only setups held overnight (next-week expiry, sized so an overnight
  gap stays within the planned loss) or crypto (no PDT). Also: if any order preview shows a
  day-trade/PDT or buying-power warning, do NOT place it and tell the user.

## 2. What may be traded
1. 0DTE SPY / QQQ options (preferred).
2. Options on any liquid stock/ETF from the Robinhood scanner "Claude — intraday movers w/ liquid options"
   (scan_id 47e13f99-78d2-4ece-a5b1-c8627a97b50e: stock/ETF, mkt cap > $5B, options volume > 30k, move ±2.5%+).
   Requirements: nearest weekly expiry; bid/ask spread ≤ ~10% of price (≤ $0.05 under $1); real volume/OI;
   NO earnings or major event before expiry. Close same day.
3. Crypto: BTC / ETH spot, buy-only (see §7).
- Always buy-to-open calls/puts only. Never sell-to-open, spreads, shorts, leverage, or stocks.
- One position at a time across options and crypto.

## 3. Sizing & day limits ("Push harder" — user's choice)
- Position size ≤ ~60% of buying power (use the buying power Robinhood reports).
- Planned loss per trade ≤ $25 (stop % = $25 ÷ premium; breakout stops never wider than −40%).
- ≤ 4 trades per day; daily realized loss limit $40 (never open a trade that could push past it).
- After a win keep trading, but protect the day: next trade's planned loss ≤ max($10, 50% of today's net profit).
  If today's profit ≥ $25, stop once it falls below 50% of the day's peak.
- No revenge trades — every re-entry needs a NEW valid setup.
- Prefer contracts with delta ~0.15–0.35 over cheap lottery tickets.

## 4. Daily bias (first scan of the day)
- Run market-breadth-analyzer and uptrend-analyzer (skills/ in this repo; no API key).
- Check scheduled macro events; no entries within ~15 min before a major release.
- Bias BULLISH (calls) / BEARISH (puts) / NEUTRAL (either, stronger confirmation). Single stocks: follow their own VWAP trend.

## 5. Entries (10:00 AM – 2:00 PM ET)
A) Breakout: 9:30–10:00 opening range + VWAP (5-min). Calls on a 5-min close above OR/consolidation high and
   above VWAP, holding; puts on a 5-min close below OR low/failed-bounce low and below VWAP.
   After fill: rest a GFD stop_market sell-to-close at the planned stop.
B) Scalp ("take even a small 5–10% squeeze"): 1-min structure — VWAP reclaim/loss with 2–3 higher lows
   (lower highs for puts), range-edge bounce, or sharp 1-min burst with volume. Re-check the quote right before
   buying; if the option already jumped, wait for a small pullback and buy at/below mid.
   After fill: rest a limit sell-to-close at ~+10–12%; stop −15–20% or underlying 1-min close back through VWAP/level (I enforce it).
- Orders: limit near mid; review_option_order then place_option_order with a fresh ref_id.
- Scanning when flat: SPY/QQQ + the scanner, shortlist 2–3 strongest above-VWAP / weakest below-VWAP names.
  Once per 5-min bar (≈10 min in a midday lull, 5 min near a trigger). Stop scanning at 2:00 PM ET.

## 6. Once money is in — 100% caution
- Check every 30 seconds while money is in (Monitor tick every 30s; send_later 1-min as backup): option bid/ask, underlying, 1-min and 5-min bars.
- Protect gains: ≥ +15% → never close below breakeven +3%; ≥ +30% → lock at least half; +100% → always sell.
- Take profit on any stall: 2 one-minute bars against, lower high/higher low after a push, key level, macro event, afternoon decay.
- Exit immediately if the underlying closes a 1-min bar back through VWAP/the breakout level.
- Exit early = cancel the resting order first (confirm 'cancelled'), then sell at the bid.
- All options closed by 3:30 PM ET; never let one expire or be exercised.

## 7. Crypto (off-hours: weekdays 2 PM–9:30 AM ET, weekends all day)
- BTC/ETH spot, buy-only, only when the 1h trend is not down and the 15m trend is up (price > EMA21 > EMA50, RSI 50–70);
  enter on a pullback or fresh 12-bar-high breakout. Candles: Coinbase public API (exchange.coinbase.com).
- Size ≤ 60% of crypto buying power; stop just below the 15m 12-bar swing low; planned loss incl. ~1% spread ≤ $25;
  counts toward the daily limits.
- Rest a gtc stop order right after the fill. +2% → stop to breakeven; +4% → trail to lock ~half; take profit at
  15m trend roll-over or +6–8%. Sell any open crypto at the 9:17 AM weekday check (frees capital for options).

## 8. Reporting
- Tell the user only on orders, fills, exits or notable changes — one short message, no hourly "nothing new" chatter.
- 3:20 PM ET weekdays: close out options, email the daily report (real Robinhood numbers only).
- 4:37 PM ET weekdays: strategy research; at most one tactic change per day, logged below. Never change §1–§3 rules on my own.

## Change log
- 2026-11-05 — 0DTE SPY/QQQ only, buy-only; max loss $10–15; one trade/day; daily email.
- 2026-11-05 — $20 on volatile days; multiple trades after a loss; other liquid options + crypto allowed; capital put to work.
- 2026-11-05 — "Push harder": 60% size, $25/trade, 4 trades, $40 day limit; resting broker stops; crypto off-hours.
- 2026-11-06 — scalps for small 5–10% squeezes; keep trading after a win with day-gain protection; 1-min in-trade checks.
- 2026-11-06 — user: check every 30 seconds when money is in.
- 2026-11-06 — market-wide scanner for the whole tradable universe; PDT removed (user); "$1,000/week" long-term goal.
- 2026-11-06 — rulebook created; all routines read it each run to keep context small.
- 2026-11-06 — day-trade guard: max 3 round trips per rolling 5 business days (cap reached 11-06).
