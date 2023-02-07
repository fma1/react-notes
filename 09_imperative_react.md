## Imperative React

So we now that we React is inherently going to be a declarative library but there are times we need to use our components in an imperative way and React gives us an escape hatch to do that. The goal here is to add a click handler to this reset button to reset the count and the text in the custom input.

```js
import Counter from './Counter';
import CustomInput from './CustomInput';
import './App.css';

export default function App() {
  return (
    <>
      <Counter />
      <CustomInput placeholder="Type something..." />  
      <button>
        Reset
      </button>
    </>
  );
}
```

And now we can quickly look at the implementation of the components.

```js
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

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

```js
import { useState } from 'react';

export default function CustomInput(props) {
  const [value, setValue] = useState('');

  return (
    <input
      {...props}
      value={value}
      onChange={event => setValue(event.target.value)}
      style={{color: 'red'}} />
  );
};
```

So let's come over to `App` and think about what we can do to reset the state of both of these components. We could lift the state up from `Counter` and `CustomInput` into `App` and pass it back into those components into props. Not necessarily wrong but separates state from component it is related to. Doesn't make sense to have `Counter` state in higher `App` component. In a more complicated example it could balloon out of control.

So what we want is an imperative way to control these components. We want to be able a click event handler to the button. What do we want to do inside this function? `Counter.reset()` and `CustomInput.reset()`. But we can't do that currently. If we want a reference to the `Counter` and the `CustomInput`, where should we start? Well we probably want a Ref so we'll import `useRef`. And add `counterRef` and `customInputRef` and pass them to the components like so. But remember, Refs are going to set equal to a DOM node and DOM nodes know nothing about React state. React state is handled by React.

```js
import { useRef } from 'react';
import Counter from './Counter';
import CustomInput from './CustomInput';
import './App.css';

export default function App() {
  const counterRef = useRef();
  const customInputRef = useRef();

  return (
    <>
      <Counter ref={counterRef}/>
      <CustomInput ref={customInputRef} placeholder="Type something..." />  
      <button>
        Reset
      </button>
    </>
  );
}
```


Now we need to go into `Counter` component and `CustomInput` component and add a forward ref.

And what we have seen in the past, taken the ref parameter and to be the attribute to one of these elements but it won't achieve what we want. We don't want it to be an equal to a DOM node. We want to send as the ref current value is a function to reset the state. And this can be achieved with a hook is called `useImperativeHandle`. It takes in the ref, and then it takes in a function. This function will return an object, the current value of the ref. One other point is here we can add a dependency array. If we only wanted to calculated value on mount, we could use an empty array. The array works just like in `useEffect`. But usually we don't need the dependency array.

We wrapped the `Counter` component in `forwardRef()`. We used `useImperativeHandle` to control what the `ref` values are, and created this object that has a reset function to reset the count.

```js
import { forwardRef, useImperativeHandle, useState } from 'react';

export default forwardRef(function (props, ref) {
  const [count, setCount] = useState(0);

  useImperativeHandle(ref, () => ({
    reset: () => setCount(0)
  }));

  return (
    <>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
      <p>Count: {count}</p>
    </>
  );
});
```

Now we need to do the same thing for `CustomInput`.

```js
import { forwardRef, useImperativeHandle, useState } from 'react';

export default forwardRef(function (props, ref) {
  const [value, setValue] = useState('');

  useImperativeHandle(ref, () => ({
    reset: () => setValue('')
  }));

  return (
    <input
      {...props}
      value={value}
      onChange={event => setValue(event.target.value)}
      style={{color: 'red'}} />
  );
});
```

So now we come over to `App`. We want the `counterRef` and `customInputRef` and we want to call `reset()`.

```js
import { useRef } from 'react';
import Counter from './Counter';
import CustomInput from './CustomInput';
import './App.css';

export default function App() {
  const counterRef = useRef();
  const customInputRef = useRef();

  return (
    <>
      <Counter ref={counterRef}/>
      <CustomInput ref={customInputRef} placeholder="Type something..." />  
      <br />
      <br />
      <button onClick={() => {
        counterRef.current.reset();
        customInputRef.current.reset();
      }}>
        Reset
      </button>
    </>
  );
}
```

Now we can go to browser. We can click Increment and type something and we can click Reset and see they reset. So we can see it's working. Really at the core useImperativeHandel is really a hook allows the components we have refs for, in an imperative way. This is an escape hatch though. You don't want to do this all over the place because you'll get away from the declarative nature of React.
