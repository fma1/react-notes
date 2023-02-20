## Debugging React

A lot of React debugging is same thing as standard HTML and JS. But we are going to be looking at React specific debugging tools. We have 2 counters, one with an initial value prop of 5 and the other having no initial value prop, and this Counter is same as before.

```js
import { useState } from 'react';

export default function App() {
  return (
    <>
      <Counter initialValue={5} />
      <Counter />
    </>
  );
}

function Counter({initialValue = 0}) {
  const [ count, setCount ] = useState(initialValue);

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

The first thing I want to discuss is in `index.js`. So what is this `React.StrictMode` we have been using? Well, Strict Mode is meant to help you find issues in your code. Mostly it gives you warnings in the console if you use various deprecated functions. There are also lifecycle methods for classes we should no longer use.

```js
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

There's something else Strict Mode does called double invocation. Let me show you want I mean.

I'm going to create a counter variable, and this is not going to be state. Just a variable at global scope of this module. Let's call it `renderCount`. Every time this function is called, I'm going to increment `renderCount` and show its value on the screen.

```js
import { useState } from 'react';

let renderCount = 0;

export default function App() {
  renderCount++;
  return (
    <>
      <Counter initialValue={5} />
      <Counter />
      <p>Render Count: {renderCount}</p>
    </>
  );
}

function Counter({initialValue = 0}) {
  const [ count, setCount ] = useState(initialValue);

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

So we get a Render Count of 2. But why is this 2? The App component should have run one time. This is because of Strict Mode. This only happens in development and not in production. While you're developing your app, your functions for your component will be invoked twice. 

You might be thinking, what is the purpose? It's to avoid side effects. Side effects should not be in the main function component. By doing things twice, oh I did this thing in a place I shouldn't have done it. Essentially React is not ever guaranteeing us when we are going to run these functions. So we should not have code in these components that depend on being run at certain times.

So this should be in a `useEffect()`.

But anyways if we were to remove `<React.StrictMode>` from `index.js` we would get a Render Count of 1. So again, if we remove Strict Mode, we get rid of double invocation but we should always use it because it helps with debugging.

---  

Next I want to show something called the `Profiler`, and we import that from React.

This is a component. And we can simply wrap any component in a Profiler. So let's change the fragment to a Profiler. What the Profiler does is keep track of when a component is rendering. One note is the Profiler only works in development mode and in production mode the Profiler is ignored. The first property it'll take is an `id` and then an `onRender` property, which take in a function. But this is a misnomer. It doesn't run on render but commit, so it'll run after the actual render phase. So it'll run when React is done running the main function.

```js
import { useState, Profiler } from 'react';

let renderCount = 0;

export default function App() {
  renderCount++;
  console.log('rendering');

  return (
    <Profiler id="App" onRender={() => console.log('commit')}>
      <Counter initialValue={5} />
      <Counter />
      <p>Render Count: {renderCount}</p>
    </Profiler>
  );
}

function Counter({initialValue = 0}) {
  const [ count, setCount ] = useState(initialValue);

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

And now I can save this, refresh and you can see "rendering" and then "commit". Now admittedly I don't use the Profiler too much but if you are dealing with some hard to find bug, this can make sure your component is only rendering when you expect it to you.

---

So you can see in my DevTools I have a Components tab and a Profiler tab. The Components tab is similar to the Elements tab but instead of showing the HTML elements it's showing our React components.


So you can see we have our App, our Profiler and our 2 different Counters. And there are various things we can look at.

![01][17/01.png]
![02][17/02.png]
![03][17/03.png]
![04][17/04.png]
![05][17/05.png]
![06][17/06.png]
![07][17/07.png]
![08][17/08.png]
![09][17/09.png]
![10][17/10.png]
![11][17/11.png]
![12][17/12.png]
![13][17/13.png]
![14][17/14.png]
![15][17/15.png]
![16][17/16.png]
![17][17/17.png]
![18][17/18.png]


