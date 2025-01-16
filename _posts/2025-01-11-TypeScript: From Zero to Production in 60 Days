---
title: "TypeScript: From Zero to Production in 60 Days"
date: 2025-01-11 08:42:00 -500
categories: [ Bending Spoons, TypeScript]
tags: [production, RxJS, Redux, my journey]
image: "https://raw.githubusercontent.com/cristinaponcela/cristinaponcela.github.io/refs/heads/main/assets/img/typescript-journey.jpg"
---


## Learning TypeScript in 2 Months: What Worked for Me

Learning TypeScript has been an interesting experience (read "b*"). It's a superset of JavaScript, so previous experience with it helped the transition be smoother, but there are still a lot of aspects that, once learned, prove to be very advantageous above JS. This isn’t a beginner’s guide, but rather a collection of tips, observations, and examples that I found interesting while learning TypeScript, and may resonate with you if you have tamed this beast too.


### Promises and Async/Await

Ah yes, the bane of a developer's existence, asynchronous logic. Especially when working with Redux, which gets very angry when it has to handle anything other than a synchronous flow of actions (insert: middleware). So one of the first things I revisited was awaits, and the cool datatype that TypeScript uses for the result: Promises.

A Promise in JavaScript is an object that acts as a placeholder for data that isn’t immediately available. Promises don’t actually pause the execution of your code; instead, they represent a value that will be resolved (or rejected) in the future. 

Note that await is used to make asynchronous code behave synchronously.

Here’s a simple example:

```typescript
async function fetchDataWithAsyncAwait(): Promise<string> {
  const data =  new Promise<string>((resolve, reject) => { // or await Promise<string> to wait for resolution
    setTimeout(() => {
      resolve("Data fetched successfully");
    }, 2000);
  });
  return data;
}

fetchDataWithAsyncAwait()
  .then((data) => console.log(data))
  .catch((error) => console.error(error));
```


### TypeScript’s Strictness (or lack thereof) and Type Inference

TypeScript is famous for being a strictly typed language, but newer versions have introduced what, for me, is its crown jewel: powerful type inference. It makes the TS experience much more developer-friendly. 

This means that you don’t always need to explicitly annotate types - you can get away with types `any` or `unkown` as long as the flow is clear enough for TypeScript to infer them.

Introduce wacky behaviour: even when your IDE shows a compilation error due to lack of typing, the code may still run successfully, provided the inferred types match the ones passed (scary consequences if not... crash alert!).

Here’s an interesting case:

#### Example 1: Typing Error

This code would flag as an error in your IDE and show a compilation error:

```typescript
import { combineEpics } from "redux-observable";
import fetchCategoriesEpic from "./services/categories/categories.epics";
import { fetchCoursesEpic } from "./services/courses/courses.epics";
import fetchReviewsEpic from "./scenes/CoursePage/services/reviews/reviews.epics";
import debounceSearchEpic from "./services/debounceSearch/debounceSearch.epics";

const rootEpic = combineEpics(
  fetchCategoriesEpic,
  fetchCoursesEpic,
  fetchReviewsEpic,
  debounceSearchEpic
);

export default rootEpic;
```

In case you're interested, the error would look something like:

```
Argument of type 'Epic<FetchCoursesAction, FetchCoursesAction, { route: RouteState; currentCategory: CurrentCategoryState; ... 4 more ...; debounceSearch: DebounceSearchState<...>; }, any>' is not assignable to parameter of type 'Epic<FetchCategoriesAction, FetchCategoriesAction, { route: RouteState; currentCategory: CurrentCategoryState; ... 4 more ...; debounceSearch: DebounceSearchState<...>; }, any>'.
  Type 'FetchCategoriesAction' is not assignable to type 'FetchCoursesAction'.
    Type 'FetchCategoriesRequestAction' is not assignable to type 'FetchCoursesAction'.
      Type 'FetchCategoriesRequestAction' is not assignable to type 'FetchCoursesFailureAction'.
        Types of property 'type' are incompatible.
          Type '"FETCH_CATEGORIES_REQUEST"' is not assignable to type '"FETCH_COURSES_FAILURE"'.ts(2345)

```

