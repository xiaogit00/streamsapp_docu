---
title: Currency Filter Button
weight: 1
---
# Currency Filter Button

The currency filter button is the button on the right of our Streams page.

![](https://res.cloudinary.com/dl4murstw/image/upload/v1645006030/Screenshot_2022-02-16_at_6.06.42_PM_lzeiov.png)

In terms of component hierarchy, it is the following:

App > RightPane > HeaderBlock > nominalCurrencyButton


---
Okay what is it. It is a ToggleButton - a materialUI component, with slightly customized styles.

So the Component itself is a `<ToggleButtonGroup>`. The ToggleButtonGroup contains two ToggleButtons.

There is first of all an `alignment` state, which is set to left by default. We also initialize useDispatch(). We have a handleAlignment() event handler which is passed to onChange prop of ToggleButtonGroup. Within the handler, we set event.target.innerText (which is the text that says 'SGD or USD') as the newGlobalDenom - afterwhich, we dispatch() the newGlobalDenom to the store using changeDenom() action handler. The reducer we use is the globalDenomReducer().

``` javascript
const globalDenomReducer = (state = 'SGD', action) => {
    switch(action.type) {
    case 'CHANGE_DENOM': {
        const newDenom = action.currency
        return newDenom
    }
    default:
        return state
    }
}
```

So, in a nutshell, what is happening is this. The button is a component that has an onChange event handler attached to it. When you click on one, one of the ToggleButtons within will be selected. Each of those buttons have a `value` attached to it, which is 'left' and 'right' respectively.

This value is passed and set **onClick** in the event handler and extracted from the element as 'innerText'.

``` javascript
const handleAlignment = (event, newAlignment) => {

    const newGlobalDenom = event.target.innerText
    //Set the state to newGlobalDenom
    dispatch(changeDenom(newGlobalDenom))
    if (newAlignment !== null) {
        setAlignment(newAlignment)
    }
}
```

That's about it! The rest is quite intuitive, no need to hold your hand. 
