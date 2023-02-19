## Error Handling

So here we have an App component and it has an h1 that says Hello World and it has this Buggy component. From the output, we get nothing due to the error. If an error is thrown anywhere throughout the React tree, we are going to get nothing. And that is not a very good user experience. You could expect a very complex React tree. It might throw an error if a service it sends a request to is not up.

```js
export default function App() {
  return (
    <>
      <h1>Hello World</h1>
      <Buggy />
    </>
  );
}

function Buggy() {
  throw new Error('error');
  return <h1>Buggy</h1>;
}
```

How can we handle this? We can do something called an error boundary. What will happen in the event that one of its children will throw an error.

However, to do this we will need to add a Class-Based Component. We will call this class `ErrorBoundary`. This is the standard name we tend to use.

So every class-based component need a `render()` method. And when we render this component, there are two options. Either there is an error or there isn't. And we will hold this in state. So if there is an error, we will return some fallback, some component that can come from props. And of course you could return anything here, but I will allow this fallback to come props. But if there is no error, we want `ErrorBoundary` to return its children.

Okay, now we need to define the state. We could define it at the top-level or within a constructor. Let's do it top-level. Okay, we need a way to know if there is an error. Traditionally we do it using try-catch but try-catch is imperative. But we need this in a declarative way for React. 

So for this React actually has a lifecycle method we can use. It will be a static method called `getDerivedStateFromError()` is going to take in some error. And this method needs to return the new state. And `hasError` is going to be true. We could pass into the state the error itself, but for now let's leave it with `hasError`.

So let's recap a bit of what this class is actually doing. It has some state called `hasError`. In the event there is an error in the children, it will change the state to `hasError: true`. If there is an error, there is a fallback. Otherwise it returns a children. So now let's test it. Let's wrap any components which may have an error so we will wrap the `Error` component. And we need to pass in a component. Let's make the fallback component say there was an error.

```js
import { Component } from 'react';

export default function App() {
  return (
    <>
      <h1>Hello World</h1>
      <ErrorBoundary fallback={<h1>There was an error.</h1>}>
        <Buggy />
      </ErrorBoundary>
    </>
  );
}

function Buggy() {
  throw new Error('error');
  return <h1>Buggy</h1>;
}

class ErrorBoundary extends Component {
  state = { hasError: false }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback;
    }
    return this.props.children;
  }
}
```

Okay now we save this and it renders. We get "Hello World". The error does not stop other components from rendering. And we get "There was an error." And if we comment out the error then it shows "Buggy".

One thing to keep in mind the `ErrorBoundary` only catches errors in its children. It also can't catch errors in itself

So these would still not work.

```js
export default function App() {
  throw new Error('oh no');
  return (
    <>
      <h1>Hello World</h1>
      <ErrorBoundary fallback={<h1>There was an error.</h1>}>
        <Buggy />
      </ErrorBoundary>
    </>
  );
}
``` 

```js
class ErrorBoundary extends Component {
  state = { hasError: false }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  render() {
    throw new Error('oh no');
    if (this.state.hasError) {
      return this.props.fallback;
    }
    return this.props.children;
  }
}
```

So there is one more lifecycle method we can use here. In the case there are some side effects, maybe we want to log out the error to our server, we have another method we can use. `componentDidCatch(error, errorInfo)`. `error` will be the error and `errorInfo` will be some info about the component that actually threw the error. 

While `getDerivedStateFromError()` runs during the render phase, `componentDidCatch()` does not run until the commit phase later down the line.

We might maybe want to save this error to our server we would have a `logErrorToServer(error, errorInfo)`. But this is for side effects.

So to sum things up:

1. You need to use a class. Not a hook for error handling currently.
2. Use this `getDerivedStateFromError()` lifecycle method to change state.
3. Just conditionally render something in the event of an error. If there's an error there needs to be a fallback otherwise render the children.
