## Store [Back](./../react_redux.md)

The **Store** is the object that bring both **Actions** and **Reducers** together.

Store will be responsible for:

- holding application state
- exposing interface of accessing to state: `getState()`
- exposing interface for getting action dispatching: `dispatch(action)`
- exposing interface for registering listener on the dispatching event: `subscribe(listener)`
- handling unregistering of listeners.

### Create a store

Remember that there is **only one single store** in a Redux application. To create a producer, what you need is reducer.

