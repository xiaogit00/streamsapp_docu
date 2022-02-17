---
title: Trades Table
weight: 1
---
# Trades Table

The trades table contains all the trades that the user has inputted.

![](https://res.cloudinary.com/dl4murstw/image/upload/v1634716681/Screenshot_2021-10-20_at_3.57.53_PM_p0bsu1.png)

## Component Hierarchy
Like `<StreamsTable>`, the `<TradesTable>` component sits within the `<ContentContainer>` component.

Unlike `<StreamsTable>`, the TradesTable component is relatively straightforward.

## TradesTable component breakdown

I am mainly using a couple of Styled components from Material UI to generate the table: `TableContainer`, `Table`, `TableHead`, `TableRow`, and `StyledTableCell`.

Then, for the data, I am using an effect hook to get trades, sorting them, and storing them into `sortedTrades`.

I then map through `sortedTrades` to generate `<StyledTableRow>s` and rendering them within `<TableBody>` component.

Now, the default display is actually the else statement, with straightforward rendering of StyledTableRows with trade data within them. The presence of the if statement is to handle the editing process.

## Editing the trades

First, you'll notice that the entire JSX rendering is wrapped in a if/else statement:

``` jsx {hl_lines=[2]}
{sortedTrades.map((trade) => {
    if (trade.id===tradeEditId) {
        ...
    } else {}
```
tradeEditId is a state. It is a state that is filled whenever the edit button is clicked. If the edit button is clicked, the code within the if condition renders *input* fields within each StyledTableCell. The `name` prop of the input field is assigned to the trade attribute name, and the value is whatever data that is in the tradeEditData itself, which, is set to the trade object selected.

(tradeEditData is also a state. It is first set to an empty object {}. When the edit button is clicked, tradeEditData is set to the trade object targetted itself.)

Now, the input fields are controlled states. When we change it, the handleInputChange handler is invoked. The handleInputChange handler stores the targetted object's name and value. If name is price - i.e. we're editing the price field - we update the value (not displayed) field of the trade object as well. We do the same for the amt field. Other than that, all other input changes will just change the `name` and `value` of the trade object and keep everything else the same.

``` javascript
const handleInputChange = e => {
    const { name, value } = e.target

    if (name === 'price') {
        const valueField = value * tradeEditData.amt
        setTradeEditData({
            ...tradeEditData,
            [name]: value,
            value: valueField
        })
    } else if ( name === 'amt') {
        const valueField = tradeEditData.price * value
        setTradeEditData({
            ...tradeEditData,
            [name]: value,
            value: valueField
        })
    } else {
        setTradeEditData({
            ...tradeEditData,
            [name]: value
        })
    }
}
```

The code above is quite elegant!

Apart from that, I think this is all there is to the tradeTables. There are a couple more event handlers to deal with saving, but those are pretty self-explanatory.
