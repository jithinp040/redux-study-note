# redux-study-note


# Redux Recommended Way

<https://redux.js.org/tutorials/fundamentals/part-8-modern-redux>. The internal
workings are described below

# Redux
## Important tips on Redux-React
### where to store data =\> redux/local
1. In a React + Redux app, your global state should go in the Redux store, and
    your local state should stay in React components.

2. If you're not sure where to put something, here are some common rules of
    thumb for determining what kind of data should be put into Redux:

    - Do other parts of the application care about this data?

    - Do you need to be able to create further derived data based on this
        original data?

    - Is the same data being used to drive multiple components?

    - Is there value to you in being able to restore this state to a given
        point in time (ie, time travel debugging)?

    - Do you want to cache the data (ie, use what's in state if it's already
        there instead of re-requesting it)?

    - Do you want to keep this data consistent while hot-reloading UI
        components (which may lose their internal state when swapped)?

### Side Effects

Redux reducers must never contain "side effects". A "side effect" is any change to state or behavior that can be seen outside of returning a value from a function. Some common kinds of side effects are things like:

- Logging a value to the console

- Saving a file

- Setting an async timer

- Making an AJAX HTTP request

- Modifying some state that exists outside of a function, or mutating arguments to a function

- Generating random numbers or unique random IDs (such as Math.random() or Date.now())

Thunk functions can be used for both **asynchronous and synchronous** logic.
Thunks provide a way to write any reusable logic that needs access to *dispatch
and getState*.

### **[Flux Standard Actions" convention](<https://github.com/redux-utilitiesflux-standard-action#motivation>)**

or "FSA". This is a suggested approach for how to organize fields inside of
action objects, so that developers always know what fields contain what kind of
data. The FSA pattern is widely used in the Redux community, and in fact you've
already been using it throughout this whole tutorial.

The FSA convention says that:

- If your action object has any actual data, that "data" value of your action should always go in action.payload

- An action may also have an action.meta field with extra descriptive data

- An action may have an action.error field with error information

So, *all* Redux actions MUST:

- be a plain JavaScript object

- have a type field

And if you write your actions using the FSA pattern, an action MAY

- have a payload field

- have an error field

- have a meta field

### Normalization of Data is necessary

in larger Redux apps, it is common to store data in a normalized state
structure. "Normalization" means:

- Making sure there is only one copy of each piece of data

- Storing items in a way that allows directly finding items by ID

- Referring to other items based on IDs, instead of copying the entire item

For example, in a blogging application, you might have Post objects that point
to User and Comment objects. There might be many posts by the same person, so if
every Post object includes an entire User, we would have many copies of the same
User object. Instead, a Post object would have a user ID value as post.user, and
then we could look up User objects by ID as state.users[post.user].

## Summary of Standard Redux Practices

- **Action creator functions encapsulate preparing action objects and thunks**

  - Action creators can accept arguments and contain setup logic, and return
        the final action object or thunk function

- **Memoized selectors help improve Redux app performance**

  - Reselect has a createSelector API that generates memoized selectors

  - Memoized selectors return the same result reference if given the same
        inputs

- **Request status should be stored as an enum, not booleans**

  - Using enums like 'idle' and 'loading' helps track status consistently

- **"Flux Standard Actions" are the common convention for organizing action
    objects**

  - Actions use payload for data, meta for extra descriptions, and error for
        errors

- **Normalized state makes it easier to find items by ID**

  - Normalized data is stored in objects instead of arrays, with item IDs as
        keys

- **Thunks can return promises from dispatch**

  - Components can wait for async thunks to complete, then do more work

This means we typically organize our data as objects instead of arrays, where
the item IDs are the keys and the items themselves are the values.

# My Notes on Redux

## 1. Create reducers

They should not contain any sort of async actions A single reducer contains
Syntax:

```js
 function reducer(state=initialState,action){
 switch(action.type){ //action.type string format: statename/actionname
  case action.type: {return ...state,action.payload};
  default: ...state
 }
 }
```

## 2. Combine reducers

```js
 import { combineReducers } from 'redux';
 const app/RootReducerName = combineReducers({importedReducers});
```

## 3. Create Store

```js
 import { createStore,compose,applyMiddleware } from 'redux'
 
 const store = createStore(app/RootReducerName,InitialStateIfPresent,enhancer/middleware);
 where 
  middleware = applyMiddleware(middleware1,middleware2,toMiddlewareN)
  enhancer = compose(enhancer1,enhancer2,toEnhancerN)

 middleware syntax: //This supports all async actions.
  
  const middlewareName = StoreAPI => next => action => {
   ...lines to execute
   return next(action); //Removing this line destroys access of state to successive middleware
  }

// To use ReduxDevTools: 
//Not doing this will make reduxDevTools to have no data

import { composeWithDevTools } from 'redux-devtools-extension'

const composedEnhancer = composeWithDevTools(

// EXAMPLE: Add whatever middleware you actually want to use here

applyMiddleware(print1, print2, print3)

// other store enhancers if any

)
```

## Async Logic and Data Fetching

### Redux middleware were designed to enable writing logic that has side effects. Side effect is something which is mentioned in **tip 3.**

The normal implementation of async function middleware for redux and its data
flow is found in the following website.

