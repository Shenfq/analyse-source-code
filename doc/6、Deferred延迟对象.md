jQuery的延迟对象主要作用是回调函数的异步处理，是一种Promises/A规范的一种实现。具体什么是Promises/A规范大家可以看看[这里](http://blog.csdn.net/qq_16339527/article/details/53097891)。

在jQuery中通过$.extend()方法扩展了两个对象。
    
    jQuery.extend(
    	Defferred: function(){},   //延迟对象
    	when:function() {}		   //when
    );

在看延迟对象的源码之前，我们应该先看看延迟对象是如何使用的。

当我们创建一个新的延迟对象后，通常给它定义两个回调函数，一个是失败后的回调函数（使用fail方法），一个是成功后的回调函数（使用done方法），当然你可以用这两个方法定义多个成功或失败的回调。定义这些对调后就要使用另外两个方法对这些回调进行激活（resolve和reject）。
延迟对象还有一个then方法用来添加回调，接受三个回调，第一个表示成功后的回调，第二个表示失败后 的回调，最后一个是进行中的回调，相当于done和fail方法的简写。

还是看看具体代码：

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


看了上面这个例子，再一步步看源码就没那么吃力了。

然后可以看看该方法的具体结构

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


在创建延迟对象的方法中，首先声明了一个数组，该数组存储的是deferred的三种状态：
1. 成功
2. 失败
3. 进行


	tuples = [
		        这三个数组每一项分别代表着不同的含义：
				执行操作, 添加回调的方法名, 监听队列, 结束状态
				[ "resolve", "done", jQuery.Callbacks("once memory"), "resolved" ],
				[ "reject", "fail", jQuery.Callbacks("once memory"), "rejected" ],
				[ "notify", "progress", jQuery.Callbacks("memory") ]
			]

然后使用each方法对这个数组进行了遍历

	jQuery.each( tuples, function( i, tuple ) {
			var list = tuple[ 2 ],  //list为一个Callbacks对象，且resolve和reject的Callbacks状态为  'once memory'
				stateString = tuple[ 3 ];  //状态信息。只有成功(done)和失败(fail)才有状态信息

			// promise[ done | fail | progress ] = list.add   Deferred的done和fail方法就是状态为  'once memory' 的Callbacks对象的add方法。
			promise[ tuple[1] ] = list.add;

			// Handle state
			if ( stateString ) {//默认为每个结束态添加了三个回调函数
				list.add(function() {  //这里表示当状态为resolve时，就注销fail上的回调，锁定progress；状态为reject时，注销done上的回调，锁定progress
					// state = [ resolved | rejected ]   改变状态
					state = stateString;

				// [ reject_list | resolve_list ].disable; progress_list.lock     使用了异或方法,  0^1==1   1^1==0
				}, tuples[ i ^ 1 ][ 2 ].disable, tuples[ 2 ][ 2 ].lock );
			}
			//扩展了6个方法： resolve | reject | notify  、   resolveWith | rejectWith | notifyWith    就是Callbacks中的fire方法和fireWith方法
			// deferred[ resolve | reject | notify ]
			deferred[ tuple[0] ] = function() {
				deferred[ tuple[0] + "With" ]( this === deferred ? promise : this, arguments );
				return this;
			};
			deferred[ tuple[0] + "With" ] = list.fireWith;// deferred[ resolveWith | rejectWith | notifyWith ]
		});