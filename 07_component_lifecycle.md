## Component LifeCycle

So this is the main `App` component. We can see it has this state with the intial state being true. Then we return a fragment with a button that calls `setIsShown()` when clicked and shows the Counter based on the `isShown` state.

So we can see that `Counter` is similar to counters in the past. The first button isn't going to seem like it does much but we'll see what it does. `Re-Render` just toggles this boolean state. This boolean isn't necessarily being used for anything. It just forces the component to re-render. And we have this button called Increment and works like buttons we have seen before.

We can click `Increment` to increment the count. And we can click `Hide Counter` to hide the counter and click `Show Counter` to show it. Because it creates a new version of that Counter the count is reset. We can click `Re-render` but it doesn't seem like it does much.

```js
import { useState } from 'react';
import Counter from './Counter';
import './App.css';

export default function App() {
  const [isShown, setIsShown] = useState(true);

  return (
    <>
      <button onClick={() => setIsShown(!isShown)}>
        {isShown ? 'Hide Counter' : 'Show Counter')
      </button>
      {isShown ? <Counter /> : null}
    </>
  );
}
```

```js
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  const [bool, setBool] = useState(false);

  return (
    <>
      <div className="counter">
        <button onClick={() => setBool(!bool)}>Re-Render</button>
        <button onClick={() => setCount(count + 1)}>Increment</button>
        <p>Count: {count}</p>
      </div>
    </>
  );
}
```

This how the component lifecycle is going to work in React:
`Mount (initial updates) -> updates (re-render) -> unmount`

The first stage is going to be Mount. This stage is just the initial render. So this is when the component is added to the screen for the first time. When we click `Show Counter`, that is mounting a new component. 

And the next stage is going to be for updates. This is for whenever we re-render the component. So whenever the state or props are changing. But it will not mount the component. Changes the values and updates the DOM.

The last stage is unmounting. Unmounting is when the component leaves the screen. And it unmounts it no longer exists anymore. When we click the `Hide Counter` it is unmounting. When we click `Show Counter` it is mounting. Whenever we click `Increment` or `Re-Render` it is updating.

### Hooking into the Component Lifecycle

Okay, but sometimes we need to hook into this component lifecycle to do something. And we can see how useEffect works. So we need to make sure we aren't calling it conditionally, and we don't need the return value. And it will take in another function as a parameter, but it should not be an async function.

So now what `useEffect` is going to do, is going to run after every render. So the mount or any updates is going to cause the `useEffect` function to run.

```js
import { useEffect, useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  const [bool, setBool] = useState(false);

  useEffect(() => {
    console.log('render');
  });

  return (
    <>
      <div className="counter">
        <button onClick={() => setBool(!bool)}>Re-Render</button>
        <button onClick={() => setCount(count + 1)}>Increment</button>
        <p>Count: {count}</p>
      </div>
    </>
  );
}
```

So you can see immediately "render" has logged out to console. When we click `Increment` or `Re-Render`, "render" is logged out. Now a key point is if I click `Hide Counter`, it did not call the callback. So unmounting does not render. But if I click `Show Counter`, "render" is logged out because mounting includes a render step.

### Hooking only *some* renders

Most of the time, we want with `useEffect` is call it on some renders, not all renders.

In this `useEffect`, I'm going to add a 2nd argument called the dependency array, which will determine when the effect will run. However, here it will only run when something in the dependency array changes (however it will always run on mount). So as we see it only run on mount and never again when we click `Re-Render` or `Increment`.

```js
import { useEffect, useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  const [bool, setBool] = useState(false);

  useEffect(() => {
    console.log('mounted');
  }, []);

  useEffect(() => {
    console.log('render');
  });

  return (
    <>
      <div className="counter">
        <button onClick={() => setBool(!bool)}>Re-Render</button>
        <button onClick={() => setCount(count + 1)}>Increment</button>
        <p>Count: {count}</p>
      </div>
    </>
  );
}
```

Now if we change the first `useEffect` to only render when `bool` is `true`, it only shows up when we press `Re-Render`.

```js
useEffect(() => {
  console.log('pressed re-render');
}, [bool]);
```

Now these functions passed to `useEffect` can also return a function used as a cleanup function. Only if you click `Hide Counter`, will "unmount" be logged out.

```js
useEffect(() => {
  console.log('mounted');

  return () => console.log('unmount');
}, []);
```

But let's see what happens if we have a dependency array.

So when we run it, we get "count changed" as it runs on mount. Then when we hide the counter, we get "cleanup count changed". Whenever I click `Increment`, you will see first the "cleanup count changed" and then you will get the "count changed". This is very usable for subscriptions. You want to break the connection in a cleanup function.

```js
useEffect(() => {
  console.log('count changed');

  return () => console.log('cleanup count changed');
}, [count]);
```

### Infinite loops with `useEffect`

Last thing is a common error is to create an infinite loop in the `useEffect`. So now what's going to happen? Well we have the count as the dependency array, whenever count changes, this effect needs to run. But the count changes inside of the effect. So it's going to cause the effect to run again and going to cause an infinite loop.

```js
useEffect(() => {
  // Break infinite loop
  if (count > 20) return;
  setCount(count + 1);
}, [count]);
```

### `useLayoutEffect`

So I want to show what is known as a layout effect. If count is 3, I want to set count to be 4 and skip 3. And now this seems like it should work if the count becomes 3, it will be updated to be 4. We can run this. Increment the count. 1, 2. And becomes 4. Seems to have worked. However if we watched very slowly it is actually going to flash the number 3.

```js
useEffect(() => {
  if (count === 3) { 
    setCount(4);
  }
}, [count]);
```

So I am going to create a while loop to stall this.

Now and when I increment to 4, you see it flashed 3 for a little bit of time. Reason is `useEffect` run asynchronously after the render EndPaint. Runs after the component has been painted to the screen. And this will cause 3 to be on the screen for the moment. For the vast majority of effects, it can be an issue.

```js
import { useEffect, useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  const [bool, setBool] = useState(false);

  useEffect(() => {
    if (count === 3) { 
      setCount(4);
    }
  }, [count]);

  useEffect(() => {
    console.log('render');
  });

  const startTime = new Date();
  while (new Date() - startTime < 100) {}

  return (
    <>
      <div className="counter">
        <button onClick={() => setBool(!bool)}>Re-Render</button>
        <button onClick={() => setCount(count + 1)}>Increment</button>
        <p>Count: {count}</p>
      </div>
    </>
  );
}
```

So the fix for this is `useLayoutEffect` because it runs synchronously so the render will not run until the `useLayoutEffect` is done running. So it won't flash 3. But you should avoid using it unless needed because running synchronously means it can cause things to run more slowly.

```js
useLayoutEffect(() => {
  if (count === 3) { 
    setCount(4);
  }
}, [count]);
```
