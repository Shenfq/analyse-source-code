当调用jQuery的Callback方法时，会返回一个对象，该对象里包含了很多的方法和属性。
创建该对象时，会接受一个参数，该参数是一个字符串，具体分为一下四种情况：


- once: 确保这个回调列表只执行（ .fire() ）一次(像一个递延 Deferred).
- memory: 保持以前的值，将添加到这个列表的后面的最新的值立即执行调用任何回调 (像一个递延 Deferred).
- unique: 确保一次只能添加一个回调(所以在列表中没有重复的回调).
- stopOnFalse: 当一个回调返回false 时中断调用
	
```javascript
var optionsCache = {};
//在创建Callbacks之前会创建一个对象用来缓存之前设定的属性值。
function createOptions( options ) {
	var object = optionsCache[ options ] = {};
	jQuery.each( options.match( core_rnotwhite ) || [], function( _, flag ) {
		object[ flag ] = true;
	});
	return object;
}


jQuery.Callbacks = function( options ) {

// Convert options from String-formatted to Object-formatted if needed
// (we check in cache first)
//检查该属性是否已经创建过，如果有则直接从optionCache缓存中获取
options = typeof options === "string" ?
	( optionsCache[ options ] || createOptions( options ) ) :
	jQuery.extend( {}, options );

//此处定义了一些self对象将会使用的属性
var memory,fired,firing,firingStart,firingLength,firingIndex,
	list = [], //一个回调函数的队列
	stack = !options.once && [],//一个可重复被激活的队列
	fire = function( data ) {
		//该方法用于激活一个回调函数
	},

	//创建的self对象就是我们的Callbacks对象，该对象下挂在了几个方法，如：
	//add,remove,fire,fireWith。
	
	self = {
		add: function() {  
			// 将一个回调函数添加到list队列
			return this;
		},
		remove: function() {
			// 将传入的回调函数从list队列移除
			return this;
		},
		has: function( fn ) {//检测list队列是否含有该回调函数
			return fn ? jQuery.inArray( fn, list ) > -1 : !!( list && list.length );
		},
		empty: function() {//检测list队列是否为空
			list = [];
			firingLength = 0;
			return this;
		},
		disable: function() {
			list = stack = memory = undefined;
			return this;
		},
		disabled: function() {
			return !list;
		},
		lock: function() {
			stack = undefined;
			if ( !memory ) {
				self.disable();
			}
			return this;
		},
		locked: function() {
			return !stack;
		},
		fireWith: function( context, args ) {   
			
			return this;
		},
		fire: function() {   //调用fireWith方法 激活添加的回调函数
			self.fireWith( this, arguments );
			return this;
		},
		fired: function() {
			return !!fired;
		}
	};


//最后返回self对象。
return self;
};
```


其实该对象主要使用的方法就是add、fire。
下面主要分析下add和fire的源码：

**add:**

```javascript
add: function() {  // 将一个回调函数添加到list队列
	if ( list ) {
		// First, we save the current length
		var start = list.length;
		(function add( args ) {
			jQuery.each( args, function( _, arg ) {
				var type = jQuery.type( arg );//这里调用了之前extend的type方法，返回变量的具体类型
				if ( type === "function" ) {  //判断传入的参数是否为一个函数
					if ( !options.unique || !self.has( arg ) ) {  //如果callbacks对象是unique类型，且该回调函数已经添加则不重复添加
						list.push( arg );   //将符合条件的参数添加到list数组后
					}
				} else if ( arg && arg.length && type !== "string" ) {  //如果该参数是个类数组，则递归
					// Inspect recursively
					add( arg );
				}
			});
		})( arguments );

		if ( firing ) {//如果添加回调函数时，正在激活状态，则将激活队列的末尾加上该回调函数
			firingLength = list.length;
		} else if ( memory ) {  //如果callbacks对象是memory类型的，传入的回调函数直接被激活，并使用之前的值
			firingStart = start;
			fire( memory );
		}
	}
	return this;
}

```

----------

**fire：**

fire一共定义了两次，一次实在self对象中，在初始化self对象之前也定义了一个fire函数，其实观察源码后就会发现，self对象中的fire方法最后就是调用的之前定义的fire函数。

```javascript
fireWith: function( context, args ) {   
	if ( list && ( !fired || stack ) ) {
		args = args || [];
		args = [ context, args.slice ? args.slice() : args ];
		if ( firing ) {
			//如果当前正处于激活状态
			stack.push( args );
		} else {
			fire( args );//这里可以发现最后调用的之前定义的fire函数
		}
	}
	return this;
},
// Call all the callbacks with the given arguments
fire: function() {   
	//调用该方法其实就调用fireWith方法
	self.fireWith( this, arguments );
	return this;
}
```

最后看看self之前定义的fire函数主要实现了什么功能：

```javascript
fire = function( data ) {
	//只有memory为true时才记录data
	memory = options.memory && data;
	fired = true;
	firingIndex = firingStart || 0;
	firingStart = 0;
	firingLength = list.length;
	firing = true;//将状态设置为激活状态
	for ( ; list && firingIndex < firingLength; firingIndex++ ) {
		if ( list[ firingIndex ].apply( data[ 0 ], data[ 1 ] ) === false && options.stopOnFalse ) {
			memory = false; // 阻止未来可能由于add所产生的回调
			break;  //如果Callbacks对象是stopOnFalse类型，当有回调函数返回false时直接中断
		}
	}
	//当回调列表的回调函数全部执行完后，将firing置为false
	firing = false;
	if ( list ) {
		if ( stack ) {//如果有可重复被激活的队列,且队列有回调，则激活其中的函数
			if ( stack.length ) {
				fire( stack.shift() );//且激活一个删除一个
			}
		} else if ( memory ) { //如果有记忆，清空回调队列
			list = [];
		} else {
			self.disable();
		}
	}
}
```





----------

在jQuery中，Callbacks经常用来为异步操作提供支持。
经常会创建一个 once和memory 类型的Callbacks对象。该对象中的回调只要fire一次，后面添加的回调都会按照第一次fire的参数自动激活，且每次激活完毕后会清空回调队列。

如：

```javascript
function fn1(val) {
	console.log(val);
}

var Def = $.Callbacks('once memory');

Def.add( fn1 );
Def.fire( "1" );  //1

//之后的fire都没有作用，因为每次add都会自动激活且清空回调队列
Def.add( fn1 );  //1
Def.fire( "2" );

Def.add( fn1 );  //1
Def.fire( "3" );

Def.add( fn1 );  //1
Def.fire( "4" );
```
