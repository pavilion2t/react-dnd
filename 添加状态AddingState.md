  #  Adding the State 

We want to make the Knight draggable可拖拽的.   
It's a noble goal, but we need to see past it.   
What we really mean is that we want to keep the current knightPosition in some kind of state storage存储, and have some way to change it.

Because setting up this state requires some thought, we won't try to implement实现 dragging at the same time.  
Instead, we'll start with a simpler implementation.   
We will move the Knight when you click a particular Square, but only if this is allowed by the Chess rules.   
Implementing this logic should give us enough insight洞察力 into managing the state, so we can replace clicking with the drag and drop once we've dealt with that.

React is not opinionated 固执己见的 about the state management or the data flow;  
you can use Flux, Redux, Rx or even Backbone nah, avoid fat models and separate your reads from writes.

I don't want to bother with installing or setting up Redux for this simple example, so I'm going to follow a simpler pattern.  
It won't scale as well as Redux, but I also don't need it to.   
I have not decided on the API for my state manager yet, but I'm going to call it Game, and it will definitely need to have some way of signaling data changes to my React code.

Since I know this much, I can rewrite my index.js with a hypothetical 假设的 Game that doesn't exist yet.   
Note that this time, I'm writing my code in blind, not being able to run it yet. This is because I'm still figuring out the API:

```
import React from 'react';
import ReactDOM from 'react-dom';
import Board from './Board';
import { observe } from './Game';

const rootEl = document.getElementById('root');

observe(knightPosition =>
  ReactDOM.render(
    <Board knightPosition={knightPosition} />,
    rootEl
  )
);
```

What is this observe function I import?   
It's just the most minimal最低的 way I can think of to subscribe to a changing state.   
I could've made it an EventEmitter but why on Earth even go there when all I need is a single change event?   
I could have made Game an object model, but why do that, when all I need is a stream of values?

Just to verify 核实 that this subscription API makes some sense, I'm going to write a fake Game that emits random positions:  
```
export function observe(receive) {
  setInterval(() => receive([
    Math.floor(Math.random() * 8),
    Math.floor(Math.random() * 8)
  ]), 500);
}
```

Nothing feels as good as being back into the rendering game!

This is obviously not very useful.   
If we want some interactivity 交互性, we're going to need a way to modify 修改 the Game state from our components.   
For now, I'm going to keep it simple and expose a moveKnight function that directly modifies the internal state.   
This is not going to fare 经营 well in a moderately complex app where different state storages may be interested in updating their state in response to a single user action, but in our case this will suffice 足够:  

```
let knightPosition = [0, 0];
let observer = null;

function emitChange() {
  observer(knightPosition);
}

export function observe(o) {
  if (observer) {
    throw new Error('Multiple observers not implemented.');
  }

  observer = o;
  emitChange();
}

export function moveKnight(toX, toY) {
  knightPosition = [toX, toY];
  emitChange();
}
```

Now, let's go back to our components.   
Our goal at this point is to move the Knight to a Square that was clicked. One way to do that is to call moveKnight from the Square itself.   
However, this would require us to pass the Square its position. Here is a good rule of thumb 经验法则:  
  > If a component doesn't need some data for rendering, it doesn't need that data at all.   
  
The Square does not need to know its position to render.   
Therefore, it's best to avoid coupling 耦合 it to the moveKnight method at this point.   
Instead, we are going to add an onClick handler to the div that wraps the Square inside the Board:   

```
import React from 'react';
import PropTypes from 'prop-types';
import Square from './Square';
import Knight from './Knight';
import { moveKnight } from './Game';

/* ... */

renderSquare(i) {
  const x = i % 8;
  const y = Math.floor(i / 8);
  const black = (x + y) % 2 === 1;

  const [knightX, knightY] = this.props.knightPosition;
  const piece = (x === knightX && y === knightY) ?
    <Knight /> :
    null;

  return (
    <div key={i}
         style={{ width: '12.5%', height: '12.5%' }}
         onClick={() => this.handleSquareClick(x, y)}>
      <Square black={black}>
        {piece}
      </Square>
    </div>
  );
}

handleSquareClick(toX, toY) {
  moveKnight(toX, toY);
}
```

We could have also added an onClick prop to Square and used it instead, but since we're going to remove the click handler in favor of the drag and drop interface later anyway, why bother.   
The last missing piece right now is the Chess rule check. The Knight can't just move to an arbitrary square, it is only allowed to make L-shaped moves. I'm adding a canMoveKnight(toX, toY) function to the Game and changing the initial position to A2 to match the Chess rules:   

```
let knightPosition = [1, 7];

/* ... */

export function canMoveKnight(toX, toY) {
  const [x, y] = knightPosition;
  const dx = toX - x;
  const dy = toY - y;

  return (Math.abs(dx) === 2 && Math.abs(dy) === 1) ||
         (Math.abs(dx) === 1 && Math.abs(dy) === 2);
}
```

Finally, I'm adding a canMoveKnight check to the handleSquareClick method:   

```
import { canMoveKnight, moveKnight } from './Game';

/* ... */

handleSquareClick(toX, toY) {
  if (canMoveKnight(toX, toY)) {
    moveKnight(toX, toY);
  }
}
```

