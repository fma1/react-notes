## Component Lists

So what we mean by Component Lists is how we take an array of data into a list of JSX components that can be added to the DOM?

So we have the state called `items` which is an empty array and we have this `newItem` state which will be an empty string. And then we have this unordered list. And then we have an input and a button. So the input is going to be a text input, controlled by the `newItem` state. And when the button is clicked on is going to spread the previous items, add `newItem` and reset the `newItem` string. 

```
import { useState } from 'react';

export default function App() {
  const [items, setItems] = useState([]);
  const [newItem, setNewItem] = useState('');

  return (
    <>
      <ul>
        { /* TODO */ }
      </ul>

      <input
        type="text"
        value={newItem}
        onChange={(event) => setNewItem(event.target.value)} />
      
      <button onClick={() => {
        setItems([...items, newItem]);
        setNewItem('');
      }}>
        Add List Item
      </button>
    </>
  );
}
```

Now what we want to do is create a list item for each item in the `items` array. One thing that's nice about React is inside of a return value, we can include an array. It can be an array of elements. For example inside of these curly braces, we can add an array. And inside of this array we can add different elements.

```js
<ul>
  { [ 
    <li>Hello</li>,
    <li>World</li>
  ] }
</ul>
```

You can see this is going to work but we get a warning message in the console. Equal child in the list should have a unique key prop. And the key prop is going to be a unique identifier. So we can add key props as so.

```js
<ul>
  { [ 
    <li key="hello">Hello</li>,
    <li key="world">World</li>
  ] }
</ul>
```

Okay. What's the point? Whenever we have an array, React is going to assume this array is some dynamic data. What happens if that dynamic data changes?

So let's add another list item. So in this case, React previously knew it had "hello" and "world", but now there's another list item in the middle, which makes sense because you saw me typing it out.

But React doesn't know if we added a list item in the middle, or maybe we took the last item, and changed its text to be "Test". In most cases, the output will be the same, we need a way to make sure React knows which item is which so it updates and mounts the correct component.

```js
<ul>
  { [ 
    <li key="hello">Hello</li>,
    <li>Test</li>,
    <li key="world">World</li>
  ] }
</ul>
```

Instead of using these hardcoded items, we need to use these items from this array. And we can convert these strings into list items.

And we can say `items.map()` and for each item we want to return a list item. And this return value is going to be some JSX. The text is going to be `item` and now we need the unique key prop, which is also going to be `item`. So it gives a list of items.

```js
<ul>
  { items.map(item => {
    return (
      <li key={item}>{item}</li>
    );
  }) }
</ul>
```

So we can simply add `hello` and `world` and this works as we expect. One cave-at is if you add a duplicate item, we are going to get a warning message that 2 children have the same key. They both end up with the key of world. So you have to make sure you have unique keys. Most time we are iterating through data through server and they have unique IDs. 

As a backup, one thing is you can use an index. And we can use that index of `i` as a key.

```js
<ul>
  { items.map((item, i) => {
    return (
      <li key={i}>{item}</li>
    );
  }) }
</ul>
```

Save this, and refresh this page, and add "hello" and "world" and "world". It will allow us to have duplicates because of the index being used as a key. But using an index as a key it should be used as a last resort, because it can run into issues React losing track of which element is which especially when items are added in the middle of last.

Last thing I want to talk about is Fragments. So let's save this.

```js
<ul>
  { items.map((item, i) => {
    return (
      <>
        <li key={i}>{item}</li>
        <li key="test">Test</li>
      </>
    );
  }) }
</ul>
```

Now we had the same issue as before. Even though both list items have a key, the Fragment itself does not have a key.

This is where you can't use the `<></>` shorthand for fragment because if you do `<key={i}>` it thinks key is the tag name. So we need to import `Fragment` and use `item` as the key. Now let's save it.

```js
import { useState, Fragment } from 'react';
<ul>
  { items.map((item, i) => {
    return (
      <Fragment key={item}>
        <li>{item}</li>
        <li>Test</li>
      </Fragment>
    );
  }) }
</ul>
```

Let's test it and it works. We add `hello` and `world`. `hello` and `test` are part of one Fragment, same with `world` and `test`. And we don't get the error about the key anymore. So only the Fragments are what need the key prop.

Make sure any element directly in that list has a `key` prop.
