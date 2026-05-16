---
title: "DART Scraping and One-Second Trading — Korean Equity Automation, 2009–2015"
date: 2026-05-17 10:00:00 +0900
categories: [Trading, Systems]
tags: [korean-equity, systematic-trading, dart-scraping, linear-regression, cybosplus]
---

## Context: Who I Was in 2009

In 2009 I was a senior researcher at a fabless security camera company called INICS. My stack was FPGA Verilog HDL, embedded C/C++, ModelSim, and MATLAB. I designed auto-exposure, auto-focus, and Smart IR algorithms for CCTV cameras. My daily tools were oscilloscopes, CRT monitors, and power supplies — not databases or web servers.

On the PC side, I barely touched anything beyond MATLAB for algorithm verification and Visual Studio 6.0 for RS232 debugging. I understood object-oriented programming in theory but had never shipped a desktop application. Today I work across .NET, Python, Node.js, and MySQL as a full-stack engineer — but back then, I was a complete beginner at application-level programming.

Then I read *Currency Wars* (화폐전쟁), picked up a few books on stock trading strategies, and wanted to verify whether those strategies actually worked. That curiosity is what pulled me into systematic trading.

What follows is the story of what I built and learned between 2009 and 2015.

---

## The Only API Available

At the time, Korea had exactly one retail-accessible brokerage API: Daishin Securities' **CybosPlus** — a COM-based interface that exposed historical price data, real-time order book quotes, and chart feeds. Other brokerages released their APIs shortly after, but in 2009 CybosPlus was it.

I pulled all available historical price data, subscribed to real-time quote streams, and started experimenting — with real money. No backtesting, no simulation. Ignorance makes you brave.

---

## Three Discoveries That Cost Me Hundreds in Commissions

After burning through enough brokerage fees to buy a decent used car, I discovered three things that no textbook mentioned:

1. **Historical data, real-time quotes, and chart data can disagree with each other.** They're sourced from different internal pipelines and occasionally diverge — especially around market open and volatile moments.

2. **You cannot capture every real-time tick during rapid execution bursts.** The COM callback mechanism drops events when the queue saturates. Your local record of what happened is always an approximation.

3. **Implementing a realistic order book execution model is a medium-term project in itself.** The gap between "place a market order" and "simulate the actual fill behavior of a multi-level order book with queue priority" is enormous.

There were many other lessons, but these three left the deepest mark.

At that point, I finally started using my brain.

---

## The DART Insight

Prices go up because people buy. They go down because people sell. So what makes people decide to buy or sell? They read news — and **disclosure filings** (공시).

Korea's equivalent of SEC EDGAR is called [DART](https://dart.fss.or.kr/). I discovered its existence only after paying hundreds of thousands of won in wasted commissions.

There's a lesson my graduate school days taught me: when you feel like everyone else already knows something you don't, that's the perfect moment to start studying.

Now I finally understood why simulation mattered. I began analyzing, obsessively, the correlation between filing timestamps on DART and subsequent price movements.

---

## Building the One-Second Trading System

The patterns were clear:

