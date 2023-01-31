```jsx
export default function App() {
  return (
    <>
      <h1>Hello Connor</h1>
      <h1>Hello Clement</h1>
    </>
  );
}
```

What is the motivation behind Props? We have a h1 tag with Hello Connor and a h1 tag with Hello Clement. Clearly there is some redundancy here. We can pass in a parameter as what is known as props. Props is an object.

```jsx
export default function App() {
  return (
    <>
      <Hello name="Connor" />
      <Hello name="Clement" />
    </>
  );
}

function Hello(props) {
  return <h1>Hello {props.name}</h1>;
}
```

You can still we get the same output if we run this.

What some people find annoying is because we pass a single props variable, no way to look at this function and see what props the function expects. We could use Typescript or we could use a comment, but one of the built-in ways is that instead of naming the parameter props, we destructure it.

```jsx
export default function App() {
  return (
    <>
      <Hello name="Connor" />
      <Hello name="Clement" />
    </>
  );
}

function Hello({name = "User"}) {
  return <h1>Hello {name}</h1>;
}
```

So now name is going to be User. There is also a defaultProps property doing the same thing as well.

```jsx
export default function App() {
  return (
    <>
      <Comment username="Conner" time={(new Date()).toString()}>
        <h1>Hello World</h1>
        <p>This is a comment</p>
      </Comment>
    </>
  );
}

function Comment({username, time, children}) {
  return (
    <section>
      <p>{username} commented at {time}</p>
      {children}
    </section>
  );
}
```

So the child tags are going to be automatically added to the `children` key. Now there is no `Comment` tag in the actual HTML. If we look at the source and inspect, we won't see it. We would see the actual `<section>` and vanilla HTML. All the browser knows is HTML. All of our custom components can be thought of as functions that return HTML elements.
