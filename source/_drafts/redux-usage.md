---
title: redux-usage
tags:
---

# Introduction

## Motivation

- state manage
- mutation and asynchronicity

## Core Concepts

- state: model without setters
- action: change something in state (what happend)
- reducer: to tie state and action together `(state, action) => state` (how state change)
- split state and reducer to small part, and then combine them together.

## Three Principles

- Single source of truth
  - eaiser to debug and inspect
  - persist app's state
- State is read-only
  - any view or network callbacks need make a intent to change state
  - changes are cenralized and one by one in a strict order(no race conditions)
  - actions can be logged, serialized, stored, and replayed
- Changes are made with pure functions

## Prior Art

- Flux
- Elm
- Immutable
- Baobab
- RxJS

# Basic

## Actions

- payloads of information that send to app store
- the only source of information for the store
- actions are plain JavaScript objects that must have a `type` property
- *action creators*
  - we do not dispatch when invoked
  - bound action creator that automatically dispatches
  - access `dispatch` by `store.dispatch` or `react-redux`'s `connect()`s
  - `bindActionCreators(Function | Object, Function) => Function | Object `
  - can be async and have side-effect

## Reducers

- state shape: data stae + UI state
- keep state as normalized as possible, without any nesting.
- `todosById: { id -> todo }` and `todos: array<id>`
- pure function: `(previousState, action) => newState`, no pure operations:
  - Mutate its argmuents
  - side effect
  - non-pure function, like `Date.now()` or `Math.random()`
- do not mutate the state, use `Object.assign({}, ...)` or ES6 `object spread operator`
- return previous `state` in the `default` case
- splitting reducers:
  - reducer composition: `combineReducers()`

## Store

- holds application state
- allows access to state via `getState()`
- allows state to be updated via `dispatch(action)`
- registers listeners via `subscribe(listener)`
- handles unregistering of listeners via the function returned by `subscribe(listener)`

## DataFlow

- strict unidirectional data flow


## Usage with React

- presentational and container components

|   | Presentational Components | Container Components |
|---|---|---|
| Purpose  | How things look (markup, styles)  | How things work()data fetching, state updates  |
| Aware of Redux  | No  | Yes  |
| To read data  | Read data from props  | Subscribe to Redux state  |
| To change data  | Invoke callbacks from props  | Dispatch Redux actions  |
| Are written  | By hand  | Usually generated by React Redux  |

- `connect({mapStateToProps, mapDispatchToProps})`
  - `<Provider>`
  - `context`

## Async Actions

- need middleware to deal async actions
- default dispatch can only deal plain object actions.
- middleware wrapper store's `dispatch()` make it able to deal with async actions
- async actions creators
  - redux-thunk: dispatch function action
  - redux-promise: dispatch promise action

## Middleware

- compose `dispatch`
- action -- dispatch (middleware1 --> middleware2 ... --> middlewaren) --> reducer