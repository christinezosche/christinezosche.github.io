---
layout: post
title:      "Event handling, state, and asynchronicity in React"
date:       2021-02-14 21:17:11 -0500
permalink:  event_handling_state_and_asynchronicity_in_react
---


React provides some very handy tools and shortcuts to build interactive web apps. One of those is the use of state in components to manage data. Using state, we can render our components (or not render them!) depending on whether certain conditions are met.

For example, say we are building a very simple app with a button and a counter keeping track of how many times we’ve clicked that button.

We can start building our app like so:

```
import React, { Component } from "react";

class CounterApp extends Component {

    constructor () {
        super()
        this.state = {
            clicks: 0,
            hasBeenClicked: false
        };
    }

    render () {
        return (
            <div>
            <h1>{this.state.clicks}</h1>
            <button>Click Me!</button>
            </div>
        )
    }

}

export default CounterApp

```

Simple enough. Note that our button doesn’t do anything just yet, except look pretty. To show how we can use state to determine what our component renders, let’s have our button and counter render only when a start button has been clicked:

```
import React, { Component } from "react";

class CounterApp extends Component {

    constructor () {
        super()
        this.state = {
            clicks: 0,
            hasBeenClicked: false
        };
    }

    render () {
        if (this.state.hasBeenClicked === false) {
            return (
                <div>
                <button>Click Me First!</button>
                </div>
            )
        }

        else {
        return (
            <div>
            <h1>{this.state.clicks}</h1>
            <button>Click Me!</button>
            </div>
        )
        }
    }

}

export default CounterApp

```

This is a good start. Now, all we need to do is have our start button change the hasBeenClicked property in the component’s state to “true” to render our counter: 

```
import React, { Component } from "react";

class CounterApp extends Component {

    constructor () {
        super()
        this.state = {
            clicks: 0,
            hasBeenClicked: false
        };
    }

    toggleHasBeenClicked = () => {
        this.setState({
            hasBeenClicked: true
        })
    }

    render () {
        if (this.state.hasBeenClicked === false) {
            return (
                <div>
                <button onClick={this.toggleHasBeenClicked}>Click Me First!</button>
                </div>
            )
        }

        else {
        return (
            <div>
            <h1>{this.state.clicks}</h1>
            <button>Click Me!</button>
            </div>
        )
        }
    }

}

export default CounterApp
```

