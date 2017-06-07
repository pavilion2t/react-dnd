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
这事一个合理的缺省，因为它仿照dragged DOM节点，并且创造性地将它用作成一个拖放象征。   

It's handy that you don't have to do any drawing as the cursor moves. 
它很方便使用，光标移动的时候你不需要有任何画图操作。   

This API is also the only way to handle the file drop events.
这个API也是处理文件拖放的唯一方式。   


Unfortunately, the HTML5 drag and drop API also has some downsides. It does not work on touch screens, and it provides less customization opportunities on IE than in other browsers.

This is why the HTML5 drag and drop support is implemented in a pluggable way in React DnD. You don't have to use it. You can write a different implementation, based on touch events, mouse events, or something else entirely. Such pluggable implementations are called the backends in React DnD. Only the HTML5 backend comes with the library, but more may be added in the future.

The backends perform a similar role to that of React's synthetic event system: they abstract away the browser differences and process the native DOM events. Despite the similarities, React DnD backends do not have a dependency on React or its synthetic event system. Under the hood, all the backends do is translate the DOM events into the internal Redux actions that React DnD can process.
