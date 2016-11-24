---
layout: post
title:  "Why I enjoy working with ReactJS"
date:   2016-11-25 10:00:00
author: kjcpaas
comments: true
---

ReactJS by Facebook is one of the most popular Javascript libraries nowadays.

Many developers love it for many reasons, from efficiency to even its tools. I personally like because of how components are built in `jsx`. In one glance, one can easily see 3 very important things: how it looks like (in the DOM), how it internally works, and how it can connect to other components. As someone who loves building things and investigating how something works, this is a very, very important point for me.

Let's consider a toggle button. It's React component code will look like below:

```jsx
// components/ToggleButton.jsx

import React, { Component, PropTypes } from 'react'

class ToggleButton extends Component {
  constructor(props) {
    super(props)
    this.toggle = this.toggle.bind(this)
    this.state = {
      isPressed: props.isPressed
    }
  }

  toggle() {
    this.setState({
      isPressed: !this.state.isPressed
    })
  }

  componentDidUpdate(prevProps, prevState) {
    if(this.props.onToggle && prevState.isPressed !== this.state.isPressed) {
      this.props.onToggle(this.state.isPressed)
    }
  }

  render() {
    const buttonClass = this.state.isPressed ? 'pressed' : null
    return (
      <button className={buttonClass} onClick={this.toggle}>Click me</button>
    )
  }
}

ToggleButton.defaultProps = {
  isPressed: false
}

ToggleButton.propTypes = {
  isPressed: PropTypes.bool,
  onToggle: PropTypes.func
}

export default ToggleButton
```

By just looking at this single file, a lot of information can be known.

## How it looks like

How a component looks like is very easy to infer. One just needs to look at its `render` method.

```jsx
render() {
  const buttonClass = this.state.isPressed ? 'pressed' : null
  return (
    <button className={buttonClass} onClick={this.toggle}>Click me</button>
  )
}
```

By looking at this method, we can easily know the following about the `ToggleButton` component.
- It is a `button` element.
- It has the text **Click me**
- It has the class `pressed` when it is pressed.

Take note that this markup is not really an HTML markup even though it looks similar. The `button` tag is not an actual HTML tag but a representation of the Javascript code below:

```js
React.createElement(
  button,
  { className: buttonClass, onClick: this.toggle },
  'Click me'
)
```

For those who are new to ReactJS, seeing the markup in the same file as the logic might be weird at first since it's usually found in a separate HTML file (or another file type, depending on template engine used).

However, the biggest benefit of having the markup with the same file as the logic is that there is no need to navigate a project's file tree (or the DOM in the browser) just to look for the HTML element that corresponds to a piece of code. The time used for file navigation can be instead used for writing code. This contributes a lot to efficiency.


## How it internally works

Abstraction is very important in software engineering, in general. However, there are times when developers need to dive deeper in to the code to investogate a bug, or enhance a feature. Hence, developers need to easily understand how a component works internally.

To understand how a React component works, one must know its properties and lifecycle.

### Properties

A component's properties have 2 types -- `props` and `state`.

`props` are properties passed by the parent component, while the `state` are properties that are only accessed by the component itself.

Default values of `props` can be set by `defaultProps`, while the types can be restricted by `propTypes`. Using `propTypes` adds validation to the component to make sure the data being passed is compatible with the component.

`state` cannot be assigned directly, except inside `constructor` method where it is initialized. To update the state `this.setState` must be used.

### Lifecycle

The are 3 major steps in React's component lifecycle: **mounting**, **updating**, and **unmounting**. It's straightforward and easy to understand because of the use of `will` and `did` in method names for methods that gets executed around `render`.

Detailed information about React's component lifecycle can be found in the [docs](https://facebook.github.io/react/docs/react-component.html#the-component-lifecycle).

### So how does `ToggleButton` really work?

Let's follow the [component lifecycle](#lifecycle) when interpreting how the component in question works.

```jsx
constructor(props) {
  super(props)
  this.toggle = this.toggle.bind(this)
  this.state = {
    isPressed: props.isPressed
  }
}

ToggleButton.defaultProps = {
  isPressed: false
}
```

From this, it can be easily seen that `props.isPressed` (from the parent component) is assigned as the initial value for `state.isPressed`. If parent doesn't pass anything, `false` is assigned by default.

Next, the component is **rendered**:

```jsx
render() {
  const buttonClass = this.state.isPressed ? 'pressed' : null
  return (
    <button className={buttonClass} onClick={this.toggle}>Click me</button>
  )
}
```

When the user **clicks** on the button, it inverts `isPressed` state, and **renders** again. It **toggles** the existence of the `pressed` HTML class of the button.

```jsx
toggle() {
  this.setState({
    isPressed: !this.state.isPressed
  })
}
```

Since the button is toggled (and therefor **updated**), the `onToggle` function from the parent is called.

```jsx
componentDidUpdate(prevProps, prevState) {
  if(this.props.onToggle && prevState.isPressed !== this.state.isPressed) {
    this.props.onToggle(this.state.isPressed)
  }
}
```

As demonstrated above, figuring out how a component works internally is very easy to do because the steps the component take is very clear.

## How it can connect to other components

Knowing how `ToggleButton` can connect to other components can easily be seen by just looking at `defaultProps` and `propTypes`.

```jsx
ToggleButton.defaultProps = {
  isPressed: false
}

ToggleButton.propTypes = {
  isPressed: PropTypes.bool,
  onToggle: PropTypes.func
}
```

The following information are easily obtainable:
- It accepts `isPressed` and `onToggle` properties, both optional. (if `isRequired` is changed, it's a required property)
- It has `false` as default for `isPressed`.
- It only accepts `boolean` value for `isPressed`, and `function` for `onToggle`.

Since the needed properties are now known, the `ToggleButton` component can be used inside another component:

```jsx
import React, { Component, PropTypes } from 'react'
import ToggleButton from 'components/ToggleButton'

class ParentComponent extends Component {
  render() {
    const onToggle = () => console.log('toggled')
    return (
      <div>
        <p>I have the button</p>
        <ToggleButton isPressed onToggle={onToggle} />
      <div>
    )
  }
}
```

And yes, `ToggleButton` is used just like how an HTML tag is used!

Seeing components used like this makes it look neat. It's also easy to see where a component and logic goes in your DOM.


## Conclusion

ReactJS, for me, is built for builders like myself. The component is well designed because of easy integration with other components. We can also easily 'tinker', if needed be, since we can easily know a lot (if not all) of information about that component by just looking at its file.

It reminds me of the Lego blocks I used to play when I was a child. After all, maybe the biggest reason I enjoy working with ReactJS is that it reminds me of the fun I had as a child.