[Redux Fundamentals, Part 6: Async Logic and Data Fetching \|
Redux](https://redux.js.org/tutorials/fundamentals/part-6-async-logic)

Redux offers async call middleware using **Redux "Thunk" middleware**. The thunk
middleware allows us to write functions that get dispatch and getState as
arguments. The thunk functions can have any async logic we want inside, and that
logic can dispatch actions and read the store state as needed.

**Writing async logic as thunk functions allows us to reuse that logic without
knowing what Redux store we're using ahead of time.**

INFO

The word **"thunk"** is a programming term that means **"a piece of code that
does some delayed work"**. For more details, see these posts:

- [What the heck is a thunk?](https://daveceddia.com/what-is-a-thunk/)

- [Thunks in Redux: the
    basics](https://medium.com/fullstack-academy/thunks-in-redux-the-basics-85e538a3fe60)

### Syntax for writing thunk function is as follows

```js
export async function functionName(dispatch, getState) {
  const response = await asyncFunction('/fakeApi/')
  dispatch({ type: actionName', payload: response })
}
```

A notable advantage in using redux is **dispatching an action immediately
updates the store**, we can also call getState in the thunk to read the updated
state value after we dispatch.

In case of ‘POST’ method, while writing thunk function we're going to run into a
problem: since we're writing the thunk as a separate function in separate file,
the code that makes the API call doesn't know what the input data is supposed to
be. For eg:

```js
async function thunkPostFunction(dispatch, getState) {
  // ❌ We need to have the data, but where is it coming from?
  const postData = { data }
  const response = await client.post('/fakeApi', {data: postData })
  dispatch({ type: actionName', payload: response })
}
```

This is solved by writing a function which receives an input and returns a
function which is a thunk function. **Anonymous function is recommended for
writing thunks.**

```js
async function addData(data) {
  return async function thunkPostFunction(dispatch, getState){
  const postData = { data }//Data is passed correctly
  const response = await client.post('/fakeApi', {data: postData })
  dispatch({ type: actionName', payload: response })
}
}
```

Normally, the API call might take a while to resolve. In that case, it's common
to show some kind of a loading spinner while we wait for the response to
complete.

This is usually handled in Redux apps by:

- Having some kind of "loading state" value to indicate the current status of
    a request

- Dispatching a "request started" action *before* making the API call, which
    is handled by changing the loading state value

- Updating the loading state value again when the request completes to
    indicate that the call is done

The UI layer then shows a loading spinner while the request is in progress, and
switches to showing the actual data when the request is complete.

Tip: **Try to write the state file as a enum, or string instead of Boolean to
respond to several kinds of error.**

## For the complete Notes: go To <https://redux.js.org/tutorials/fundamentals/>

## 5. Redux and React

```js
import {useSelector,useDispatch, Provider, shallowEqual } from 'react-redux'
```

### Read state from Store

1. useSelector lets your React components read data from the Redux store.

    - Helps in automatic subscription.

        - Makes strict check, so causes unnecessary re-render.

        - Multiple useSelector to get separate values is recommended

            useSelector accepts a single function, which we call a selector
            function. A selector is a function that takes the entire Redux store
            state as its argument, reads some value from the state, and returns
            that result.eg:

```js
const selectStateVariable = state => state.variable
inside React component:
const state = useSelector(selectStateVariable,optionalShallowEqualComparator) or useSelector(state => state.variable,optionalShallowEqualComparator)
```

Redux recommends using **memorized selectors like** **reselect library’s createSelector module** instead of useSelector in case of array.map usage in useSelector (this causes re-render every time). **The createSelector API will generate memoized selector functions**. createSelector accepts one or more "input selector" functions as arguments, plus an "output selector", and returns the new selector function. Every time you call the selector:

- All "input selectors" are called with all of the arguments

- If any of the input selector return values have changed, the "output
    selector" will re-run

- All of the input selector results become arguments to the output selector

- The final result of the output selector is cached for next time

```js
createSelector(
  // First, pass one or more "input selector" functions:
  state => state.variable, //The state can vary
  // Then, an "output selector" that receives all the input results as arguments
  // and returns a final result value
  variable => variable.map(item => item.id) // This can have any sort of logic like filter.
)
```

This actually behaves **a bit differently** than the shallowEqual comparison function does. Any time the state array changes, we're going to create a new array as a result. That includes any immutable updates to items like toggling their completed field, since we have to create a new array for the immutable update. createSelector allows multiple state to be extracted at once but this is not recommended
> **TIP**: Memoized selectors are only helpful when you actually derive additional values from the original data. If you are only looking up and returning an existing value, you can keep the selector as a plain function.

#### **Read Dispatcher from Store**

useDispatch returns the store.dispatch(action) function, which is used to dispatch actions. **Returning the dispatch object via function which returns just an object {type:’’ , payload: data} is recommended by redux.**

```js
const dispatch = useDispatch()
dispatch({ type: 'state/action', payload: {} })
```

#### Give Store to app

> Provider Component passes the store to the components.

In Top-Most Component:

```html
<Provider store={storeFile}> //storeFile implies createStore()...
<App/>
</Provider>
```

P.S: I really believe that the notes supplied in redux website is far better
than what I have written, this will be useful only for me, I guess.
