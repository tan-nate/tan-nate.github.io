---
layout: post
title:      "Rendering changes in React with setState and componentDidUpdate"
date:       2020-03-05 18:21:53 +0000
permalink:  rendering_changes_in_react_with_setstate_and_componentdidupdate
---


Two concepts that I became much more familiar with through trial and error while working on my React project were setState and componentDidUpdate. 

One problem that I frequently encountered was that I sometimes had trouble triggering actions to occur at the right point in the component lifecycle. Often, I would update state in a component, and place an action in componentDidUpdate, hoping that the state change would be captured, and the correct action would be triggered. This didn't always work. 

I discovered that setState takes a callback function as its second argument from this [article](https://medium.learnreact.com/setstate-takes-a-callback-1f71ad5d2296). If, for whatever reason, you just can't get your action to catch, don't be afraid to use that callback. It will fire as soon as the state is updated. 

For most other updates to my components that I needed to occur after state or prop changes, I utilized the componentDidUpdate method. I soon discovered that componentDidUpdate takes two arguments: prevProps and prevState, in that order. You can set up conditions for prevProps or prevState in componentDidUpdate, such that certain actions only fire when those conditions are met. 
