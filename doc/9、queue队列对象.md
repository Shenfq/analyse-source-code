队列是我们常见的一种线性结构，在队尾进行入队操作，在队头进行出队操作。其特点是先进先出，即先插入的元素先出队。

jQuery中引入的队列还是为了管理函数的异步调用，js中充斥着大量的异步调用，这样会把我们的算法分解的支离破碎，之所以的出现promises约定就是为了解决js的大量异步操作带来的异步管理的混乱。而queue方法不同于deferred方法，它的逻辑是将所有的异步操作的函数放在一个数组中，然后对该数组进行出队操作，按顺序执行回调数组中的回调函数。

其实该方法主要是为后面的animation方法提供内部支持的，就像Deferred为ajax方法提供内部支持。

先来介绍一下queue的架构：

	$.extend({
		queue: function(){},  //入队
		dequeue: function(){},  //出队
		_queueHooks: function(){}  //钩子函数，当出队完毕时，清空缓存对象的数据
	});

	$.fn.extend({
		queue: function(){},
		dequeue: function(){},
		delay: function(){},
		clearQueue: function(){},
		promise: function(){}
	});



$.queue(elem, type, data)，该方法接受三个参数：

- elem：表示队列将缓存到那个对象上；这里的缓存其实是调用了jQ内部的Data。之前在介绍data方法时，jQ内部实例化了两个Data对象，data\_user和data\_priv，data方法主要调用了data\_user，本以为data\_priv没什么用，是提供给未来版本jQ使用的，没想到queue方法就是调用data\_priv来缓存队列的，果然jQ里面的代码还是复用度很高的没什么冗余。

- type：表示队列的名字；我们已经知道queue是通过data方法把队列缓存在jQuery.cache上，所以这个type就是data中的key。如果传入的type为空，默认的type为fx。

- data：表示要缓存到队列的函数；可以是单个函数，也可以是函数数组。

	>该方法的使用情况一共有如下几种：  
	>$(elem): 返回当前默认队列（'fx'）下的缓存队列   
	>$(elem,type): 返回当前type队列下的缓存队列    
	>$(elem,type,data):   
	>①、data为一个函数时，先判断当前type下有没有缓存队列，没有队列就新建一个队列，把data缓存到新队列，如果存在就push到队列中。   
	>②、data为一个函数数组时，不管之前有没有缓存队列，重新缓存队列。
	


$.dequeue(elem, type)，该方法接受两个参数：

- elem：表示出队队列的缓存节点

- type：表示出队队列的名字



后面jQuery又通过$.fn.extend把queue和dequeue又一次扩展到了实例方法下，就像之前的data一样，jQuery即拓展了data的静态方法又拓展了data的实例方法。其实在实际情况中，我们通常只会使用到jQuery的实例方法，其静态方法其实是为实例方法提供支持的，毕竟jQuery是一个DOM操作为主的库。再让我们来看看实例方法下的queue与静态方法下的queue有什么区别。
