## React Basics

### React

- Declarative
  - Describe how the UI should look, not all the implementation details (https://stackoverflow.com/a/49162630)
- Component-based
  - Reusable pieces of the UI, like our own custom HTML tags
- Unidirectional data flow
  - React is going to be dynamic and change in response to some data
  - But the data is only going to flow in one direction, from parent to child and not vice versa

### JSX

- **J**ava**S**cript **X**ML
- Javascript syntax extension for inlining HTML or any XML elements in Javascript code
- When used with React, JSX will compile into some React code

```jsx
const element = (
  <p id="Hello">
    Hello <em>World!</em>
  </p>
);
```

`React.createElement(type, [props], ...children)`

```js
var element = React.createElement(
  'p',
  { id: 'Hello' },
  'Hello ',
  React.createElement('em', null, 'World!')
);
```

The first element is going to be the type which is `p` here. Then we have an optional parameter called props which is going to be an object, and these are any attributes added to the element. So here we have the key of `id` and the value being `Hello`. And now we have a rest parameter which allows `React.createElement()` to take multiple extra parameters, which are going to be the children. So you see the first child is `Hello `, with a space at the end. And the second element is the `em` tag which is going to be another call to `React.createElement()`. And likewise that child element has the type of `em`. And it has no props so that is going to be null, and it has a child of `World!`. And that's it. We won't normally use `React.createElement()` because we will always use JSX.

### ReactDOM

- Package for inserting element into the dom
  - `ReactDOM.render(element, DOMContainer)`

`ReactDOM.render()` takes in a react element, some element we created using JSX. And it takes in the DOMContainer. Append this element to the DOMContainer. It usually going to be the root `div` but it can also be some deeper nested DOM container if you only want part of the page to be using React.

### Functional Components

- Functions that return a React element (JSX)

```jsx
function SayHello() {
  return <p>Hello World</o>
}

function App() {
  return (
    <div>
      <SayHello />
      Welcome to React!
    </div>
  );
}
```

So you see in `App` we have `SayHello` here as a custom component that can be reused like any HTML tag.
