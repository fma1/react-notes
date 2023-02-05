## Refs

We have some state initialized to 0, called `seconds`. And then we have this `startTimer` function which calls `setInterval` to update the `seconds` by 1 every second. It is note that we are using the function syntax with `setSeconds` to make sure we don't have stale state.

Our goal is to implement the `stopTimer` function. And we have 2 buttons which will start and stop the timer.

```js
import { useState } from 'react';

export default function App() {
  const [seconds, setSeconds] = useState(0);

  const startTimer = () => {
    setInterval(() => {
      setSeconds(currSeconds => currSeconds + 1);
    }, 1000);
  };

  const stopTimer = () => {

  };

  return (
    <>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
      <p>seconds: {seconds}</p>
    </>
  );
}
```

In `stopTimer`, we need to call `clearInterval()` and we need the timer id from `setInterval`. And then we need some place to storing this id, and in scope for `startTimer` and `stopTimer`. We could store it at the top of the component as `let timerID` and `timerID = setInterval(...)` and `clearInterval(timerID)`. But then we click on stop, it is not going to stop.

So `timerID` is in scope from both `stopTimer` and `startTimer`, which is why we got no error message. But we make `timerID` undefined here at the top of `App()`. And we start timer, we set `timerID`. But then we call `setSeconds`, and React will re-render, so `timerID` is going to be set again to be `undefined`. So `timerID` is just going to be undefined in all future renders. So we want some value against persistent re-renders we need to use state.

```js
import { useState } from 'react';

export default function App() {
  let timerID;
  const [seconds, setSeconds] = useState(0);

  const startTimer = () => {
    timerID = setInterval(() => {
      setSeconds(currSeconds => currSeconds + 1);
    }, 1000);
  };

  const stopTimer = () => {
    clearInterval(timerID);
  };

  return (
    <>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
      <p>seconds: {seconds}</p>
    </>
  );
}
```

So let's change this to use `useState()`. 

```js
import { useState } from 'react';

export default function App() {
  const [seconds, setSeconds] = useState(0);
  const [timerID, setTimerID] = useState(null);

  const startTimer = () => {
    setTimerID(setInterval(() => {
      setSeconds(currSeconds => currSeconds + 1);
    }, 1000));
  };

  const stopTimer = () => {
    clearInterval(timerID);
  };

  return (
    <>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
      <p>seconds: {seconds}</p>
    </>
  );
}
```

And we can see it is working, but it is a little suboptimal. The reason for that is we call `setTimerID` when we start to interval. It is going to cause a re-render, but we don't need to re-render at this time but we are adding a re-render. But there are times when this can be more dramatic. It could be causing an infinite loop of re-renders in `useEffect`.

So how can we solve this? Well, we can make the initial state given to `useState` an object, and as long as we keep using the same object, then React is not going to re-render. So we can have a `current` property on the object and change everything to use that property.

```js
import { useState } from 'react';

export default function App() {
  const [seconds, setSeconds] = useState(0);
  const [timerID, setTimerID] = useState({
    current: null;
  });

  const startTimer = () => {
    timerID.current = setInterval(() => {
      setSeconds(currSeconds => currSeconds + 1);
    }, 1000);
  };

  const stopTimer = () => {
    clearInterval(timerID.current);
  };

  return (
    <>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
      <p>seconds: {seconds}</p>
    </>
  );
}
```

And we can see it is still working the same way but the big difference is we didn't call `setTimerID` so we save a re-render. But admittedly it is not the cleanest code. We have a `setTimerID()` we're not using and we have this object we're not updating so it feels like we're hacking `useState()`.

Because of that, React has this functionality built-in, and it's called a Ref. We can simply say `const timerID = useRef(...)`. And now this is going to work just like `useState()` with that object but it just returns that object that's current to whatever we set it to, instead of returning an array with the initial value and the setter function.

```js
import { useRef, useState } from 'react';

export default function App() {
  const [seconds, setSeconds] = useState(0);
  const timerID = useRef(null);

  const startTimer = () => {
    timerID.current = setInterval(() => {
      setSeconds(currSeconds => currSeconds + 1);
    }, 1000);
  };

  const stopTimer = () => {
    clearInterval(timerID.current);
  };

  return (
    <>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
      <p>seconds: {seconds}</p>
    </>
  );
}
```

