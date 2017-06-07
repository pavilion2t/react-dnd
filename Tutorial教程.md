  # Tutorial 

Now that you've read the overview, it's the adventure time! Even if you have not, you can skip it for now, because ⅔ of our time we'll be busy identifying and building normal React components, just like in the classic Thinking in React tutorial. Adding the drag and drop support is just the icing on the cake.

In this tutorial, we're going to build a Chess game with React and React DnD. Just kidding! Writing a full-blown Chess game is totally out of scope of this tutorial. What we're going to build is a tiny app with a Chess board and a lonely Knight. The Knight will be draggable according to the Chess rules.

We will use this example to demonstrate the data-driven approach of React DnD. You will learn how to create a drag source and a drop target, wire them together with your components, and change their appearance in response to the drag and drop events.

If you're new to React and know a thing or two about it, but yet have to gain some experience building components, this tutorial can also serve as an introduction to the React mode of thinking and the React workflow. If you are a seasoned React developer and only came here for the drag and drop part, feel free to skip to the third and the final chapter of this tutorial.

Enough talk! It's time to set up a build workflow for our little project. I use Webpack, you might be using Browserify. I don't want to get into that now, so just set up an empty React project in whatever way is most convenient for you. If you're feeling lazy, you are free to clone React Hot Boilerplate and work on top of it. In fact, that's what I'm going to do myself.

In this tutorial, the code examples are available simultaneously in ES5, ES6, and ES7. If you want to follow along using ES6 or ES7, you will need to set up a compilation step using Babel. It's easy to make it work with the tool of your choice so we're going to skip this step, too, and assume you've dealt with it and you are ready to write code now. The boilerplate project I linked to before already includes Babel.

The app we're going to build is available as an example on this website.
