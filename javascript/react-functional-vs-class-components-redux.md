## Functional VS Class components in React-Redux

_Originally published on Jun 10, 2023 [here](https://medium.com/@jannden/functional-vs-class-components-in-react-redux-4aa7ac432d1c)._

---

This article shows a minimalistic React-Redux app in both functional and class components, so that you can see the differences in syntax.

If you want more insights into Functional and Class components in React, see my previous article.

Task for this article is to use React with Redux to create a counter with:

- a button to increase a number by a value from an input,
- a button to decrease a number by a value from an input,
- an option to change color asyncroniously.

### Redux setup

Setting up Redux is the same for Functional and Class React components. So the Redux code will be the same for both cases.

You can read more about Redux in my [Crash Course](https://medium.com/@jannden/9f4becebea69).

```react
import { createStore, combineReducers, applyMiddleware } from "redux";
import ReduxThunk from "redux-thunk";
import { composeWithDevTools } from "@redux-devtools/extension";

// Actions
const INCREASE = "INCREASE";
const DECREASE = "DECREASE";
const CHANGE_COLOR = "CHANGE_COLOR";

// Action creators
export const increaseByNumber = (customNumber) => ({
  type: INCREASE,
  payload: { customNumber: customNumber }
});
export const decreaseByNumber = (customNumber) => ({
  type: DECREASE,
  payload: { customNumber: customNumber }
});
const changeColor = () => ({
  type: CHANGE_COLOR
});
export const changeColorAsync = () => {
  return function (dispatch) {
    return setTimeout(() => dispatch(changeColor()), 700);
  };
};

// Reducers
const counterReducer = (state = 0, action) => {
  switch (action.type) {
    case INCREASE:
      return state + action.payload.customNumber;
    case DECREASE:
      return state - action.payload.customNumber;
    default:
      return state;
  }
};
const colorReducer = (state = "red", action) => {
  if (action.type === CHANGE_COLOR && state === "red") {
    return "green";
  }
  return "red";
};

// Combine reducers
const rootReducer = combineReducers({
  counter: counterReducer,
  color: colorReducer
});

// Store
export const store = createStore(
  rootReducer,
  composeWithDevTools(applyMiddleware(ReduxThunk))
);
```

### Functional Component

```react
import React from "react";
import { useDispatch, useSelector } from "react-redux";

import { increaseByNumber, decreaseByNumber, changeColorAsync } from "./store";

export const FunctionalApp = () => {
  // Simple useState for the number input
  const [customNumber, setCustomNumber] = React.useState(1);
  const handleCustomNumberChange = (e) => {
    setCustomNumber(Number(e.target.value));
  };

  // With useDispatch, we can access the dispatch function in any component/file
  const dispatch = useDispatch();

  // We call the reducers and pass them actions created by action creators
  // Notice here that we use simple dispatch(), which is thanks to the useDispatch hook. If we haven't used the useDispatch hook, we could use store.dispatch(), but then store would have to be passed to this component as a prop
  const handleDecrease = () => dispatch(decreaseByNumber(customNumber));
  const handleIncrease = () => dispatch(increaseByNumber(customNumber));
  const handleColorChange = () => dispatch(changeColorAsync());

  // In order to read state, we use hook useSelector, which calls the store and returns desired state
  // Similarly as with dispatch(), we could instead use simply store.getState(), but then, we would have to have store passed to this component as a prop
  const counter = useSelector((state) => state.counter);
  const color = useSelector((state) => state.color);

  return (
    <div>
      <h3>Functional Component</h3>
      <button onClick={handleColorChange}>Change color</button>
      <h1 style={{ color }}>{counter}</h1>
      <p>Change count by:</p>
      <button onClick={handleDecrease}>-</button>
      <input
        type="number"
        onChange={handleCustomNumberChange}
        value={customNumber}
      />
      <button onClick={handleIncrease}>+</button>
    </div>
  );
};
```

### Class Component

```react
import React from "react";
import { connect } from "react-redux";

import { increaseByNumber, decreaseByNumber, changeColorAsync } from "./store";

class NumberInput extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      customNumber: 1
    };
    this.handleDecrease = this.handleDecrease.bind(this);
    this.handleIncrease = this.handleIncrease.bind(this);
    this.handleCustomNumberChange = this.handleCustomNumberChange.bind(this);
  }

  handleCustomNumberChange(e) {
    this.setState({ customNumber: Number(e.target.value) });
  }

  handleDecrease() {
    return this.props.decreasing(this.state.customNumber);
  }
  handleIncrease() {
    return this.props.increasing(this.state.customNumber);
  }

  render() {
    return (
      <>
        <p>Change count by:</p>
        <button onClick={this.handleDecrease}>-</button>
        <input
          type="number"
          onChange={this.handleCustomNumberChange}
          value={this.state.customNumber}
        />
        <button onClick={this.handleIncrease}>+</button>
      </>
    );
  }
}

class ColorSwitch extends React.Component {
  handleColorChange() {
    return this.props.dispatch(changeColorAsync());
  }

  render() {
    return (
      <button onClick={() => this.handleColorChange()}>Change color</button>
    );
  }
}

class Counter extends React.Component {
  render() {
    return <h1 style={{ color: this.props.color }}>{this.props.counter}</h1>;
  }
}

/*** SETTING UP REDUX FOR CLASS COMPONENTS ***/

// This allows us to specify, which state departments can a component use
const mapStateToProps = function (state) {
  return { color: state.color, counter: state.counter };
};

// This allows us to specify, which action creators can dispatch within a component use
// Notice, how we changed the name of the action dispatch for the component NumberInput
// Compare it to the component ColorSwitch, for which we haven't specified any allowed action creator, so that component has access to the whole dispatch()

const mapDispatchToPropsNumberInput = (dispatch) => {
  return {
    increasing: (customNumber) => dispatch(increaseByNumber(customNumber)),
    decreasing: (customNumber) => dispatch(decreaseByNumber(customNumber))
  };

  // The above would be the same as the shorter version bellow:
  // return mapDispatchToPropsNumberInput = Redux.bindActionCreators({increasing: increaseByNumber, decreasing: decreaseByNumber}, dispatch)
  // The only use case for bindActionCreators is when you want to pass some action creators down to a component that isn't aware of Redux, and you don't want to pass dispatch or the Redux store to it.
};

// Now we wrap the React Class components with Redux, so that we can actually use all the stuff
const ReduxColorSwitchContainer = connect()(ColorSwitch);
const ReduxCounterContainer = connect(mapStateToProps)(Counter);
const ReduxNumberInputContainer = connect(
  null,
  mapDispatchToPropsNumberInput
)(NumberInput);

// And we now render the components
export class ClassApp extends React.Component {
  render() {
    return (
      <div>
        <h3>Class Component</h3>
        <ReduxColorSwitchContainer />
        <ReduxCounterContainer />
        <ReduxNumberInputContainer />
      </div>
    );
  }
}
```

### Conclusion

This short article compares Redux in a React app using Functional and Class components.

If you found it helpful, please click the clap 👏 button. Also, feel free to comment! I'd be happy to help :)