Even though TypeScript flags an error at compile time (because `rootEpic` is not typed), the code still runs in JavaScript after transpilation. This is a reminder that TypeScript adds a layer of safety but doesn’t enforce runtime checks unless explicitly coded.


#### Example 2: Hack

Now at this point, if I didn't work in a company that highly values code health and best practices, I would be inclined to use a hack and ignore this. Let TypeScript do the job, I'm lazy and I didn't expect the output to behave weirdly so inference should be right:

```typescript
import { combineEpics, Epic } from "redux-observable";
import fetchCategoriesEpic from "./services/categories/categories.epics";
import { fetchCoursesEpic } from "./services/courses/courses.epics";
import fetchReviewsEpic from "./scenes/CoursePage/services/reviews/reviews.epics";
import debounceSearchEpic from "./services/debounceSearch/debounceSearch.epics";

const rootEpic = combineEpics(
  fetchCategoriesEpic,
  fetchCoursesEpic,
  fetchReviewsEpic,
  debounceSearchEpic
);

export default rootEpic as Epic<any, any, any, any>;
```

Note that at this point, ESLint would peek its head out of the corner and stare at you in disappointment. Womp womp.

```bash
Unexpected any. Specify a different type.eslint@typescript-eslint/no-explicit-any
```

And if your linting config doesn't allow type `any` (which I recommend you set to be strict), it would directly throw an error (say goodbye to merging that PR).


#### Example 3: Correct

So as much as TypreScript is annoying because it makes you work a bit harder, it is ultimately much safer. Plus, if you can't type your functions properly, perhaps you should rethink what you're doing and make sure you understand the logic and flow.

Correct example in this case would be:

```typescript
import { combineEpics, Epic } from "redux-observable";
import fetchCategoriesEpic from "./services/categories/categories.epics";
import { fetchCoursesEpic } from "./services/courses/courses.epics";
import fetchReviewsEpic from "./scenes/CoursePage/services/reviews/reviews.epics";
import debounceSearchEpic from "./services/debounceSearch/debounceSearch.epics";
import { RootState } from "./rootReducer";
import { FetchCategoriesAction } from "./services/categories/categories.ducks";
import { FetchCoursesAction } from "./services/courses/courses.ducks";
import { FetchReviewsAction } from "./scenes/CoursePage/services/reviews/reviews.ducks";
import { DebounceSearchActions } from "./services/debounceSearch/debounceSearch.ducks";

export type RootAction =
  | FetchCategoriesAction
  | FetchCoursesAction
  | FetchReviewsAction
  | DebounceSearchActions;

export type RootEpic = Epic<RootAction, RootAction, RootState | void>;

const rootEpic: RootEpic = combineEpics(
  fetchCategoriesEpic as RootEpic,
  fetchCoursesEpic as RootEpic,
  fetchReviewsEpic as RootEpic,
  debounceSearchEpic as RootEpic
);

export default rootEpic;
```

Plus, the syntax to correctly type functions looks neat, no? Like:

```typescript
import { createStore, applyMiddleware } from "redux";
import { createEpicMiddleware, EpicMiddleware } from "redux-observable";
import rootReducer, { RootState } from "./rootReducer";
import rootEpic, { RootEpic } from "./root.epics";
import { RootAction } from "./root.epics";

// const epicMiddleware = createEpicMiddleware();
// becomes:
const epicMiddleware: EpicMiddleware<RootAction, RootAction, RootState> =
  createEpicMiddleware<RootAction, RootAction, RootState>();

const persistedState =
  typeof window !== "undefined" && window.localStorage
    ? JSON.parse(localStorage.getItem("reduxState") || "{}")
    : {};

const store = createStore(
  rootReducer,
  persistedState,
  applyMiddleware(epicMiddleware)
);

// epicMiddleware.run(rootEpic);
// becomes:
epicMiddleware.run(rootEpic as RootEpic);

store.subscribe(() => {
  if (typeof window !== "undefined" && window.localStorage) {
    localStorage.setItem("reduxState", JSON.stringify(store.getState()));
  }
});

export type AppDispatch = typeof store.dispatch;
export type { RootState };
export default store;
```

Now I'm just dropping Redux snippets because I can. This took me too much effort to learn not to. Also yes, I'm Classic Redux instead of Toolkit as a choice. I hate it too, thanks.


