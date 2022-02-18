---
title: Data Input 
weight: 1
---

# Data Input in Streamsapp
Data input in Streamsapp is done via the Stream/Trade modal forms.

As specified in the portion on SpeedDials, the Modal forms are triggered via a `dispatch()` call attached to the event handler of the SpeedDialAction buttons. This dispatch() call sends the action of TOGGLE_STREAM/TRADE..etc to the modalReducer.

It's worthwhile looking at this set up more in detail.

# modalReducer

The ModalReducer has an initialState variable, which is an object with the following properties set to false: streamModalOpen/tradeModalOpen/orderModalOpen

When the TOGGLE_STREAM action is received, it triggers one of the states open or closed.

Importantly, the entire reason why the modals can be triggered is because the `<Modal>` component is rendered within the speedDial. With that said..

# Component Hierarchy for modals
...ContentContainer>SpeedDial>Modals

# Modal Component

Parked under /floatingActionButtonAdd, the modal component is an interesting one. In part because it is an 'invisible' one. It's visibility is triggered by the state of the modalStates selected from Store.

``` javascript
...
const modalStates = useSelector(state => state.modals)
...
```
The condition for rendering the modal is simply: if modalStates include a true value, then check which of the modalStates is returned true, and render the appropriate modal.

That's about it!

There are some further logics for each of those specific modals, which are important to get into, in another section.
