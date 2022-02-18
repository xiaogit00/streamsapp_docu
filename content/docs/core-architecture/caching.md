---
title: Caching
weight: 3
---
# Caching


## Introduction to Caching
A really important feature that Streamsapp provides is getting live price data using API. However, there are currently request limits for the plans I am using to fetch data for the app. One way to work around that is to use browser caching.

## How Streams Price Data Caching works
The main idea here is, when I make requests to my live price endpoints, I want to check whether the data is present in localStorage first. If it is present and not stale (checked against a time based condition), then I shall serve the data directly from localStorage. If not, I will proceed to make a request.

The way this is done is as follows.

### Request Interceptor

First, you set up the requestInterceptor within currentPriceService like so:

```javascript {hl_lines=[4]}
const fetchPriceForStock = (ticker) => {
    console.log('currentPriceService is entered')
    const fetchCurrentStockPriceURL = `/api/streams/currentstockprice/${ticker}`
    interceptorService.useAxiosRequestInterceptor()
    interceptorService.useAxiosResponseInterceptor()
    const response = axios.get(fetchCurrentStockPriceURL, {
        headers: {
            Authorization:`bearer ${token}`
        }
    })
    ...
```
This way, all requests sent using axios go through the *requestHandler*.

### Request Handler

The requestHandler is defined in `interceptorService.js` in the following manner:

```javascript
function requestHandler(request) {
    if (request.method === 'GET' || 'get') {
        var checkIsValidResponse = cache.isValid(request.url || '')
        if (checkIsValidResponse.isValid) {
            request.headers.cached = true
            request.data = JSON.parse(checkIsValidResponse.value || '{}')
            return Promise.reject(request)
        }
    }
    return request
}
```
There's quite a number of things going on here.

First, there is a conditional check for request type. If the request method is not GET, the handler will bypass everything and return the request untouched.

If it is a GET request, then this line
```javascript
var checkIsValidResponse = cache.isValid(request.url || '')
```

will check if there is a valid cache. The way this is done is by calling the `isValid()` method defined within the `cache` module (*cacheHandler.js*).

The isValid() method

```javascript {hl_lines=[2, 30]}
function isValid(key) {
    var value = localStorage.getItem(key)
    //if Empty
    if (value === null) {
        return {
            isValid: false,
        }
    }
    //Get timestamp
    var values = value.split(SEPARATOR)
    var timestamp = Number(values[1])

    //If timestamp not valid
    if (Number.isNaN(timestamp)) {
        return {
            isValid: false,
        }
    }
    //Make timestamp into date
    var date = new Date(timestamp)

    //If date is not a valid date
    if (date.toString() === 'Invalid Date') {
        return {
            isValid: false,
        }
    }

    //If current date minus cache date is less than CACHE_INTERVAL, means ISVALID*, return values[0]
    if ((Date.now() - date.getTime()) < CACHE_INTERVAL) {
        return {
            isValid: true,
            value: values[0],
        }
    }

    //else, its not valid. We remove the localStorage, and return isValid:False.
    localStorage.removeItem(key)
    return {
        isValid: false,
    }
}
```

contains a series of checks on the `value` variable, defined to be the results of *localStorage.getItem(key)*. It:

1. Checks whether `value` is empty
2. Checks whether if we can split value to get timeStamp.
3. Checks that the result of the split is not Nan.  
4. Convert the timestamp to date, and check whether date is valid.
5. **Checks whether data is stale.** If not, `isValid` is true.
6. Else, remove localStorage key, and set `isValid` to be false.

Steps 1-4 are simply data cleaning to support Step 5, which is the main purpose of this module.

### If cache valid

Now, back to our requestHandler.

```javascript {hl_lines=[4]}
function requestHandler(request) {
    if (request.method === 'GET' || 'get') {
        var checkIsValidResponse = cache.isValid(request.url || '')
        if (checkIsValidResponse.isValid) {
            request.headers.cached = true
            request.data = JSON.parse(checkIsValidResponse.value || '{}')
            return Promise.reject(request)
        }
    }
    return request
}
```