When we load the page, we get something like this:
![](https://i.imgur.com/49CJnjD.png)

Then, when we click the “Click Me First!” button, we get this:
![](https://i.imgur.com/g9sGJwL.png)

We’re on our way. Now, all we need to do is add some functionality to the “Click Me!” button. We can follow a similar principle as we did before, updating the component’s state on click and re-rendering our component based on our new state.

```
import React, { Component } from "react";

class CounterApp extends Component {

    constructor () {
        super()
        this.state = {
            clicks: 0,
            hasBeenClicked: false
        };
    }

    toggleHasBeenClicked = () => {
        this.setState({
            hasBeenClicked: true
        })
    }

    addClick = () => {
        this.setState(previousState => {
            return {
            clicks: previousState.clicks + 1
            }
        })
}

    render () {
        if (this.state.hasBeenClicked === false) {
            return (
                <div>
                <button onClick={this.toggleHasBeenClicked}>Click Me First!</button>
                </div>
            )
        }

        else {
        return (
            <div>
            <h1>{this.state.clicks}</h1>
            <button onClick={this.addClick}>Click Me!</button>
            </div>
        )
        }
    }

}

export default CounterApp

```

You may have noticed that the addClick method we wrote is slightly different from the toggleHasBeenClicked method: both update state, but addClick uses something called “previousState”. The reason we do this is because setState is asynchronous. In other words, React gets around to setting state whenever it finds time to do it; it doesn’t happen instantaneously. Thus, if we were to write our addClick method like so:

```

    addClick = () => {
        this.setState({
            clicks: this.state.clicks + 1
         )}
    }

```

It’s possible that multiple setState calls could be grouped together, and our component state could be changed before this setState method executes, breaking our app. This is not a problem in our app, since it’s extremely simple and no setState calls are being grouped. But it’s good practice anyway to use previousState instead of this.state in case a problem arises in the future.

To better illustrate the issues caused by this asynchronicity, let’s say we want to have our app display “Boom!” when we hit the tenth click. You may think we can write our code this way: 

```

import React, { Component } from "react";

class CounterApp extends Component {

    constructor () {
        super()
        this.state = {
            clicks: 0,
            hasBeenClicked: false,
            explosion: false
        };
    }

    toggleHasBeenClicked = () => {
        this.setState({
            hasBeenClicked: true
        })
    }

    addClick = () => {
        this.setState(previousState => {
            return {
            clicks: previousState.clicks + 1
            }
        })
        this.checkForExplosion()
    }

    checkForExplosion = () => {
        if (this.state.clicks === 10) {
            this.setState({
                explosion: true
            })
        }
    }

    render () {
        if (this.state.hasBeenClicked === false) {
            return (
                <div>
                <button onClick={this.toggleHasBeenClicked}>Click Me First!</button>
                </div>
            )
        }

        else if (this.state.explosion === true) {
            return (
                <h1>Boom!</h1>
            )
        }

        else {
        return (
            <div>
            <h1>{this.state.clicks}</h1>
            <button onClick={this.addClick}>Click Me!</button>
            </div>
        )
        }
    }

}

export default CounterApp

```

This seems logical enough. At first glance, it appears as if the following actions are happening in order: When we click “Click Me!”, the number of clicks in our component state is incremented by one, and then, we immediately check for whether this number of clicks is equal to 10. If so, state.explosion is set to “true”, our component re-renders, displaying “Boom!”. But this is not what happens. Instead, we get this:

[](https://youtu.be/LxFm9M43-fw)

That’s not right! It shows the number 10 on our counter, and then, on the 11th click, our app renders “Boom!”.  Why??

![](https://i.imgur.com/CiYNsn1l.png)

Looking at our devtools, we can see that our state has updated to show the number of clicks as 10, but our “explosion” boolean has not updated to “true”.

If we click the button one more time, we get the following:

![](https://i.imgur.com/rWI1HHbl.png)

“explosion” now equals “true”, but our number of clicks is 11. Clearly, our setState calls are not happening instantaneously. The easiest fix to this problem is to avoid using this.state.explosion altogether. Having unnecessary setState calls gets messy not only because the state may not be updated when we want, but each update to state causes a re-render, which could lead us into a rabbit hole of infinite loops and/or unnecessary rerenders.

We can write our component like this:

```
import React, { Component } from "react";

class CounterApp extends Component {

    constructor () {
        super()
        this.state = {
            clicks: 0,
            hasBeenClicked: false
        };
    }

    toggleHasBeenClicked = () => {
        this.setState({
            hasBeenClicked: true
        })
    }

    addClick = () => {
        this.setState(previousState => {
            return {
            clicks: previousState.clicks + 1
            }
        })
    }

    render () {
        if (this.state.hasBeenClicked === false) {
            return (
                <div>
                <button onClick={this.toggleHasBeenClicked}>Click Me First!</button>
                </div>
            )
        }

        else if (this.state.clicks === 10) {
            return (
                <h1>Boom!</h1>
            )
        }

        else {
        return (
            <div>
            <h1>{this.state.clicks}</h1>
            <button onClick={this.addClick}>Click Me!</button>
            </div>
        )
        }
    }

}

export default CounterApp
```

You may be wondering why we don’t see the number “10” appear on our counter before we see “Boom!”. After all, doesn’t our component display the number of clicks? The reason for this is that it never gets the chance to display the number “10”. When the component’s state is updated, React triggers a re-render (which simply means that the “render” method runs again). Thus, when this.state.clicks is updated to 10, React revisits the “render” method. It sees the conditional “else if (this.state.clicks === 10) …” and renders the return value of that conditional (Boom!). It never even reaches the counter, which is found in the “else” statement. The DOM updates, that is, the counter and “Click Me!” button are removed and replaced by “Boom!”

[](https://youtu.be/aHsDkidyccw)


