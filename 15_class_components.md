## Class Components

In React, so far we have seen functional components. But components can also be classes. In the early days of React before we had React Hooks, we had to use classes whenever we needed to work with the React lifecycle directly.

With Modern React, you don't see class based components so much.

So we have this code we've seen before I want to change it into a class.

```js
import { useState } from 'react';

export default function App() {
  return (
    <>
      <Counter startingCount={10} />
      <Counter />
    </>
  );
}

function Counter({startingCount = 0}) {
  const [ count, setCount ] = useState(startingCount);

  return (
    <>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
      <p>Count: {count}</p>
    </>
  );
}
```

Okay so the first thing we want to do. First thing is we want to change this from a function into a class. And a class does not take parameters. And we want to extend the `Component` class which comes from React. And we can remove this `useState()` because we can't use it in a class. We can't really add any hooks in a class. In a class component, we need one method called `render()` which works just like functional components. The return value should be the element we want to render.

```js
import { Component } from 'react';

export default function App() {
  return (
    <>
      <Counter startingCount={10} />
      <Counter />
    </>
  );
}

class Counter extends Component {
  render() {
    return (
      <>
        <button onClick={() => setCount(count + 1)}>
          Increment
        </button>
        <p>Count: {count}</p>
      </>
    );
  }
}
```

Okay so now we have a completed component. We have an issue because `count` and `setCount` are not defined.

The way this is going to work is we are just going to make an instance variable for our state. And in this case it's going to be an object, and this object is going to have a count with a default variable of 0, if we don't have a `startingCount` in props. 

```js
class Counter extends Component {
  state = {
    count: this.props.startingCount ?? 0
  }

  render() {
    ...
  }
}
```

Note that we can do this a different way with a `constructor()`. One other thing we need to do is to call the super constructor.

```js
import { Component } from 'react';

export default function App() {
  return (
    <>
      <Counter startingCount={10} />
      <Counter />
    </>
  );
}

class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = {
      count: props.startingCount ?? 0
    };
  }

  render() {
    return (
      <>
        <button onClick={() => setCount(count + 1)}>
          Increment
        </button>
        <p>Count: {count}</p>
      </>
    );
  }
}
```

So now we have our state but this click event handler is not using it correctly. So when we want to change state, we have a method we call. We don't ever want to manually change `this.state`. Instead we call `this.setState()`, which is similar to the setter returned by `useState()`, but the difference is this is going to take in an entire state object. So we want to set the `count` to `count + 1`.

```js
<button onClick={() => {
  this.setState({
    count: this.state.count + 1
  });
}}>
  Increment
</button>
<p>Count: {this.state.count}</p>
```

I should note that if you use a standard function here instead of an arrow function, this code is not going to work.

```js
<button onClick={function () {
  this.setState({
    count: this.state.count + 1
  });
}}>
  Increment
</button>
<p>Count: {this.state.count}</p>
```

You will see we get some error messages.
`Cannot read properties of undefined (reading setState)`

And it is trying to read `setState`. We are reading `setState` from `this` and it looks like `this` is `undefined`. It is because when we create a new normal function it is creating a new `this` context. And using an arrow function does not create a new `this` context. Most of the time, we want to use an arrow function, we if we want to do it this way, we will need to use `bind()`.

```js
<button onClick={function () {
  this.setState({
    count: this.state.count + 1
  });
}.bind(this)}>
  Increment
</button>
<p>Count: {this.state.count}</p>
```

And so save and refresh, and this works. But for readability I am going to use the arrow function.

So next, I want to look at what we call lifecycle methods. In React Hooks, we can use `useEffect()` to hook into the React lifecycle. But with class components we can use different methods to hook into the lifecycle instead.

So the first lifecycle method we will look at is `componentDidMount()`. This works essentially the same as having a `useEffect(..., [])`. Let's add a `console.log()`.

```js
class Counter extends Component {
  componentDidMount() {
    console.log('mounted');
  }

  render() {
    ...
  }
}
```

And you can see "mounted" logged twice. If I increment, "mounted" does not log again because it only happens on mount.

If we wanted it to run some code on every update, we can use another method called `componentDidUpdate()`. It will take in two different parameters, `prevProps` and `prevState`. Let's log these values out.

```js
class Counter extends Component {
  componentDidUpdate(prevProps, prevState) {
    console.log(prevProps, prevState);
  }

  render() {
    ...
  }
}
```

So you can see we get "mounted" twice, but nothing from the update yet. It does not run on mount, only update. So if we can increment the second prop, we get an empty object and a count of 0. If you click again, we get an empty object and a count of 1. Likewise for the first counter, we get the `startingCount: 10` in the object and a count of 10. And if we increment we get the same props but a count of 11.

```js
class Counter extends Component {
  componentDidUpdate() {
    console.log('mounted');
  }

  render() {
    ...
  }
}
```

We also have a lifecycle method for unmount called `componentWillUnmount()` and will run when a component is about to unmount.

So we'll create a new button with conditional rendering so we can see the method in action.

```js
import { Component, useState } from 'react';

export default function App() {
  const [ shouldRender, setShouldRender ] = useState(false);
 
  return (
    <>
      <Counter startingCount={10} />
      {shouldRender && <Counter />}
      <button onClick={() => setShouldRender(!shouldRender)}>
        Toggle Counter 
      </button>
    </>
  );
}

class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = {
      count: props.startingCount ?? 0
    };
  }

  componentWillUnmount() {
    console.log('unmounting')
  }

  render() {
    return (
      <>
        <button onClick={() => {
          this.setState({
            count: this.state.count + 1
          });
        }}>
          Increment
        </button>
        <p>Count: {this.state.count}</p>
      </>
    );
  }
}
```