While Refs are this specific functionality within React, you can think of them as syntactic sugar to passing in an object to `useState()` and never calling the setter function. The primary purpose is the value will persist across renders, but changing the value will never cause a re-render.

---  

What I want to show is the most common use cases for using Refs. I want to change this to an input and a button. We need an `onClick` handler so let's set that.

```js
import { useRef, useState } from 'react';

export default function App() {
  const focusInput = () => {
    input.focus();
  }

  return (
    <>
      <input />
      <button onClick={focusInput}>Focus</button>
    </>
  );
}
```

And to focus an input in Javascript is `input.focus()`. But we need to get the input, but because this is not React functionality but DOM functionality we need the actual input from the DOM. Now we could use `DOM.getElementById()` or something similar, but that's not the *React* way to do things. With React, I would get these values out of JSX.

And we can use a Ref for that. Inside of the input, we are going to add an attribute `ref` and set it to `inputRef`. So what this does is Reacts sets the `inputRef` current value to be the DOM node associated with the JSX element once created and added to the DOM.

```js
import { useRef, useState } from 'react';

export default function App() {
  const inputRef = useRef(null);
  console.log(inputRef);

  const focusInput = () => {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={focusInput}>Focus</button>
    </>
  );
}
```

So if we click on `Focus`, it highlights the input, which is what focusing the input looks like in the browser. 

One key point is at the top of this function, before DOM node is created, inputRef is going to be null. On that initial render, you cannot use the `inputRef`. So you would have to do it in a useEffect or something of that nature.

Ref attributes can be only added to nodes, but not custom components by default. Let me show you what I mean.

```js
import { useRef, useState } from 'react';

export default function App() {
  const inputRef = useRef(null);
  console.log(inputRef);

  const focusInput = () => {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={focusInput}>Focus</button>
    </>
  );
}

function MyInput(props) {
  return <input {...props} style={{color: 'red'}} />
}
```

So now you see we get an error that function components cannot be given Refs. If I try to click on Focus, it is not going to work. The error message is we tried to called focus, but `inputRef.current` is `null`. We solve this is with a forwardRef. We wrap the entire function call to `forwardRef`. And we close out parentheses at the end of the function. We need to save the return value of `forwardRef()`. It returns the same component but with ability to pass in ref, and `ref` will be a second parameter. We need to use the ref. We need to tell the return value which DOM element is going to use the ref. And we do that by passing the `ref` to that ref attribute.



```js
import { forwardRef, useRef, useState } from 'react';

const MyInput = forwardRef(function (props, ref) {
  return <input {...props} style={{color: 'red'}} ref={ref} />
});

export default function App() {
  const inputRef = useRef(null);
  console.log(inputRef);

  const focusInput = () => {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={focusInput}>Focus</button>
    </>
  );
}
```

And now clicking on Focus is going to focus this input but we can see by typing this is our custom component with color of red. Now, one note is this idea of having an input that is not being controlled by any state and being controlled using refs, is known as an uncontrolled component. Most of the time we don't want to do that. We want controlled components with state. Also sometimes we might want the ref on a controlled component.

The final thing I want to look at is a Callback Ref. Up to this point, we've used `useRef` to create a Ref. I'm going to comment out the focus button. And instead of our input ref. I will say `handleRef` and define this as a function`. And what this function is going to do is take in the DOM node. Now it's going to be called with the DOM node on mount. 

```js
import { forwardRef, useRef, useState } from 'react';

const MyInput = forwardRef(function (props, ref) {
  return <input {...props} style={{color: 'red'}} ref={ref} />
});

export default function App() {
  const inputRef = useRef(null);
  console.log(inputRef);

  const focusInput = () => {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={handleRef} />
      {/*<button onClick={focusInput}>Focus</button>*/}
    </>
  );
}

function handleRef(domNode) {
  console.log(domNode);
}
```

And we show it does not log out again if we type in the input so it does not get called on every render. It will also run on unmount, but will take in `null` as a parameter. Not common but can be useful from time to time. 
