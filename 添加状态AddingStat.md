  #  Adding the State 

We want to make the Knight draggable.   
It's a noble goal, but we need to see past it.   
What we really mean is that we want to keep the current knightPosition in some kind of state storage, and have some way to change it.

Because setting up this state requires some thought, we won't try to implement dragging at the same time.  
Instead, we'll start with a simpler implementation.   
We will move the Knight when you click a particular Square, but only if this is allowed by the Chess rules.   
Implementing this logic should give us enough insight into managing the state, so we can replace clicking with the drag and drop once we've dealt with that.

React is not opinionated about the state management or the data flow;  
you can use Flux, Redux, Rx or even Backbone nah, avoid fat models and separate your reads from writes.

I don't want to bother with installing or setting up Redux for this simple example, so I'm going to follow a simpler pattern.  
It won't scale as well as Redux, but I also don't need it to.   
I have not decided on the API for my state manager yet, but I'm going to call it Game, and it will definitely need to have some way of signaling data changes to my React code.

Since I know this much, I can rewrite my index.js with a hypothetical Game that doesn't exist yet.   
Note that this time, I'm writing my code in blind, not being able to run it yet. This is because I'm still figuring out the API:
