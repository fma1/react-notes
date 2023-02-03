## State

```js
export default function App() {
  let count = 0;

  return (
    <>
      <button onClick={() => count++}>
        Increment
      </button>
      <p>Count: {count}</p>
    </>
  );
}
```

In this video, we are going to be talking about state. So this example is going to be some code that doesn't work, and show us why we need state.

Here we have a count variable set to 0. And we are returning a button that simply increments the count when you click it. And then we have this paragraph that simply displays the count. But if I click on Increment nothing is going to happen. The count doesn't actually increase. So why is this? So this variable actually increments. But nothing has told React it needs to update the DOM. We need a way to tell it something has changed and we want it to re-render.

So state is going to be local to be a component, and it's going to have a setter function. And we ever call that setter function, React will re-render the component to based on that new state.

```js
import { useState } from 'react';

export default function App() {
  const [ count, setCount ] = useState(0);

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

We want call the `useState()` function and we are going to import this from react, and it's not a default export so we use curly braces. This function will take on a default value, so let's say `0`. So this function is going to return an array and we destructure that into the value and the setter function. So let's destructure that into `count` and `setCount`. So we never want to directly mutate the `count` value because React relies on you calling the setter function to know when it needs to re-render.

So now we can simply say `setCount()` and set it to `count + 1` for `onClick`. So now if we click on the `Increment` button, it works as we expect.

So this `useState()` function is what is known as a hook. Any function with `use` at the beginning is going to be a hook. These are supposed to be special functions, that *hook* into special features of React. More generally, we are interacting with what is known as the component lifecycle. The return value of `useState` has the abliity to cause the component to re-render.

One rule is that hooks can not be called conditionally. For example, this code would be a problem because `useState()` is called conditionally. You will get an error that `React Hook useState was called conditionally.` The reason this matters is that React relies on the order of these hook functions. If we use them conditionally because React knows that on every render these hook functions are going to be called in the same order.

```js
import { useState } from 'react';

export default function App() {
  if (true) {
    return;
  }

  const [ count, setCount ] = useState(0);

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

Just for reference, we could pass in a function to `useState()` versus a value like `0`. The point of this is if you have some complex computation here. And the point of being able to pass in a function rather than some value like `expensiveComputation()` is that Javascript is pass by value so it will call `expensiveComputation()` every single time but if you pass in a function, `useState()` will only invoke it when necessary for the initial render.

```js
const [ count, setCount ] = useState(() => {
  return 0;
});
```

One more key point with useState is that we can have multiple pieces of state within one component. And we can see these are incrementing separately. 

```js
import { useState } from 'react';

export default function App() {
  const [ count, setCount ] = useState(0);
  const [ otherCount, setOtherCount ] = useState(5);

  return (
    <>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
      <p>Count: {count}</p>
      <button onClick={() => setOtherCount(otherCount + 1)}>
        Increment
      </button>
      <p>Other Count: {otherCount}</p>
    </>
  );
}
```

I lastly want to show that state is local to a certain component. We can also have a starting prop `startingCount`.

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
  const [ otherCount, setOtherCount ] = useState(startingCount + 5);

  return (
    <>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
      <p>Count: {count}</p>
      <button onClick={() => setOtherCount(otherCount + 1)}>
        Increment
      </button>
      <p>Other Count: {otherCount}</p>
    </>
  );
}
```

Now let's try updating `count` 3 times. You might expect it to increment the count by 3. But it only increments by one. The `Counter` won't re-render until the function is done running and the call stack is empty, the counter is not going to re-render. So all these 3 calls to `setCount` have the same `count` value.

```js
<button onClick={
  () => {
    setCount(count + 1)
    setCount(count + 1)
    setCount(count + 1)
  }>
  Increment
</button>
```

Now if we wanted our `setCount`s to use the value that was added before, and that's by passing in a function to `setCount`. If we try this, you can see we do is increment by 3.

```js
<button onClick={
  () => {
    setCount(prevCount => prevCount + 1)
    setCount(prevCount => prevCount + 1)
    setCount(prevCount => prevCount + 1)
  }>
  Increment
</button>
```

One other thing I wanted to point is we can have `useState` take in an object. But the important thing is React will not re-render unless we pass in a new object. So we would have to use the spread syntax here.

```js
function Counter({startingCount = 0}) {
  const [ count, setCount ] = useState({num: startingCount});

  return (
    <>
      <button onClick={() => setCount({...count, otherCount: 0})}>
        Increment
      </button>
      <p>Count: {count}</p>
    </>
  );
}
```

Another thing is I want to point out that you can lift state up. So what if we wanted both of these Counters to use the same state? Well we know data only moves from the parent to the children. So state needs to be high up in the parent tree to be sent to all children. So now you'll see that the child components are sharing state.

```js
import { useState } from 'react';

export default function App() {
  const [ count, setCount ] = useState(0);

  return (
    <>
      <Counter count={count} setCount={setCount} />
      <Counter count={count} setCount={setCount} />
    </>
  );
}

function Counter({count, setCount}) {
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

Let's come back over here once again. What I want to show now is what we call a Controlled Component. This `Counter` function is taking a setter function as a prop, and using whatever the parent says as a setter.But the most common use of Controlled Components is input. Now for this input tag, `onChange` is going to call `setValue()`. So now the input element is being controlled by this state here. When we type into this input. Whenever we type into input, we set the state here, which is causing the component to re-render. And thus we get the input with the new value.


```js
import { useState } from 'react';

export default function App() {
  const [ count, setCount ] = useState(0);
  const [ value, setValue ] = useState('');

  return (
    <>
      <input
        type="text"
        onChange={(event) => setValue(event.target.value} />
      <Counter count={count} setCount={setCount} />
      <Counter count={count} setCount={setCount} />
    </>
  );
}

function Counter({count, setCount}) {
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

Now what I want to show is an alternative to `useState` called `useReducer`. So it returns an array with `state` and dispatch. It takes in a reducer function and the initial state. So this reducer function takes in the previous state, and an action. These actions normally have a `type` property, so we `switch` on it, and return new state based on the action.

So now we change to have a `onClick` property that returns a function which calls the dispatch function. And the dispatch function takes in an action, which has a `type` property` and a `num` property. And now we see they both increment by `1`. By changing the `num` property in the second `Counter` to 10, we can now increment by 10.

```js
import { useReducer } from 'react';

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + action.num};
    case 'decrement':
      return {count: state.count - action.num};
    default:
      throw new Error('Unknown action type');
  }
}

export default function App() {
  const [state, dispatch] = useReducer(reducer, {
    count: 0
  });

  return (
    <>
      <Counter 
        count={state.count} 
        onClick={() => 
          dispatch({
            type: 'increment',
            num: 1
        })} />
      <Counter 
        count={state.count} 
        onClick={() => 
          dispatch({
            type: 'increment',
            num: 10
        })} />
    </>
  );
}

function Counter({count, onClick}) {
  return (
    <>
      <button onClick={onClick}>
        Increment
      </button>
      <p>Count: {count}</p>
    </>
  );
}
```

