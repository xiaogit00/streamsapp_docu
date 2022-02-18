---
title: Navigation Menu
weight: 1
---

# Navigation Menu

The Navigation Menu of Streamsapp is the menu on the left. It is a static one that is wrapped in the `<LeftNav />` base component. To maintain it, we'll need to know the general component hierarchy.


## Component Hierarchy
![](https://res.cloudinary.com/dl4murstw/image/upload/v1638669212/Screenshot_2021-12-05_at_8.50.09_AM_rkmvgd.png)
First, we need to know that every `<LeftNav />` on the site is rendered in the *routes.js* file.

*Routes.js*
```html {hl_lines=[4,5, 12,13, 16,17]}
<Router>
    <Switch>
        {/* Home Page */}
        <Route exact path="/">
            <LeftNav />
            <div style={{fontFamily:'menlo',fontSize:'2em', margin:'30px'}}>Dashboard feature coming soon!</div>
            <p style={{fontFamily:'menlo',fontSize:'1em', margin:'30px'}}> In the mean time, listen to some jazz while keying in your trades...</p>
            <iframe style={{marginLeft:'30px'}} width="800" height="430" src="https://www.youtube.com/embed/u-32Wr8Gxzk" title="YouTube video player" frameBorder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowFullScreen></iframe>
            <button onClick={logoutHandler}>Log Out</button>
        </Route>
        {/* React Exercises Page */}
        <Route path="/streams">
            <LeftNav />
            <RightPane type="streams"/>
        </Route>
        <Route path="/trades">
            <LeftNav />
            <RightPane type="trades"/>
        </Route>
        <Redirect from="*" to="/" />
    </Switch>
</Router>
```

As you can see, our site's navigation menu is present as `<LeftNav />` within each page.

Opening up the `<LeftNav/>` component, we see that it contains NavigationMenu, which itself contains NavHeader and NavSections.

*LeftNav*
```javascript
const LeftNav = () => {


    return (
        <div className='left-nav'>
            <NavigationMenu />
        </div>
    )
}

export default LeftNav
```
*NavigationMenu*
```javascript
const NavigationMenu = () => {

    return (
        <div className='navigation-menu'>
            <NavHeader />
            <NavSections />
        </div>
    )
}
```
There are two more children components down the line, MenuItems, and MenuItem, which contain the actual Menu Items.




---
## Components on the page  

{{< figure src="https://res.cloudinary.com/dl4murstw/image/upload/v1638669215/pic_lrgzjh.jpg" width="50%" >}}

Above is the break down of the components on the page. As you can see, the entire purple bar is `<LeftNav />`. On the top, we have `<NavHeader />` which we use to store the site logo. Below that, we have `<NavSection />` component, which contains three `<NavSection>`s. The first section contains Dashboard, the second section is titled *Track*, and the third section is *Overview*. Within each of them, there are the `<MenuItems>` component which itself contains multiple `<MenuItem>`s.