### Integrating TypeScript with React, Redux, and RxJS

TypeScript ✨shines✨ when working with React, Redux, and RxJS. It makes large-scale applications more manageable by catching bugs early and providing excellent developer tooling.

#### React Advantages
React’s component-based architecture is great for building reusable and testable UI components. TypeScript enhances this by:
- Adding strict typing to props and state, reducing the likelihood of passing incorrect data.
- Offering better IDE support with autocompletion and inline documentation.

Example of a typed React functional component:

```typescript
import React from "react";

type ButtonProps = {
  label: string;
  onClick: () => void;
};

const Button: React.FC<ButtonProps> = ({ label, onClick }) => {
  return <button onClick={onClick}>{label}</button>;
};

export default Button;
```

#### Redux Advantages
Redux provides a predictable state container for managing application state. TypeScript ensures that actions, reducers, and state structures are consistent, which is crucial for large applications.

Here’s an example of a Redux slice using TypeScript:

```typescript
// Action Types
const INCREMENT = "INCREMENT";
const DECREMENT = "DECREMENT";
const SET = "SET";

// Action Creators
export const increment = () => ({
  type: INCREMENT,
});

export const decrement = () => ({
  type: DECREMENT,
});

export const set = (value: number) => ({
  type: SET,
  payload: value,
});

// Initial State
type CounterState = {
  value: number;
};

const initialState: CounterState = { value: 0 };

// Reducer
const counterReducer = (state = initialState, action: { type: string; payload?: number }): CounterState => {
  switch (action.type) {
    case INCREMENT:
      return { ...state, value: state.value + 1 };
    case DECREMENT:
      return { ...state, value: state.value - 1 };
    case SET:
      return { ...state, value: action.payload ?? 0 };
    default:
      return state;
  }
};

export default counterReducer;
```

And because I miss Toolkit:

```typescript
import { createSlice, PayloadAction } from "@reduxjs/toolkit";

type CounterState = {
  value: number;
};

const initialState: CounterState = { value: 0 };

const counterSlice = createSlice({
  name: "counter",
  initialState,
  reducers: {
    increment(state) {
      state.value += 1;
    },
    decrement(state) {
      state.value -= 1;
    },
    set(state, action: PayloadAction<number>) {
      state.value = action.payload;
    },
  },
});

export const { increment, decrement, set } = counterSlice.actions;
export default counterSlice.reducer;
```

#### RxJS Advantages
RxJS simplifies asynchronous programming by using observable streams. This is particularly useful for handling asynchronous logic, API calls, and most importantly: *cancellations*. I'll let the guy who created t [explain this one](https://www.youtube.com/watch?v=AslncyG8whg) better than me: bonus points for Neflix. What a talk.

#### Example: Ducks and Epics
If you’re using Redux and epics with RxJS, I highly recommend you use the ducks pattern (beautifully typed by TypeScript, of course).

Here’s a quick example using RxJS for debouncing an action:

```typescript
import { ofType } from "redux-observable";
import { debounceTime, map } from "rxjs/operators";
import { Epic } from "redux-observable";
import { Action } from "redux";

// Action Types
const SEARCH = "SEARCH";
const SEARCH_SUCCESS = "SEARCH_SUCCESS";

// Action Creators
export const search = (query: string) => ({ type: SEARCH, payload: query });
export const searchSuccess = (results: any[]) => ({ type: SEARCH_SUCCESS, payload: results });

// Epic
export const searchEpic: Epic<Action> = (action$) =>
  action$.pipe(
    ofType(SEARCH),
    debounceTime(300), // Debounce API calls by 300ms
    map((action: any) => searchSuccess([`Results for: ${action.payload}`]))
  );
```

See that?! One line of code! One. Line. Of. Code.

Gets better for cancellations, but I'll let you play around to figure that one out.


### Final Thoughts

As always, the key to being a good developer is knowing which tools you can use to be more efficient. I have become quite fond of RxJS. Of course, it is quite challenging to learn, and only kind of necessary for me right now because I'm working on a streaming service (cancellation in one line sounds quite juicy right about now). But still, TypeScript is great because you can pair it up with great libraries, and let the harder syntax and logic to the poor open source creators.

