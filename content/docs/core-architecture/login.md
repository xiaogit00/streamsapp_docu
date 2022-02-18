---
title: Login
weight: 1
---
# Login

## Beaking down the login architecture
One of the key parts to understanding the login architecture is to examine what happens immediately when we visit the site root. For instance, when we visit *https://app.streamsapp.xyz*, we want one of the first operations to be checking for user token within localStorage. If it exists, we'd want to redirect the user to the 'private' (logged in) section of the site. If not, we'd want to redirect the user to the public *login* page.


## App.js
Much of the login logic is handled by app.js.
{{<details "app.js" open>}}
```javascript {hl_lines=[6, "8-12"]}
//***app.js****
import useToken from 'components/useToken'

const App = () => {

    const { token, setToken } = useToken()

    if(!token && window.location.pathname !== '/signup') {
        return <Login setToken={setToken} />
    } else if (!token && window.location.pathname === '/signup') {
        return <SignUp />
    }

    return (... <Routes/>  ...)
}
```
{{</details>}}

There are two main things for us to note here.

```javascript
const { token, setToken } = useToken()
```

First, the destructured variables, *token* and *setToken*, helps us manipulate token storage and retrieval (I'll touch on this later).



Second, the *if-else* statement
```javascript
if(!token && window.location.pathname !== '/signup') {
    return <Login setToken={setToken} />
} else if (!token && window.location.pathname === '/signup') {
    return <SignUp />
}
```
checks for the presence of tokens, and handles for two cases where token *doesn't exist*.

The first condition directs users to `<Login/>` page if url isn't '/signup.'

The second condition directs users to `<SignUp/>` page if url is '/signup.'


Finally, the return at the end

```javascript
return (... <Routes/>  ...)
```
tells us that if token exists, then direct users straight to `<Routes>`.

### **Examining the useToken() function**

Key to understanding the login architecture is this line:
```javascript
const App = () => {

const { token, setToken } = useToken()
```  

Let us look at useToken.js closely.


***useToken.js***
{{<expand "useToken.js">}}
```javascript {hl_lines=[9,"15-17"]}
//useToken.js
import { useState } from 'react'

export default function useToken() {
    const getToken = () => {
        const tokenString = localStorage.getItem('token')
        return tokenString
    }
    const [token, setToken] = useState(getToken())

    const saveToken = userToken => {
        localStorage.setItem('token', userToken)
        setToken(userToken)
    }
    return {
        setToken: saveToken,
        token
    }
}
```
{{</expand>}}

First, we note that the following line:
```javascript
const [token, setToken] = useState(getToken())
```
creates a state variable for token.  

Second, we note that the `useToken()` function has two methods: `getToken()` and `saveToken()`, and that it **returns an object** with two properties: `setToken` and `token`.

The `getToken()` method gets the token from localStorage, whereas `saveToken()` expects a `userToken` and saves it to localStorage, and calls the `setToken()` state hook to update state of token within the component.  


Now, going back to our initial app.js, reproduced here for convenience:

{{<expand app.js>}}
``` javascript {hl_lines=[6, "8-12"]}
//***app.js****
import useToken from 'components/useToken'

const App = () => {

    const { token, setToken } = useToken()

    if(!token && window.location.pathname !== '/signup') {
        return <Login setToken={setToken} />
    } else if (!token && window.location.pathname === '/signup') {
        return <SignUp />
    }

    return (... <Routes/>  ...)
}
```
{{</expand>}}

When *app.js* is entered, *useToken()* is ran.

The state hook `const [token, setToken] = useState(getToken())` calls *getToken()* and sets the value of state variable token. Reminder: getToken() simply gets token from localStorage.

If no token exists, we direct the user to login page, where we pass setToken function reference as prop.

### **setToken()**
One way to understand the nuances of this system is to follow the logic of the setToken() function. In some ways, the setToken function is really interesting.

{{< columns >}} <!-- begin columns block -->
## app.js

```javascript
...
const { token, setToken } = useToken()
...
return <Login setToken={setToken} />
...
```

<---> <!-- magic separator, between columns -->

## useToken.js
```javascript
...
const [token, setToken] = useState(getToken())
...
const saveToken = userToken => {
    localStorage.setItem('token', userToken)
    setToken(userToken)
}
return {
    setToken: saveToken,
    token
}
...
```

{{< /columns >}}

Within app.js, it is first defined to be the function reference of the object returned from useToken(), effectively pointing to *saveToken()*. Then, looking at *saveToken()*, we note that this function sets the userToken to localStorage as well as state component state.

setToken() is only invoked when login component is rendered - it is passed as prop to Login component. The overall picture therefore, is this: the useToken() function creates the setToken *function reference* which is then passed down into Login component as prop, so that the user login detail can be saved to localStorage and token state is updated.

### Login component

The login component receives the setToken as prop, and sends a post request to *api/login* via *newLogin* service.

***Login.js***
```javascript {hl_lines=[1, 6]}
export default function Login({ setToken }) {
...

        try {
            const token = await newLogin(loginCredentials)
            setToken(token)
            ...
```

If successful, a token will be returned. This token is passed into setToken, which sets localStorage as well as token state variable, triggering a rerender of app.

Now, on the second iteration `app.js` is ran, there will be a token, and *<Routes />* component is returned.  

## Summary

To summarize, the login flow can be visualized as follows:

![](https://res.cloudinary.com/dl4murstw/image/upload/v1638526524/login_architecture_xbidnl.jpg)


1. App looks for token `useToken()`

2. Attempted token retrieval from localStorage using `getToken()`

3. [If-else] If token found, direct to `<Routes/>` (private section). Else, direct to `<Login/>`

4. User enters login details, clicks submit. Login component makes call to `api/login`, returning the token. `setToken(token)` is called, saving token to localStorage and setting token state

5. A rerender is triggered - bringing us back to (3).
