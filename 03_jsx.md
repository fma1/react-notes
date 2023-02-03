```js
import './App.css';
import React from 'react';

export default function App() {
  return (
    <h1>Hello World</h1>
    <p>Test</p>
  );
}
```

What's the error here? We need to always be returning a single JSX element.

```js
import './App.css';
import React from 'react';

export default function App() {
  return (
    <div>
      <h1>Hello World</h1>
      <p>Test</p>
    </div>
  );
}
```


But we are just adding extra elements to the DOM. It will slow down the page and make the job of accessibility tools such as screen readers harder.

React has something to solve this called a Fragment. So we can change the `div` to be a `React.Fragment` element and we can change the closing tag as well. We can save it and it will seem nothing has changed. But now we no longer have that extra `div`. Basically `React.Fragment` will just be removed whenever the element is appended to the DOM.

```js
import './App.css';
import React from 'react';

export default function App() {
  return (
    <React.Fragment>
      <h1>Hello World</h1>
      <p>Test</p>
    </React.Fragment>
  );
}
```

There is a shorthand for this. We can use empty tags and there is going to be no difference the vast majority of the time.

There are two differences:  

1. We can delete the React import since we don't need it.
2. There is a prop called a key prop but there's no way a prop to these empty elements but we can add that prop with the `React.Fragment` syntax.

```js
import './App.css';

export default function App() {
  return (
    <>
      <h1>Hello World</h1>
      <p>Test</p>
    </>
  );
}
```

So for now you can think of these two syntaxes as the same.

So now let's talk about how we can use JS expressions inside of our JSX.  

```js
import './App.css';

export default function App() {
  const name = 'Conner';
  return (
    <>
      <h1>Hello {name.toUpperCase()}</h1>
      <p>Test</p>
    </>
  );
}
```

So whenever we use this curly brace syntax, anything inside of that braces is just Javascript.

Let's try a different example that shows Error on Success based on whether there was an error.

```js
export default function App() {
  const error = true;
  // const error = false;
  if (error) {
    return <h1>Error</h1>;
  }
  return <h1>Success<h1>;
}
```

We could also do it with a ternary.

```js
export default function App() {
  const error = true;
  // const error = false;
  return error ? <h1>Error</h1> : <h1>Success</h1>;
}
```

We can also do it another way with ternaries;

```js
export default function App() {
  const error = true;
  // const error = false;
  return (
    <>
      {error ? <h1>Error</h1> : null};
      {!error ? <h1>Success</h1> : null};
    </>
  );
}
```

We could also change `null` to `undefined` and get the same result.

We could do it another way with short circuiting.

```js
export default function App() {
  const error = true;
  // const error = false;
  return (
    <>
      {error && <h1>Error</h1>};
      {!error || <h1>Success</h1>};
    </>
  );
}
```

```js
export default function App() {
  const error = true;
  // const error = false;
  return <h1>{error ? 'Error' : 'Success'}</h1>;
}
```

We can also use Javascript for props.

```js
export default function App() {
  const props = {
    id: 'input',
    type: 'text',
    maxLength: 3
  };
  return (
    <>
      <label htmlFor="input">Input:</label>
      <input {...props} />
    <>
  );
}
```

```js
export default function App() {
  return (
    <>
      <p style={{
        color: 'red',
        text-align: 'center',
        font-size: 48px
      }}>Hello World</p>
    </>
  );
}
```
