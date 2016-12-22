写过jquery插件的人都知道可以通过jquery提供的extend可以对jquery对象进行扩展，而且该方法不仅可以对jquery对象扩展，还能给一个对象添加新的属性和方法，这个在后面会介绍。

**通过不同的方式调用extend扩展的方法也不同：**

- 通过 $.extend() 扩展的是静态方法；
- 而通过 $.fn.extend() 扩展的是实例方法。


写过jQuery插件的通过应该都知道，很多时候我们都是使用extend来为jQuery对象添加插件的。
> 插件的写法：

	;(function($){
		$.fn.extend({
			Firstplus: function() {}
		});

		//这样写的话插件的使用方法就是：$('div').Firstplus();

		$.extend({
			Secondplus: function() {}
		});
		//这样写的话插件的使用方法就是：$.Secondplus();
	})($);



查看源码的第285行，$.extend()和$.fn.extend()调用的其实是同一个函数，那么他们实现的功能为什么不同呢？

    jQuery.extend = jQuery.fn.extend = function() {}   //源码285行

主要是因为这个方法都是将传入的对象扩展到了this上。

- $.extend( xx ) 的this指向的是jQuery对象，所以此时扩展的是jQuery对象，可以直接通过$.xx 的方式调用。
- $.fn.extend( xx ) 的this指向的是jQuery对象的prototyep，所以此时扩展的jQuery对象的原型，实例化的jQuery对象可以调用所有的jQuyer原型上的方法,可以直接通过 $().xx 的方式调用。



以下是extend的三种不同用法：

1. jQuery.extend(  [ object ] )
	>将传入的 object 扩展到 this 对象上

2. jQuery.extend( target, [ object1 ,... objectN ] )
	>将后面传入的 object1 到 objectN 扩展到 target 对象上

3. jQuery.extend( [ deep ], target, [object1,... objectN ] )
	>如果传入了 deep 参数表示递归后面传入object1到objectN对象然后在扩展到target，这样同名的属性名就不会被覆盖了。

具体看下源码分析：

	jQuery.extend = jQuery.fn.extend = function() {
	//定义了一些变量
	var options, name, src, copy, copyIsArray, clone,
		target = arguments[0] || {},  //target用来存储传入的第一个对象（目标对象）
		//这个target不仅表示要进行合并的目标对象，也可以表示要扩展到jquery上的对象
		i = 1,  //i用来表示target后面传入的对象是arguments的第几个参数
		length = arguments.length,   
		deep = false;  //deep变量表示，是否进行深度拷贝

	//先进行了一系列的if判断，来初始化参数，判断到底是要扩展jQuery还是对传入的对象进行扩展

	//如果第一个参数是布尔类型，则表示是否深度拷贝
	if ( typeof target === "boolean" ) {
		deep = target;
		target = arguments[1] || {};  //将目标对象置为传入的第二个参数，如果没有置为空对象
		// skip the boolean and the target
		i = 2;   //将i置为2
	}

	//如果target不是一个对象，将其置为空对象
	if ( typeof target !== "object" && !jQuery.isFunction(target) ) {
		target = {};
	}

	//如果arguments和i相等，表示要扩展的jquery对象，让target指向this
	//这个this具体指向什么之前已经探讨过了
	if ( length === i ) {  
		target = this;
		--i;  //且让i--，这时arguments[i]表示的才是要扩展到jquery上的对象
	}

	for ( ; i < length; i++ ) { //开始遍历传入的 arguments
		// 只有参数不为空时才执行后面的操作
		if ( (options = arguments[ i ]) != null ) {
			// 对对象进行扩展，无论传入的target对象，还是jQuery对象，都使用同样的方法进行扩展
			for ( name in options ) {  //遍历传入的对象的属性
				src = target[ name ];
				copy = options[ name ];
				//防止target和obj的某个属性指向的是同一对象进入死循环
				if ( target === copy ) {  
					continue;
				}

				//是否进行深度拷贝
				//这里说的深度拷贝，和我们平时说的深度拷贝有点区别；这里指的当obj的某个属性是一个对象时，是否对这个属性继续遍历，如果不进行遍历的话，会直接覆盖target对象的同名属性
				if ( deep && copy && ( jQuery.isPlainObject(copy) || (copyIsArray = jQuery.isArray(copy)) ) ) {
					if ( copyIsArray ) {  //如果要拷贝的obj属性为数组
						copyIsArray = false;
						clone = src && jQuery.isArray(src) ? src : [];

					} else {  //如果要拷贝的obj属性为对象
						clone = src && jQuery.isPlainObject(src) ? src : {};
					}

					// 进行递归，直到属性不再为一个对象或数组
					target[ name ] = jQuery.extend( deep, clone, copy );
				} else if ( copy !== undefined ) {//如果不进行深拷贝且当前obj的属性不为空
					//扩展target对象，且和当前obj属性名相同。
					target[ name ] = copy;
				}
			}
		}
	}

	return target;   最后返回target对象，如果扩展的jquery对象，则返回的就是jquery对象
	};




通过extend函数，可以看出extend函数通过许多的 if 判断，实现了许多不同的功能：
- 扩展多个对象到一个对象上；
- 扩展多个对象到一个对象上，并对要进行扩展的对象进行遍历，防止将同属性名的对象覆盖；
- 扩展一个对象到jquery对象上；
- 扩展一个对象到jquery的原型对象上。


后面还可以看到 jq 通过这个方法扩展了很多工具方法到jQuery对象上，不得不说 jq 的结构还是很紧凑的，一个方法不仅提供给外部使用，在内部也多次使用，这不就是传说中的高内聚么，看样子看源码对自己还是有很大的提升的，我是不是又涨见识，哈哈哈哈。