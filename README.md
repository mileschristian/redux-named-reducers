# redux-named-reducers
Access redux state anywhere in your code through reducers

## Usage

`npm install redux-named-reducers --save`

```js
import { createNamedReducer } from 'redux-named-reducers'

const initialState = {
  state1: '',
  state2: ''
}

function reducer(state = initialState, action) {
  //reducer logic goes here
}

export const moduleA = createNamedReducer({
  moduleName: 'moduleA',
  reducer: reducer
})
```

Add the enhancer

```js
import { namedReducerEnhancer } from 'redux-named-reducers'
import { moduleA } from './moduleA'

const store = createStore(moduleA, composeEnhancers(namedReducerEnhancer(moduleA), ...otherEnhancers))
```

Access the state from anywhere in your code

```js
import { getState } from 'redux-named-reducers'
import { moduleA } from './moduleA'

const state1 = getState(moduleA.state1)
const state2 = getState(moduleA.state2)
```
