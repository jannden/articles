## Functional VS Class components in React

_Originally published on Jun 1, 2023 [here](https://medium.com/@jannden/functional-vs-class-components-in-react-d8bc7d019b07)._

---

If you come across an old repository, it might come in handy to know "the old way". So this article compares Functional and Class components.

### Functional component

```react
const FunctionButton = () => {
  const [count, setCount] = React.useState(0)
  const handleClick = () => setCount((prevCount) => prevCount +1)
  return (
    <button onClick={handleClick}>
      Functional button clicks: {count}
    </button>
  )
}
```

### Class component without bind

```react
class ClassButton extends React.Component {
  constructor() {
    super()
    this.state = {
      count: 0,
    }
  }
  clickHandler() {
    this.setState((prevState) => ({ count: prevState.count + 1 }))
  }

  render() {
    return (
      <button onClick={() => this.clickHandler()}>
        Class button clicks: {this.state.count}
      </button>
    )
  }
}
```

### Class component with bind

```react
class ClassButtonBind extends React.Component {
  constructor() {
    super()
    this.state = {
      count: 0,
    }
    // BIND
    // When we use 'bind', invoking the function in the JSX is simpler
    // and faster for processing:
    this.clickHandler = this.clickHandler.bind(this)
  }
  clickHandler() {
    this.setState((prevState) => ({ count: prevState.count + 1 }))
  }

  render() {
    return (
      <button onClick={this.clickHandler}>
        Class (with bind) button clicks: {this.state.count}
      </button>
    );
  }
}
```

### Using Redux

See my next article to see how to use Redux in React with Functional and Class components.

### Conclusion

This short article compares two different approaches to creating React components.

If you found it helpful, please click the clap 👏 button. Also, feel free to comment! I'd be happy to help :)
