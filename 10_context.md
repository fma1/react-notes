## Context

In this video we'll be talking about Context. Context is another form of state within React. Here what we have is an `App`, which has some state for user. The current user is Steve. And we're going to toggle between the user being Steve and the user being Joe. And we have a `toggleUser()` function for that. We have a Profile that takes in the user as a prop. And a button that toggles the user.

```js
import { useState } from 'react';
import Profile from './Profile';

const steve = {
  name: 'Steve',
  course: 'Math'
};

const joe = {
  name: 'Joe',
  course: 'English'
};

export default function App() {
  const [user, setUser] = useState(steve);

  const toggleUser = () => {
    if (user === steve) {
      setUser(joe);
    } else {
      setUser(steve);
    }
  }

  return (
    <main>
      <Profile user={user} />
      <button onClick={toggleUser}>Toggle User</button>
    </main>
  );
}
```

And then we can see the profile as well. We can see it returns a `WelcomeBanner` and a `Course`, which both take in the user as a prop.

```js
import WelcomeBanner from './WelcomeBanner';
import Course from './Course';

export default function Profile({ user }) {
  return (
    <>
      <WelcomeBanner user={user} />
      <Course user={user} />
    </>
  );
}
```

And these two components are pretty simple.

```js
export default function WelcomeBanner({ user }) {
  return <h1>Hello {user.name}</h1>;
}
```

```js
export default function Course({ user }) {
  return <p>Your course is {user.course}</p>;
}
```

So we can run it and we can see a greeting for the user and the course for the user. And we can toggle the user and that changes the state in `App` and the props propagate and everything updates.

This is fine and works. This is called lifting state up. The `App` component is the lowest component that has both components as children. And we're using prop drilling, sending that state or prop deeply down a React tree. In this case it's not a big deal but it could get complicated very fast.

Context is just that. Create some state accessible by all children of a specific component. Let's see how we can create a Context.

We call `createContext()` and it takes in a parameter similar to `useState()`, the default value.

```
import { createContext } from 'react';

export const UserContext = createContext({
  user: null,
  course: null
});
```

So let's import that `UserContext` inside of `App.js`. And we'll use it as a JSX element. But specifically `UserContext.Provider`. The provider is the component that creates the state and passes down the state toe children. And the only thing that needs the `user` prop is `Profile` so we only need to put `Profile` there. And then all we need do is add a value prop to the `Context.Provider`, which will be equal to the `user`. And we can delete `user` as a prop to `Profile` but it won't work just yet.

```js
import { useState } from 'react';
import Profile from './Profile';
import { UserContext } from './UserContext';

const steve = {
  name: 'Steve',
  course: 'Math'
};

const joe = {
  name: 'Joe',
  course: 'English'
};

export default function App() {
  const [user, setUser] = useState(steve);

  const toggleUser = () => {
    if (user === steve) {
      setUser(joe);
    } else {
      setUser(steve);
    }
  }

  return (
    <main>
      <UserContext.Provider value={user}>
        <Profile />
      </UserContext.Provider>
      <button onClick={toggleUser}>Toggle User</button>
    </main>
  );
}
```

So we need update all the components to use the context. We will make a call to the `useContext()` hook function. And then we pass in the `UserContext` to `useContext()`.

So `useContext()` gets the user from the UserContext. It looks for the first instance of a Context Provider of this type in the tree above and gets the value from that state and we can use it just like if it were a prop.

```js
import WelcomeBanner from './WelcomeBanner';
import Course from './Course';
import UserContext from './UserContext';
import { useContext } from 'react';

export default function Profile() {
  const user = useContext(UserContext);
  return (
    <>
      <WelcomeBanner user={user} />
      <Course user={user} />
    </>
  );
}
```

We can run it and it works but we are still using props in `WelcomeBanner` and `Course` so we can change those. And we don't even need `user` in `Profile`.

```js
import WelcomeBanner from './WelcomeBanner';
import Course from './Course';

export default function Profile() {
  return (
    <>
      <WelcomeBanner />
      <Course />
    </>
  );
}
```

```js
import { UserContext } from './UserContext';
import { useContext } from 'react';

export default function WelcomeBanner() {
  const user = useContext(UserContext);
  return <h1>Hello {user.name}</h1>;
}
```

```js
import { UserContext } from './UserContext';
import { useContext } from 'react';

export default function Course() {
  const user = useContext(UserContext);
  return <p>Your course is {user.course}</p>;
}
```

So we can click on `Toggle User` and it toggles between the two users. So the important thing to note here is we have a `Profile` that doesn't even need to know the Context exists, so they don't need to be passing props down and we can make these components much simpler.

Now I want to move this logic toggling the user into the Context itself.

So I'm going to come over to `UserContext.js`, create a component that we can export. So rather than using `UserContext.Provider` we will use this `UserContextProvider` component. Now let's move all the React state logic here. And now we need to add `user` and `toggleUser` to the value of the context provider.

```
import { createContext, useState } from 'react';

const steve = {
  name: 'Steve',
  course: 'Math'
};

const joe = {
  name: 'Joe',
  course: 'English'
};

export const UserContext = createContext({
  toggleUser: null,
  user: {
    name: null,
    course: null
  }
});
 
export function UserContextProvider({children}) {
  const [user, setUser] = useState(steve);

  const toggleUser = () => {
    if (user === steve) {
      setUser(joe);
    } else {
      setUser(steve);
    }
  }

  return (
    <UserContext.Provider value={user, toggleUser}>
      {children}
    </UserContext.Provider>
  );
}
```

So now we can update `App.js`. We're going to add `AppInternal` because `UserContextProvider` needs to be a parent.

```js
import Profile from './Profile';
import { UserContext, UserContextProvider } from './UserContext';
import { useContext } from 'react';

export default function App() {
  return (
    <main>
      <UserContextProvider>
        <AppInternal />
      </UserContextProvider>
    </main>
  );
}

function AppInternal() {
  const { toggleUser } = useContext(UserContext);
  return (
    <>
      <Profile />
      <button onClick={toggleUser}>Toggle User</button>
    </>
  );
}
```

Now we just need to update `WelcomeBanner` and `Course` to destructure the result of `useContext()`.

Let's do a quick recap of everything. In `App.js` we return this main element and inside is the `UserContextProvider` as well `AppInternal`. And `UserContextProvider` provides its state to all of its children. And `AppInternal` is going to get the `toggleUser` and give it to the button, and then we get to `Profile`, which doesn't use the context. And then we get to `WelcomeBanner` and `Course` which only use the `user` from the context.

So in `UserContext`, we create the context and pass in the default value. The initial value doesn't matter but we add it in anyways. Finally we have the `UserContextProvider`, which is going to be a custom component that keeps track of some state and has the `toggleUser` value. And the return value is going to provide the `user` and `toggleUser` values to all of its children.

You might see different patterns in different places, like how we created a custom component. In some applications, they might break this into multiple aspects. One Context having the `toggleUser` and one context having a `user`. What's important is what a Context does is it creates state that is accessible by all children of the Context Provider. And it is going to work just like standard state. When state is changed, components will re-render, but without prop drilling. Often this creates much cleaner code. Much of the big use cases will be this like a signed-in use, or theming. Dark Mode or Light Mode. Change CSS styles based on that information.
