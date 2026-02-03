# React Redux --- Crash Course for the Impatient

_Originally published on May 11, 2023 [here](https://medium.com/@jannden/react-redux-2023-crash-course-f93b19330e11)._

---

This article explains how to use Redux in a React app. It's part three of a five-part series on Redux:

1.  [Redux](https://medium.com/@jannden/9f4becebea69),
2.  [Redux Middleware](https://medium.com/@jannden/26746ca4bbf7),
3.  React Redux (this article),
4.  [RTK Redux Toolkit](https://medium.com/@jannden/bb4edb5177dd),
5.  [RTK Query](https://medium.com/@jannden/rtk-query-2023-crash-course-49f1dcb652e7).

By the end, you will learn:

- How to connect Redux to a React app
- How to optimize your app
- How to decide where to store state

TL;DR --- You can see [React Redux in action in my CodeSanbox](https://codesandbox.io/s/06-react-redux-cafe-with-loading-w0ksyf).

### Mini React boilerplate

We have this tiny React app.

```react
import { createRoot } from "react-dom/client";

const rootElement = document.getElementById("root");
const root = createRoot(rootElement);

root.render(
  <div>
    <h1>Hello World</h1>
    <h2>Regards from the tiniest React boilerplate!</h2>
  </div>
);
```

Let's see, how we can add Redux to it.

### How to connect Redux to React

Connect Redux to your React app in just 3 steps:

1.  Wrap the React App in <Provider store={store}>
2.  Use the hook useSelector((state) => state) to get state
3.  Use the hook useDispatch() to dispatch actions

All three are imports from the react-redux package.

This is how a minimalistic React-Redux app looks like. Open my [CodeSandbox](https://codesandbox.io/s/02-tiniest-react-redux-boilerplate-gssp0d) to see it in action.

```react
import { createRoot } from "react-dom/client";
import { createStore } from "redux";
import { Provider, useSelector, useDispatch } from "react-redux";

const reducer = (state = 0, action) => {
  if (action.type === "ADD") {
    return state + 1;
  }
  return state;
};
const store = createStore(reducer);

const App = () => {
  const count = useSelector((state) => state);
  const dispatch = useDispatch();
  const handleClick = () => dispatch({ type: "ADD" });
  return <button onClick={handleClick}>{count}</button>;
};

const rootElement = document.getElementById("root");
const root = createRoot(rootElement);

root.render(
  <Provider store={store}>
    <h1>The tiniest React + Redux boilerplate!</h1>
    <App />
  </Provider>
);
```

### Code structure

Redux has no direct opinion on how your project should be structured. One suggestion is:

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*yAHrJtQwihFgJf62LcpmBA.png)

PS. It's entirely possible (and encouraged) for a reducer defined in one folder to respond to an action defined in another folder.

### React --- Redux optimization

You might ask why to worry about optimization at this point.

The hook *useSelector *re-renders the element (and all of it's children), if it returns a new value. It checks whether the value is new with the reference equality (===).

That might significantly hinder the performance of your React app if you are not careful. That's why we will look at ways to prevent possible optimization problems now.

### Optimization with Memo

Using React.memo() prevents re-rendering the child elements if:

- their props don't change,
- their consts don't change (such as from fetching other changed state slices with useSelector).

So you could easily optimize this way:

```react
import { memo } from React

const ChildComponent = () => {
  return <div />
}

export const ChildComponentMemo = memo(ChildComponent)
```

However, *memo()* doesn't prevent the parent element from re-rendering. That's why we should look for a different solution.

### Optimization with ShallowEqual

*ShallowEqual *is a second parameter for the *useSelector *hook, which prevents the element from re-rendering, if the shallow equality is kept. Look at this example:

```react
import { shallowEqual, useSelector } from 'react-redux'
import { ChildComponent } from './ChildComponent'

const ParentComponent = () => {
  const data = useSelector((state) => state.data, shallowEqual)
  return (
    <>
      {data.map((item, index) => <ChildComponent key={index} item={item} /> )}
    </>
  )
}
```

This example will not re-render if *data* is a not-changed array of primitive values such as *[ 1, 2, 3 ]*.

But it will re-render if *data* is a not-changed array of objects such as *[ { id: 1 }, { id: 2 }, { id: 3 } ]* because the instances of objects are considered new even if their content hasn't changed.

### Passing down the smallest amount of state

We see the importance of shallow equality. So in order to prevent re-renders, let's try to pass down the smallest amount of information.

In our case, it would be an array of IDs instead of an array of objects.

Each *ChildComponent *would then fetch the additional necessary data again through *useSelector* based on the received ID.

```react
import { shallowEqual, useSelector } from 'react-redux'
import { ChildComponent } from './ChildComponent'

const selectIds = (state) => state.data.map((item) => item.id)

export const ParentComponent = () => {
  const listOfIds = useSelector((state) => selectIds(state), shallowEqual)

  return (
    <>
      {listOfIds?.map((id, index) => <ChildComponent key={index} id={id} />)}
    </>
  )
}
```

### Code examples for optimization

Compare this [optimized example](https://codesandbox.io/s/04-react-redux-cafe-optimized-htm59e) to this [not-optimized one](https://codesandbox.io/s/03-react-redux-cafe-not-optimized-8cjvgg).

### Loading states

Redux shouldn't hold every type of state. If state is relevant only to a particular component, consider using simpler ways to manage it.

A great example is a component loading state. It's often better to use a simple useState() hook to manage it. It's not necessary to push it into Redux global state.

```react
const Component = () => {
  const [loadingPayment, setLoadingPayment] = useState(false)

  const dispatch = useDispatch()

  const handlePay = async () => {
    setLoadingPayment(true)
    await dispatch(payAction()) // Dispatch with async action creator
    setLoadingPayment(false)
  }

  return loadingPayment ? <Spinner /> : <Content />
}
```

Compare an [example with loading](https://codesandbox.io/s/06-react-redux-cafe-with-loading-w0ksyf) and [without](https://codesandbox.io/s/05-react-redux-cafe-async-without-loading-1sbpkd).

### Conclusion

Hope you got a good grasp on React Redux. You can open [the final CodeSandbox](https://codesandbox.io/s/06-react-redux-cafe-with-loading-w0ksyf) to see it in action.

We will continue with Redux Toolkit in the next article.

If you found this article helpful, please click the clap 👏 button. Also, feel free to comment! I'd be happy to help :)
