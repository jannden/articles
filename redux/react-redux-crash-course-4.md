# Redux Toolkit (RTK) --- Crash Course for the Impatient

_Originally published on May 18, 2023 [here](https://medium.com/@jannden/redux-toolkit-rtk-2023-crash-course-bb4edb5177dd)._

---

This article explains Redux Toolkit (RTK). It's part four of a five-part series on Redux:

1.  [Redux](https://medium.com/@jannden/9f4becebea69),
2.  [Redux Middleware](https://medium.com/@jannden/26746ca4bbf7),
3.  [React Redux](https://medium.com/@jannden/f93b19330e11),
4.  RTK Redux Toolkit (this article),
5.  [RTK Query](https://medium.com/@jannden/rtk-query-2023-crash-course-49f1dcb652e7).

By the end of this article about Redux Toolkit, you will learn:

- Use Redux Toolkit declaratively
- Use the Slice Logic of RTK
- Create a React app with RTK

TL;DR --- You can see Redux Toolkit in action in my [CodeSanbox](https://codesandbox.io/s/11-react-redux-cafe-rtk-async-dtgd80).

### What is RTK

Redux Toolkit simplifies most Redux tasks, prevents common mistakes, and makes it easier to write Redux applications.

### How to set it up

Store is created with *configureStore()*, which gives many benefits over the old *createStore()* that you might remember from the previous articles of this series.

Among other things, the new *configureStore()* automatically adds async capabilities, as well as Redux Devtools Extension support. And we don't have to use *combineReducers()* anymore.

We will have a look a minimalistic RTK app using *createAction *and *createReducer*.

*CreateReducer *takes the default state as the first parameter and function with *builder *as the second parameter. We use *builder *to create reducer logic.

Have a look at this minimalistic app and then we will explain everything step by step:

```react
import {
  configureStore,
  createAction,
  createReducer,
  current
} from "@reduxjs/toolkit";

console.clear();

const increment = createAction("increment");

const counterReducer = createReducer({ count: 0 }, (builder) => {
  builder.addCase(increment, (state, action) => {
    console.log("old state", current(state).count);
    state.count++;
    console.log("updated state", current(state).count);
  });
});

const store = configureStore({ reducer: { counter: counterReducer } });

store.dispatch(increment());
```

### Reducer builder

Thanks to the *builder *object provided by the RTK library, we can chain different rules for the reducer logic:

- addCase matches an action instance
- addMatcher matches a more broader requirements that are passed in as a function
- addDefaultCase helps when we need to address any other cases

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*iYOspwLgnWqv9jRHCUSyxg.png)

### Using Actions

When calling actions, we can pass the function any parameters.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*8sTACUnt9jucTB1CupLVyw.png)

### Additional logic for createAction

If we want additional logic for creating an *Action*, we can pass a function as the second parameter to *createAction*.

This function can then customize the payload object.

Notice the use of *nanoid()* which is a useful way of generating unique IDs.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*9ZFhooOrRcN7RefvYTww-g.png)

### Slice logic

Redux toolkit has a second option for creating actions and reducers.

Using *createSlice*, the actions and reducers are automatically taken care of.

### createSlice

Slices are like islands of logic around one part of state.

Slice receives:

- name
- initialState
- reducers

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*eWsZAyvpkfpFUQ3oSIqaKA.png)

Notice, that the reducers in RTK can directly modify the state. That was unthinkable in pure Redux (do you remember the principle of immutability from the previous parts of this series?).

Now we don't have to worry about immutability anymore.

### Reducers and Action Creators from Slices

Created slice gives us access to related reducers and action creators.

Notice that action creators are usually de-structured from the actions object.

In contrast, the reducer is only one, so it's not getting de-structured.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*hU4EVoXp35jfUDfke9S5sg.png)

### current

Since RTK uses IMMER library to update state, it's not possible to console.log state from a reducer.

However, we can use current() for this purpose.

It helps with de-bugging.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*fxFbnySDAcQ5o0yJNEsGnQ.png)

### Tiny RTK app with slice

This is a minimalistic RTK app using *createSlice:*

```react
import { configureStore, createSlice, current } from "@reduxjs/toolkit";

const counter = createSlice({
  name: "counter",
  initialState: { count: 0 },
  reducers: {
    increment: (state, action) => {
      console.log("old state", current(state).count);
      state.count++;
      console.log("updated state", current(state).count);
    }
  }
});

const { increment } = counter.actions;
const counterReducer = counter.reducer;

const store = configureStore({ reducer: { counter: counterReducer } });

store.dispatch(increment());
```

### extraReducers

A reducer created from a slice might need to respond to an action from another slice.

That's when extraReducers come in handy.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*yCm9v8e2n3nEYCy0McSl5A.png)

### Generating unique IDs in createSlice

An action can be divided into:

- reducer
- prepare

Prepare is the place to adjust the payload including the generation of random values.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*XnAsxywcqRtXhlvDejsmVw.png)

### createAsyncThunk

This is basically an async action creator.

```react
import {
  configureStore,
  createSlice,
  createAsyncThunk
} from "@reduxjs/toolkit";

export const fetchFromServer = async () => {
  const output = 2;
  const delay = 2000;
  return new Promise((resolve) => setTimeout(() => resolve(output), delay));
};

export const increment = createAsyncThunk(
  "counter/increment",
  async ({ base }) => {
    const multiplicator = await fetchFromServer();
    return { incrementAmount: base * multiplicator };
  }
);

const counterSlice = createSlice({
  name: "counter",
  initialState: { count: 0 },
  extraReducers(builder) {
    builder
      .addCase(increment.pending, (state, action) => {})
      .addCase(increment.fulfilled, (state, action) => {
        state.count = action.payload.incrementAmount;
      })
      .addCase(increment.rejected, (state, action) => {});
  }
});

const store = configureStore({ reducer: counterSlice.reducer });

store.subscribe(() => console.log(store.getState()));

store.dispatch(increment({ base: 10 }));
```

### Conclusion

Hope you got a good grasp on Redux Toolkit. You can open the final [CodeSandbox](https://codesandbox.io/s/11-react-redux-cafe-rtk-async-dtgd80) to see it in action.

We will continue with RTK Query in the next article.

If you found this article helpful, please click the clap 👏 button. Also, feel free to comment! I'd be happy to help :)
