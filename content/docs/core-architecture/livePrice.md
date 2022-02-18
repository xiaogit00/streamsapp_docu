---
title: Live Prices
weight: 1
---

# Live Prices
{{< figure src="https://res.cloudinary.com/dl4murstw/image/upload/v1638775198/Screenshot_2021-12-06_at_3.13.53_PM_hxxill.png">}}
## Overview
Streamsapp relies on third party API providers to get live prices for assets.

## Component responsible
When `<StreamsTable />` component is reached, it renders one `<TableRow />` for every stream.  

The responsibility for calling API to get live asset price lies with `<TableRow />`. `<TableRow />` uses *useEffect()* to get the data.  Within the effect hook, we make calls to our backend API using our CurrentPriceService.

```javascript {hl_lines=[9, 17, 25, 30]}
const TableRow = ({individualStream, trades, globalDenom, num}) => {

    const swapsCurrentPrices = []
    const [currentPrice, setCurrentPrice] = useState(0)

    const stream = streamData(individualStream, trades, globalDenom)


    useEffect(() => {
        console.log('useEffect is entered ')

        if (individualStream.assetClass === 'Stocks') {

            const getCurrentPrice = async () => {
                const baseDenom = individualStream.exchangePriceDenom

                const stockPrice = currentPriceService.fetchPriceForStock(individualStream.ticker)

                const conversionRate = exchangeService.exchange(baseDenom, globalDenom)

                let values = await Promise.all([stockPrice, conversionRate])
                const newPrice = values[0][0].open * values[1].conversion_rate
                setCurrentPrice(newPrice)
            }
            getCurrentPrice().catch(e => {
                console.log('[TableRow]There\'s a problem with getting current price for: ', individualStream.asset)
                console.log('[TableRow]this is the error request if it\'s cached:', e)
            })
        } else if (individualStream.assetClass === 'Crypto') {
            currentPriceService.fetchPriceForCrypto(individualStream.coinId, globalDenom)
                .then(response => {
                    setCurrentPrice(response[individualStream.coinId][globalDenom.toLowerCase()])
                })
                .catch(err => {
                    console.log(err)
                    setCurrentPrice('Failed')
                })
        }
    }, [globalDenom])

```

The `fetchPriceForStock()` and `fetchPriceForCrypto()` functions of the `currentPriceService` help us get current price. `fetchPriceForStock()` has one argument, *ticker*.

`fetchPriceForCrypto()` expects the *coinId* and *globalDenom*.

## CurrentPriceService

Within my current configuration, the currentPriceService service module makes calls to my internal backend API, which in turn makes calls to the external APIs to get the price.

The .fetchPriceForCrypto() method within currentPriceService

```javascript
const fetchPriceForStock = (ticker) => {
    console.log('currentPriceService is entered')
    const fetchCurrentStockPriceURL = `/api/streams/currentstockprice/${ticker}`
    // interceptorService.useAxiosRequestInterceptor()
    // interceptorService.useAxiosResponseInterceptor()
    const response = axios.get(fetchCurrentStockPriceURL, {
        headers: {
            Authorization:`bearer ${token}`
        }
    })
    return response.then(res => {
        console.log(`[CurrenPriceService] response for ${ticker} is fetched, and it is:`, res.data)

        return res.data
    } ).catch(err => {
        console.log(`There is an error in fetching for ${ticker}, the error is: `, err)

    })
}
```
makes a call to the backend `api/streams/currentstockprice/${ticker}`.

The backend controller, *streams.js*, has a route defined

```javascript
streamsRouter.get('/currentstockprice/:ticker', async (request, response) => {
    const token = getTokenFrom(request)
    const decodedToken = jwt.verify(token, process.env.SECRET)
    if (!token || !decodedToken.id) {
        return response.status(401).json({ error: 'token missing or invalid'})
    }
    const marketStackURL = `https://api.marketstack.com/v1/eod/latest?access_key=${process.env.marketStackToken}&symbols=${request.params.ticker}`
    console.log('this is reached in Get Router')

    const data = await axios.get(marketStackURL)
    if (data) {
        response.json(data.data.data)
    } else {
        console.log('There is an error getting data from marketStack')
        response.status(404).end()
    }
```

which makes an axios call to marketStack API.

This chain thus looks something like this:


{{< figure src="https://res.cloudinary.com/dl4murstw/image/upload/v1638775198/Screenshot_2021-12-06_at_3.13.53_PM_hxxill.png">}}

Finally, when the data is returned to TableRow, we update the state variable currentPrice.