Remember, if `isValid` is passed, the variable `checkIsValidResponse` will contain an object with the following attributes: `{isValid: true, value: values[0]}`, where `values[0]` is the response data.

Because the cache is valid, we would want to return whatever data that's in cache instead of making the actual request.  **On the request level, we need to use the requestHandler to *block* this request (terminating it prematurely) the same time, return the data stored in `checkIsValidResponse.value`.**

To do this, we first set `request.headers.cached = true`. Then, we set the `data` attribute for request to be the cache value stored, like so:

```javascript
request.data = JSON.parse(checkIsValidResponse.value || '{}')
```

Finally, we reject the promise, with the request object sent.

```javascript
return Promise.reject(request)
```

Here, it's important to remember which promise the interceptor will reject to serve cache data. In our case, it is the following request defined within fetchPriceForStock():

```javascript {hl_lines=[6, 15,16]}
const fetchPriceForStock = (ticker) => {
    console.log('currentPriceService is entered')
    const fetchCurrentStockPriceURL = `/api/streams/currentstockprice/${ticker}`
    interceptorService.useAxiosRequestInterceptor()
    interceptorService.useAxiosResponseInterceptor()
    const response = axios.get(fetchCurrentStockPriceURL, {
        headers: {
            Authorization:`bearer ${token}`
        }
    })
    return response.then(res => {
        return res.data
    } ).catch(err => {
        console.log(err)
        if (err.data) {
            return err.data
        } else {
            console.log(`There is an error in fetching for ${ticker}, the error is: `, err)
        }

    })

}
```
Within the `fetchPriceForStock` function definition, we first call a GET request. We return res.data if there is no error, and return `err.data` if there is an error.

This function is called within `TableRow.js`:

```javascript {hl_lines=[4]}
const getCurrentPrice = async () => {
    const baseDenom = individualStream.exchangePriceDenom

    const stockPrice = currentPriceService.fetchPriceForStock(individualStream.ticker)
```

Now here is the part where I last left off. The confusions in error handling has left me unable to implement a successful caching mechanism. I will try to fix it moving forward.

## Implementation of Caching for streams price data

First, we start off with `<TableRow>`.

```javascript
//Stocks
const getCurrentPrice = async () => {
    const baseDenom = individualStream.exchangePriceDenom

    const stockPrice = currentPriceService.fetchPriceForStock(individualStream.ticker)

    const conversionRate = exchangeService.exchange(baseDenom, globalDenom)

    return await Promise.all([stockPrice, conversionRate])

}

getCurrentPrice()
    .then(res => {
        const newPrice = res[0][0].open * res[1].conversion_rate
        setCurrentPrice(newPrice)
    })
    .catch(e => {
        console.log('[TableRow]There\'s a problem with getting current price for: ', individualStream.asset)
        console.log('[TableRow]this is the error request if it\'s cached:', e)
    })
```
Here, we have defined an async function `getCurrentPrice`.

This function makes two async calls to the two APIs to get the price.

If either promise is rejected, .catch() statement will be run.

---

## Unit Testing for Caching feature

To test whether my `fetchPriceForStock()` async function works as expected with `requestInterceptor` and `responseInterceptor` mounted,, I've decided to set up a unit test for this function. I'll document what exactly goes on under the hood, for the results here are very demonstrative.

First, I created a test cache button on the home page.

```javascript
import React from 'react'
import currentPriceService from 'services/currentPriceService'
import interceptorService from 'services/interceptorService'

const TestCacheButton = () => {

    const buttonHandler = async () => {
        interceptorService.useAxiosRequestInterceptor()
        interceptorService.useAxiosResponseInterceptor()
        try {
            const response = await currentPriceService.fetchPriceForStock('AAPL')
            console.log(response)
        } catch (err) {
            console.log(err.message)
        }


    }

    return (
        <>
            <button onClick={buttonHandler}>Test Cache Button</button>
        </>
    )
}

export default TestCacheButton

```

This button mounts request and response inteceptors, and calls `fetchPriceForStock()` when clicked.

This is what is shown on console.log:

