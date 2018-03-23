jQuery的延迟对象主要作用是回调函数的异步处理，是一种Promises/A规范的一种实现。具体什么是Promises/A规范大家可以看看[这里](http://blog.csdn.net/qq_16339527/article/details/53097891)。

在jQuery中通过$.extend()方法扩展了两个对象。

 ```javascript   
jQuery.extend(
	Defferred: function(){},   //延迟对象
	when:function() {}		   //when
);
```

在看延迟对象的源码之前，我们应该先看看延迟对象是如何使用的。

当我们创建一个新的延迟对象后，通常给它定义两个回调函数，一个是失败后的回调函数（使用fail方法），一个是成功后的回调函数（使用done方法），当然你可以用这两个方法定义多个成功或失败的回调。定义这些对调后就要使用另外两个方法对这些回调进行激活（resolve和reject）。
延迟对象还有一个then方法用来添加回调，接受三个回调，第一个表示成功后的回调，第二个表示失败后 的回调，最后一个是进行中的回调，相当于done和fail方法的简写。

还是看看具体代码：

```javascript
var Def = $.Defferred();
Def.done(function(val){
	alert('成功;'+val);
}).fail(function(val){
	alert('失败;'+val);
});

Def.then(function(val){
	alert('成功;'+val);
},function(val){
	alert('失败;'+val);
});	//	这个作用与上面相同

Def.resolve(1);   //通过这个方法可以调用传入done函数的回调，传入resolve的值提供每一个成功回调函数
```

看了上面这个例子，再一步步看源码就没那么吃力了。

然后可以看看该方法的具体结构

```javascript
Deferred: function( func ) {
	var tuples = [	],  //用来表示延迟对象三个状态的数组
		state = "pending",
		promise = {   //创建一个promise对象
			state: function() { },
			always: function() { },
			then: function( /* fnDone, fnFail, fnProgress */ ) {  },
			promise: function( obj ) {
				return obj != null ? jQuery.extend( obj, promise ) : promise;
			}
		},
		deferred = {};

	promise.pipe = promise.then;
	
	jQuery.each( tuples, function( i, tuple ) { 
		//遍历tuples数组，并给deferred对象绑定相应添加回调和改变状态的方法
	});

	promise.promise( deferred );//将deferred对象与之前声明的promise对象合并

	if ( func ) {  
		//如果给该方法传入了一个函数，那么调用该函数，传递的参数为当前的deferred对象
		func.call( deferred, deferred );
	}

	return deferred;
}
```

在创建延迟对象的方法中，首先声明了一个数组，该数组存储的是deferred的三种状态：
1. 成功
2. 失败
3. 进行

从这个数组我们可以知道，Deferred对象是基于Callbacks对象的，所以如果要弄懂Deferred对象首先要对Callbacks对象有清晰的认识。

```javascript
tuples = [
			这三个数组每一项分别代表着不同的含义：
			执行操作, 添加回调的方法名, 监听队列, 结束状态
			[ "resolve", "done", jQuery.Callbacks("once memory"), "resolved" ],
			[ "reject", "fail", jQuery.Callbacks("once memory"), "rejected" ],
			[ "notify", "progress", jQuery.Callbacks("memory") ]
		]
```


然后使用each方法对这个数组进行了遍历，分别创建了三个Callbacks对象，done方法是向成功的回调队列添加回调，fail是向失败的回调队列添加回调，而progress是向进行中的回调队列添加回调。

```javascript
jQuery.each( tuples, function( i, tuple ) {
	var list = tuple[ 2 ],  //list为一个Callbacks对象，且resolve和reject的Callbacks状态为  'once memory'，该状态表示回调队列fire一次都每次添加的回调都会自动激活然后清空回调队列
		stateString = tuple[ 3 ];  //状态信息。只有成功(done)和失败(fail)才有状态信息

	// promise[ done | fail | progress ] = list.add   Deferred的done和fail方法就是状态为  'once memory' 的Callbacks对象的add方法。
	promise[ tuple[1] ] = list.add;

	// Handle state
	if ( stateString ) {//默认为每个结束态添加了三个回调函数，只要状态改变这三个回调函数都会被激活
		list.add(function() {
			// state = [ resolved | rejected ]   改变状态
			state = stateString;

		// 这里表示当状态为resolve时，就注销fail上的回调，锁定progress；状态为reject时，注销done上的回调，锁定progress     使用了异或方法,  0^1==1   1^1==0
		}, tuples[ i ^ 1 ][ 2 ].disable, tuples[ 2 ][ 2 ].lock );
	}
	//扩展了6个方法： resolve | reject | notify  、   resolveWith | rejectWith | notifyWith    就是Callbacks中的fire方法和fireWith方法
	// deferred[ resolve | reject | notify ]
	deferred[ tuple[0] ] = function() {
		deferred[ tuple[0] + "With" ]( this === deferred ? promise : this, arguments );
		return this;
	};
	// deferred[ resolveWith | rejectWith | notifyWith ]
	deferred[ tuple[0] + "With" ] = list.fireWith;
});
```

下面再看看之前定义的promise对象定义的几个方法：
promise一共定义了四个方法：
1. state   返回当前延迟对象的状态；
2. always  将传入的回调同时注册到成功队列和失败队列；
3. then    传入三个回调，分别注册到成功队列、失败队列和进行中队列；
4. promise	返回延迟对象的一个promise对象，该对象去掉了延迟对象改变状态的几个方法，只能用添加回调到延迟对象。


```javascript
promise = {
	state: function() {
		return state;//获取状态
	},
	always: function() {
		将传入的回调函数同时绑定到失败和成功队列，同时返回promise对象
		deferred.done( arguments ).fail( arguments );
		return this;
	},
	then: function( /* fnDone, fnFail, fnProgress */ ) {  //该函数创建了一个新的Deferred对象并返回受限的promise对象
		var fns = arguments;
		return jQuery.Deferred(function( newDefer ) {
			jQuery.each( tuples, function( i, tuple ) {
				var action = tuple[ 0 ],
					fn = jQuery.isFunction( fns[ i ] ) && fns[ i ];
				// 该函数添加到旧deferred对象的callbacks的对调队列，这个函数是沟通新的def对象和旧def对象的桥梁，旧的def对象状态改变时该函数会被调用
				deferred[ tuple[1] ](function() {//deferred[ tuple[1] ]() ===  $.Callbacks('once memory').add()
					var returned = fn && fn.apply( this, arguments );   //激活回调并获取返回值
					if ( returned && jQuery.isFunction( returned.promise ) ) { //如果回调返回的是一个promise对象
						returned.promise()
							.done( newDefer.resolve )
							.fail( newDefer.reject )
							.progress( newDefer.notify );
					} else {//如果是一个字符串，将该返回值作为下一个回调的参数进行传递
						newDefer[ action + "With" ]( this === promise ? newDefer.promise() : this, fn ? [ returned ] : arguments );
					}
				});
			});
			fns = null;
		}).promise();
	},
	// Get a promise for this deferred  从deferred对象获得一个promise对象
	// If obj is provided, the promise aspect is added to the object  如果传入了obj，表示将promise上的方法添加到obj上
	promise: function( obj ) {
		return obj != null ? jQuery.extend( obj, promise ) : promise;
	}
}
```



之前也说了jQuery的延迟对象其实是对Promise/A规范的一种实现，用来将js无数回调函数进行整理，让代码逻辑更加清晰和使代码扁平化。如果你还在为js的回调地狱苦恼的话不妨试试用延迟对象来管理你的js回调，不过在使用延迟对象之前一定要理解透彻Callbacks对象是怎么管理回调队列的。前前后后为了理解Callbacks和Deferred两个对象花了几个星期，仍然有很多没有理解透彻的地方，希望通过阅读这些源码可以让自己慢慢进步吧。哈哈，又给自己喝了一碗鸡汤。。。