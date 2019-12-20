# Lazy-Props

Help yourself when working with async props.
```
npm i lazy-props
```

## Motivation

When working with `React`, or any view library, some of your components will need to request or build props asynchronously. Most of the time this is done in a `useEffect` or `componentWillMount` function.

```js
import React from 'react'

//Component needing users
const MyComponent = ({users}) => (
  <div>
    { users.map(user => 
        <div>
          {user.name}
        </div>
      )
    }
  </div>
)

export default MyComponent

//in another file
import React, {useState, useEffect, lazy, Suspense} from 'react'

import { getUsers } from '../users'

const LazyMyComponent = import('./MyComponent')
const MyComponent = lazy(() => LazyMyComponent)

const App = () => {
  //Initializing users to undefined, sometimes []
  const [users, setUsers] = useState()

  //actualy requesting users, updating the view when done
  useEffect(() => {
    getUsers().then(usrs => setUsers(usrs))
  }, [])

  return (
    <Suspense fallback={<div>Loading</div>}>
      <MyComponent users={users} />
    </Suspense>
  )
}

export default App
```

As pointed above, `users` is initilized to undefined. In this setup, you will render `MyComponent` with no users. Then, when the server answer actually come back, you will update the ui with actual users.

This is bad because your user is seeing an empty screen withtout any notification. As anyone, I would think "Is this normal?" or "Probably a bug right?".

To avoid this, some people actually wait for `users` to not be undefined.

```js
...
return (
    <Suspense fallback={<div>Loading</div>}>
      { users === undefined 
        ? <div>Loading</div>
        : <MyComponent users={users} />
      }
    </Suspense>
  )
...
```
That's better in terms of user experience, but now you have to loading one after the other. It may seems fine in this simple case, but real applications get real messy. We would like to avoid duplication.

The idea behind `lazy-props` is __"Why not just the `lazy` and `Suspense` provided to us ?"__

```js
import React, {useState, useEffect, lazy, Suspense} from 'react'
import { lazy-props } from 'lazy-props'

import { getUsers } from '../users'

const LazyMyComponent = import('./MyComponent')

const App = () => {
  
const MyComponent = lazy(() => lazyProps(LazyMyComponent, getUsers(), 'users'))

  return (
    <Suspense fallback={<div>Loading</div>}>
      <MyComponent />
    </Suspense>
  )
```

As you can see, using `lazy-props` in coordination with `lazy` and `Suspense` we get a unified way of handling promises for our props.

## Typescript

`lazy-props` was designed in Typescript and then written in Javascript for portability. Typing is provided with the library and is as tight and type-safe as possible.

## When should I use it ?

That's a good question. You may have understand that the library will create a new Component each type you call it. It's not that big of a problem, you will probably end up with re-creating dozen of components each second, some more will not make a differences.

Nevertheless, you should notes that `lazy-props`is most intended for one-of-time calls. Typically calls that you would have make in a useEffect with `[]` as last parameter. Because this kind of calls will be made only when a component is created and not when it's state changes.