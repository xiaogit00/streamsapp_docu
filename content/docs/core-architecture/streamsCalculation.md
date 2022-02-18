---
title: Streams Calculation
weight: 1
---
# Streams Calculation

Okay we've reached quite a meaty section of the documentation, one that I already have forgotten. This just includes looking at each data point and figuring out how they're derived and what's the logics to take note of.

So obviously, there's the
- avg purchase price.
- purchase value.
- current value.
- total returns,
- weight

## Avg Purchase Price
This is an import field that tells us the unit price of the stream. The formula for this is:
> Avg Purchase Price = Total Spent Open Nominal / Total Amt Purchased

One thing to note about the current set up is that Avg Purcahse Price is dependent on the total spent of **open** trades. Within the current set up, 'closed' trades are added as another row in the trades database, instead of specifying a 'close amount' from an existing trade. That is why, if we go by this user flow, it makes sense to filter for Open trades. Another thing to note is the word 'nominal': the avg purchase price is calculated at runtime, based on globalDenom set..

All these can be found in streamData.js - where most of the figures are generated at runtime.

## Purchase Value
Purchase Value is derived from:
> Purchase Value = Avg Purchase Price * Total Amt Purchased

## Current Value
This is calculated live from the JSX section of 'tableRow.js', supplied directly as prop for StreamBar.
> Current Value = Total Amt * Current Price

## Total Returns
The calculation for totalReturns is a bit trickier.

The first step is to determine whether there are swaps. If there are swaps, then it gets more complicated. If there are no swaps, we pass the 'strea' object, along with trades, currentPrice and globalDenom into the `returnsData()` function.

This function calculates the following fields:
### realizedReturnsPercentage
> Realized Returns (%) = Avg Close Price / Avg Purchase Price

Why is it AvgClosePrice / Avg Purchase price?

Let's say, I bought 3 Xiaomi Stock at $10 each. Avg Purchase price is $10. When I sold 1 at $12, the AvgClosePrice is $12. The realized returns is 20%. I then need to calculate the weight of closed:

### weightClosed
> Weight Closed = Total Amt Sold / Total Amt Purchased  

Pretty intuitive!

With both of these, I can calculate the Absolute returns for the closed portion:

### Closed Trades Absolute return
> Closed Trade Absolute Returns = Realized Returns (%) * Weight Closed.

I do the same for Open trades, and derive the following fields: `unrealizedReturnsPercentage`, `weightOpen`, `openTradeReturnsAbsolute`.

Finally: my stream returns is simply a summation of the closed trades returns (absolute) and open trade returns (absolute).

``` javascript
//returnsData.js

const returnsData = (stream, trades, currentPrice, globalDenom) => {
    const realizedReturnsPercentage = ((stream.avgClosePrice() / stream.avgPurchasePrice()) - 1)*100 || 0
    // console.log("realizedReturnsPercentage",realizedReturnsPercentage)
    const weightClosed = stream.totalAmtSold() / stream.totalAmtPurchased()
    // console.log("weightClosed",weightClosed)
    const closedTradeReturnsAbsolute = realizedReturnsPercentage * weightClosed
    // console.log("closedTradeReturnsAbsolute",closedTradeReturnsAbsolute)
    const unrealizedReturnsPercentage = ((currentPrice / stream.avgPurchasePrice()) - 1)*100
    // console.log("currentPrice",currentPrice, stream.avgPurchasePrice())
    const weightOpen = (1 - weightClosed)
    // console.log("unrealizedReturnsPercentage",unrealizedReturnsPercentage)
    const openTradeReturnsAbsolute = unrealizedReturnsPercentage * weightOpen
    // console.log("openTradeReturnsAbsolute", openTradeReturnsAbsolute)
    const streamReturns = closedTradeReturnsAbsolute + openTradeReturnsAbsolute
    // console.log("openTradeReturnsAbsolute",openTradeReturnsAbsolute)

    return streamReturns
}

```
