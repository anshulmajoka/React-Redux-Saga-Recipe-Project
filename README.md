
# Getting Started with React Saga

 - React Saga
 - [Dispatching Action](#dispatching-an-action-to-saga)
<a name="top"></a>
## React Saga

Basically it is a middeware that allows to handle  a lot of api calls or Async operation in a very clean way. Initially there is a boilerplate code but once that is done it's very easy to write async operations such as API call.

Key Components of Saga
 - [Action](#action)
 - [Reducer](#reducer)
 - [Saga](#saga)
 - [Root Reducer](#rootreducer)
 - [Store](#store)
 - Api
 
### Action
It is a plain javascipt object that **describes an event** or an intention to change the state in your application.
**Redux-Saga listens for these actions** and performs side effects, such as asynchronous data fetching or handling complex business logic, based on the action types.
Actions typically have a `type` property that indicates the type of action being performed, and they may also include a `payload` property that carries additional data needed to process the action. 

```jsx harmony
actionsTypes.js
export const FETCH_RECIPE_START = "FETCH_RECIPE_START";
export const FETCH_RECIPE_SUCCESS = "FETCH_RECIPE_SUCCESS";
export const FETCH_RECIPE_FAIL = "FETCH_RECIPE_FAIL";
   ```

### Reducer
A reducer in the context of Redux (including when used with Redux-Saga) is a **pure function** that takes the **current state and an action as arguments** and **returns a new state**. Reducers specify how the application's state changes in response to actions sent to the store. Reducers are essential for managing state changes in a Redux-based application.

```jsx harmony
reducer.js
import * as types from "./actionTypes";

const initialState = {
  recipes: [],
  error: null,
  loading: false,
};

const recipeReducer = (state = initialState, action) => {
  switch (action.type) {
    case types.FETCH_RECIPE_START:
      return {
        ...state,
        loading: true,
      };
    case types.FETCH_RECIPE_SUCCESS:
      return {
        ...state,
        loading: false,
        recipes: action.payload,
      };
    case types.FETCH_RECIPE_FAIL:
      return {
        ...state,
        loading: false,
        error: action.payload,
      };
    default:
      return state;
  }
};

export default recipeReducer;
   ```
   [Back To Top](#getting-started-with-react-saga)
### Sagas
A saga is a generator function that handles side effects in your Redux application. Sagas listen for dispatched actions and perform asynchronous tasks such as data fetching, calling APIs, or executing complex business logic. They make it easier to manage side effects, especially asynchronous ones, in a predictable and organized manner.
-  (*) operator means it’s generator function
-   yield is simlar to ES7 async wait

```jsx harmony
a-simple-saga.js
import { call, put, takeEvery } from 'redux-saga/effects';
import axios from 'axios';

// Worker saga
function* fetchData(action) {
  try {
    const response = yield call(axios.get, `/api/data/${action.payload.id}`);
    yield put({ type: 'FETCH_DATA_SUCCESS', payload: response.data });
  } catch (e) {
    yield put({ type: 'FETCH_DATA_FAILURE', message: e.message });
  }
}

// Watcher saga
function* watchFetchData() {
  yield takeEvery('FETCH_DATA_REQUEST', fetchData);
}

export default watchFetchData;

   ```

```jsx harmony
sagas.js
import { takeLatest, all, put, fork, call } from "redux-saga/effects";
import * as types from "./actionTypes";
import { getRecipes } from "./api";

export function* onLoadRecipeAsync({ query }) {
  try {
    console.log("query", query);
    const response = yield call(getRecipes, query);
    yield put({ type: types.FETCH_RECIPE_SUCCESS, payload: response.data });
  } catch (error) {
    yield put({ type: types.FETCH_RECIPE_FAIL, payload: error });
  }
}

export function* onLoadRecipe() {
  yield takeLatest(types.FETCH_RECIPE_START, onLoadRecipeAsync);
}

const recipeSaga = [fork(onLoadRecipe)];

export default function* rootSaga() {
  yield all([...recipeSaga]);
}
   ```
   [Back To Top](#getting-started-with-react-saga)
### RootReducer
Root reducer is a single reducer function that combines all the individual reducers used in the application into one central reducer. This is typically done using the `combineReducers` function from Redux. The root reducer is then passed to the Redux store, enabling the store to manage the state of the entire application
```jsx harmony
import { combineReducers } from 'redux';
import userReducer from './userReducer';
import postsReducer from './postsReducer';

const rootReducer = combineReducers({
  user: userReducer,
  posts: postsReducer,
});

export default rootReducer;
```
```jsx harmony
import { combineReducers } from "redux";
import recipeReducer from "./reducer";

const rootReducer = combineReducers({
  data: recipeReducer,
});

export default rootReducer;
```
[Back To Top](#getting-started-with-react-saga)
### Store
A store is a centralized repository for the state of your application. It holds the entire state tree and allows for state management across different components. The store is created using Redux's `createStore` function and can be enhanced with middleware, such as Redux-Saga, to handle asynchronous actions and side effects.
In this example:

-   `rootReducer` combines all reducers.
-   `rootSaga` is the root saga that combines all sagas.
-   `sagaMiddleware` allows Redux-Saga to intercept actions and perform side effects.
```jsx harmony
a-simple-store.js
import { createStore, applyMiddleware } from 'redux';
import createSagaMiddleware from 'redux-saga';
import rootReducer from './reducers';
import rootSaga from './sagas';

const sagaMiddleware = createSagaMiddleware();

const store = createStore(
  rootReducer,
  applyMiddleware(sagaMiddleware)
);

sagaMiddleware.run(rootSaga);

export default store;
```

```jsx harmony
store.js
import { applyMiddleware, createStore } from "redux";
import logger from "redux-logger";
import createSagaMiddleware from "redux-saga";
import rootReducer from "./root-reducer";
import rootSaga from "./sagas";

const sagaMiddleWare = createSagaMiddleware();

const middleware = [sagaMiddleWare];

if (process.env.NODE_ENV === "development") {
  middleware.push(logger);
}

const store = createStore(rootReducer, applyMiddleware(...middleware));

sagaMiddleWare.run(rootSaga);

export default store;
```
[Back To Top](#getting-started-with-react-saga)
## Dispatching an action to Saga
To dispatch an action from a component in a Redux application, you typically use the `useDispatch` hook provided by React-Redux.
```jsx harmony
import React from 'react';
import { useDispatch } from 'react-redux';

const MyComponent = () => {
  const [query, setQuery] =  useState("chicken");
  const [search, setSearch] =  useState("")
  
  const { recipes } =  useSelector((state) =>  state.data);
  const updateSearch = () => {
    setQuery(search);
    setSearch("");
  };
  
  const dispatch = useDispatch();
  useEffect(() => {
    dispatch({ type: types.FETCH_RECIPE_START, query });
  }, 

  return (
   <form className={classes.root} noValidate autoComplete="off">
        <TextField
          id="outlined-basic"
          variant="outlined"
          type="text"
          value={search}
          onChange={(e) => setSearch(e.target.value)}
        />
        <Button
          variant="contained"
          color="primary"
          style={{ width: "80px", height: "50px" }}
          onClick={updateSearch}
        >
          Search
        </Button>
      </form>
  );
};

export default MyComponent;
```

[Back To Top](#getting-started-with-react-saga)
