---
title: Logout
weight: 1
---
# Logout

The logout feature is fairly straightforward. It is simply an event handler that's tied to a logout button.

*Event Handler*
```javascript
const logoutHandler = () => {
    window.localStorage.removeItem('token')
    window.location.reload()
}
```
*Logout Button*
```html
<button onClick={logoutHandler}>Log Out</button>
```

As of this stage of the development, the logout button is only on the home page, which means it's located in `routes.js`. In future tweaks, it'll at a corner of all pages.

---

## Update 6th Dec

I've changed the logOut button to be on the LeftNav bar.

There are a few things to note:

### Component Creation
First, to create the logout button, I created a new section within `<NavSections />` component called *section4*.

```javascript
const section4 = {
    id:4,
    label: false,
    name: 'loggedInUser',
    menuItems: [{
        logoUrl: 'https://res.cloudinary.com/dl4murstw/image/upload/v1638760878/user_1_b87gap.png',
        text: 'Logout',
        notPage: true
    }]
}

```

This creates a new button section with *one* menu item, the details of which are within the menuItems attribute.

### Positioning
My next challenge is to move the menuItem to the bottom of the bar.

I needed to not target the menuItem, but the entire section div. Within my architecture, the classname selector is defined in navSection here: `const sectionClass = 'flexbox-item nav-section' + section.id + '-flexitem'`. Hence, to target it, I've added the following in App.css:

```css
.nav-section4-flexitem {
    width: 160px;
  padding: 11px;
  /* border: 1px solid grey; */
  position: absolute;
  bottom: 0px;

}
```

After tinkering around for a bit, I realized that `position:absolute` and `bottom: 0px` seemed the easiest way to go for this element.

Next, I'll need to attach the logOut event handler to the button.

### Event Handling
There was a slight problem with event handling. My logOut menu item expects a URI because within MenuItem, the menuInfo.uri is tied to a href tag.

```html
<a href={menuInfo.uri}><span style={spanStyle}></span> </a>
```

This wouldn't work if I want to supply a logout event handler.

I've decided to create a hot fix for this by creating another `notPage`attribute to the menuItems field within `navSections.js`.

Then, I created a conditional render within JSX of `<MenuItem>` component.

```html {hl_lines=["6-12"]}
<div style={menuItemContainerStyle}>
    <img src={menuInfo.logoUrl}
        height='20px' width='20px'
        alt="S logo"/>
    <span style={menuText}>{menuInfo.text}</span>
    {!menuInfo.notPage &&
        <a href={menuInfo.uri}><span style={spanStyle}></span> </a>
    }

    {menuInfo.notPage &&
        <a href={'/'} onClick={logoutHandler}><span style={spanStyle}></span> </a>
    }

</div>
```

This basically says that for every menuItem, if notPage is true (as in the case for my louOut button), then supply the ahref tag with the handler, instead of just the URI.

That's about it. Logout button done for now. 
