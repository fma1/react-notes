## Custom Hooks

So let's look at this example. We have this increment button as well as the count and the previous render count. So if I click Increment the count becomes 1. If I increment again, the count becomes 2 and the previous render count becomes 1.

Then we have a text input and a previous render text. If I type a letter a, the previous render count now becomes 2. This isn't previous count state value, it's the value from the previous render. And then if I type another `a` here, you will see the previous render text is now `a`. Now if I increment, we have count of 3, previous render count of 2, but previous render text is `aa` because on the previous render the text didn't change. If I add another `a` the previous render text is still `aa`.

So first of all, we have our state for the count and the text. And then we have two different `useEffects` and refs. We have a ref for the previous count. When it changes it does not a cause a re-render. Whenever the count changes, we save the previous count. We set `prevCount.current` equal to the current count. Same exact thing for text.

We have a paragraph to display the current and previous count. Same thing for the text.

This code all works and there's nothing wrong with it. There's some redundancy. Two different `useEffects` that do the same thing but with different refs. And then we have 2 different refs associated with 2 different effects. In traditional programming, if you saw this redundancy you want to create a helper function. And that is all a custom hook. A helper function with hooks.

```js
import { useEffect, useRef, useState } from 'react';

export default function App() {

  const [count, setCount] = useState(0);
  const [text, setText] = useState('');

  const prevCount = useRef();
  useEffect(() => {
    prevCount.current = count;
  }, [count]);

  const prevText = useRef();
  useEffect(() => {
    prevText.current = text;
  }, [text]);

  return (
    <>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <p>Count: {count}</p>
      <p>Previous render count: {prevCount.current}</p>
      
      <input
        value={text}
        onChange={(event) => setText(event.target.value)} />
      <p>Previous render text: {prevText.current}</p>
    </>
  );
}
```

So let's write a helper function to remove the redundancy. Whenever we have a helper function to use hooks, we should preface it with `use`. Since our function is going to call hook functions, we want React to apply the same linking rules and it does that with the prefix `use`.

And now with any helper function, take some values specific to an instance of this function and move them to parameters. We have `prevText` but no longer text specific so we can rename to `prevRef`. And then we have this `text`, so we can have a parameter that can just be `value`. And the last thing we do is we need to return this previous value. `return prevRef.current` since we don't need the entire object.

```js
function usePrevious(value) {
  const prevRef = useRef();
  useEffect(() => {
    prevRef.current = value;
  }, [value]);

  return prevRef.current;
}
```

And now we can replace our redundant code. And we can change the JSX is to not use `.current` we no longer need the `.current` since our custom hook returns the current value.

```js
import { useEffect, useRef, useState } from 'react';

export default function App() {

  const [count, setCount] = useState(0);
  const [text, setText] = useState('');

  const prevCount = usePrevious(count);
  const prevText = usePrevious(text);

  return (
    <>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <p>Count: {count}</p>
      <p>Previous render count: {prevCount}</p>
      
      <input
        value={text}
        onChange={(event) => setText(event.target.value)} />
      <p>Previous render text: {prevText}</p>
    </>
  );
}

function usePrevious(value) {
  const prevRef = useRef();
  useEffect(() => {
    prevRef.current = value;
  }, [value]);

  return prevRef.current;
}
```

And we can see this works the same way as before. So that's all there is to custom hook. The core of it is we are writing helper function for React hooks. And you will need to prefix the helper function with `use` and that helper function becomes a React hook.

Normally, we want to write hooks for two reasons

- when we have redundant code
- when we have long code and we want to make it more readable