And if we save, we can toggle the button. And we toggle the button to hide the component we can see the console logging "unmounting".

There's one last lifecycle method called `shouldComponentUpdate()` and this takes in `nextProps` and `nextState`. It works very similarly to the React `memo()` we have seen before. Whenever props or state change, it will get the nextProps and nextState. If it returns true, the component will render. If it returns false, the component will not re-render.

So let's try making it return `nextState.count < 3`. If the count is 3 or higher the update will not happen.

```js
class Counter extends Component {
  shouldComponentUpdate(nextProps, nextState) {
    console.log(nextProps, nextState);
    return nextState.count < 3;
  }

  render() {
    ...
  }
}
```

If I click increment, it works as expected up to 2. If I click increment one more time, nothing will happen because it does not update anymore. Because of the `console.log()`, we can see that we are still logging out the `nextProps` and `nextState` even though increment doesn't do anything after 2. It's just the component is not updating even though the state is updating.

The last thing is refs. In a functional component we used `useRef()`. In class-based components we will use `createRef()`. And then we need to use the ref. If we want to add the ref to our button, we can simply set the ref property on the button equal to the ref.

```js
import { Component, createRef, useState } from 'react';

export default function App() {
  const [ shouldRender, setShouldRender ] = useState(false);
 
  return (
    <>
      <Counter startingCount={10} />
      {shouldRender && <Counter />}
      <button onClick={() => setShouldRender(!shouldRender)}>
        Toggle Counter 
      </button>
    </>
  );
}

class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = {
      count: props.startingCount ?? 0
    };
    this.buttonRef = createRef();
  }

  componentDidMount() {
    console.log('mounted');
    console.log('buttonRef in componentDidMount():');
    console.log(this.buttonRef);
  }

  componentDidUpdate(prevProps, prevState) {
    console.log(prevProps, prevState);
  }

  componentWillUnmount() {
    console.log('unmounting')
  }

/*
  shouldComponentUpdate(nextProps, nextState) {
    return nextState.count < 3;
  }
*/

  render() {
    console.log('buttonRef in render() method:')
    console.log(this.buttonRef);
    return (
      <>
        <button ref={buttonRef} onClick={() => {
          this.setState({
            count: this.state.count + 1
          });
        }}>
          Increment
        </button>
        <p>Count: {this.state.count}</p>
      </>
    );
  }
}
```

So we can see `buttonRef` holds `current: button` in `componentDidMount()` but is an object with `current: null` initially for render(), and then later when we increment we can see `current: button`. Keep in mind if you set a ref to a DOM Node it will not be set in the initial call to `render()`.

What I want to do next is look at a context. This is going to be a `Theme` that keeps track of the mode, whether it's light mode or dark mode. Let's import `createContext()`.

So first of all we need to add a static variable and this is going to be called `contextType`. This is the return value of `createContext()`. 

If we add `console.log(this.context)` anywhere inside this component, we can see it returns the context value from that `contextType`.

So we can say a paragraph with a Theme which will be `this.context.mode`. Now this does work but we can only consume one context at a time. 

What if we want multiple contexts? We will say `Theme.Consumer`. It takes in a function as the children. The function takes in a parameter of the current context. So in this case, our paragraph that says Theme.

```
const Theme = createContext({
  mode: 'dark'
})
```

```js
import { Component, createContext, createRef, useState } from 'react';

const Theme = createContext({
  mode: 'dark'
});

export default function App() {
  const [ shouldRender, setShouldRender ] = useState(false);
 
  return (
    <>
      <Theme.Provider value={{mode: 'dark'}}>
        <Counter startingCount={10} />
        <hr />
        {shouldRender && <Counter />}
        <button onClick={() => setShouldRender(!shouldRender)}>
          Toggle Counter 
        </button>
        </Theme.Provider>
    </>
  );
}

class Counter extends Component {
  static contextType = Theme;

  constructor(props) {
    super(props);
    this.state = {
      count: props.startingCount ?? 0
    };
    this.buttonRef = createRef();
  }

  componentDidMount() {
    console.log('mounted');
    console.log('buttonRef in componentDidMount():');
    console.log(this.buttonRef);
  }

  componentDidUpdate(prevProps, prevState) {
    console.log(prevProps, prevState);
  }

  componentWillUnmount() {
    console.log('unmounting')
  }

/*
  shouldComponentUpdate(nextProps, nextState) {
    return nextState.count < 3;
  }
*/

  render() {
    console.log('buttonRef in render() method:')
    console.log(this.buttonRef);
    return (
      <>
        <button ref={this.buttonRef} onClick={() => {
          this.setState({
            count: this.state.count + 1
          });
        }}>
          Increment
        </button>
        <p>Count: {this.state.count}</p>
        {
          /* <p>Theme: {this.context.mode}</p> */
        }
        <Theme.Consumer>
          {
            context => <p>Theme: {context.mode}</p>
          }
        </Theme.Consumer>
      </>
    );
  }
}
```

We can save and you can see it is working as before.

Most of the time, you would want to use functional components, but nothing wrong with class-based components. You can do the same in both but syntax is different and class-based tends to be more verbose.
