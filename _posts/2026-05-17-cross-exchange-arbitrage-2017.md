---
title: "Cross-Exchange Arbitrage — When a 3% Margin Accidentally Rode an 8x Bull Run (2017)"
date: 2026-05-17 01:00:00 +0900
categories: [Trading, Crypto]
tags: [cryptocurrency, arbitrage, bithumb, binance, bittrex, blockchain-transfer, csharp]
---

*This is Part 2. [Part 1](/posts/dart-scraping-and-one-second-trading/) covered DART-based Korean equity automation (2009–2015).*

---

## The Setup

This happened in 2017 — the year the Asian cryptocurrency boom truly started.

At the time I was running a mining operation. "Running" is generous — it was about 90% automated, and I just monitored it occasionally. While figuring out how to convert mined coins to Korean Won, I noticed that the same coin was trading at wildly different prices on Bithumb (Korean exchange) and Bittrex (US exchange).

I later learned people called this the "Kimchi Premium." Korean exchanges consistently carried a 20–50% price premium over overseas exchanges. The asymmetry was created by capital controls, KRW KYC barriers, and an explosion of Korean retail demand — all layered on top of each other.

In [Part 1](/posts/dart-scraping-and-one-second-trading/), I described chasing an *information* asymmetry and hitting a wall when another information asymmetry (insider trading) neutralized it. This story is the companion piece. This time the target was a *price* asymmetry — and the story of how pursuing it accidentally captured a *time* asymmetry that dwarfed the original intent.

---

## Building the Routing Engine

The start was simple. I connected to the Bithumb API, Bittrex API, and Poloniex API one after another. Later I added Binance and Upbit.

Sometimes the optimal path was straightforward: send mined ETH to Bithumb, sell for KRW. But sometimes it was better to send ETH to Bittrex, sell for USDT, buy Litecoin with USDT, transfer Litecoin to Bithumb, and sell for KRW — because the margin through that route was 3–15% higher.

The system computed every possible route across all overlapping coins between exchanges. Buy → transfer → sell → buy → transfer → sell. Even with just two exchanges the combinations are large; add more exchanges and they explode. For every route, it deducted fees, average transfer time, and slippage, then sorted the remaining net profit in real time. The most profitable route executed automatically.

Built in .NET C# WinForms with MySQL. A stack that would make anyone laugh today. But it moved an average of **35 billion KRW (~$26M USD) per month** in trading volume through those exchanges.

