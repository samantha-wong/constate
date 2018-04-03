<p align="center">
  <img src="logo/logo.svg" alt="constate logo" width="300" />
</p>
<br /><br />

<p align="center">
  <a href="https://github.com/diegohaz/nod"><img alt="Generated with nod" src="https://img.shields.io/badge/generator-nod-2196F3.svg?style=flat-square" /></a>
  <a href="https://npmjs.org/package/constate"><img alt="NPM version" src="https://img.shields.io/npm/v/constate.svg?style=flat-square" /></a>
  <a href="https://travis-ci.org/diegohaz/constate"><img alt="Build Status" src="https://img.shields.io/travis/diegohaz/constate/master.svg?style=flat-square" /></a>
  <a href="https://codecov.io/gh/diegohaz/constate/branch/master"><img alt="Coverage Status" src="https://img.shields.io/codecov/c/github/diegohaz/constate/master.svg?style=flat-square" /></a>
</p>
<br /><br />

> context + state = constate

~2KB React state management library that lets you work with local state and scale up to global state with ease when needed. 

Demo: https://codesandbox.io/s/7p2qv6mmq

## Install

```sh
npm i -S constate
```

## Usage

**Table of Contents**:
  - [Local state](#local-state)
  - [Global state](#global-state)
  - [Composing state](#composing-state)
  - [Testing](#testing)

### Local state

You can start by creating your `State` component:
```jsx
import React from "react";
import { State } from "constate";

export const initialState = {
  count: 0
};

export const actions = {
  increment: amount => state => ({ count: state.count + amount })
};

export const selectors = {
  getParity: () => state => (state.count % 2 === 0 ? "even" : "odd")
};

const CounterState = props => (
  <State
    initialState={initialState}
    actions={actiosn}
    selectors={selectors}
    {...props}
  />
);

export default CounterState;
```
> Note: the reason we're exporting `initialState`, `actions` and `selectors` is to make [testing](#testing) easier.

Then, just use it elsewhere:
```jsx
const CounterButton = () => (
  <CounterState>
    {({ count, increment, getParity }) => (
      <button onClick={() => increment(1)}>{count} {getParity()}</button>
    )}
  </CounterState>
);
```

### Global state

Whenever you need to share state between components and/or feel the need to have a global state, you can pass a `context` property to `State` and wrap your app with `Provider`:
```jsx
const CounterButton = () => (
  <CounterState context="foo">
    {({ increment }) => <button onClick={() => increment(1)}>Increment</button>}
  </CounterState>
);

const CounterValue = () => (
  <CounterState context="foo">
    {({ count }) => <div>{count}</div>} 
  </CounterState>
);

const App = () => (
  <Provider>
    <CounterButton />
    <CounterValue />
  </Provider>
);
```

### Composing state

This is still React, so you can pass new properties to `CounterState`, making it really composable.

First, let's change our `CounterState` so as to receive new properties:

```jsx
const CounterState = props => (
  <State
    {...props}
    initialState={{ ...initialState, ...props.initialState }}
    actions={{ ...actions, ...props.actions }}
    selectors={{ ...selectors, ...props.selectors }}
  />
);
```

Now we can pass new `initialState`, `actions` and `selectors` to `CounterState`:

```jsx
export const initialState = {
  count: 10
};

export const actions = {
  decrement: amount => state => ({ count: state.count - amount })
};

const CounterButton = () => (
  <CounterState initialState={initialState} actions={actions}>
    {({ decrement }) => <button onClick={() => decrement(1)}>Decrement</button>}
  </CounterState>
);
```

Those new members will work even if you use `context`.

### Testing

`actions` and `selectors` are pure functions. Testing is pretty straightfoward:

```js
import { initialState, actions, selectors } from "./CounterState";

test("initialState", () => {
  expect(initialState).toEqual({ count: 0 });
});

test("actions", () => {
  expect(actions.increment(1)({ count: 0 })).toEqual({ count: 1 });
  expect(actions.increment(-1)({ count: 1 })).toEqual({ count: 0 });
});

test("selectors", () => {
  expect(selectors.getParity()({ count: 0 })).toBe("even");
  expect(selectors.getParity()({ count: 1 })).toBe("odd");
});
```

## License

MIT © [Diego Haz](https://github.com/diegohaz)
