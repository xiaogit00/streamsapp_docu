---
title: Redux Architecture
weight: 1
---
# Redux Architecture

The purpose of a redux is to act as a state storage. It enables you to separate state management from component rendering hierarchy. In the context of my app, I used Redux to manage the following class of states:  

1. Stream data
2. Trade Data
3. Modal toggling data
4. Global Denom
5. Login state

## Implementation
The basics of implementation takes us to `index.js`. Within the root of your app, you are already importing `Provider` from redux, as well as `store`. Provider is wrapped around the entire App so that you can access the store.

``` javascript
ReactDOM.render(
    <Provider store={store}>
        <App />
    </Provider>,)
```

`store.js` within utils is where you create your store. Using the `combineReducers()` function, you add all the reducers you want to call. Then, you create the store using the `createStore()` function, providing the thunk middleware so that you're dispatching to the store with every call to the service.

```javascript
//store.js
const reducer = combineReducers({
    streams: streamReducer,
    trades: tradeReducer,
    globalDenom: globalDenomReducer,
    modals: modalReducer
    // loggedIn: loggedInReducer
})

const store = createStore(
    reducer,
    composeWithDevTools(
        applyMiddleware(thunk)
    )
)
```

Okay, that's about it! The rest is just using the `useSelector()` function, or the `dispatch()` function, as appropriate to pull the data into the project.
