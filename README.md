# redux-named-reducers

Access redux state anywhere in your code through reducers

## Usage

`npm install redux-named-reducers --save`

Create a named reducer

```js
import { createNamedReducer } from "redux-named-reducers";

const initialState = {
  state1: "",
  state2: ""
};

function reducer(state = initialState, action) {
  //reducer logic goes here
}

export const moduleA = createNamedReducer({
  moduleName: "moduleA",
  reducer: reducer
});
```

Add the enhancer

```js
import { createStore, compose } from "redux";
import { namedReducerEnhancer } from "redux-named-reducers";
import { moduleA } from "./moduleA";

const store = createStore(moduleA, compose(namedReducerEnhancer(moduleA), ...otherEnhancersOrMiddleware));
```

Access the state from anywhere in your code

```js
import { getState } from "redux-named-reducers";
import { moduleA } from "./moduleA";

function someFunction() {
  const state1 = getState(moduleA.state1);
  const state2 = getState(moduleA.state2);
}
```

Use it to directly access state in mapDispatchToProps()

```js
const mapDispatchToProps = dispatch => {
  return {
    onClick: () => {
      dispatch(someAction(getState(moduleA.state1)));
    }
  };
};
```

## Usage with combineReducers

Just pass the root reducer to the enhancer

```js
import { combineReducers, createStore, compose } from "redux";
import { namedReducerEnhancer } from "redux-named-reducers";
import { moduleA } from "./moduleA";
import { moduleB } from "./moduleB";

const rootReducer = combineReducers({
  [moduleA.moduleName]: moduleA,
  [moduleB.moduleName]: moduleB
});

const store = createStore(rootReducer, compose(namedReducerEnhancer(rootReducer), ...otherEnhancersOrMiddleware));
```

## Access external state from other reducers

Add an externalState option with a list of external state with default values

```js
export const moduleA = createNamedReducer({
  moduleName: "moduleA",
  reducer: reducer,
  externalState: { extState1: null, extState2: "" }
});
```

In the main app link the external state to other modules

```js
moduleA.extState1 = moduleB.state1;
moduleA.extState2 = moduleC.state1;
```

You can then access the external state from the local module

```js
const extState1 = getState(moduleA.extState1);
const extState2 = getState(moduleA.extState2);
```

## Usage with reselect

Each accessible state is just a selector so you can use it in reselect (since version 1.0.6 only)

```js
import { createSelector } from 'reselect';
import { moduleA } from "./moduleA";

const mySelector = createSelector(
  [ moduleA.state1, moduleA.state2 ],
  (state1, state2) => {
    //selector logic goes here  
  }
```

## Notes

getState() currently works for first level state properties only. To access nested state you must do:

```js
getState(moduleA.state1).subState1;
```