The source code for the two main systems is public: [`cross-exchange-arbitrage`](https://github.com/kdkkh/cross-exchange-arbitrage) — includes a 17-minute video walkthrough of the predecessor semi-automatic version.

---

## The KYC Grind Nobody Talks About

Connecting to an API wasn't the end of the story. New accounts had tiny limits — nowhere near enough for serious volume.

Every exchange required ID photos, selfies, and some demanded English-translated resident registration documents — with my face and the document in the same frame. Only after passing these checks did the trading limit unlock to the tens-of-millions-of-dollars range. This took days to weeks per exchange, and I had to clear it on *every single one* before the real game could start.

From the outside this sounds like a story about code making money. In reality, half of it was taking selfies and obtaining government documents in English and uploading them again and again. That's the trust foundation. The code ran on top of it.

---

## Transfer Speed as a Variable

Operating the system taught me something that only becomes obvious through experience: **coin transfer speed varies enormously**.

XRP was fast. BCH was painfully slow. BTC was slow too. At some point my routing engine started factoring in average transfer time per coin as a variable — not just price spread. Fast coins became the workhorse for moving assets between exchanges. This is why XRP was the de facto reserve asset for inter-exchange transfers in 2017.

---

## The Transfer Window Problem

Even with everything automated, I couldn't step away. The reason: **transfer windows**.

Once a trade starts, the coin leaves Exchange A and takes minutes to hours — sometimes days — to arrive at Exchange B. During that time, the asset is fully exposed to market movements. And I had dozens of these transfer windows running simultaneously, 24 hours a day. Some land at 3 AM. Some at noon. Some at midnight.

Transfers get stuck. Exchange servers go down. Withdrawals get suspended without notice. Solve one problem, another appears. It never ends.

I never believed the "sleep when you're dead" crowd. Whenever someone claimed they worked hard on 4 hours of sleep, I assumed they were lying. Then I did it myself — 4 to 5 hours a night for months. It turns out you can.

---

## Scale

Bithumb dropped fees to roughly 1/10th of standard when monthly volume exceeded a certain threshold (I recall it being around 10–20 billion KRW). My monthly volume averaged around **35 billion KRW**. Eventually Bithumb created a separate VIP member site, and I traded from there.

The initial capital was approximately **35 million KRW (~$26K USD)**. Over about five months of operation, the net profit from spread arbitrage alone exceeded **200 million KRW (~$150K USD)** — roughly a 6x return on the designed mechanism only. The unintended payoffs described below are *not* included in that number.

I know this reads like a brag post. I don't want it to be one. So let me talk about what actually matters — the part I didn't design.

---

## The Unintended Payoffs

The system's *designed* revenue was the inter-exchange price spread: 3–15% per transaction. That was the intended mechanism.

But while operating, two *unintended* revenue streams appeared alongside it — and they were larger than the spread income.

### Unintended Payoff #1: Forced Exposure to the Bull Run

There were always assets in transit. Dozens of trades in-flight simultaneously across five exchanges. Now consider what 2017 was: Bitcoin went from 3 million KRW in Q1 to 25 million KRW by year-end. XRP went from 300 KRW to over 4,000. Altcoins were worse — coins worth tens of won getting pumped to 1,000 by retail apes.

A trade initiated for a 3% spread margin could end up +30% or -30% by the time the transfer completed — depending on what the market did during the transfer window. Over enough transactions, that distribution converges to the market's average return. And that year's market average was **8x**.

All I built was an arbitrage infrastructure. But that infrastructure, by its nature, forced continuous market exposure across all major coins, on all major exchanges, 24 hours a day. It accidentally rode the 2017 bull run.

### Unintended Payoff #2: The Bitcoin Hard Fork

In the summer of 2017, the Bitcoin hard fork happened — the Bitmain/Wu Jihan mining faction split from the Core development team. The result: every BTC holder received an equal amount of Bitcoin Cash (BCH). My arbitrage-held BTC duplicated into BCH. BCH started around 600K KRW, crashed to the 100K range, then spiked hard later that year.

---

## Summary of What Actually Happened

What I built was "a system that captures price asymmetry between exchanges."

But the system's **exposure surface** was enormous: five exchanges, maximum-tier limits, assets distributed 24/7, dozens of transfer windows running concurrently, positions spread across every major coin. So almost every event that happened in that market — the bull run, the hard fork, the altcoin pumps — landed directly on that exposure surface.

The *intended* profit was a small part. The *unintended* profit was far larger.

I won't state the total amount. The facts above give enough for anyone to estimate.

---

## The Scene I Can't Forget

Sometime between summer and winter of 2017.

I watched the top of an order book on Bithumb get swept clean in under a minute. I don't remember which coin it was. Crypto is famous for volatility, but I'd never seen an order book like that. Buy-side orders were being emptied line by line, every few seconds. Then DASH, sitting next to it, started getting swept the same way. DASH more than doubled in price within one minute.

I turned the system off. It was almost the first time I'd manually shut it down during operation. I was scared. I just stared at the screen blankly. I didn't turn it back on until the next day.

The rumor afterward was that Chinese capital exited in a wave following a government announcement. I never verified this independently.

---

## External Risk Wasn't the Only Kind

Once I left mining pool rewards sitting in the pool wallet too long without distributing them to my personal wallets or exchange hot wallets. The pool got compromised. I lost approximately **8 million KRW (~$6K USD)**. This wasn't a risk the system caught — it was operator negligence. Mined coins need to be periodically swept out of pool wallets, and I got lazy for a stretch.

---

## Why I Stopped

I stopped the arbitrage system between autumn and winter of 2017. The reasons were layered:

- The 3%+ spreads that had been common stopped appearing.
- More coins became transfer-restricted on Korean exchanges.
- Market prices were rising to levels that felt disconnected from reality, while prominent voices — scam, tax haven, unproductive speculation — accumulated.

The primary reason: **the asymmetry itself got arbitraged away**. Arbitrage requires asymmetry to exist. When it disappears, the game ends. Same principle as Part 1 — I don't play below 50% expected value.

Mining continued a bit longer. I shut it down in summer 2018. The direct cause was profitability collapse from the crash and negative media coverage of mining operations. But what truly made me let go was something else.

---

## The Regulatory Environment

While operating the mining farm, I was simultaneously studying mining pool mechanics, mining software, blockchain protocols, and the evolution of new coins. My conviction that this knowledge would have a viable future *in Korea* kept weakening.

Limited to my direct experience: when new technology creates friction with established interests in Korea, regulation moves to kill the technology rather than accommodate it. Two examples are enough.

**TADA** (a ride-hailing service) launched in 2018 and was effectively banned by legislation within 18 months. The same business model — Uber — faced fierce incumbent resistance in the US too (taxi medallions, dispatch unions), but was *allowed to survive* long enough to reshape the industry. One died. One lived. This isn't about code quality or founder charisma. It's about regulatory environment.

**Zigbang** started as a direct-transaction platform between landlords and tenants — its founding pitch was to disrupt real estate agents. In practice, it became a digital storefront for the very agents it intended to disrupt. Direct transactions are nearly nonexistent.

If you want to cite Samsung or Hyundai as counterexamples — those companies are products of an era before today's incumbent power structures existed. They were built when Korea had almost nothing that could be called an entrenched interest. They're answers from a different era, not counterexamples to today's question.

This isn't a policy argument. It's one operator's assessment of the environment he worked in. And that assessment was the real reason I stepped away at the end of 2017.

---

## What Part 2 Adds to Part 1

Looking back: Part 1's ending was an external limit — my mechanism was neutralized by another mechanism (informed traders with advance knowledge). Part 2's ending has two layers. The market's asymmetry getting arbitraged away is one layer — an external fact. My assessment of the environment I was working in is the other.

The scene where DASH doubled in a minute and I turned off the system — that's the same kind of closure. An admission that my mechanism, in front of certain market events, is a system where the operator ultimately has to step back and let go.

If there's one lesson from this chapter, it's this: **build a system with enough exposure surface, and payoffs you never designed for will follow alongside the ones you did.** Part 1 was about distinguishing mechanism from coincidence. Part 2 is about what happens when your mechanism is large enough that coincidence gets captured along with it.

---

## References

- Source code (two C# systems, public): [`cross-exchange-arbitrage`](https://github.com/kdkkh/cross-exchange-arbitrage)
- 17-minute video walkthrough of predecessor system (Korean narration, English auto-subtitles via YouTube CC): included in the repository README above
