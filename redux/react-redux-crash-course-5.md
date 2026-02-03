# RTK Query --- Crash Course for the Impatient

_Originally published on May 26, 2023 [here](https://medium.com/@jannden/rtk-query-2023-crash-course-49f1dcb652e7)._

---

This article explains RTK Query. It's the last part of a five-part series on Redux:

1.  [Redux](https://medium.com/@jannden/9f4becebea69),
2.  [Redux Middleware](https://medium.com/@jannden/26746ca4bbf7),
3.  [React Redux](https://medium.com/@jannden/f93b19330e11),
4.  [RTK Redux Toolkit](https://medium.com/@jannden/bb4edb5177dd),
5.  RTK Query (this article).

By the end of this article about Redux Toolkit Query, you will learn how to set it up and use it.

TL;DR --- You can see RTK Query in action in my [CodeSanbox](https://codesandbox.io/s/12-example-of-rtk-query-torm76).

### What is RTK Query

Redux Toolkit Query is data fetching and caching solution designed for Redux applications.

RTK Query is included within the installation of the core Redux Toolkit package. It is available via either of the two entry points:

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*GsRnn9EulUe5BLhWqy0tVw.png)

`createApi()` receives:

- _reducerPath_, which is for naming purposes only
- *baseQuery* with *fetchBaseQuery* that specifies the path to the API
- *endpoints *which use *builder.query* for fetching information from the server or *builder.mutation* for updating information on the server

Here is an example how it looks:

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*Gcebn9bN3WxqXKKEqBmguQ.png)

### Cache

A "tag" is a string or small object that lets you name certain types of data, and invalidate portions of cache.

When a cache tag is invalidated, RTK Query will automatically refetch the endpoints that were marked with that tag.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*TimlelsxY3h97A_3VmS8aA.png)

In addition to *tags*, there are other ways to update state even more efficiently. If you want to find out more, read about [optimistic](https://redux-toolkit.js.org/rtk-query/usage/manual-cache-updates#optimistic-updates) and [pessimistic](https://redux-toolkit.js.org/rtk-query/usage/manual-cache-updates#pessimistic-updates) updates.

### Hooks

RTK Query automatically creates hooks for React apps.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*-S0PEuDukHtyIiGoRcWyTg.png)

Which can then be easily used inside of components.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*Rb2Uwk3EAlkYxKchwLOqvQ.png)

### How to add RTK Query to your Redux store

To create a store, we use configureStore from RTK.

Notice, that we also have to add middleware in order to use RTK query.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*Wepo_pd_ON4nsIRRjTV0Vw.png)

### Conclusion

Hope you got a good grasp on the basics of Redux Toolkit Query. You can open my [CodeSandbox](https://codesandbox.io/s/12-example-of-rtk-query-torm76) to see it in action.

This completes the five-part series on Redux.

If you found this article helpful, please click the clap 👏 button. Also, feel free to comment! I'd be happy to help :)
