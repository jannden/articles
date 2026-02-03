# Redux Middleware --- Crash Course for the Impatient

_Originally published on May 4, 2023 [here](https://medium.com/@jannden/redux-middleware-2023-crash-course-26746ca4bbf7)._

---

This article explains how to use Middleware in Redux. It's the second part of a five-part series on Redux:

1.  [Redux](https://medium.com/@jannden/9f4becebea69),
2.  Redux Middleware (this article),
3.  [React Redux](https://medium.com/@jannden/f93b19330e11),
4.  [RTK Redux Toolkit](https://medium.com/@jannden/bb4edb5177dd),
5.  [RTK Query](https://medium.com/@jannden/rtk-query-2023-crash-course-49f1dcb652e7).

By the end, you will learn:

- what is Middleware and how does it work in Redux,
- how to do async operations in Redux thanks to middleware,
- how to use Redux Thunk.

TL;DR --- You can see Middleware and Thunk in action in my [CodeSanbox](https://codesandbox.io/s/redux-crash-course-cafe-p13gmv).

### What is Middleware

*Middleware *sits in between the *dispatch *method and the *reducers*.

Going back to the metaphor from the previous article that compares Redux to a coffe shop:

Customers *(action creators)* of a coffee shop make an order *(action)* that is taken by a waiter *(dispatch)* who brings it to the manager *(reducer)*. *Middleware *would be a detour that the waiter would take before getting to the manager.

For example, the waiter can stop on his way and send an inquiry to the accounting department to ask for the price of one cup of coffee *(asynchronious operation)*. Once he gets the response, he will proceed to the manager.

### Why we need middleware

Middleware does what the reducers cannot.

Reducers must be pure functions and cannot create side effects, such as:

- logging a value to the console,
- saving a file,
- using async operations,
- generating unique random values (such as *Math.random()* or *Date.now()*)

Middleware enables logic that has side effects.

### How does middleware look like

Here is a minimalistic example how to create a middleware and how to hook it up to the *store*.

```react
import { createStore, applyMiddleware } from "redux";

const customMiddleware = (store) => (next) => (action) => {
  // ... some logic
};

const store = createStore(
  reducer,
  defaultState, // optional
  applyMiddleware(customMiddleware) // optional
);
```

Did you notice the strange notation *(store) => (next) => (action)*?

The middleware receives a store, then returns a function that receives a next function and returns another function that receives an action.

Middleware uses syntax of curried functions. It looks weird, but it's nothing to worry about.

### What is a curried function

Currying simply means evaluating functions with multiple arguments and decomposing them into a sequence of functions with a single argument. This example says it all:

```react
// Noncurried version
const add = (a, b, c) => {
  return a + b + c;
};
console.log(add(2, 3, 5)); // 10

// Curried version
const curriedAdd = (a) => {
  return (b) => {
    return (c) => {
      return a + b + c;
    };
  };
};
console.log(curriedAdd(2)(3)(5)); // 10
```

### Minimalistic Redux App with Middleware

Let's go back to the minimalistic example of a Redux app from the previous article. We will add a middleware that uses console.log() to inform us that state changed.

```react
import { createStore, applyMiddleware } from "redux";

const reducer = (state = 0, action) => {
  switch (action.type) {
    case "ADD":
      return state + 1;
    default:
      return state;
  }
};

export const customMiddleware = (store) => (next) => (action) => {
  console.log("default state:", store.getState());
  next(action);
  console.log("updated state:", store.getState());
  return;
};

const store = createStore(reducer, applyMiddleware(customMiddleware));

store.dispatch({ type: "ADD" });
```

### Async Middleware

Let's use the middleware for an async call now.

Notice how middleware dispatches data obtained from server.

```react
const serverRequest = () => {
  // Simulating a promise from a server
  const result = 1;
  return new Promise((resolve) => setTimeout(() => resolve(result), 2000));
};

const customMiddleware = (store) => (next) => (action) => {
  switch (action.type) {
    case "SERVER":
      console.log(store.getState());
      serverRequest().then((response) => {
        store.dispatch({ type: "ADD", payload: response });
        console.log(store.getState());
      });
      break;
    default:
      next(action);
  }
};
```

This middleware starts to look too much like a reducer. A cleaner approach would be to write this async logic in an Action Creator. But Redux expects *dispatch* to receive a plain object, not an async function. So we will rewrite the middleware to recognize when we pass a plain action object and when we pass an actual async function.

This way, we can extract the async logic to the Action Creator.

```react
export const customMiddleware = (store) => (next) => (action) => {
  // If the "action" is actually a function instead...
  if (typeof action === "function") {
    // then call the function and pass on dispatch and getState
    return action(store.dispatch, store.getState);
  }

  // Otherwise, it's a normal action - send it onwards
  return next(action);
};

// Possibility 1: normal action
const normalAction = () => ({ type: "ADD", payload: 1 });
store.dispatch(normalAction());

// Possibility 2: asynchronious action
const asyncFunctionAction = () => (dispatch, getState) => {
  console.log("current state", getState());
  serverRequest().then((response) => {
    dispatch({ type: "ADD", payload: response });
    console.log("updatedstate", getState());
  });
};
store.dispatch(asyncFunctionAction());
```

### Redux Thunk

What we have created in the previous step is actually a simple version of Redux Thunk.

We can use the [redux-thunk](https://www.npmjs.com/package/redux-thunk) package out of the box for simplicity.

```react
import { createStore, applyMiddleware } from "redux";
import thunkMiddleware from "redux-thunk";
import { customMiddleware } from "./custom-middleware";

const store = createStore(
  reducer,
  applyMiddleware(customMiddleware, thunkMiddleware)
);
```

### Conclusion

Hope you got a good grasp on using middleware in Redux. You can open my [CodeSandbox](https://codesandbox.io/s/redux-crash-course-cafe-p13gmv) to see various examples of Redux implementations.

We will continue with Redux Toolkit (RTK) in the next article.

If you found this article helpful, please click the clap 👏 button. Also, feel free to comment! I'd be happy to help :)
