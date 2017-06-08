  # Tutorial 教程

Now that you've read the overview, it's the adventure time! Even if you have not, you can skip it for now, because ⅔ of our time we'll be busy identifying and building normal React components, just like in the classic Thinking in React tutorial. Adding the drag and drop support is just the icing on the cake.锦上添花

In this tutorial, we're going to build a Chess game with React and React DnD. Just kidding! Writing a full-blown成熟的 Chess game is totally out of scope超出范围 of this tutorial. What we're going to build is a tiny app with a Chess board and a lonely Knight. The Knight will be draggable according to the Chess rules.

We will use this example to demonstrate the data-driven approach of React DnD. You will learn how to create a drag source and a drop target, wire them together with your components, and change their appearance in response to the drag and drop events.
你会学会怎么创造一个拖放源和拖放目标，用组件把他们牵线在一起，根据拖放事件来改变他们的外观。

If you're new to React and know a thing or two about it, but yet have to gain some experience building components, this tutorial can also serve as an introduction to the React mode of thinking and the React workflow. If you are a seasoned老练的 React developer and only came here for the drag and drop part, feel free to skip to the third and the final chapter of this tutorial.

Enough talk! It's time to set up a build workflow for our little project. I use Webpack, you might be using Browserify. I don't want to get into that now, so just set up an empty React project in whatever way is most convenient for you. If you're feeling lazy, you are free to clone React Hot Boilerplate样板文件 and work on top of it. In fact, that's what I'm going to do myself.

In this tutorial, the code examples are available simultaneously同时地 in ES5, ES6, and ES7. If you want to follow along using ES6 or ES7, you will need to set up a compilation step using Babel. It's easy to make it work with the tool of your choice so we're going to skip this step, too, and assume you've dealt with it and you are ready to write code now. The boilerplate project I linked to before already includes Babel.

The app we're going to build is available as an example on this website.

   ##  Identifying the Components识别组件   
  
  We're going to start by creating some React components first, with no thoughts of the drag and drop interaction. Which components is our Lonely Knight app going to be made of? I can think of a few:

  * Knight, our lonely knight piece;
  * Square, a single square on the board; 正方形
  * Board, the whole board with 64 squares.
  
Let's consider their props.

  * Knight probably needs no props. It has a position, but there's no reason for the Knight to know it, because it can be positioned by being placed into a Square as a child.

  * It is tempting吸引人的 to give Square its position via props, but this, again, is not necessary, because the only information it really needs for the rendering is the color. I'm going to make Square white by default, and add a black boolean prop. And of course Square may accept a single child: the chess piece that is currently on it. I chose white as the default background color to match the browser浏览器 defaults.

  * The Board is tricky. It makes no sense to pass Squares as children to it, because what else could a board contain? Therefore it probably owns the Squares. But then, it also needs to own the Knight because this guy needs to be placed inside one of those Squares. This means that the Board needs to know the knight's current position. In a real Chess game, the Board would accept a data structure describing all the pieces, their colors and positions, but for us, a knightPosition prop will suffice足够. We will use two-item arrays as coordinates坐标, with [0, 0] referring to the A8 square. Why A8 instead of A1? To match the browser coordinate orientation. I tried it another way and it just messed with my head too much.
  
Where will the current state live? I really don't want to put it into the Board component. It's a good idea to have as little state in your components as possible, and because the Board will already have some layout logic, I don't want to also burden it with managing the state.

The good news is, it doesn't matter at this point. We're just going to write the components as if the state existed somewhere, and make sure that they render correctly when they receive it via props, and think about managing the state afterwards!

   ## Creating the Components创建组件 

I prefer to start bottom-up自底向上的；从细节到总体的, because this way I'm always working with something that already exists. If I were to build the Board first, I wouldn't see my results until I'm done with the Square. On the other hand, I can build and see the Square right away without even thinking of the Board. I think that the immediate feedback loop即时反馈循环 is important (you can tell that by another project I work on).


   ### 第一步：创建骑士    
     
In fact I'm going to start with the Knight. It doesn't have any props at all, and it's the easiest one to build:
```
import React, { Component } from 'react';

export default class Knight extends Component {
  render() {
    return <span>♘</span>;
  }
}
```
Yes, ♘ is the Unicode knight! It's gorgeous极好的. We could've made its color a prop, but in our example we're not going to have any black knights, so there is no need for that.

It seems to render fine, but just to be sure, I immediately changed my entry point to test it:
```
import React from 'react';
import ReactDOM from 'react-dom';
import Knight from './Knight';

ReactDOM.render(<Knight />, document.getElementById('root'));
```
I'm going to do this every time I work on another component, so that I always have something to render. In a larger app, I would use a component playground like cosmos so I'd never write the components in the dark.

