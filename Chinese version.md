  #  Overview总览  
  
  
React DnD is unlike most of the drag and drop libraries out there, and it can be intimidating if you've never used it before.  
React DnD不像其他的拖拉库，如果你从未使用过它，可能还会望而生畏。  

However, once you get a taste of a few concepts at the heart of its design, it starts to make sense.  
然而，一旦你掌握了它核心设计的一些概念，就容易多了。  

I suggest you read about these concepts before the rest of the docs.  
我建议你阅读其他文档前，先了解这些概念。  

Some of these concepts resemble the Flux and Redux architectures.
这些概念有些类似Flux和Redux。  

This is not a coincidence, as React DnD uses Redux internally.
这不是巧合，而是React DnD本质上使用Redux。


  ## Backends 

React DnD is built on top of the HTML5 drag and drop API.
React DnD是基于H5的拖放API的。    

It is a reasonable default because it screenshots the dragged DOM node and uses it as a “drag preview” out of the box.
这是一个合理的缺省，因为它仿照dragged DOM节点，并且创造性地将它用作成一个拖放象征。   

It's handy that you don't have to do any drawing as the cursor moves. 
它很方便使用，光标移动的时候你不需要有任何画图操作。   

This API is also the only way to handle the file drop events.
这个API也是处理文件拖放的唯一方式。   


Unfortunately, the HTML5 drag and drop API also has some downsides. It does not work on touch screens, and it provides less customization opportunities on IE than in other browsers.
不幸的是，H5的拖放API也有下降趋势。它不能在触屏上使用，而且对IE较少适配。   


This is why the HTML5 drag and drop support is implemented in a pluggable way in React DnD.
这也是为什么React DnD使用可插入的方式来支持H5的拖放。  

You don't have to use it. You can write a different implementation, based on touch events, mouse events, or something else entirely. Such pluggable implementations are called the backends in React DnD. Only the HTML5 backend comes with the library, but more may be added in the future.

The backends perform a similar role to that of React's synthetic event system: they abstract away the browser differences and process the native DOM events.后端扮演了一个和React综合事件系统类似的角色：它们摘要了浏览器的不同，加工了原生DOM事件。  

Despite the similarities, React DnD backends do not have a dependency on React or its synthetic event system.   
Under the hood, all the backends do is translate the DOM events into the internal Redux actions that React DnD can process.
在底层，所有后端做的事情就是将DOM事件转换成Redux的内部活动，这样React DnD就可以处理了。


  ## Items and Types 

Like Flux (or Redux), React DnD uses data, and not the views, as the source of truth.   

When you drag something across the screen, we don't say that a component, or a DOM node is being dragged.   
当你在屏幕上拖动某物时，我们不会说是在拖动一个组件或是一个DOM节点。   

Instead, we say that an item of a certain type is being dragged.
我们会说在拖动某个类型的事项。   


What is an item?   
An item is a plain JavaScript object describing what's being dragged.  
For example, in a Kanban board application, when you drag a card, an item might look like { cardId: 42 }.   
比如说，在一个看板的应用中，当你拖动一张卡片，那个事项可能长这样 { cardId: 42 }。  

In a Chess game, when you pick up a piece, the item might look like { fromCell: 'C5', piece: 'queen' }. 
在国际象棋中，当你拿起一个棋子，那个事项可能长这样 { fromCell: 'C5', piece: 'queen' }。   


Describing the dragged data as a plain object helps you keep the components decoupled and unaware of each other.
把拖放的数据描述成一个简单的事物，可以保持组件解耦，不相互影响。


What is a type, then? A type is a string (or a symbol) uniquely identifying a whole class of items in your application. In a Kanban board app, you might have a 'card' type representing the draggable cards and a 'list' type for the draggable lists of those cards. In Chess, you might only have a single 'piece' type.

Types are useful because, as your app grows, you might want to make more things draggable, but you don't necessarily want all the existing drop targets to suddenly start reacting to the new items. The types let you specify which drag sources and drop targets are compatible. You're probably going to have an enumeration of the type constants in your application, just like you may have an enumeration of the Redux action types.
