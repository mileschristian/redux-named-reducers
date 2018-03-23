# redux-named-reducers

[![npm](https://img.shields.io/npm/v/redux-named-reducers.svg?style=flat-square)](https://www.npmjs.com/package/redux-named-reducers)

Access redux state anywhere in your code through reducers, and optionally split your reducers into smaller 'reduce' functions that operate per action

## Usage

`npm install redux-named-reducers --save`

Create a named reducer

```js
import { createNamedReducer } from "redux-named-reducers";

//define all the state you will access in initial state
const initialState = {
  state1: "default1",
  state2: "default2"
};

function reducer(state = initialState, action) {
  //reducer logic goes here
}

export const moduleA = createNamedReducer({
  moduleName: "moduleA", //optional module name
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
  const moduleName = moduleA.moduleName;
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

## Alternate method of declaring reducers

Create a named reducer passing an initial state option

```js
import { createNamedReducer } from "redux-named-reducers";

export const moduleA = createNamedReducer({
  initialState: initialState
});
```

Create a reducer for each action or group of actions using 'reduce' function

```js
//notice no need for '...state' just return the states which changed
moduleA.reduce(actions.ACTION1, action => {
  state1: action.param1,
  state2: action.param2
})

//if you are just changing state to a constant you can return an object directly
moduleA.reduce(actions.ACTION2, { state1: "nextState"} )

//multiple actions supported
moduleA.reduce([actions.ACTION3, actions.ACTION4], { state2: "nextState"} )

//...as well as 'redux-actions' like action creators that have a toString() method
moduleA.reduce(myActionCreator1, { state1: "nextState"} )

//'state' is not given because you can access it using the getState() method
moduleA.reduce(toggleState1, () => ({
  state1: !getState(moduleA.state1)
})
```

## Usage with combineReducers

Just pass the root reducer to the enhancer

```js
import { combineReducers, createStore, compose } from "redux";
import { namedReducerEnhancer } from "redux-named-reducers";
import { moduleA } from "./moduleA";
import { moduleB } from "./moduleB";

const rootReducer = combineReducers({
  moduleA,
  moduleB
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
(Note: Since version v1.0.7 you must use linkState() to link states, old method is no longer supported)

```js
import { linkState } from "redux-named-reducers";

linkState(moduleA.extState1, moduleB.state1);
linkState(moduleA.extState2, moduleC.state1);
```

You can then access the external state from the local module

```js
const extState1 = getState(moduleA.extState1);
const extState2 = getState(moduleA.extState2);
```

## Usage with reselect

Each named reducer state is just a selector so you can use it in reselect (since version 1.0.7 only)

```js
import { createSelector } from 'reselect';
import { moduleA } from "./moduleA";
import { moduleB } from "./moduleB";

const mySelector = createSelector(
  [ moduleA.state1, moduleA.extState1, moduleB.state2 ],
  (state1, extState1, state2) => {
    //selector logic goes here  
  }  
```

You can then use the selector as normal, or if you link it to your state then you can access it anywhere in your code

```js
linkState(moduleA.extState1, mySelector);

const derivedState = getState(moduleA.extState1);
```

## Notes

getState() currently works for first level state properties only. To access nested state you must do:

```js
getState(moduleA.state1).subState1;
```
