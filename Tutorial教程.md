  # Tutorial 教程

Now that you've read the overview, it's the adventure time! Even if you have not, you can skip it for now, because ⅔ of our time we'll be busy identifying and building normal React components, just like in the classic Thinking in React tutorial. Adding the drag and drop support is just the icing on the cake.锦上添花

In this tutorial, we're going to build a Chess game with React and React DnD. Just kidding! Writing a full-blown成熟的 Chess game is totally out of scope超出范围 of this tutorial. What we're going to build is a tiny app with a Chess board and a lonely Knight. The Knight will be draggable according to the Chess rules.

We will use this example to demonstrate the data-driven approach of React DnD. You will learn how to create a drag source and a drop target, wire them together with your components, and change their appearance in response to the drag and drop events.
你会学会怎么创造一个拖放源和拖放目标，用组件把他们牵线在一起，根据拖放事件来改变他们的外观。

If you're new to React and know a thing or two about it, but yet have to gain some experience building components, this tutorial can also serve as an introduction to the React mode of thinking and the React workflow. If you are a seasoned老练的 React developer and only came here for the drag and drop part, feel free to skip to the third and the final chapter of this tutorial.

Enough talk! It's time to set up a build workflow for our little project. I use Webpack, you might be using Browserify. I don't want to get into that now, so just set up an empty React project in whatever way is most convenient for you. If you're feeling lazy, you are free to clone React Hot Boilerplate样板文件 and work on top of it. In fact, that's what I'm going to do myself.

In this tutorial, the code examples are available simultaneously同时地 in ES5, ES6, and ES7. If you want to follow along using ES6 or ES7, you will need to set up a compilation step using Babel. It's easy to make it work with the tool of your choice so we're going to skip this step, too, and assume you've dealt with it and you are ready to write code now. The boilerplate project I linked to before already includes Babel.

The app we're going to build is available as an example on this website.


  # Identifying the Components 识别组件
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
