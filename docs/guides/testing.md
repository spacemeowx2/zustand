---
title: Testing
description: How to test your new store
nav: 9
---

## Resetting state between tests in **react-dom**

When running tests, the stores are not automatically reset before each test run.

Thus, there can be cases where the state of one test can affect another. To make sure all tests run
with a pristine store state, you can mock `zustand` during testing and use the following code to
create your store:

```js
import { create as actualCreate } from 'zustand'
// const { create: actualCreate } = jest.requireActual('zustand') // if using jest
import { act } from 'react-dom/test-utils'

// a variable to hold reset functions for all stores declared in the app
const storeResetFns = new Set()

// when creating a store, we get its initial state, create a reset function and add it in the set
export const create = (createState) => {
  const store = actualCreate(createState)
  const initialState = store.getState()
  storeResetFns.add(() => store.setState(initialState, true))
  return store
}

// Reset all stores after each test run
beforeEach(() => {
  act(() => storeResetFns.forEach((resetFn) => resetFn()))
})
```

The way you mock a dependency depends on your test runner/library.

In [jest](https://jestjs.io/), you can create a `__mocks__/zustand.js` and place the code in that
file. If your app is using `zustand/vanilla` instead of `zustand`, then you'll have to place the
above code in `__mocks__/zustand/vanilla.js`.

### TypeScript usage

If you are using zustand, as documented in [TypeScript Guide](./typescript.md), use the following
code:

```ts
import { create as actualCreate, StateCreator } from 'zustand'
// if using Jest:
// import { StateCreator } from 'zustand';
// const { create: actualCreate } = jest.requireActual<typeof import('zustand')>('zustand');
import { act } from 'react-dom/test-utils'

// a variable to hold reset functions for all stores declared in the app
const storeResetFns = new Set<() => void>()

// when creating a store, we get its initial state, create a reset function and add it in the set
const create =
  () =>
  <S>(createState: StateCreator<S>) => {
    const store = actualCreate(createState)
    const initialState = store.getState()
    storeResetFns.add(() => store.setState(initialState, true))
    return store
  }

// Reset all stores after each test run
beforeEach(() => {
  act(() => storeResetFns.forEach((resetFn) => resetFn()))
})
```

## Resetting state between tests in **react-native** and **jest**

You should use the following code in the `__mocks__/zustand.js` file (the `__mocks__` directory
should be adjacent to node_modules, placed in the same folder as node_modules, unless you
configured roots to point to a folder other than the project root [jest docs: mocking node modules](https://jestjs.io/docs/manual-mocks#mocking-node-modules)):

```js
import { act } from '@testing-library/react-native'
const { create: actualCreate } = jest.requireActual('zustand')

// a variable to hold reset functions for all stores declared in the app
const storeResetFns = new Set()

// when creating a store, we get its initial state, create a reset function and add it in the set
export const create = (createState) => {
  const store = actualCreate(createState)
  const initialState = store.getState()
  storeResetFns.add(() => store.setState(initialState, true))
  return store
}

// Reset all stores after each test run
beforeEach(() => {
  act(() => storeResetFns.forEach((resetFn) => resetFn()))
})
```

If the `jest.config.js` has `automock: false`, then you need to do the following in `jest.setup.js`:

```js
jest.mock('zustand')
```
