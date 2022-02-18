---
title: Streams Table
weight: 1
---
# Streams Table

Streams table is the most important feature of this site. It is a one page visual interface to keep track of the performance of various 'streams' of income.

This is what it looks like:

![](https://res.cloudinary.com/dl4murstw/image/upload/v1634873545/Screenshot_2021-10-20_at_3.55.29_PM_vrpqp1.png)

The component hierarchy of `<StreamsTable>` is as follows:
![](https://res.cloudinary.com/dl4murstw/image/upload/v1644992688/Screenshot_2022-02-16_at_2.24.00_PM_cdcolc.png)
*App > Routes > RightPane > ContentContainer > StreamsTable*

## StreamsTable Component

`<StreamsTable>` component is the top level component containing the logic to fetch all streams and trades to be displayed.

It does so by calling `initializeStreams()` and `initializeTrades()` function within useEffect.

``` jsx
    useEffect(() => {
        dispatch(initializeStreams())
    }, [dispatch])

    useEffect(() => {
        dispatch(initializeTrades())
    }, [dispatch])
```

Both these functions are **reducers**, meaning they will get the data from services and store it into the store.

`initializeStreams()` reducer is a function imported from *streamReducer.js* and it is responsible for calling `streamService.getAll()` and storing all the data in dispatch.

``` javascript
//streamReducer.js
    const streams = await streamService.getAll()
            dispatch({
                type: 'INIT_STREAMS',
                data: streams
            })
```

Although it calls .getAll(), it does not fetch **all** of the data in the database. What happens on the backend is this:
- Within `stream.js` (service), an axios call is made to `api/streams`, passing along the token in the authorization header.

``` javascript
const token = localStorage.getItem('token')
const getAll = async () => {

    try {
        const response = await axios.get(baseURL, {
            headers: {
                Authorization:`bearer ${token}`
            }
        })
        return response.data
    }
```

- Within the backend, the token is retrieved within the route, decoded, and stored. The id of the decoded token is used to filter a call made to the model.
``` javascript {hl_lines=[4,5,11]}
//streams router

streamsRouter.get('/', async (request, response) => {
    const token = getTokenFrom(request)
    const decodedToken = jwt.verify(token, process.env.SECRET)
    console.log('process.env.SECRET:',process.env.SECRET)
    if (!token || !decodedToken.id) {
        return response.status(401).json({ error: 'token missing or invalid'})
    }

    const streams = await Stream.find({'userId': decodedToken.id})
    response.json(streams)
})
```

- We then call the Stream model using the .find() method, supplying the 'userId' as the `decodedToken.id`


What this implies is that only the Streams relevant to the User are returned from our backend service.

We now return to the StreamsTable component.  After data is fetched, `<TableHead>` and multiple `<TableRow>s` components are rendered.

That's about all the responsibility of StreamsTable.

**In summary, the streamsTable component fetches the data with effectHooks, dispatches them to store, and feeds them into TableRow components.**

## TableHead Component
Before diving into `<TableRow>` component, let's take a brief look at `<TableHead>`. TableHead is a holder container that contains TableHeadItems. Each of these items are columns, with their column name, column width, and spanID specified within `<TableHeadItems>` component. Each of these then become rendered as divs with a span tag inside. Pretty straightforward component for generating table headers.


## TableRow Component
TableRow is where a lot of the action is. Seriously a lot of action.

Birthed by the `<StreamsTable>` component,  Each `<TableRow>` is a component that receives the following props: `individualStream` (stream data), `trades`, `globalDenom`, and `num`.

First, it calls `streamData(individualStream, trades, globalDenom)`, which is a function imported from *calcs/tableRow/streamData* that does further processing of the streams object.

The function produces an object with the following attributes:
- `tradesInStream`: an array of trades within the stream
- `filteredTrades`: trade objects filtered from tradesInStream
- `baseAsset`: main asset of the stream
- `openTrades`: open trade objects
- `closedTrades`: closed trade objects
- `latestDate`: date of the latest open trade.
- `totalSpentOpenNominal`: This is an important field that standardizes the value fields of all the trades associated with the stream into one denominated value, as specified by the globalDenom. It does so by calling the nominal_value() function. It then adds everything up into one value: subsequently displayed as *Purchase Value* within the StreamsTable.
- `totalCashedOutNominal`: this looks through all the closed trades within the stream, and sums up the value for all of them. As the term suggests, this signifies the 'value' of whatever that's already been cashed out.
- `totalAmtPurchased`: this is the total **amount** (in units) of whatever asset purchased, calculated from trades.
- `totalAmtSold`: this is the total **amount** (in units) of whatever asset sold, calculated from trades.
- `avgPurchasePrice`: calculated from totalSpentOpenNominal/totalAmtPurchased
- `avgPurchaseValueNominal`: avgPurchasePrice/totalAmtPurchased

In essence, the streamData function does a lot of the calculation from raw individualStream and trade data, and returns the values that I'd wanna see on the page as an object.

This object I store in the variable 'stream' that is exposed within TableRow.

### Fetching live prices within tableRow
Within TableRow component, there is an effects hook that calls functions within the currentPriceService to get live prices using the API. I wouldn't touch on them much here.

### Swaps

Subsequently, within `<TableRow>`, there are logics to deal with swaps information. The design philosophy behind swaps can be found here <insert link>. But the main idea here is: if the individualStream contains swaps, then call swapsData() to format the swaps data. This will also be dealt with in the section on swaps. For now, we just need to know that this function returns an object with the following attributes:
- `totalAmtSwappedinSwapped`: total amount denominated in swap asset
- `totalAmtSwappedBase`: total amount denominated in base asset
- `avgPurchasePriceInBaseAsset`: avg purchase price in base asset
- `closedAmtSwapBase`: how much (amt) of swap is closed.
- `openAmtSwapBase`: how much (amt) of swap is open
- `netOpenSwapAmtInBaseAsset`: how much is netOpen
- `currentValueofSwapOpenNominal`
- `weightOpen`
- `weightClosed`
- `unrealizedReturnsSwapAbsolute`
- `realizedClosedReturnsSwapPercentage`
- `realizedClosedReturnsSwapAbsolute`

So the reason why this logic is more complex is because every single stream that contains swaps will have their own logic for realized and unrealized streams and trades. I won't get into the logic here.

The goal of each `<TableRow>` component is to calculate the returns figure, as well as the weights.

Together, `stream`, the variable containing all the stream info I need, and `weights`, are passed as props into the `<StreamBar>` component.


## StreamBar Component
Streambar component receives all the props and renders them. Quite straightforward.

In addition, the coloring behind is done by the `<ProgressBar>` component.

## ProgressBar component
The `<ProgressBar>` component receives the weights (to generate proportion of colors), avgClosePrice, and realizedReturns as props (to generate the avg ROI texts for each stream).

---

And there we have it! Everything for the Streams table.

That's quite a bit of components broken down, but hopefully it's straightforward enough. 
