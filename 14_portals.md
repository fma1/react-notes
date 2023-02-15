## Portals

So let's start by looking at this example. We have some state `isHidden` which is a boolean. Clicking the button toggles the `isHidden` state. If `isHidden` is false, we are going to render a Modal. We have a paragraph with the className of other. So if we come over the browser and click Show Modal you can see it shows the modal. The modal is being created but it is below the other content. 

```js
import { useState } from 'react';
import './App.css';

export default function App() {
  const [isHidden, setIsHidden] = useState(true);

  return (
    <>
      <div className="container">
        <button onClick={() => setIsHidden(!isHidden)}>
          {isHidden ? 'Show Modal' : 'Hide Modal'}
        </button>
        {isHidden || < Modal />}
      </div>

      <p className="other">
        Other Content
      </p>
    </>
  );
}

function Modal() {
  return <p className="modal">Modal</p>;
}
```

So something is probably wrong with our CSS. So if we look here. We have a ruleset for the container. The other paragraph as well as the modal. Both the container and the other paragraph have position relative and both have a z-index. So the container has a z-index of 1 and other has z-index of 1. Modal will be position fixed and z-index of 2. So you're saying modal has a higher z-index so it would make sense that the Modal is on top. So why is this not working?

The issue has to do with Stacking Contexts. Not going to go into huge detail here though as we have a whole other video on it. Idea is pretty simple. We have container div and other graph The other pagraph is above the  container div because it has a higher z-index. Because the Modal is inside the container div. Even though it has its own stacking context, it is not being compared. Anything inside of the container is going to be below other content.

```js
.container {
  position: relative;
  z-index: 0;
  background-color: lightgreen;
  height: 100px;
  padding: 10px;
}

.other {
  position: relative;
  z-index: 1;
  background-color: orange;
  height: 100px;
  padding: 10px;
}


.modal {
  position: fixed;
  z-index: 2;
  background-color: lightblue;
  width: 90vw;
  height: 75vh;
  padding: 10px;
  top: 5vw;
  left: 50%;
  transform: translateX(-50%);
}
```

Okay. How can we solve this? We could move the Modal outside of the container. Let's pretend we can't do that, needs access to Context Provider. Okay, we can change the CSS then. For the sake of demonstration, let's pretend we need to keep those CSS declarations. 

The solution is a portal. Take a React element leave where it is but when it is rendered on the DOM change where it actually renders. So let's come over to our `index.html` file. Let's add another div. Let's put the Modal inside of this `modal-root` div.

```html
<body>
    <div id="root"></div>
    <div id="modal-root"></div>
</body>
```

Now we can come back to `App.js` and we need to create a portal to move it into this new div. We need to import a new function `createPortal()` from `react-dom`. As a reminder, `react-dom` is the package we get the render() function from. This package is the main package as the bridge between the React tree and the DOM tree.

Okay, so `createPortal()` works similar to `render()` but meant to be used inline. So for example in here our Modal() instead of just returning our paragraph, we can return a call to `createPortal()`. And the first parameter would be a React element, this paragraph, and the second parameter would be the DOM node to append the element.

So whenever we use the Modal component, instead of the paragraph directly in the DOM where it is in the React tree, instead append it to this location in the root.

```js
import { useState } from 'react';
import { createPortal } from 'react-dom';
import './App.css';

export default function App() {
  const [isHidden, setIsHidden] = useState(true);

  return (
    <>
      <div className="container">
        <button onClick={() => setIsHidden(!isHidden)}>
          {isHidden ? 'Show Modal' : 'Hide Modal'}
        </button>
        {isHidden || < Modal />}
      </div>

      <p className="other">
        Other Content
      </p>
    </>
  );
}

function Modal() {
  return createPortal(
    <p className="modal">Modal</p>,
    document.getElementById('modal-root')
  );
}
```

So save and run this and click on Show Modal. And we can see this is working.

We can view this in this inspector and we have the `root` and the `modal-root`. If we open the modal-root, here is our Modal. If we open the standard root, we do not have the modal. Even though it is inside the container in the React tree, it is under `modal-root` in the DOM tree.

As one final point, when using portals, that element is still in its standard  place in the React tree. It still has access to props and context providers. And events will still bubble based on the events in the React. Let's set a click handler for `container.

```js
<div className="container" onClick={() => {
  console.log('Clicked on container');
}}>
  ...
</div>
```

And save again and run. Now if we click on the Modal even when we are over the other paragaph we can see it still registers the click handler because the Modal is under the container in the React tree.

Using a portal does not change anything except for the actual location of the element when it gets rendered to the DOM.

Common scenarios are modals and tooltips but there are other use cases.
