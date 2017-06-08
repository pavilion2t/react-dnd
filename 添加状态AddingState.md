  #  Adding the State 

We want to make the Knight draggable我们希望骑士是可拖拽的.   
It's a noble goal, but we need to see past it. 这是个宏伟的目标，但我们需要越过他。 
What we really mean is that we want to keep the current knightPosition in some kind of state storage, and have some way to change it.  
事实上，我们真正要做的是把当前的knightPosition保存为某种状态存储，然后有办法来改变它。

Because setting up this state requires some thought, we won't try to implement实现 dragging at the same time.   
因为设立这些状态需要深思熟虑，所以我们暂时不会实现拖放。  

Instead, we'll start with a simpler implementation.  我们会从一个简单实现开始。
We will move the Knight when you click a particular Square, but only if this is allowed by the Chess rules.   
我们会根据国际象棋规则，当你点击特定的正方形，就移动骑士。   

Implementing this logic should give us enough insight洞察力 into managing the state, so we can replace clicking with the drag and drop once we've dealt with that.  
实现这个逻辑应该会给我们足够的洞察力来管理状态，然后我们就可以用拖放来代替点击。

React is not opinionated about the state management or the data flow;   
React在状态管理和数据流方面不是固执己见的。  

you can use Flux, Redux, Rx or even Backbone nah, avoid fat models and separate your reads from writes.

I don't want to bother with installing or setting up Redux for this simple example, so I'm going to follow a simpler pattern.  
It won't scale as well as Redux, but I also don't need it to.   
I have not decided on the API for my state manager yet, but I'm going to call it Game, and it will definitely need to have some way of signaling data changes to my React code.  
我还没有想好状态管理的API，但是已经决定给他命名为Game，毫无疑问，他需要传递一些数据改变的信号给我的React代码。

Since I know this much, I can rewrite my index.js with a hypothetical 假设的 Game that doesn't exist yet.     
既然我知道了这些，我可以重写index.js 包含一个暂时还不存在的Game。  

Note that this time, I'm writing my code in blind, not being able to run it yet. This is because I'm still figuring out the API:  
声明这个时候，我还只是在盲写，不能调试。因为我还在思考API：  


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

What is this observe function I import?  我引入的这个 observe 函数是什么？
It's just the most minimal way I can think of to subscribe to a changing state. 这是我能想到的收集状态改变的最简洁的方式。  

I could've made it an EventEmitter but why on Earth even go there when all I need is a single change event?   
我本来可以使用一个EventEmitter，但是我只需要简单的变化事件，为什么even go there？？？    

I could have made Game an object model, but why do that, when all I need is a stream of values?  
我本来可以把Game当成一个实物模型，但是我只需要改变的数据流，所以为什么要那样做呢？  

Just to verify 核实 that this subscription API makes some sense, I'm going to write a fake Game that emits random positions:    
只要核实这个订阅API行得通，我就会写一个假的Game来产生随机位置：  

```
export function observe(receive) {
  setInterval(() => receive([
    Math.floor(Math.random() * 8),
    Math.floor(Math.random() * 8)
  ]), 500);
}
```

Nothing feels as good as being back into the rendering game!
没有比这感觉更好的了，我们又回到了渲染的Game。  

This is obviously not very useful. 显然 这不是非常有用。  

If we want some interactivity, we're going to need a way to modify the Game state from our components.   
如果我们需要一些交互性，我们需要在组件里找到一种方式来修改Game的状态。  

For now, I'm going to keep it simple and expose a moveKnight function that directly modifies the internal state.   
现在，我要尽量简洁地使用moveKnight函数来直接修改内部状态。  

This is not going to fare 经营 well in a moderately complex app where different state storages may be interested in updating their state in response to a single user action, but in our case this will suffice 足够  
在比较复杂的APP里，这个方式行不通，因为一个简单的用户操作可能会导致不同的状态存储都更新它们的状态，但是在我们这个例子中是够用了。

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

Now, let's go back to our components.  现在我们回到组件。  

Our goal at this point is to move the Knight to a Square that was clicked.  
现在我们的目标是点击方块就把骑士移动过去。   

One way to do that is to call moveKnight from the Square itself.   
However, this would require us to pass the Square its position. Here is a good rule of thumb 经验法则:  
  > If a component doesn't need some data for rendering, it doesn't need that data at all.   
  
The Square does not need to know its position to render.   
Therefore, it's best to avoid coupling 耦合 it to the moveKnight method at this point.   
Instead, we are going to add an onClick handler to the div that wraps the Square inside the Board:     
我们打算在包裹着Square的div上添加onClick事件：  


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

The last missing piece right now is the Chess rule check.  

The Knight can't just move to an arbitrary square, it is only allowed to make L-shaped moves.  

I'm adding a canMoveKnight(toX, toY) function to the Game and changing the initial position to A2 to match the Chess rules:   

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

