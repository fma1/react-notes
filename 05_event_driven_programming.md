## Event Driven Programming

```js
export default function App() {
  return (
    <button>Click Me!</button>
  );
}
```

So here we have a component that returns a button with "Click Me!" We haven't added any event listeners so clicking it would do nothing. We would use the `addEventListener()` in vanilla Javascript but that won't work here. We don't have an easy reference to the DOM node of the button, and even we could get it, that wouldn't be the React way to do things. The biggest reason is that when we use JSX, DOM nodes are being created by React. When a component needs to re-render, React might decide to remove and re-add that DOM node so our event listeners would disappear. So we should avoid doing any direct DOM manipulation on DOM nodes that React creates.

The way we're going to do this is using attributes or props.

```js
export default function App() {
  return (
    <button onClick={() => console.log('Clicked')}>Click Me!</button>
  );
}
```

We can also create a named function for this.

```js
export default function App() {
  const handleClick = () => console.log('Clicked')

  return (
    <button onClick={handleClick}>Click Me!</button>
  );
}
```

And it works the same. Note that we are re-creating the handleClick function every single time.

```js
const handleClick = () => console.log('Clicked')

export default function App() {
  return (
    <button onClick={handleClick}>Click Me!</button>
  );
}
```

And now when we use event handlers in React they work just like they do in vanilla JS. And we can get an event as the parameter and pass that in to `console.log()` and we'll get something similar to vanilla JS, except it's a `SyntheticBaseEvent`. Works same as standard event most of the time. But this adds consistency between browsers versus the native events can be different in browsers. But if you do need the native event, you can do `event.nativeEvent`.

What would happen if we tried to add a click event or any event handler to a custom component? Remember, the browser controls event listeners but the browser doesn't know about custom components. So we can't add event listeners to our custom components since they are compiled to native HTML tags.

```js
const handleClick = () => console.log('Clicked');

export default function App() {
  return (
    <button onClick={handleClick}>Click Me!</button>
  );
}

export function MyButton(props) {
  return (
    <button style='color:red;'>
      {props.children}
    </button>
  );
}
```

But the problem here is we're not doing anything with `onClick` in MyButton so clicking the button doesn't do anything.

```js
const handleClick = () => console.log('Clicked');

export default function App() {
  return (
    <button onClick={handleClick}>Click Me!</button>
  );
}

export function MyButton({...props}) {
  return (
    <button style='color:red;' {...props} />
  );
}
```

I recommend using this spread method because if you just do `onClick={onClick}` it won't work for any other event handlers.
