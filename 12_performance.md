## Performance

Two main categories of performance:
1. How do we make renders more efficient?
2. How can we minimize the number of renders?

### How do we make renders more efficient?

So if we have less renders and our renders take less time, our website will be more performant. Let's take a look at this example. Two pieces of state, `num` and log value. `num` will be 10 and `logValue` will be empty string.

For the return value, we can see have an h1 that says Fib and the number and the fib of that number. And then we have an input to change the number. Another text input that is controlled by the `logValue` state. A button that logs the `logValue`.

```js
import { useState } from 'react';
import MyButton from './MyButton';

export default function App() {
  const [num, setNum] = useState(10);
  const [logValue, setLogValue] = useState('');

  return (
    <>
      <h1>Fib {num} is {fib(num)}</h1>
      <input
        type="number"
        value={num}
        onChange={(event) => setNum(parseInt(event.target.value))}
      />

      <br />
      <br />
      
      <input
        type="text"
        value={logValue}
        onChange={(event) => setLogValue(event.target.value)}
      />
      
      <MyButton onClick={() => {
        console.log(logValue)
      }}>Log Value</MyButton>
    </>
  );
}

function fib(n) {
  if (n == 2) return 1;
  if (n == 1) return 0;
  return fib(n-2) + fib(n-1);
}
```

And `MyButton` is a very basic component

```js
export default function MyButton(props) {
  return <button {...props} style={{color: 'red'}} />;
}
```

And `fib` is a very basic implementation of the Fibonacci function. So this function takes in a number and returns a new number.

Let's go over to the browser. If I increase this, the number actually changes. And same thing if I decrease it. And then we have `logValue` input, and it logs out `Hello` if I type `Hello` and click the button.

So first of all, we said want to minimize the amount of time a render takes. So what is happening in each render? We create this big return value. One key point is we calculate `fib()` of the number in the input even if the number doesn't change.

So the way we can fix this is by using memoization. Sometimes we don't need to calculate this function again. And if those parameters are not matched, we will not calculate it again and we will just use the previous value. We can do this with another hook called `useMemo`.

`useMemo` is going to take in a function. We can have a function that calls `fib(num)`. Take in this function, and take its return value, and make it be the return value of `useMemo`. So `fibValue` is going to set equal to `fib(num)`. But `useMemo()` will only callback function if something in a dependency array has changed. In this case, we want to pass in the number. If the number didn't change on this render, we can use the value from the previous render.

I'm going to add a `console.log()` to keep track of how often this function is running.

```js
import { useMemo, useState } from 'react';
import MyButton from './MyButton';

export default function App() {
  const [num, setNum] = useState(10);
  const [logValue, setLogValue] = useState('');

  const fibValue = useMemo(() => {
    console.log('calculating fib value');
    return fib(num);
  }, [num]);

  return (
    <>
      <h1>Fib {num} is {fibValue}</h1>
      <input
        type="number"
        value={num}
        onChange={(event) => setNum(parseInt(event.target.value))}
      />

      <br />
      <br />
      
      <input
        type="text"
        value={logValue}
        onChange={(event) => setLogValue(event.target.value)}
      />
      
      <MyButton onClick={() => {
        console.log(logValue)
      }}>Log Value</MyButton>
    </>
  );
}

function fib(n) {
  if (n === 2) return 1;
  if (n === 1) return 0;
  return fib(n-2) + fib(n-1);
}
```

We can refresh the page. Whenever we change the number, we calculate the `fib(num)` again. Even if we go back down to a previous number. So if you wanted to save every value from `fib()` you would have to implement that yourself.


What's important when we type in `hello` for the log value, `fibValue` was not calculated again because nothing changed in the dependency array. However if we added `logValue` to the dependency array, we'd see with every letter typed we'd calculate `fib(num)` again.

**To sum up this idea**, whenever you have something very complicated that doesn't change on every render you can use the `useMemo` hook to potentially to save the component from needing to calculate that complicated value all over again when it hasn't changed.

### How do we limit the number of renders?

So let's come over to `MyButton.js` and I'll add a `console.log()` statement. Let's come over to the browser and refresh.

```js
export default function MyButton(props) {
  console.log('Rendering MyButton');

  const startTime = new Date();

  while (new Date() - startTime < 1000) {}

  return <button {...props} style={{color: 'red'}} />;
}
```

So let's refresh the page. So we can see whenever we change the input for `logValue`, the `Rendering MyButton` message is printed out to the console.

And now this might seem like a big of a deal, but what if `MyButton` was very slow to render?

So we can create a `while` loop using `new Date()`, and let's say make it take 1000 ms.  

```js
export default function MyButton(props) {
  console.log('Rendering MyButton');

  const startTime = new Date();

  while (new Date() - startTime < 1000) {}

  return <button {...props} style={{color: 'red'}} />;
}
```

Save, refresh the page. So you can actually see it is actually causing the page to take longer to load. And whenever this render needs to happen, it's going to take some time before the page will update. Of course you shouldn't have a component that takes a full second to render, but this is just to show the issue in a more extreme manner.

So how can we fix this?

