---
layout: post
title: "React/Redux for the uninitiated"
date: 2017-03-27 11:00:00
author: kristm
comments: true
image: https://c2.staticflickr.com/4/3294/2298750408_57553092a6_o.jpg
---

In the latest [developer survey of StackOverflow](http://stackoverflow.com/insights/survey/2017/), React is the most loved framework and Javascript is the most popular language.<sup>[1](#footnote1)</sup>
I hear the words _New Hotness_ thrown a lot. Surprisingly, former Hotness Coffeescript has now been relegated to the most hated list.

Backbone nostalgia aside, it does make me raise a question why most developers today choose to make their web frontend increasingly complex. Should all client side code be
obligatorily heavy just because the technology is there? Or the hardware can (presumably) handle it? Probably not. But rightly or wrongly it seems that's where we are right now. So in order to be part of the steamroller instead of the road<sup>[2](#footnote2)</sup> I'll attempt to grok a react/redux app by stripping it to its bare essential.

### ES6 and Babel

Forget about webpack and all the fancy superflous tooling, React at its core is just ES6. And because it's not supported in all browsers yet, we're going to need a transpiler to manage the ES5 translation<sup>[3](#footnote3)</sup>. That would be Babel. You're also not going to be messing with the DOM as far as React is concerned. It has its own virtual DOM<sup>[4](#footnote4)</sup>. So at the very basic, we're going to be using 3 things<sup>[5](#footnote5)</sup>:

```
<script src="https://cdnjs.cloudflare.com/ajax/libs/react/15.4.2/react.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/react/15.4.2/react-dom.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/6.23.1/babel.min.js"></script>
```

For the actual code, let's just make a simple event handler to see how things work

```
<div id="container">
</div>

<script type="text/babel">
  ReactDOM.render(
    <div>
      <h1>
        HELLO
      </h1>
      <button onClick={function(){ console.log('awyeah'); }}>Click me</button>
    </div>,
    document.getElementById('container'));
</script>
```

So far, so good. All of the action takes place in that render call. So this will be like the program's entry point. It takes 2 arguments, the app markup and the root node. To take things further, we would need to extract the markup<sup>[6](#footnote6)</sup> from there and build a proper React class.

```
<script type="text/babel">
var Hello = React.createClass({
  render: function() {
    return (
      <div>
        <h1>
          HELLO
        </h1>
        <br />
        <button onClick={function(){ console.log('awyeah'); }}>Click me</button>
      </div>
    )
  }
})
ReactDOM.render(
  <Hello />,
   document.getElementById('container'));
</script>
```

That `<script type="text/babel">` is making me feel hip already. To make things really fly, let's add in the State.

### State

One of the big deal about react is something called the State. It's like a data representation of the app. I like to think of it as a switchboard that shows and controls all of the app's behavior. Best way to illustrate it is probably by code.

Say you want to show a modal. In event-driven thinking, this could be accomplished by adding a button event handler to insert the modal into your DOM. But in react, you only need to turn the modal "on" from the State (In this case, turning it "on" is merely toggling the css class of the modal).

```
<script type="text/babel">
var Hello = React.createClass({
  getInitialState: function() {
    return { showModal: false }
  },
  toggleModal: function() {
    this.setState({
      showModal: !!!this.state.showModal
    })
  },
  render: function() {
    return (
      <div>
        <h1>
          HELLO
        </h1>
        <br />
        <button onClick={this.toggleModal}>Click me</button>
        <div class="modal" className={this.state.showModal ? '' : 'hidden'}>I Modal</div>
      </div>
    )
  }
})
ReactDOM.render(
  <Hello />,
   document.getElementById('container'));
</script>
```

Okay, so the State seem to be just a big list of variables that represents various ehem, states of the app.  Here we're using the State to represent the visibility of the modal. Any changes in that state will instantly be reflected in our render method. Suffice to say, one of the reason for react's success is its Virtual DOM implementation<sup>[7](#footnote7)</sup>. 

### Redux

Though not in the same developer survey, one of the things that seem to excite a lot of developers (I know) today is Redux. It's basically a souped-up State when React's State is not enough<sup>[8](#footnote8)</sup>. The website tells it better, Redux is a predictable state container for Javascript apps (which has got to be one of the best written product bylines I've seen).
To be honest, I'm not sure when React State is not enough or when Redux should come in<sup>[9](#footnote9)</sup>. But just for the purposes of illustration, let's see what it takes to convert our trivial app to Redux.
First, we'll need to load the redux library.

```
<script src="https://cdnjs.cloudflare.com/ajax/libs/redux/3.6.0/redux.min.js"></script>
```

```
<script type="text/babel">
  const Hello = React.createClass({
    getInitialState: function() {
      return store.getState();
    },
    toggleModal: function() {
      const { showModal } = store.getState()
      store.dispatch({ type: showModal ? 'HIDE' : 'SHOW' })
    },
    render: function() {
      return (
        <div>
          <h1>
            HELLO
          </h1>
          modal: {store.getState().showModal.toString()}
          <br />
          <button onClick={this.toggleModal}>Click me</button>
          <div id="modal" className={store.getState().showModal ? '' : 'hide'}>
            i'm modal
          </div>
        </div>
      )
    }
  })

  const reducer = function(state, action) {
    let initialState = { showModal: false }
    switch(action.type) {
      case 'SHOW':
        return { showModal: true };
      case 'HIDE':
      default:
        return initialState;
    }
  }

  const { createStore } = Redux;
  const store = createStore(reducer)

  const render = function() {
    ReactDOM.render(
      <Hello />,
      document.getElementById('container')
    )
  }

  store.subscribe(render);
  render();
</script>
```

Whoa! Our trivial app got a lot more complicated without really doing anything new. Instead of the State, we now have something called the Store, which probably means State on Steriods. What happened to our `toggleModal` though? It uses something called dispatch. It's because you can't manipulate the Store directly, it has to go through the Redux dispatcher. Anyhow, our state accessors got a lot longer. So there's this thing called the `Provider` which supports something called `Contexts`. It can help make the state more accessible, but we need to add another library

```
<script src="https://cdnjs.cloudflare.com/ajax/libs/react-redux/5.0.3/react-redux.min.js"></script>
```

now we change our render a bit to use the Provider

```
const render = function() {
  ReactDOM.render(
    <ReactRedux.Provider store={store}>
      <Hello />
    </ReactRedux.Provider>,
    document.getElementById('container')
  )
}
```

To use Contexts, we add this to our React class

```js
contextTypes: {
  store: React.PropTypes.object
}
```

then instead of calling the global store, we can access it via the local context (and while we're at it, why we don't make full use of our store to remove all hardcoded values)

```js
getInitialState: function() {
  const { store } = this.context
  return store.getState();
},
toggleModal: function() {
  const { store } = this.context
  const state = store.getState()

  store.dispatch({ type: state.showModal ? 'HIDE' : 'SHOW' })
},
render: function() {
  const { store } = this.context
  const state = store.getState()

  return (
    <div>
      <h1>
        {state.name}
      </h1>
      <button onClick={this.toggleModal} className={state.label}>{state.label}</button>
      <div id="modal" className={state.showModal ? '' : 'hide'} onClick={this.toggleModal}>
        i'm modal
      </div>
    </div>
  )
}
```

Now we need to update our reducer to fill in our new store values

```js
const reducer = function(state, action) {
  let initialState = { showModal: false, name: 'HELLO', label: 'show' }
  switch(action.type) {
    case 'SHOW':
      return { ...state, showModal: true, label: 'hide' };
    case 'HIDE':
    default:
      return initialState;
  }
}
```

Funny, our code keeps getting longer and longer (and colors!) and no single feature has been added.

### Summary

It's obvious that React/Redux is built for client side heavy apps that do a lot of state processing. There's just too much overhead to justify the costs for a simple app. An app to be usable must first be available after all. And I mean _will download quickly and without much ceremony_. The technology is there. But devs should also be asking _when_ to use it. It's a constant balancing act between pragmatism and overengineering.

Final demo source: [http://codepen.io/anon/pen/LyPagR](http://codepen.io/anon/pen/LyPagR)

**Notes**

<small>
<a name="footnote1">1</a>: [http://stackoverflow.com/insights/survey/2017/](http://stackoverflow.com/insights/survey/2017/#technology-most-loved-dreaded-and-wanted-frameworks-libraries-and-other-technologies)<br>
<a name="footnote2">2</a> _"Once a new technology rolls over you, if you’re not part of the steamroller, you’re part of the road."_ - Steward Brand<br>
<a name="footnote3">3</a> [https://www.quora.com/What-exactly-is-BabelJs-Why-does-it-understand-JSX-React-components](https://www.quora.com/What-exactly-is-BabelJs-Why-does-it-understand-JSX-React-components)<br>
<a name="footnote4">4</a> [http://stackoverflow.com/questions/21109361/why-is-reacts-concept-of-virtual-dom-said-to-be-more-performant-than-dirty-mode](http://stackoverflow.com/questions/21109361/why-is-reacts-concept-of-virtual-dom-said-to-be-more-performant-than-dirty-mode)<br>
<a name="footnote5">5</a> I came late to this party, but prior to [version 0.14 ReactDOM was part of the React core](https://facebook.github.io/react/blog/2015/10/07/react-v0.14.html)<br>
<a name="footnote6">6</a> In a typical React app this would go in a JSX file<br>
<a name="footnote7">7</a> [Why is React's concept of Virtual DOM said to be more performant than dirty model checking?](http://stackoverflow.com/questions/21109361/why-is-reacts-concept-of-virtual-dom-said-to-be-more-performant-than-dirty-mode)<br>
<a name="footnote8">8</a> Okay, maybe this is a gross oversimplification. Redux takes after [Flux](http://redux.js.org/docs/introduction/PriorArt.html).<br>It's also said that it's one of the best implementation of flux that's [not really flux](https://twitter.com/andrestaltz/status/616271392930201604)<br>
<a name="footnote9">9</a> There's an adequately comprehensive series of video tutorials from Redux's author himself [here](https://egghead.io/courses/getting-started-with-redux)
</small>
