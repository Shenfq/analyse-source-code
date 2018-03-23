队列是我们常见的一种线性结构，在队尾进行入队操作，在队头进行出队操作。其特点是先进先出，即先插入的元素先出队。

jQuery中引入的队列还是为了管理函数的异步调用，js中充斥着大量的异步调用，这样会把我们的算法分解的支离破碎，之所以的出现promises约定就是为了解决js的大量异步操作带来的异步管理的混乱。而queue方法不同于deferred方法，它的逻辑是将所有的异步操作的函数放在一个数组中，然后对该数组进行出队操作，按顺序执行回调数组中的回调函数。

其实该方法主要是为后面的animation方法提供内部支持的，就像Deferred为ajax方法提供内部支持。

先来介绍一下queue的架构：

```javascript
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
```


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


看过了这两个方法是如何使用的，再来看看这两个方法是如何来实现的。

```javascript
jQuery.extend({
	queue: function( elem, type, data ) {//创建一个函数队列，队列通过data方法在缓存elem上，type为该队列的名字，默认为'fx'
		var queue;
		if ( elem ) {
			type = ( type || "fx" ) + "queue";//如果传入的队列名为空，则默认为fx
			//这里可以看到queue函数队列缓存在了Data的实例data_priv下。
			queue = data_priv.get( elem, type );//先获取当前对象下缓存的队列
			if ( data ) {
				if ( !queue || jQuery.isArray( data ) ) {  //如果当前elem没有缓存过函数队列，则将队列进行缓存；如果data是数组则会把之前的覆盖掉
					//其实我们可以看到queue方法是通过data来提供支撑的，将函数队列缓存在的入口缓存在elem对象上，这也就是为什么我们使用queue方法时，要传入一个节点的原因
					queue = data_priv.access( elem, type, jQuery.makeArray(data) );
				} else {
					queue.push( data );//如果该节点下已经缓存了函数队列，则直接push
				}
			}
			return queue || [];
		}
	},

	dequeue: function( elem, type ) {
		type = type || "fx";

		var queue = jQuery.queue( elem, type ), //先获取队列
			startLength = queue.length,
			fn = queue.shift(),//找到队列的第一个函数
			hooks = jQuery._queueHooks( elem, type ),
			next = function() {//该方法是函数队列出队时的第一个参数，调用该参数就相当于立即进行下一次出队操作
				jQuery.dequeue( elem, type );
			};

		// 判断队首是不是一个inprogress字符串
		if ( fn === "inprogress" ) {
			fn = queue.shift();
			startLength--;
		}

		if ( fn ) {
			// 让animation队列添加后能自动出队
			if ( type === "fx" ) {
				queue.unshift( "inprogress" );
			}
			delete hooks.stop;
			fn.call( elem, next, hooks );//调用队首的函数，让this指向elem，并传入两个参数，next和hooks
			//在该函数中调用next表示对该队列进行出队操作，而hooks的作用是让该队列提前结束。
		}

		if ( !startLength && hooks ) {//如果该函数是队列的最后一个函数，调用hook.empty
			hooks.empty.fire();//清空队列
		}
	},
	_queueHooks: function( elem, type ) {//添加一个回调方法，当队列回调全部调用完毕后，清空队列
		var key = type + "queueHooks";
		return data_priv.get( elem, key ) || data_priv.access( elem, key, {
			empty: jQuery.Callbacks("once memory").add(function() {
				data_priv.remove( elem, [ type + "queue", key ] );
			})
		});
	}
});
```


后面jQuery又通过$.fn.extend把queue和dequeue又一次扩展到了实例方法下，就像之前的data一样，jQuery即拓展了data的静态方法又拓展了data的实例方法。其实在实际情况中，我们通常只会使用到jQuery的实例方法，其静态方法其实是为实例方法提供支持的，毕竟jQuery是一个DOM操作为主的库。再让我们来看看实例方法下的queue与静态方法下的queue有什么区别。