![](https://res.cloudinary.com/dl4murstw/image/upload/v1638864905/Screenshot_2021-12-07_at_4.14.28_PM_gufzaq.png)

### Observations
Below is the sequence of events:
1. Button clicked
2. Interceptors mounted
3. Try block executed
4. `fetchPriceForStock()` method called (as can be seen from 'currentPriceService is entered')
5. `axios.get()` is called, and RequestHandler is triggered immediately.
![](https://res.cloudinary.com/dl4murstw/image/upload/v1638865371/Screenshot_2021-12-07_at_4.14.28_PM_gufzaq_ygawmq.png)
6. RequestHandler performs `isValid()` check. The request object is:
{{<details "request object">}}
```javascript
{
    "url": "/api/streams/currentstockprice/AAPL",
    "method": "get",
    "headers": {
        "Accept": "application/json, text/plain, */*",
        "Authorization": "bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6IkxlaWJpbmciLCJpZCI6IjYxNjUyNjE2MjllMTEwOGU5MmVmYjE5YSIsImlhdCI6MTYzNDgwNDE5OX0.QKpJPoYSuNLtK8dUorbEAw4GaCXfeRYmKiNSJWT3eIg"
    },
    "transformRequest": [
        null
    ],
    "transformResponse": [
        null
    ],
    "timeout": 0,
    "xsrfCookieName": "XSRF-TOKEN",
    "xsrfHeaderName": "X-XSRF-TOKEN",
    "maxContentLength": -1,
    "maxBodyLength": -1
}
```
{{</details>}}
7. `isValid()` check fails (as we've not cached it before), hence 'serving fresh from API' is displayed. Request sent to API.
8. ResponseHandler triggered. The response object is: (I've highlighted some important fields to note)
{{<details "response object">}}
```javascript {hl_lines=[2, 21, 23, 34, 35, 36]}
{
    "data": [
        {
            "open": 164.29,
            "high": 167.8799,
            "low": 164.28,
            "close": 165.32,
            "volume": 107496982,
            "adj_high": null,
            "adj_low": null,
            "adj_close": 165.32,
            "adj_open": null,
            "adj_volume": null,
            "split_factor": 1,
            "dividend": 0,
            "symbol": "AAPL",
            "exchange": "XNAS",
            "date": "2021-12-06T00:00:00+0000"
        }
    ],
    "status": 200,
    "statusText": "OK",
    "headers": {
        "access-control-allow-origin": "*",
        "content-length": "262",
        "content-type": "application/json; charset=utf-8",
        "date": "Tue, 07 Dec 2021 07:54:17 GMT",
        "etag": "W/\"106-vsIBFwsWPx+waUzG4NHGPvig5ho\"",
        "server": "Cowboy",
        "vary": "Accept-Encoding",
        "via": "1.1 vegur",
        "x-powered-by": "Express"
    },
    "config": {
        "url": "/api/streams/currentstockprice/AAPL",
        "method": "get",
        "headers": {
            "Accept": "application/json, text/plain, */*",
            "Authorization": "bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6IkxlaWJpbmciLCJpZCI6IjYxNjUyNjE2MjllMTEwOGU5MmVmYjE5YSIsImlhdCI6MTYzNDgwNDE5OX0.QKpJPoYSuNLtK8dUorbEAw4GaCXfeRYmKiNSJWT3eIg"
        },
        "transformRequest": [
            null
        ],
        "transformResponse": [
            null
        ],
        "timeout": 0,
        "xsrfCookieName": "XSRF-TOKEN",
        "xsrfHeaderName": "X-XSRF-TOKEN",
        "maxContentLength": -1,
        "maxBodyLength": -1
    },
    "request": {}
}
```
{{</details>}}
9. `response.config.method` is GET, and after doing the following checks
```javascript {hl_lines=["3-6"]}
if (response.config.method === 'GET' || 'get') {
        console.log('[ResponseHandler-InterceptorService]', response)
        if (response.config.url && isURLInWhiteList(response.config.url)) {
            console.log('[ResponseHandler-InterceptorService]storing in cache')
            cache.store(response.config.url, JSON.stringify(response.data))
        }
    }
    return response
```
**`response.data` is stored in cache.**

10. reponseHandler returns `response` object. **This is the final output of the function call.** The promise response `const response = await axios.get()` within `fetchPriceForStock()` method is resolved, and the function returns `response.data`, which is:
{{<details "Output of fetchPriceForStock()">}}
```javascript
[
    {
        "open": 164.29,
        "high": 167.8799,
        "low": 164.28,
        "close": 165.32,
        "volume": 107496982,
        "adj_high": null,
        "adj_low": null,
        "adj_close": 165.32,
        "adj_open": null,
        "adj_volume": null,
        "split_factor": 1,
        "dividend": 0,
        "symbol": "AAPL",
        "exchange": "XNAS",
        "date": "2021-12-06T00:00:00+0000"
    }
]
```
{{</details>}}

**This data point is now stored in localStorage**.

![](https://res.cloudinary.com/dl4murstw/image/upload/v1638866406/Screenshot_2021-12-07_at_4.39.51_PM_oqo3gc.png)


### A word about the whitelist
The responseHandler will only cache responses whose response.config.url is in the whitelist as defined within `interceptorService.js`.

Within the responseHandler, there is a check for whether the response.config.url is in whitelist.

```javascript
if (response.config.url && isURLInWhiteList(response.config.url)) {
            cache.store(response.config.url, JSON.stringify(response.data))
```

Looking at the `isURLInWhiteList(url)` function

```javascript
function isURLInWhiteList(url) {
    return whiteList.includes(url.split('/')[2])
}
```
we observe that it splits the url input by '/' and returns the second argument. For instance, in the case of `https://v6.exchangerate-api.com/api`, the second arg would be *v6.exchangerate-api.com*.

In my case, I want to whitelist `/api/streams/currentstockprice/AAPL`, which means the second arg is 'streams'.

I'll thus need to add that to the whitelist within `interceptorService.js`.

```javascript
const whiteList = [
    'shielded-reaches-54541.herokuapp.com',
    'v6.exchangerate-api.com',
    'streams'
]
```

### Checking that data is retrieved from cache

The more important question for us here, is figuring out what happens when there is cached data.

**Observations**  
Sequence of events
![](https://res.cloudinary.com/dl4murstw/image/upload/v1638867947/Screenshot_2021-12-07_at_5.03.57_PM_zaiwed.png)



1. isValid check within `requestHandler` to pass
```javascript
if (request.method === 'GET' || 'get') {
        var checkIsValidResponse = cache.isValid(request.url || '')
        if (checkIsValidResponse.isValid) {
```
2. `request.headers.cached` set to true
```javascript
if (checkIsValidResponse.isValid) {
            console.log('[RequestHandler-InterceptorService] serving cached data:')
            request.headers.cached = true
            request.data = JSON.parse(checkIsValidResponse.value || '{}')
            console.log('cached data:', request.data)
            return Promise.reject(request)
        }
```
3. `request.data` is retrieved from localStorage and parsed appropriately
```javascript
request.data = JSON.parse(checkIsValidResponse.value || '{}') //checkIsValidResponse contains cached data
```
4. Return a rejected Promise.
5. Here, and it is crucial behavior of interceptor, **control is deferred to errorHandler**. This errorHandler is defined when we mounted interceptor.

```javascript
const useAxiosResponseInterceptor = () => {
    console.log('axios response interceptor is mounted')
    return (
        axios.interceptors.response.use(
            response => responseHandler(response),
            error => errorHandler(error)
        )
    )
}
```

The errorHandler
```javascript
function errorHandler(error) {
    // console.log('[ErrorHandler-InterceptorService] error.headers.cached:', error.headers.cached)
    if (error.headers.cached === true) {
        console.log('[ErrorHandler-InterceptorService] got cached data in response, serving it directly: ', error)
        return Promise.resolve(error)
    }
    return Promise.reject(error)
}
```

checks whether 'error' is actually due to cache with this logic: `if (error.headers.cached === true)`. If it is, it resolves the Promise. If not, it rejects it, hence triggering .catch() statement of fetchPriceForStock().

**Main observation: in most cases, when data is serving from cache, the errorHandler will *resolve* the error before it reaches the parent call**, thereby ensuring that the .catch() within currentPriceService.js is not executed.

`fetchPriceForStock()` will still output response.data (which is in fact `error.data`, since the Promise.reject(request) sends the request object into errorHandler), and testCacheButton receives the response neatly.

## Bug: 7 Dec 2021

Symptom:
The first time I press test cache button, everything runs smoothly and cache is saved. The second time I press Test Cache Button, I get
```
TypeError: Cannot read properties of undefined (reading 'method')
    at responseHandler (interceptorService.js:31)
    at interceptorService.js:57
    at async Object.fetchPriceForStock (currentPriceService.js:18)
    at async buttonHandler (testCacheButton.js:11)
```

**Observations of Bug:**  
Here is the console:
![](https://res.cloudinary.com/dl4murstw/image/upload/v1638869467/Screenshot_2021-12-07_at_5.30.25_PM_btqhwq.png)

What's interesting is that everything is okay all the way up to the point of serving cached data. Which means, requestInterceptor ran, checked that cache isValid, served data object, rejected Promise, deferring control to ErrorHandler, which dutifully served the cache data.

However, there is this curious behavior where control is then defered to ResponseHandler, as seen in blue. The response object sent to responseHandler does not have `response.config.method`
{{<details "Response object">}}
```javascript
{
    "url": "/api/streams/currentstockprice/AAPL",
    "method": "get",
    "headers": {
        "common": {
            "Accept": "application/json, text/plain, */*"
        },
        "delete": {},
        "get": {},
        "head": {},
        "post": {
            "Content-Type": "application/x-www-form-urlencoded"
        },
        "put": {
            "Content-Type": "application/x-www-form-urlencoded"
        },
    ...
}
```
{{</details>}}

Hence the following code within responseHandler fails:

```javascript {hl_lines=[3]}
function responseHandler(response) {

        if (response.config.method === 'GET' || 'get') {
            console.log('[ResponseHandler-InterceptorService]', response)
            if (response.config.url && isURLInWhiteList(response.config.url)) {
                console.log('[ResponseHandler-InterceptorService]storing in cache')
                cache.store(response.config.url, JSON.stringify(response.data))
            }
    }

    return response
```

**Cause:**
The bug is caused by the if statement: `if (response.config.method === 'GET' || 'get')`. This conditional check presupposes that `response.config.method` exists. If it doesn't exist, an exception will be thrown - which is not what you want. The desired behavior is for `if` to be skipped if condition not satisfied, not for it to throw an exception.


**Fix:**
I wrapped a simple `if(response.config)` around the `if` statement above. Response Handler now looks like this:

```javascript {hl_lines=[3]}
function responseHandler(response) {

    if (response.config) {
        if (response.config.method === 'GET' || 'get') {
            console.log('[ResponseHandler-InterceptorService]', response)
            if (response.config.url && isURLInWhiteList(response.config.url)) {
                console.log('[ResponseHandler-InterceptorService]storing in cache')
                cache.store(response.config.url, JSON.stringify(response.data))
            }
        }
    }

    return response
}
```
In addition, I've also wrapped `if (error.headers.cached)` within errorHandler because a similar exception is thrown here when I just did the fix for responseHandler.

```javascript {hl_lines=[2]}
function errorHandler(error) {
    if (error.headers.cached) {
        if (error.headers.cached === true) {
            console.log('[ErrorHandler-InterceptorService] got cached data in response, serving it directly: ', error)
            return Promise.resolve(error)
        }
    }

    return Promise.reject(error)
}
```

**Fixed in file(s):**
*interceptorService.js*


**Time taken to resolve bug:**
20min

**Lessons:**  
Conditional statements that take the form of `if (object.attribute === true )` (i.e. checks for the value of the attribute) *must* first satisfy the condition that said attribute is defined in the first place. If not, a TypeError exception will be thrown where it states that properties are not defined.