We can actually memoize the entire component. And essentially what this is going to do, is if the props have not changed, we want to use the old version of the component instead of rendering it again. We need to import the `memo()` function from React. It's very similar to the `forwardRef()` function. 

And this concept of these functions that take in a component and returns a new component is known as a Higher-Order Component. It doesn't matter but normally with HOCs you would use an anonymous function.

Now what this saying, in the case where the props have not changed, use the previous version of this component. You can customize this behavior, and pass in a function. This function would take in the old props and the new props and returns if they are equal.

```js
import { memo } from 'react';

export default memo(function (props) {
  console.log('Rendering MyButton');

  const startTime = new Date();

  while (new Date() - startTime < 1000) {}

  return <button {...props} style={{color: 'red'}} />;
}, areEqual);

function areEqual(oldProps, newProps) {
  return true;
}
```

This would mean the component never needs to re-render, because you are saying previous props and new props are always the same. A note is that when you tell React it does not re-render, that does not mean React will not re-render. It is simply a performance increase that will are telling React it *can* make. But you should not rely on this for not re-rendering. But for now I will delete the `areEqual` function and the second parameter.

But this isn't going to work quite yet. If we come over to `App.js`. We are passing in a function as a prop, so in every render in our `App` component, we are passing in a new function even if it does the same thing.

Well, we can use `useMemo` again. Again, `useMemo` takes in a function and we will return this nested function, if this `logValue` changes. Now we can say `onClickLog` here for `MyButton`'s `onClick` value. So in the case that the `logValue` has not changed, `MyButton` is going to exact same function and because of `memo` it is not going to re-render.

```js
import { useMemo, useState } from 'react';
import MyButton from './MyButton';

export default function App() {
  const [num, setNum] = useState(10);
  const [logValue, setLogValue] = useState('');

  const fibValue = useMemo(() => {
    console.log('calculating fib value');
    return fib(num);
  }, [num]);

  const onClickLog = useMemo(() => {
    return () => () => console.log(logValue);
  }, [logValue]);

  return (
    <>
      <h1>Fib {num} is {fibValue}</h1>
      <input
        type="number"
        value={num}
        onChange={(event) => setNum(parseInt(event.target.value))}
      />

      <br />
      <br />
      
      <input
        type="text"
        value={logValue}
        onChange={(event) => setLogValue(event.target.value)}
      />
      
      <MyButton onClick={onClickLog}>Log Value</MyButton>
    </>
  );
}

function fib(n) {
  if (n === 2) return 1;
  if (n === 1) return 0;
  return fib(n-2) + fib(n-1);
}
```

We can refresh the page, take a second to render, on the initial render we are still waiting a second. It's still going to take a second when we type into the `logValue` input field, but we have essentially separated the `logValue` components from the `num` components.

One more thing I wanted to show is a shorthand notation for this `useMemo` and that is another function called `useCallback`

```js
const onClickLog = useCallback(
  () => () => console.log(logValue), [logValue]);
```

But now we still have this issue of how the page takes the button because of how slow `MyButton` take to load but we don't need it on the initial render.

So we can change the import statement for `MyButton`

```js
import { lazy, useCallback, useMemo, useState } from 'react';
const MyButton = lazy(() => import('./MyButton'));
```

Now this function will only run once we use `MyButton`.

And we can conditionally render this `MyButton` when we need it.

```js
{ 
  logValue.length > 0 ? <MyButton onClick={onClickLog}>Log Value</MyButton>
                      : null
}
```

So let's save this and we can refresh this page and we can see it loads it instantly but then we are going to have an issue. We are going to get an error message. 

`A React component suspended while rendering but no fallback UI was specified.`

Essentially what this means we got to this point where we lazily rendered this button, but this is going to take some time so we need to tell react what it needs to show on the UI while it waits for this button.

And the way we do it is with a Component known as `Suspense`. So I will wrap `MyButton` with `Suspense` tags. And essentially what this does is that while the button is suspending or loading, we will render a fallback. And the fallback, and the `fallback` will be a prop. And I will set it to a div that says "Loading".

```js
import { lazy, useCallback, useMemo, useState, Suspense } from 'react';

{ 
  logValue.length > 0 ? 
                      <Suspense fallback={<div>Loading...</div>}>
                        <MyButton onClick={onClickLog}>Log Value</MyButton>
                      </Suspense>
                      : null
}
```

Now save and refresh this page to clear console. We can see as we change the fib number everything changes quickly. When we type letter a into `logInput` field, we see the `Loading..` to finish that 1 second render. Because of this call to `memo()` it is not going to slow down while we are changing this fib number. Because now the module is loaded, it won't show that loading div anymore when I type in a second a into the `logInput` field.

And of course, if you do have a component that has some very slow logic that actually takes a full second to run, there's nothing we can do about that long render slowing down the input when it does actually need to re-render. But you can see how we used have `memo` and `useCallback`, which is syntactic sugar for `useMemo`, plus `lazy` and `Suspense` to really optimize this component to re-render only when it absolutely needs to and also how we have used the `useMemo` hook to make renders for this component really fast. 

These are powerful tools but you should only use them when you need to and can actually hurt performance if you overuse them. But if you do find you have performance issues, you should look to these tools.