```javascript
jQuery.fn.extend({
	queue: function( type, data ) {
		var setter = 2;

		if ( typeof type !== "string" ) {
			data = type;
			type = "fx";	//默认type为fx
			setter--;
		}

		if ( arguments.length < setter ) {//获取已缓存的队列，并返回
			return jQuery.queue( this[0], type );
		}

		return data === undefined ?
			this :  //如果data为空，则直接返回当前jQuery对象
			this.each(function() {//在jQuery中，使用each遍历匹配的元素，是一种安全的惯例做法，为每个节点都进行一次缓存队列的操作
				var queue = jQuery.queue( this, type, data );//通过调用jQuery.queue()将回调缓存到队列中

				jQuery._queueHooks( this, type );//为当前节点添加一个hooks，确保当想要清空队列时，能及时清空

				if ( type === "fx" && queue[0] !== "inprogress" ) {  //如果type为默认值时，且队首不为"inprogress"
					jQuery.dequeue( this, type );//添加缓存队列后，自动出队，为animation函数提供方便
				}
			});
	},
	//我们可以看出实例方法下的queue比静态方法下的queue，会调用each遍历jQuery对象下的节点，并给所有节点进行相同的操作
	//而且实例方法通常都会将this返回，这里的this就是jQuery对象，方便了链式调用。

	dequeue: function( type ) {//实例方法下的dequeue其实就是对jQuery对象下的每个DOM节点都进行一次出队操作，还是很好理解的
		return this.each(function() {
			jQuery.dequeue( this, type );//调用静态方法的dequue进行出队
		});
	},
	delay: function( time, type ) {//延迟执行队列中未执行的函数
		time = jQuery.fx ? jQuery.fx.speeds[ time ] || time : time;
		//jQuery.fx.speeds[ time ],这个函数作用是把一些特定的字符串转换为时间
		/*源码在8568行
		jQuery.fx.speeds = {
			slow: 600,
			fast: 200,
			// Default speed
			_default: 400
		};
		*/
		type = type || "fx";
		return this.queue( type, function( next, hooks ) {
			var timeout = setTimeout( next, time );//其实就是调用setTimeout进行延迟，时间结束后调用next
			//为hooks添加了一个stop方法，用来清除定时器。
			hooks.stop = function() {
				clearTimeout( timeout );
			};
		});
	},
	clearQueue: function( type ) {//清空队列
		return this.queue( type || "fx", [] );
	},
	// Get a promise resolved when queues of a certain type
	// are emptied (fx is the type by default)
	promise: function( type, obj ) {// 返回一个只读视图，当队列中指定类型的函数执行完毕后
		var tmp,
			count = 1,
			defer = jQuery.Deferred(),//先创建一个延迟对象
			elements = this,
			i = this.length,
			resolve = function() {
				if ( !( --count ) ) {
					defer.resolveWith( elements, [ elements ] );
				}
			};

		if ( typeof type !== "string" ) {
			obj = type;
			type = undefined;
		}
		type = type || "fx";

		while( i-- ) {
			tmp = data_priv.get( elements[ i ], type + "queueHooks" );
			if ( tmp && tmp.empty ) {
				count++;
				tmp.empty.add( resolve );//将缓存对象的resolve操作添加到队列的hooks上，当出队完毕后再调用
			}
		}
		resolve();
		return defer.promise( obj );//返回延迟对象的promise
	}
});
```


----------

源码解读的过程其实很枯燥，接下来看看queue的具体用法。

queue经常结合jQuery的动画使用，可以用来解决回调的问题。只要在每个动画结束后，再回调函数的地方调用next就可以：	

```javascript
var box1 = $('#box1');
var FUNC=[
	function(next) {box1.animate({width:'200'},next);},
	function(next) {box1.animate({width:'500'},next);},
	function(next) {box1.animate({height:'300'},next);},
	function(next) {box1.animate({left:'400'},next);},
	function(next) {box1.animate({left:'200'},next);},
	function(next) {alert('动画结束')}
];

$.queue(document, 'myAnimation',FUNC);
$.dequeue(document ,'myAnimation');
```

如果不用queue，那么写法就是这样的：

```javascript
var box1 = $('#box1');
box1.animate({width:'200'},function() {
	box1.animate({width:'500'},function() {
		box1.animate({height:'300'},function() {
			box1.animate({left:'400'},function() {
				box1.animate({left:'200'},function() {
					alert('动画结束')
				});
			});
		});
	});
});
```

其实我们不用那么复杂的使用queue，因为animate支持链式调用，在内部其实就是使用了queue，所有queue主要是一个内部的为queue提供支持的方法。
	
```javascript
var box1 = $('#box1');
box1.animate({width:'200'})
	.animate({width:'500'})
	.animate({height:'300'})
	.animate({left:'400'})
	.animate({left:'200'})
```

但是当我们要在两个动画之间进行其他的操作时就要用到queue了,例如动画过程中需要从服务器获取数据。

```javascript
var box1 = $('#box1');
box1.animate({width:'200'})
.queue(function(next) {
	$.get('http://localhost:8000/index.php?data=text',function(json) {
		console.log(json);
		next();
	});
})
.delay(5000)  //调用该方法让执行下个动画之前延迟五秒
.animate({width:'500'})
.promist().done(function() { //调用promise返回一个延迟对象，并且在queue出队结束后进行resolve操作，将延迟对象设置成成功态
	alert('animate has done');
});
```



----------

感觉越到后面看源码越来越有些吃力的感觉，不过收获还是很多，熟悉了jQuery的很多api，而且也知道了jQuery是如何解决异步回调的。jQuery定义的很多工具方法虽然都只在内部使用，但是必要的时候我们还是可以自己拿出来使用的。源码的解读也持续了差不多三个月时间了，虽然没有每天都坚持更新，但至少现在还在坚持，没有因为难而放弃。

说说现在的感受：到现在慢慢明白仅仅只盯着源码看是没办法看懂的，一定得先熟悉这些方法是如何使用的才能慢慢看懂内部的实现。而且jQuery内部的方法耦合还是挺高的，经常一个地方要调用其它好几个地方定义的工具函数，而且经常会在看到之前已经看过的工具函数时有些反应不过来，忘记这个函数的具体用法，所以还是得加强复习，有些常用的方法得做到烂熟于心。