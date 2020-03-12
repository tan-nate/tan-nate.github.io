---
layout: post
title:      "What are Redux and Thunk?"
date:       2020-03-12 22:28:43 +0000
permalink:  what_are_redux_and_thunk
---


During my project review for my final React project, it became clear that my understanding of Redux and Thunk are a bit foggy. I understood the basics: that Redux is a global data store. But the specifics of what an action is, what dispatch does, what reducers are, and what Thunk contributes, were all still foggy to me. This is an attempt to clarify that lack of understanding. 

Let's start by defining actions, state, and reducers. State stores our data, and actions modify state. Reducers are functions that take a given state and action as arguments, and modify the state according to the type of action. This is an example of a reducer:

```
function changeState(state, action){      
  switch (action.type) {
    case 'INCREASE_COUNT':
      return {count: state.count + 1}
    case 'DECREASE_COUNT':
      return {count: state.count - 1}
    default:
      return state;
  }
}
```

Depending of the type of action it is passed, this reducer will execute different modifications to the data store, or state.

Let's move on to the concept of pure functions and dispatch functions. If the function above is executed more than once, observe what happens:

```
let state = {count: 0}
 
changeState(state, {type: 'INCREASE_COUNT'})
// => {count: 1}

changeState(state, {type: 'INCREASE_COUNT'})
// => {count: 1}
```

The count is not continuing to increase. Why is that? `changeState` is continually referring to the original definition of `state`: `let state = {count: 0}`. It never reassigns that `state` variable. That's because `changeState` is a pure function, meaning it does not reassign any variable that is defined outside the function. Let's say we were to write `changeState` like this:

```
function changeState(state, action){      
  switch (action.type) {
    case 'INCREASE_COUNT':
      state.count = state.count + 1;
			return state;
    case 'DECREASE_COUNT':
      state.count = state.count - 1;
			return state;
  }
}

let state = {count: 0}
 
changeState(state, {type: 'INCREASE_COUNT'})
// => {count: 1}

changeState(state, {type: 'INCREASE_COUNT'})
// => {count: 2}
```
Now, when `changeState` is continually executed, the count is actually increasing. However, this is no longer a pure function, because the function reassigns `state`, which is declared outside the function. 

Why should the reducer be a pure function? It is a core part of the [Redux philosophy](https://redux.js.org/glossary#reducer), which to be honest is a bit over my head. But my shallow understanding is that since actions do not reassign the state variable, they can be safely used and interchanged without affecting other parts of the application. Also, it allows us to isolate state reassignment in a single dispatch function, which I'll describe in a second. 

So how can we solve this problem, allowing our reducer to remain 'pure', while also allowing it to continually update as expected, given the same action? The answer is to write a dispatch function:

```
function dispatch(action){
  state = changeState(state, action)
  return state
}

dispatch({type: 'INCREASE_COUNT'})
  // => {count: 1}
dispatch({type: 'INCREASE_COUNT'})
  // => {count: 2}
dispatch({type: 'INCREASE_COUNT'})
  // => {count: 3}
```

Now, in order to update the state object, the dispatch function must be called. So the dispatch function is not pure, but it allows the state variable to be reassigned only within this single function, rather than in multiple actions. This also provides us with an ideal place to insert our re-render function, since whenever state is updated, the dispatch function must be called:

```
function render(){
  document.body.textContent = state.count
}
 
function dispatch(action){
  state = changeState(state, action)
  render()
}
```
Let's move on to Thunk. What is Thunk and why do we need it? Let's say we defined the following action:
```
function fetchAstronauts() {
  const astronauts = fetch('http://api.open-notify.org/astros.json');
  return {
    type: 'ADD_ASTRONAUTS',
    astronauts
  };
};

dispatch(fetchAstronauts());
```
If we ran the above code, we would run into a problem, because `fetch` runs asynchronously, and `dispatch` would run before the Promise from `fetch` is received. Thunk solves this problem by allowing the action passed to `dispatch` to be a function, rather than simply an object. Therefore, we could write:
```
function fetchAstronauts() {
  return (dispatch) => {
    fetch('http://api.open-notify.org/astros.json')
      .then(response => response.json())
      .then(astronauts => dispatch({ type: 'ADD_ASTRONAUTS', astronauts }));
  };
}
```
Now, if we ran `dispatch(fetchAstronauts())`, `dispatch({ type: 'ADD_ASTRONAUTS', astronauts })` would not run until the Promise is resolved. 
