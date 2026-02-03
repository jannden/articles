# Redux --- Crash Course

_Originally published on Apr 27, 2023 [here](https://medium.com/@jannden/redux-2023-crash-course-9f4becebea69)._

---

This is a complete Crash Course for Redux. It consists of five parts:

1.  Redux (this article),
2.  Redux Middleware,
3.  React Redux,
4.  Redux Toolkit (RTK),
5.  RTK Query.

This article explains pure Redux. By the end, you will learn:

- how does Redux work,
- how to create a JS app with Redux.

TL;DR --- You can see Redux in action in my [CodeSanbox](https://codesandbox.io/s/redux-crash-course-cafe-p13gmv).

### What is Redux

The official documentation says:

> "Redux is a predictable state container for JavaScript apps."

Note that they mention Javascript apps --- instead of React apps. Nowadays, Redux is perhaps mostly used and associated with React, but it's actually a pure Javascript library, so it can work with vanilla Javascript as well as various Javascript frameworks.

### What is the purpose of Redux

Redux allows you to manage your app's state in a single place and keep changes in your app more predictable and traceable.

### When to use Redux?

Think about using Redux when:

- you have big amount of state that is needed in many places in the app
- the state is updated frequently over time
- the logic to update the state may be complex
- the app has a medium to large-sized codebase or many contributors

### What packages do you need to install

The three main tools related to pure Redux are:

- the core library --- [Redux](https://www.npmjs.com/package/redux)
- addon for using async logic (such as fetching data from server) --- [Redux Thunk](https://www.npmjs.com/package/redux-thunk)
- extension for better dev experience and debugging --- [Redux Devtools](https://www.npmjs.com/package/@redux-devtools/extension)

Note: Usually, a regular project needs all of these and even some more. So the Redux team decided to make the setup simpler and now provide a single library that takes care of all of it --- Redux Toolkit. We will talk about [Redux Toolkit](https://www.npmjs.com/package/@reduxjs/toolkit) in part 3 of these series.

Note 2: You also need to install the [React-Redux](https://www.npmjs.com/package/react-redux) package, if you want to use Redux in a React project. You will learn more in part 4 of these series.

## How does Redux work

Imagine Redux is a coffee shop. Here is the explainer:

![If Redux was a coffee shop, the "Action Creators" would be the customers, "Dispatch" would be the waiter, "Reducers" would be the managers, "Subscribe" would be the marketers, "Selectors" would be the secretaries.](https://miro.medium.com/v2/resize:fit:875/1*NlQHM1h32_aZCq3IMBs_oQ.png)

### The main principle of Redux

Redux is all about keeping information available for your app, so that it can be easily used. The information thus needs to be relevant and up to date.

When updating the information, Redux doesn't allow you to change the old information, but rather create a completely new and updated copy. That's what I describe as the main principle of pure Redux.

It means keeping the state immutable, like in this example:

```react
const tables = [
  {
    tableId: 1,
    orderList: {
      espresso: 0,
      cappuccino: 0,
      latte: 0
    }
  }
];

// How to add 3 espressos to tableId = 1

// -> DON'T do this in pure Redux:
tables[0].orderList.espresso += 3

// -> DO this (the 'immutable' way):
const updatedTables = tables.map((table) => {
  if (table.tableId === 1) {
    return {
      ...table,
      orderList: {
        ...table.orderList,
        espresso: table.orderList.espresso + 3
      }
    };
  }
  return table;
});
```

If you want a refresher on immutability, here is an amazing article about [shallow and deep copies in Javascript](https://www.freecodecamp.org/news/copying-stuff-in-javascript-how-to-differentiate-between-deep-and-shallow-copies-b6d8c1ef09cd/).

### What to import into your project

The Redux core is a very small and deliberately unopinionated library. It provides a few small API primitives (we will talk about each of them):

- createStore to actually create a Redux store
- combineReducers to combine multiple slice reducers into a single larger reducer
- compose to combine multiple store enhancers into a single store enhancer
- applyMiddleware to combine multiple middleware into a store enhancer

Other than that, all the other Redux-related logic in your app has to be written entirely by you.

## Tiny Redux App

So how does a redux app look like?

There is a big misconception saying that Redux has a lot of boilerplate. But you don't actually have to write a lot of code to use it. Here is a minimalistic example of using pure Redix in vanilla Javascript:

```react
import { createStore } from "redux";

const reducer = (state = 0, action) => {
  switch (action.type) {
    case "ORDER":
      return state + 1;
    default:
      return state;
  }
};

const store = createStore(reducer);

console.log("default state:", store.getState());

store.dispatch({ type: "ORDER" });

console.log("updated state:", store.getState());
```

That's as simple as it gets.

Now we will go back to the metaphor about the coffee shop and explain each of the parts of Redux with code examples.

### Action Creators

In the minimalistic example, we used:

```react
store.dispatch({ type: "ORDER" });
```

Back in our coffee shop metaphor, that's a waiter *dispach* taking the order *{ type: "ORDER" }*. Currently, the order is anonymous. Let's create a customer of the coffee shop that will create the order and also specify the amount of ordered coffees as *payload*:

```react
// Action Type -> what's possible to do in our coffee shop
const ORDER= "ORDER";

// Action Creator -> customer of the coffee shop creating the action
const orderAction = () => ({
  type: ORDER,
  payload: 1
});

// Dispatch -> waiter taking the order
store.dispatch( orderAction() )
```

### Reducers

Now the order has to be processed by a manager, who is called *reducer *in Redux. Reducers receive the dispatched actions and decide how to update the state, keeping in mind these two rules:

- The state must be updated immutably.
- Always return unchanged state as a default.

```react
const defaultState = 0;

const reducer = (state = defaultState, action) => {
  switch (action.type) {
    case ORDER:
      return state + action.payload;
    default:
      return state;
  }
};
```

### Combining reducers

If there are more reducers, we have to combine them.

We can rename the reducers while combining them. The names basically represent the names of different parts of state.

```react
import { combineReducers } from "redux";

const firstReducer = (state = defaultFirstState, action) => {
  // ... some logic
};

const secondReducer = (state = defaultSecondState, action) => {
  // ... some logic
};

const combinedReducer = combineReducers({
  first: firstReducer,
  second: secondReducer
});
```

### Store

Store consists of all reducers.

If a default state is set while creating the Store, it will overwrite the default state set in the Reducers.

```react
import { createStore } from "redux";
import { reducer, combinedReducer } from "./reducers";

// Simple store
const store1 = createStore(reducer);

// Store with multiple reducers created with combineReducers()
const store2 = createStore(combinedReducer);

// Store with default state
const store3 = createStore(combinedReducer, {
  first: defaultFirstState,
  second: defaultSecondState
});
```

### Selectors

A selector function is any function that accepts the state as an argument, and returns data that is based on that state.

Selectors are not required, but they are a standard pattern.

```react
const exampleState = [
  {
    tableId: 1,
    orderList: {
      espresso: 10,
      cappuccino: 0,
      latte: 0
    }
  }
];

// Get amount of espressos on a particular table
const espressosSelector = (state, desiredTable) => {
  return state.find((table) => table.tableId === desiredTable).orderList
    .espresso;
};
```

### Subscribe

Subscribe is a function that gets called when state changes

We can easily update the UI thanks to Subscribe.

```react
const handleChange = () => {
  // ... for example rendering UI
};

const unsubscribe = store.subscribe(handleChange);

// When necessary, we can unsubscribe
unsubscribe();
```

## Complete Redux Javascript app

Now let's combine all the concepts we have learned so far in a complete example of a simple Javascript app using Redux:

```react
import { createStore, combineReducers } from "redux";

// Default State
const defaultOrder = 0;
const defaultAccounting = 0;

// Action Types
const ORDER = "ORDER";
const PAY = "PAY";

// Action Creators
const orderAction = (coffeeCount) => ({
  type: ORDER,
  payload: {
    coffeeCount
  }
});
const payAction = (totalOrders) => {
  const priceOfCoffee = 10;
  return {
    type: PAY,
    payload: {
      totalOrderValue: totalOrders * priceOfCoffee
    }
  };
};

// Reducers
const orderReducer = (state = defaultOrder, action) => {
  switch (action.type) {
    case ORDER:
      return state + action.payload.coffeeCount;
    case PAY:
      return defaultOrder;
    default:
      return state;
  }
};

const accountingReducer = (state = defaultAccounting, action) => {
  if (action.type === PAY) {
    return state + action.payload.totalOrderValue;
  }
  return state;
};

// Combining reducers
const rootReducer = combineReducers({
  orders: orderReducer,
  turnover: accountingReducer
});

// Creating store
const store = createStore(rootReducer);

// Subscribing to changes
store.subscribe(() => console.log("updated state:", store.getState()));

// Dispatching actions
store.dispatch(orderAction(3));
store.dispatch(orderAction(5));

// Using selectors
const partOfStateSelector = (state, partOfState) => {
  return state[partOfState];
};
const totalOrders = partOfStateSelector(store.getState(), "orders");
store.dispatch(payAction(totalOrders));
```

## Redux Devtools extension

Redux Devtools helps with debugging Redux apps. Download [the extension](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd) from Chrome Store and then inject a *store enhancer* to your Store.

You can use the composeWithDevTools function or inject it explicitly, the result should be the same. I will show you both ways.

Once injected in your app, open the Chrome Extension and observe how the state of your application changes.

### Option 1: Using composeWithDevTools

```react
import { createStore, applyMiddleware, compose } from "redux";
import thunkMiddleware from "redux-thunk";
import { composeWithDevTools } from "@redux-devtools/extension";

const store = createStore(
  reducer,
  composeWithDevTools(applyMiddleware(thunkMiddleware))
);
```

### Option 2: Injecting Devtools explicitly

```react
import { createStore, applyMiddleware, compose } from "redux";
import thunkMiddleware from "redux-thunk";

const store = createStore(
  reducer,
  compose(
    applyMiddleware(thunkMiddleware),
    window.__REDUX_DEVTOOLS_EXTENSION__
      ? window.__REDUX_DEVTOOLS_EXTENSION__()
      : (f) => f
  )
);
```

## Conclusion

Hope you got a good grasp on pure Redux. You can open my [CodeSandbox](https://codesandbox.io/s/redux-crash-course-cafe-p13gmv) to see various examples of Redux implementations.

We will continue with Redux Middleware in the next article.

If you found this article helpful, please click the clap 👏 button. Also, feel free to comment! I'd be happy to help :)
