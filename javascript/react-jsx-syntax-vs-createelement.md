# React JSX syntax VS createElement

*Originally published on Apr 20, 2023 [here](https://medium.com/@jannden/react-jsx-syntax-vs-createelement-371c63cb63ab).*

---

As a React developer, you probably use JSX.

> JSX is a syntax extension for JavaScript that lets you write HTML-like markup inside a JavaScript file.

Did you know that it's possible to use React without JSX? There is a legacy [*createElement*](https://beta.reactjs.org/reference/react/createElement) method. It looks like this:

`React.createElement(type, props, ...children)`

Compare the following components --- one with *createElement *and one with JSX:

### Component with createElement

```javascript
function ComponentWithCreateElement() {
  const props = {
    style: { color: "red" },
    onClick: (e) => e.target.innerText = 'Clicked',
  };
  return React.createElement("div", null, [
    React.createElement("button", props, "Component with createElement"),
  ]);
}
```

### Component with JSX

```javascript
function ComponentWithJSX() {
  const style = { color: "green" };
  const handleClick = (e) => e.target.innerText = 'Clicked';
  return (
    <div>
      <button style={style} onClick={handleClick}>
        Component with JSX
      </button>
    </div>
  );
}
```

## Conclusion

This short article compares two different approaches to creating React components.

If you found it helpful, please click the clap 👏 button. Also, feel free to comment! I'd be happy to help :)
