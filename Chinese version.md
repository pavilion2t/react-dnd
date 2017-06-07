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


  ## Items

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


  ## Types 

What is a type, then?   
A type is a string (or a symbol) uniquely identifying a whole class of items in your application.   

In a Kanban board app, you might have a 'card' type representing the draggable cards and a 'list' type for the draggable lists of those cards. 在一个看板的应用中，你可能会有一个“卡片”类型，代表着可以拖动的卡片；一个“列表”类型，代表着可以拖动的列表。  


In Chess, you might only have a single 'piece' type.在国际象棋中，你可能就只有“棋子”这种类型。

Types are useful because, as your app grows, you might want to make more things draggable, but you don't necessarily want all the existing drop targets to suddenly start reacting to the new items. 类型 非常有用，因为随着APP的成长，你可能需要更多可拖放事物。但你肯定不希望所有存在的接收拖放方都突然对新事项做出反应。   


The types let you specify which drag sources and drop targets are compatible.   指定类型让你更详细了解拖放源和拖放目标是否兼容。  


You're probably going to have an enumeration of the type constants in your application, just like you may have an enumeration of the Redux action types.你可能会遍历应用的所有类型常量，就像可能会遍历Redux动作类型一样。


  ## Monitors监视器 

Drag and drop is inherently stateful. Either a drag operation is in progress, or it isn't. Either there is a current type and a current item, or there isn't. This state has to live somewhere.

React DnD exposes this state to your components via a few tiny wrappers over the internal state storage called the monitors.React DnD把这些状态暴露给你的组件，通过一些小的封装状态存储，叫监视器。  

The monitors let you update the props of your components in response to the drag and drop state changes.监视器让你根据拖放变化更新组件的props。  


For each component that needs to track the drag and drop state, you can define a collecting function that retrieves the relevant bits of it from the monitors.对于每个需要追踪拖放状态的组件，你可以定义一个函数，这个函数通过监视器检索相关的比特。

React DnD then takes care of timely calling your collecting function and merging its return value into your components' props.

Let's say you want to highlight the Chess cells when a piece is being dragged. A collecting function for the Cell component might look like this:
  ```
  function collect(monitor) {
    return {
      highlighted: monitor.canDrop(),
      hovered: monitor.isOver()
  };
}
  
  ```
It instructs React DnD to pass the up-to-date values of highlighted and hovered to all the Cell instances as props.它通知React DnD 传递最新的值（突出的和悬停的）


  ## Connectors连接器 

If the backend handles the DOM events, but the components use React to describe the DOM, how does the backend know which DOM nodes to listen to? Enter the connectors.如果后端处理DOM事件，组件使用React来描述DOM，那么后端怎么知道要监听哪个DOM节点呢？   

The connectors let you assign one of the predefined roles (a drag source, a drag preview, or a drop target) to the DOM nodes in your render function. 连接器让你在渲染函数钟，分配一个提前定义的角色给DOM节点。

In fact, a connector is passed as the first argument to the collecting function we described above. Let's see how we can use it to specify the drop target:
```
function collect(connect, monitor) {
  return {
    highlighted: monitor.canDrop(),
    hovered: monitor.isOver(),
    connectDropTarget: connect.dropTarget()
  };
}
```
In the component's render method, we are then able to access both the data obtained from the monitor, and the function obtained from the connector:
```
render() {
  const { highlighted, hovered, connectDropTarget } = this.props;

  return connectDropTarget(
    <div className={classSet({
      'Cell': true,
      'Cell--highlighted': highlighted,
      'Cell--hovered': hovered
    })}>
      {this.props.children}
    </div>
  );
}
```

The connectDropTarget call tells React DnD that the root DOM node of our component is a valid drop target, and that its hover and drop events should be handled by the backend. Internally it works by attaching a callback ref to the React element you gave it. The function returned by the connector is memoized, so it doesn't break the shouldComponentUpdate optimizations.

  ## Drag Sources and Drop Targets 
  So far we have covered the backends which work with the DOM, the data, as represented by the items and types, and the collecting functions that, thanks to the monitors and the connectors, let you describe what props React DnD should inject into your components.

But how do we configure our components to actually have those props injected? How do we perform the side effects in response to the drag and drop events? Meet the drag sources and the drop targets, the primary abstraction units of React DnD. They really tie the types, the items, the side effects, and the collecting functions together with your components.

Whenever you want to make a component or some part of it draggable, you need to wrap that component into a drag source declaration. Every drag source is registered for a certain type, and has to implement a method producing an item from the component's props. It can also optionally specify a few other methods for handling the drag and drop events. The drag source declaration also lets you specify the collecting function for the given component.

The drop targets are very similar to the drag sources. The only difference is that a single drop target may register for several item types at once, and instead of producing an item, it may handle its hover or drop.