I see my Knight on the screen! 骑士出现在屏幕上了！！！ 

  ### 第二步：创建正方形     
    
  Time to go ahead and implement the Square now.现在要开始实现正方形的功能。 Here is my first stab:
```
import React, { Component } from 'react';
import PropTypes from 'prop-types';

export default class Square extends Component {
  render() {
    const { black } = this.props;
    const fill = black ? 'black' : 'white';

    return <div style={{ backgroundColor: fill }} />;
  }
}

Square.propTypes = {
  black: PropTypes.bool
};
```
Now I change the entry point code to see how the Knight looks inside a Square:  
现在我改变入口文件来看骑士是否在正方形中：
```
import React from 'react';
import ReactDOM from 'react-dom';
import Knight from './Knight';
import Square from './Square';

ReactDOM.render(
  <Square black>
    <Knight />
  </Square>,
  document.getElementById('root')
);
```
Sadly, the screen is empty. I made a few mistakes:我做错了一些事情：

  * I forgot to give Square any dimensions so it just collapses. I don't want it to have any fixed size, so I'll give it width: '100%' and height: '100%' to fill the container.  
  * 忘记给正方形指定大小，所以它失败了。我不希望它是固定的大小，所以给它的宽高都是100%来填充容器。

  * I forgot to put {this.props.children} inside the div returned by the Square, so it ignores the Knight passed to it.  
  * 我忘记了把{this.props.children}放在正方形返回的容器里，所以它忽视了骑士。

Even after correcting these two mistakes, I still can't see my Knight when the Square is black. That's because the default page body text color is black, so it is not visible on the black Square. I could have fixed this by giving Knight a color prop, but a much simpler fix is to set a corresponding color style in the same place where I set backgroundColor. This version of Square corrects the mistakes and works equally great with both colors:
```
import React, { Component } from 'react';
import PropTypes from 'prop-types';

export default class Square extends Component {
  render() {
    const { black } = this.props;
    const fill = black ? 'black' : 'white';
    const stroke = black ? 'white' : 'black';

    return (
      <div style={{
        backgroundColor: fill,
        color: stroke,
        width: '100%',
        height: '100%'
      }}>
        {this.props.children}
      </div>
    );
  }
}

Square.propTypes = {
  black: PropTypes.bool
};
```
  ### 第三步：创建白板   
  
  Finally, time to get started with the Board! I'm going to start with an extremely naïve version that just draws the same single square:  
  
```
import React, { Component } from 'react';
import PropTypes from 'prop-types';
import Square from './Square';
import Knight from './Knight';

export default class Board extends Component {
  render() {
    return (
      <div>
        <Square black>
          <Knight />
        </Square>
      </div>
    );
  }
}

Board.propTypes = {
  knightPosition: PropTypes.arrayOf(
    PropTypes.number.isRequired
  ).isRequired
};
```
My only intention so far is to make it render, so that I can start tweaking it:  
我唯一的想法就是先把它渲染出来，然后再稍作调整：
```
import React from 'react';
import ReactDOM from 'react-dom';
import Board from './Board';

ReactDOM.render(
  <Board knightPosition={[0, 0]} />,
  document.getElementById('root')
);
```
Indeed, I can see the same single square. I'm now going to add a whole bunch of them! But I don't know where to start. What do I put in render? Some kind of a for loop? A map over some array?

To be honest, I don't want to think about it now. I already know how to render a single square with or without a knight. I also know the knight's position thanks to the knightPosition prop. This means I can write the renderSquare method and not worry about rendering the whole board just yet.

My first attempt at renderSquare looks like this:
```
renderSquare(x, y) {
  const black = (x + y) % 2 === 1;

  const [knightX, knightY] = this.props.knightPosition;
  const piece = (x === knightX && y === knightY) ?
    <Knight /> :
    null;

  return (
    <Square black={black}>
      {piece}
    </Square>
  );
}
```
I can already give it a whirl by changing render to be
```
render() {
  return (
    <div style={{
      width: '100%',
      height: '100%'
    }}>
      {this.renderSquare(0, 0)}
      {this.renderSquare(1, 0)}
      {this.renderSquare(2, 0)}
    </div>
  );
}
```
At this point, I realize that I forgot to give my squares any layout. I'm going to try Flexbox because why not. I added some styles to the root div, and also wrapped the Squares into divs so I could lay them out. Generally it's a good idea to keep components encapsulated and ignorant of how they're being laid out, even if this means adding wrapper divs.