- **Financial statements, contract announcements, and patent filings** trigger price spikes within one second of publication.
- Certain patent keywords — stem cells, clinical trials, "Phase X" — were the strongest catalysts. Stem cell research had been tainted years earlier by a major scandal in Korea, yet the keyword still moved prices harder than anything else on DART.
- **Bonus shares and rights issues** were useless signals — by the time the filing appeared, trading was restricted due to regulatory mechanisms (circuit breaker or similar suspension; the exact mechanism escapes me after 15+ years, but the practical effect was the same: you couldn't act on the information).

### Scoring System

I built a scoring model using linear regression to evaluate each filing:
- **Patents**: keyword-based weighting (stem cell, clinical trial, etc.)
- **Financial statements**: operating profit delta
- **Contract announcements**: contract value as a percentage of annual revenue

All scores were normalized into a single buy/no-buy signal. I remember the regression setup being considerably more complex than this summary suggests — multiple feature interactions, different weighting schemes per filing type — but the exact implementation details have faded after 15+ years. I won't pretend to remember what I don't.

### DART Scraping — The Technical Challenge

In 2009, DART provided no API. Refreshing the page more than a few times triggered a rate limit. But I noticed that plugging my ethernet cable into a different port on the router gave me a fresh IP — so the rate limiting was IP-based.

I purchased proxy IPs, reverse-engineered the exact rate limit conditions, and built a scraper that polled DART every 800ms across rotating proxies. This alone was a significant technical challenge for someone whose prior experience was firmware debugging over serial terminals.

### Execution Pipeline

Having worked with FPGAs, I had some instinct for parallel processing. I bought a high-spec PC and built a pipeline that processed multiple stages concurrently:
- DART scrape → HTML parse → score computation: **< 200ms**
- Score above threshold → buy order via CybosPlus: **immediate**
- Hold for 3 minutes → sell

I was so exhausted by the time I got the buy side working that the sell strategy was deliberately kept trivial. Three minutes, then out. No optimization.

Total latency from filing publication to order placement: **under one second**.

It was profitable. The exact returns are lost — my trading income was mixed with my day job salary at the time, and I never separated the accounting properly. What I remember clearly isn't the money. It's the dopamine of having built a system that *worked* — that reliably converted a public information event into a profitable trade. That feeling dominated everything else.

That feeling lasted until the Korean equity market taught me otherwise.

---

## The Wall: Information Leakage

I began observing a pattern that occurred **far more frequently than the financial news ever reported** — by my estimate, what made it into the news was maybe 1–2% of what I was seeing in practice:

- **Minutes or tens of minutes before** a positive filing appeared on DART, the stock price would surge dramatically.
- **Within 100ms of the filing publication**, the price would crash — the informed participants were dumping into the wave of algorithmic buyers like me.

Nine out of ten trades were profitable. But the one loss wiped out all nine gains. The reason was volume asymmetry: the nine profitable trades involved stocks with modest order book depth, while the one catastrophic loss involved a stock with enormous volume — the kind of volume that only appears when someone already knows what's coming.

I added defenses: before executing a buy on a positive filing, I checked the price trajectory in the preceding minutes and filtered out stocks that had already moved suspiciously. It helped — but not enough. Even among the modest-volume stocks, the same pattern seemed to exist in subtler forms. Whether it was smaller-scale informed trading or just noise from other participants, I couldn't reliably distinguish.

The problem wasn't one I could engineer around. Or maybe it was — maybe I simply hadn't studied enough to find the right filter. I still think of it as an unsolved bug, not a fundamental impossibility.

---

## Why I Stopped

The system worked as engineered. The market environment made it unsustainable. When the probability drops below 50% on a risk-adjusted basis, I don't play — whether it's a stock market or a casino. If Las Vegas offered me 50.1% expected return, I'd go.

I haven't traded Korean equities since. Not because I've sworn off investing forever, but because I don't engage with odds I can't verify.

---

## What I Took Away

This experience taught me to distinguish between **things that seem real** and **things that are real**.

**Seems real**: A news headline about a trader who made $10M with a novel strategy. What you don't see are the tens of thousands of others who tried their own strategies during the same period. Most lost money. A few made thousands, or tens of thousands. They just didn't make the news. The headline winner is a sample from the tail of a distribution, not proof that the strategy works.

**Actually real**: Building a system that exploits a measurable, repeatable information asymmetry — where you can explain the causal mechanism, not just pattern-match on outcomes. My DART system was invalidated by a specific external factor, but the underlying approach (filing publication → price movement → automated capture) was structurally sound.

The difference between the two is whether you're observing a **mechanism** or a **coincidence**. Fifteen years of engineering since then have only reinforced that distinction.

---

## Technical Summary

| Component | Implementation |
|---|---|
| Brokerage API | Daishin Securities CybosPlus (COM, C#) |
| Data scraping | DART HTML parsing, 800ms polling, rotating proxy IPs |
| Scoring model | Linear regression with keyword weighting and normalized composite score (exact feature design no longer recalled) |
| Execution latency | < 1 second from filing publication to order |
| Exit strategy | Time-based (3-minute hold) |
| Defense | Pre-filing price trajectory filter |
| Period | 2009–2015 |
| Outcome | Profitable until information leakage made risk-adjusted returns negative |
