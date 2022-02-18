---
title: Floating Action Button
weight: 1
---

# Floating Action Button

![](https://res.cloudinary.com/dl4murstw/image/upload/v1645072127/Screenshot_2022-02-17_at_12.26.23_PM_rkncrc.png)

The Floating Action Button is basically a `<SpeedDial>` component imported from Material UI. The workings of the SpeedDial component is rather unique, which I'll try to illuminate here.

## Component Hierarchy:

*App>RightPane>ContentContainer> speedDail>*

## SpeedDial Component

The floating action button is a SpeedDial component.

The general syntax of this component looks like this:
{{< details "speedDail.js" >}}
```javascript
const actions = [
    { icon: <AddchartIcon />, name: 'Add Trade', operation: 'trade' },
    { icon: <WavesIcon />, name: 'Add Stream', operation: 'stream' }
]


export default function BasicSpeedDial() {

    const dispatch = useDispatch()

    //
    const handleClick = (e, operation) => {
        e.preventDefault()
        if (operation === 'stream'){
            dispatch({type:'TOGGLE_STREAM'})
        } else if (operation === 'order') {
            dispatch({type:'TOGGLE_ORDER'})
        } else if (operation === 'trade') {
            dispatch({type:'TOGGLE_TRADE'})
        }
    }

    return (
        <>
            <Box sx={{ height: '90%',
                transform: 'translateZ(0px)',
                flexGrow: 1}}>
                <SpeedDial
                    ariaLabel="SpeedDial openIcon example"
                    sx={{ position: 'absolute', bottom: 14, left: 60,
                        '& .MuiFab-primary': {
                            backgroundColor: '#460F61',
                            '&:hover': {
                                backgroundColor: '#4E116D'
                            },
                            border: '1px solid blue'
                        }
                    }}
                    icon={<SpeedDialIcon openIcon={<EditIcon />} />}
                >
                    {actions.map((action) => (

                        <SpeedDialAction
                            key={action.name}
                            icon={action.icon}
                            tooltipTitle={action.name}

                            onClick={e => {
                                handleClick(e, action.operation)
                            }}
                        />
                    ))}

                </SpeedDial>

            </Box>
            <Modal />
        </>
    )
}

```
{{< /details >}}

Specifically, the `<BasicSpeedDial>` component is a `<Box>` component that wraps a SpeedDial component, which itself wraps a couple of `<SpeedDialAction>` components.

The `<SpeedDial>` component has an sx field, which I have customized. (Customization of MUI component styles is another topic that I should probably touch on.) Within it, I am mapping over the `actions` array to generate corresponding `<SpeedDialAction>` components- a common trick. The `actions` are arrays of objects, each object having the following property: icon, name, operation.

``` javascript
const actions = [
    { icon: <AddchartIcon />, name: 'Add Trade', operation: 'trade' },
    { icon: <WavesIcon />, name: 'Add Stream', operation: 'stream' }
]
```


Each `<SpeedDialAction>` component has the following props: key, icon, tooltipTitle, and onClick handler.

``` javascript
{actions.map((action) => (

                        <SpeedDialAction
                            key={action.name}
                            icon={action.icon}
                            tooltipTitle={action.name}

                            onClick={e => {
                                handleClick(e, action.operation)
                            }}
                        />
                    ))}
```


With these, I can specify what the icons are, what the tool tip suggests, and what happens on click.

In this specific case, onClick handler sends a dispatch() to the store with TOGGLE_STREAM and TOGGLE_ORDER as actions. These is separately handled within `modalReducer.js`, where if the state is detected to be true, opens up the respective Modal forms.

Finally, the `<Modal>` component is appended to the end of speedDial component so that the modals can be triggered.

That's about it for the SpeedDial component!
