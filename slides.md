Using Redux with React for easy state handling

![](http://logos-download.com/wp-content/uploads/2016/09/React_logo_wordmark.png) <!-- .element: class="plain" style="padding: 40px 50px; width: 300px;" -->

+

![](https://raw.githubusercontent.com/reactjs/redux/master/logo/logo-title-light.png) <!-- .element: class="plain" style="width: 400px;" -->



## ![](https://raw.githubusercontent.com/reactjs/redux/master/logo/logo-title-light.png) <!-- .element: class="plain" style="height: 1.5em; margin: -30px -20px; background-color: transparent;" --> basics

* Simplicity
  * < 100 lines of code (no error handling)
* Some practices enforced, others not
* No ties with React
  * Standalone usage & other frameworks


## Store

![](/reduxtree.png) <!-- .element: class="plain" style="height: 7em;" -->

* Contains entire **app state in one place**
  * Easy to inspect when debugging
  * Easily accessed from anywhere in code
* Tree structure, organized by concepts


## Actions

![](/reduxactions.png) <!-- .element: class="plain" style="height: 7em;" -->
* Actions describe **what happened**
  * Example: menu drawer toggled
* **State can only change via actions**
  * Easy to follow what happened


## Reducer

* Decides how state changes after action
* Returns new state based on old state and action

`(oldState, action) => newState`

```
const initialState = { toggled: false };

const buttonReducer = (state = initialState, action) => {
  switch (action.type) {
    case PRESS_BUTTON:
      return { ...state, toggled: !state.toggled };
    default:
      return state
  }
}
```



## Standalone ![](https://raw.githubusercontent.com/reactjs/redux/master/logo/logo-title-light.png) <!-- .element: class="plain" style="height: 1.5em; margin: -30px -20px; background-color: transparent;" --> usage

* In the simplest case we need:
  * Action type
  * Action creator
  * Reducer


```
// Action type, action creator
const PRESS_BUTTON = 'PRESS_BUTTON';
const pressButton = () => ({ type: PRESS_BUTTON });

// Reducer
const initialState = { toggled: false };
const buttonReducer = (state = initialState, action) => {
  switch (action.type) {
    case PRESS_BUTTON:
      return { ...state, toggled: !state.toggled };
    default:
      return state
  }
};
```


```
// Add our button reducer to the store
const store = createStore(combineReducers({
  button: buttonReducer
}));

// Store initial state looks like:
// { button: { toggled: false }}
```

Let's dispatch some actions:

```
store.dispatch(pressButton());

// Store state now:
// { button: { toggled: true }}

store.dispatch(pressButton());

// Store state now:
// { button: { toggled: false }}
```



## State handling in ![](http://logos-download.com/wp-content/uploads/2016/09/React_logo_wordmark.png) <!-- .element: class="plain" style="height: 1em; margin: -10px 0px; background-color: transparent;" -->

* Components keep track of their own state
* OK for small and simple components
* State reset if component (un)mounted

```
class MyButton extends Component {
  state = { toggled: false }; // Initial state

  render = () =>
    <div>
      <Button
        onClick={() => this.setState({
          toggled: !this.state.toggled
        })}
      />
      <div>{ `Toggled: ${this.state.toggled}` }</div>
    </div>
}
```


* App grows, gets split into smaller parts
* Need to access state across components

```
class MyButton extends Component {
  state = { toggled: false }; // Initial state
  render = () =>
    <Button onClick={() => this.setState({ ... })} />
}

// Can't access state of MyButton instance!
class ButtonStateDisplay extends Component {
  render = () =>
    <div>{ `Toggled: ${???.toggled}` }</div>
}
```


One solution:
* Keep state in parent/container component
* Pass state and functions for updating it as props

```
class MyButton extends Component { ... }
class ButtonStateDisplay extends Component { ... }

class ButtonContainer extends Component {
  state = { toggled: 'no' }; // Initial state

  render = () =>
    <div>
      <MyButton doToggle={() => this.setState({ ... })} />
      <ButtonStateDisplay toggled={this.state.toggled} />
    </div>
}
```


Problems:
* Gets messy and verbose fast, boilerplate
* What if components aren't direct descendants:

```
<!-- Sample React component tree -->
<Container>
  <Toolbar>
    <MyButton />
  </Toolbar>
  <Contents>
    <ButtonStateDisplay />
  </Contents>
</Container>
```

→ Container ends up handling everything

Is there a better way?



## State handling with ![](https://raw.githubusercontent.com/reactjs/redux/master/logo/logo-title-light.png) <!-- .element: class="plain" style="height: 1.5em; margin: -30px -20px; background-color: transparent;" -->

In addition to using component state, our React components could:

* Read from Redux store state
* Dispatch actions to modify Redux store state


react-redux `connect()` function

* Easy React + Redux integration
* Usually passed two functions as parameters:
  * `mapStateToProps`
  * `mapDispatchToProps`

```
// MyButton.jsx

const mapStateToProps = state => ({ ... });
const mapDispatchToProps = dispatch => ({ ... });

@connect(mapStateToProps, mapDispatchToProps)
class MyButton extends Component { ... }
```


`mapStateToProps`

```
// "What does our component need to read from Redux?"

// MyButton.jsx
const mapStateToProps = state => ({
  toggled: state.button.toggled,
});

@connect(mapStateToProps)
class ButtonStateDisplay extends Component {
  render = () =>
    <div>{ `Toggled: ${this.props.toggled}` }</div>
}
```

* First parameter in `connect()`
* Receives entire Redux store state
* Keys in returned object passed as props


`mapDispatchToProps`

```
// "What does our component need to modify in Redux?"

// ButtonStateDisplay.jsx
const mapDispatchToProps = dispatch => ({
  doToggle: () => dispatch(pressButton()),
});

@connect(mapStateToProps, mapDispatchToProps)
class MyButton extends Component {
  render = () =>
    <Button onClick={this.props.doToggle} />
}
```

* Second parameter in `connect()`
* Receives `store.dispatch()` function
* Keys in returned object passed as props



## Extras: Redux Setup
```
import { createStore } from 'redux';
import { Provider } from 'react-redux';

// Redux setup stuff... (details later)
const store = createStore(...)

// Wrap your app in <Provider> to make it ready for Redux:
const Root = () =>
  <Provider store={store}>
    <YourApp />
  </Provider>
```
